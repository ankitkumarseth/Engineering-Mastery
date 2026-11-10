# Transactions & Exactly-Once Semantics (EOS) in Distributed Pipelines

## 1. The Exactly-Once Problem: Why Idempotence Alone Is Not Enough

In distributed systems theory, achieving "Exactly-Once" processing over unreliable networks is considered an impossibility without atomic multi-system coordination.

In Kafka, setting `enable.idempotence = true` guarantees that a single producer writing to a single partition will not create duplicates across retries. However, modern stream processing and event-driven architectures execute the **Consume-Transform-Produce loop**:

```
+-------------------------------------------------------------------------------+
|                      THE CONSUME-TRANSFORM-PRODUCE LOOP                       |
|                                                                               |
|  [ Input Topic A ] ===> (1. Poll) ===> [ Worker Node ] ===> (2. Process)      |
|                                              |                                |
|  [ __consumer_offsets ] <== (4. Commit) <----+====> (3. Produce) ===> [ Topic B ]
+-------------------------------------------------------------------------------+
```

### The Dual-System Failure Vulnerability:
Step (3) produces messages to an output topic, while Step (4) commits the read offset back to `__consumer_offsets`.
- If the worker crashes **after Step 3 but before Step 4**: The replacement worker re-reads the input message from Topic A and produces it to Topic B a second time. **Duplicate downstream production!**
- If the worker crashes **after Step 4 but before Step 3**: The offset was advanced, but output was never written. **Data loss!**

To achieve true end-to-end Exactly-Once Semantics (**EOS**), Kafka introduces **Distributed Transactions (KIP-98)**, coupling outgoing messages and incoming offset commits into an **atomic Two-Phase Commit (2PC) unit of work**.

---

## 2. Architectural Blueprint: The Transaction Coordinator & 2PC Sequence

The following diagram illustrates the 6-phase transactional pipeline coordinating the Transaction Coordinator, the internal `__transaction_state` log, control markers, and the `read_committed` consumer isolation barrier:

```
[ Producer (transactional.id = "order-worker-1") ]
       |
       | 1. FindCoordinator & InitProducerId (Allocates PID, increments Epoch to fence zombies)
       v
[ Transaction Coordinator (Broker 102) ] <====> [ __transaction_state (50 Partitions) ]
       |
       | 2. beginTransaction()
       | 3. AddPartitionsToTxnRequest (Topic B, Partition 0)
       v
[ Output Partition (Topic B-0) ] <====== (Appends Data Batches)
       |
       | 4. sendOffsetsToTxn(Input Topic A, Offset 42)
       v
[ Consumer Coordinator ] (Buffers offsets for Transaction Coordinator)
       |
       | 5. commitTransaction()
       v
[ Transaction Coordinator ]
       |-- Phase 1: Writes PREPARE_COMMIT to __transaction_state
       |-- Phase 2: Writes physical COMMIT MARKER to Topic B-0
       |-- Phase 2: Writes physical COMMIT MARKER to __consumer_offsets
       \-- Completes: Writes COMMITTED to __transaction_state
```

---

## 3. The 6-Step Transactional Protocol Internals

### 3.1 Step 1: Discovering the Coordinator & Initializing Transactions
1. **FindCoordinator**: The producer hashes its configured `transactional.id`:
   $$\text{Coordinator Partition} = \text{Utils.abs}(\text{transactional.id.hashCode()}) \pmod{50}$$
   The broker hosting that partition of `__transaction_state` becomes the dedicated **Transaction Coordinator**.
2. **InitProducerIdRequest**:
   - The producer registers its `transactional.id`.
   - The coordinator looks up the existing state. It increments the **Producer Epoch** (e.g., from `1` to `2`) and returns the same `PID`.
   - **Zombie Fencing**: If an old, partitioned instance of `order-worker-1` is still alive and attempts to produce with Epoch `1`, the broker rejects it with `ProducerFencedException`. The old zombie process is immediately terminated.

### 3.2 Step 2: `beginTransaction()`
A purely local client-side call. Sets the internal producer transaction state to `IN_TRANSACTION`.

### 3.3 Step 3: Registering Output Partitions (`AddPartitionsToTxn`)
Before writing to any topic partition, the producer sends an `AddPartitionsToTxnRequest` to the coordinator. The coordinator appends an `ONGOING` state record to `__transaction_state`. If the producer crashes later, the coordinator knows exactly which partitions require cleanup.

### 3.4 Step 4: Routing Consumed Offsets to Transaction
The application invokes `sendOffsetsToTransaction(offsets, consumerGroupId)`:
- Instead of committing directly to `__consumer_offsets`, the producer sends the offsets to the **Transaction Coordinator**.
- The coordinator registers `__consumer_offsets` as a participant in the two-phase commit transaction.

### 3.5 Step 5: Committing the Transaction (`EndTxnRequest`)
The application invokes `commitTransaction()`:
1. **Phase 1 (Prepare Commit)**: The coordinator writes a `PREPARE_COMMIT` record to the `__transaction_state` log. Once this record hits disk across the `min.insync.replicas` quorum, the transaction is **guaranteed to succeed**, even if the coordinator crashes immediately afterward.
2. **Phase 2 (Commit Markers)**: The coordinator writes a special non-data **Commit Control Marker** (`ControlBatch`) to:
   - Every output topic-partition involved in the transaction.
   - The consumer group's partition inside `__consumer_offsets`.
3. **Completion**: The coordinator writes a final `COMMITTED` record to `__transaction_state`.

---

## 4. Consumer Isolation: `read_committed` & Last Stable Offset (LSO)

How do downstream consumers perceive ongoing transactions?

```
Topic B - Partition 0:
[ Offset 0 (Committed) ] [ Offset 1 (Txn Ongoing) ] [ Offset 2 (Txn Ongoing) ] [ Offset 3 (Uncommitted) ]
------------------------->                                                    ---------------------------->
Last Stable Offset (LSO = 1)                                                  High Watermark (HW = 4)
(Consumer isolation.level = read_committed stops here!)                       (Standard consumer reads here)
```

### 4.1 Last Stable Offset (LSO) Mechanics
- **`isolation.level = read_uncommitted` (Default)**:
  Consumers read all records up to the **High Watermark (HW)**. Ongoing, uncommitted, and eventually aborted transactions are immediately visible.
- **`isolation.level = read_committed`**:
  Consumers read only up to the **Last Stable Offset (LSO)**:
  $$\text{LSO} = \text{Offset of the earliest ongoing / uncommitted transaction}$$
  - Records belonging to aborted transactions are discarded in memory by the consumer's fetcher.
  - Control markers (commit/abort) are consumed internally by the client driver and never exposed to the user application code.

### 4.2 The "Hanging Transaction" Trap (LSO Pinning)
> [!CAUTION]
> If a buggy producer begins a transaction at offset 100, produces 1 record, and hangs indefinitely without committing or aborting:
> - The **LSO remains frozen at offset 100**.
> - Even if millions of records from other healthy producers are appended at offsets $101, 102, \dots, 1{,}000{,}000$, all `read_committed` consumers **freeze at offset 100**!
> - **The Antidote**: Configure `transaction.timeout.ms` (Broker default: `900,000` ms = 15 minutes; Client default: `60,000` ms = 1 minute). When the timeout expires, the Transaction Coordinator automatically aborts the hung transaction, appends an `ABORT` control marker, and advances the LSO, releasing consumer traffic.

---

## 5. End-to-End Exactly-Once: What It Guarantees (and What It Doesn't)

Staff candidates must clearly articulate the physical boundaries of Kafka EOS:

### What Kafka EOS DOES Guarantee:
1. **Kafka-to-Kafka Pipelines**: Reading from Kafka topic A, processing state in memory, and writing to Kafka topic B with `isolation.level = read_committed` provides mathematically provable **Exactly-Once Semantics**.
2. **Kafka Streams & Flink**: Stream state stores (RocksDB) and output topics commit atomically via the internal transaction coordinator.

### What Kafka EOS DOES NOT Guarantee:
1. **Side-Effects to External Systems**: If your worker sends an HTTP request, calls an external payment gateway, or issues an `INSERT` into an external Oracle/Postgres database inside a Kafka transaction:
   - **Kafka cannot rollback the external database or the external credit card charge** if the Kafka transaction subsequently aborts!
2. **The External Solution**: To achieve end-to-end exactly-once with external datastores:
   - Use the **Transactional Outbox Pattern with Debezium CDC**.
   - Ensure the downstream database write is strictly **idempotent** using a unique business transaction key.

---

## 6. Runnable Verification Suite

The following self-contained Java verification suite demonstrates:
1. **The Transaction Coordinator State Machine**: Simulating `initTransactions`, `beginTransaction`, `sendOffsetsToTransaction`, and 2PC commit marker generation.
2. **Zombie Producer Fencing**: Proving that an old producer epoch is rejected when an identically named producer initializes.
3. **`read_committed` vs. `read_uncommitted` Consumer Behavior**: Demonstrating LSO filtering and skipping aborted transaction batches.

```java
package com.engineering.mastery.kafka;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * TransactionalExactlyOncePipelinePOC
 *
 * Demonstrates:
 * 1. 2PC Transaction Coordinator managing __transaction_state.
 * 2. Producer epoch zombie fencing via transactional.id.
 * 3. Commit/Abort control markers in partition logs.
 * 4. Consumer isolation levels: read_uncommitted vs. read_committed (LSO barrier).
 */
public class TransactionalExactlyOncePipelinePOC {

    // =========================================================================
    // 1. Transactional Models
    // =========================================================================
    public enum TxnState { EMPTY, ONGOING, PREPARE_COMMIT, PREPARE_ABORT, COMMITTED, ABORTED }
    public enum RecordType { DATA, COMMIT_MARKER, ABORT_MARKER }

    public record PartitionRecord(long offset, RecordType type, long pid, String payload) {}

    // =========================================================================
    // 2. Transaction Coordinator & Broker Storage Simulation
    // =========================================================================
    public static class TransactionCoordinator {
        private final Map<String, Long> transactionalIdToPid = new ConcurrentHashMap<>();
        private final Map<Long, Short> pidToEpoch = new ConcurrentHashMap<>();
        private final Map<Long, TxnState> pidToTxnState = new ConcurrentHashMap<>();
        private long pidSequence = 1000L;

        public synchronized long initProducerId(String transactionalId) {
            long pid = transactionalIdToPid.computeIfAbsent(transactionalId, k -> pidSequence++);
            short epoch = (short) (pidToEpoch.getOrDefault(pid, (short) 0) + 1);
            pidToEpoch.put(pid, epoch);
            pidToTxnState.put(pid, TxnState.EMPTY);
            return pid;
        }

        public synchronized void validateProducerEpoch(long pid, short epoch) {
            short currentEpoch = pidToEpoch.getOrDefault(pid, (short) -1);
            if (epoch < currentEpoch) {
                throw new IllegalStateException(String.format(
                        "ProducerFencedException: Incoming epoch %d is fenced by active epoch %d!", epoch, currentEpoch));
            }
        }

        public synchronized void beginTransaction(long pid, short epoch) {
            validateProducerEpoch(pid, epoch);
            pidToTxnState.put(pid, TxnState.ONGOING);
        }

        public synchronized void commitTransaction(long pid, short epoch, List<SimulatedPartition> partitions) {
            validateProducerEpoch(pid, epoch);
            pidToTxnState.put(pid, TxnState.PREPARE_COMMIT);

            // Phase 2: Write COMMIT markers to all participant partitions
            for (SimulatedPartition part : partitions) {
                part.appendControlMarker(RecordType.COMMIT_MARKER, pid);
            }
            pidToTxnState.put(pid, TxnState.COMMITTED);
        }

        public synchronized void abortTransaction(long pid, short epoch, List<SimulatedPartition> partitions) {
            validateProducerEpoch(pid, epoch);
            pidToTxnState.put(pid, TxnState.PREPARE_ABORT);

            // Phase 2: Write ABORT markers to all participant partitions
            for (SimulatedPartition part : partitions) {
                part.appendControlMarker(RecordType.ABORT_MARKER, pid);
            }
            pidToTxnState.put(pid, TxnState.ABORTED);
        }

        public short getEpoch(long pid) {
            return pidToEpoch.getOrDefault(pid, (short) 0);
        }
    }

    // =========================================================================
    // 3. Simulated Partition with LSO Isolation
    // =========================================================================
    public static class SimulatedPartition {
        final List<PartitionRecord> log = new ArrayList<>();

        public synchronized void appendData(long pid, String data) {
            long offset = log.size();
            log.add(new PartitionRecord(offset, RecordType.DATA, pid, data));
        }

        public synchronized void appendControlMarker(RecordType markerType, long pid) {
            long offset = log.size();
            log.add(new PartitionRecord(offset, markerType, pid, markerType.name()));
        }

        // Simulates read_uncommitted: returns all records
        public synchronized List<PartitionRecord> readUncommitted() {
            return new ArrayList<>(log);
        }

        // Simulates read_committed: skips aborted records and control markers
        public synchronized List<PartitionRecord> readCommitted() {
            List<PartitionRecord> result = new ArrayList<>();
            Map<Long, List<PartitionRecord>> ongoingTxnBuffers = new HashMap<>();

            for (PartitionRecord rec : log) {
                if (rec.type() == RecordType.DATA) {
                    ongoingTxnBuffers.computeIfAbsent(rec.pid(), k -> new ArrayList<>()).add(rec);
                } else if (rec.type() == RecordType.COMMIT_MARKER) {
                    List<PartitionRecord> committed = ongoingTxnBuffers.remove(rec.pid());
                    if (committed != null) {
                        result.addAll(committed);
                    }
                } else if (rec.type() == RecordType.ABORT_MARKER) {
                    // Abort: silently discard buffered records!
                    ongoingTxnBuffers.remove(rec.pid());
                }
            }
            return result;
        }
    }

    // =========================================================================
    // 4. Verification Runner
    // =========================================================================
    public static void main(String[] args) {
        System.out.println("================================================================================");
        System.out.println("     KAFKA EXACTLY-ONCE SEMANTICS (EOS) & 2PC TRANSACTIONS POC                  ");
        System.out.println("================================================================================\n");

        TransactionCoordinator coordinator = new TransactionCoordinator();
        SimulatedPartition outputTopicPartition = new SimulatedPartition();

        // 1. Initialize Producer Instance 1
        String txId = "order-processor-tx-1";
        long pid = coordinator.initProducerId(txId);
        short epoch1 = coordinator.getEpoch(pid);

        System.out.println("--- [1. Initializing Producer & Epoch Fencing] ---");
        System.out.printf("Producer Registered: transactional.id='%s' -> PID=%d, Epoch=%d%n", txId, pid, epoch1);

        // 2. Successful Transaction Execution
        System.out.println("\n--- [2. Successful Transaction (Commit 2PC)] ---");
        coordinator.beginTransaction(pid, epoch1);
        outputTopicPartition.appendData(pid, "Order-101-Created");
        outputTopicPartition.appendData(pid, "Order-101-InventoryDeducted");
        coordinator.commitTransaction(pid, epoch1, List.of(outputTopicPartition));
        System.out.println("Transaction committed with physical COMMIT marker.");

        // 3. Aborted Transaction Execution
        System.out.println("\n--- [3. Aborted Transaction (Rollback 2PC)] ---");
        coordinator.beginTransaction(pid, epoch1);
        outputTopicPartition.appendData(pid, "Order-102-PaymentFailed");
        coordinator.abortTransaction(pid, epoch1, List.of(outputTopicPartition));
        System.out.println("Transaction aborted with physical ABORT marker.");

        // 4. Compare Consumer Isolation Levels
        System.out.println("\n--- [4. Consumer Isolation Level Comparison] ---");
        List<PartitionRecord> uncommittedView = outputTopicPartition.readUncommitted();
        List<PartitionRecord> committedView = outputTopicPartition.readCommitted();

        System.out.println("A. consumer.isolation.level = read_uncommitted (Exposes Dirty Records & Markers):");
        for (PartitionRecord r : uncommittedView) {
            System.out.printf("   Offset %d | Type: %-13s | PID: %d | Payload: %s%n",
                    r.offset(), r.type(), r.pid(), r.payload());
        }

        System.out.println("\nB. consumer.isolation.level = read_committed (Hides Aborted Records & Markers):");
        for (PartitionRecord r : committedView) {
            System.out.printf("   Offset %d | Type: %-13s | PID: %d | Payload: %s%n",
                    r.offset(), r.type(), r.pid(), r.payload());
        }

        // 5. Zombie Producer Fencing
        System.out.println("\n--- [5. Zombie Producer Epoch Fencing Test] ---");
        System.out.println("[SCENARIO]: Pod 1 stalls. Kubernetes spins up replacement Pod 2 with identical transactional.id...");
        long pid2 = coordinator.initProducerId(txId);
        short epoch2 = coordinator.getEpoch(pid2);
        System.out.printf("New Pod 2 Registered: PID=%d, New Epoch=%d (Monotonically Incremented!)%n", pid2, epoch2);

        System.out.println("\nZombie Pod 1 wakes up and attempts to produce with OLD Epoch 1:");
        try {
            coordinator.beginTransaction(pid, epoch1);
            System.out.println("❌ FAILED: Zombie was allowed to write!");
        } catch (IllegalStateException e) {
            System.out.printf("✓ SUCCESS: %s%n", e.getMessage());
            System.out.println("Zombie fenced out immediately! Zero data corruption.");
        }

        System.out.println("\n================================================================================");
        System.out.println("                      VERIFICATION COMPLETED SUCCESSFULLY                       ");
        System.out.println("================================================================================");
    }
}
```

---

## 7. Staff / Principal Interview Strategy

### 7.1 "Is Kafka Truly Exactly-Once?"
When an interviewer asks: *"Vendors claim Kafka has Exactly-Once Semantics. Is it actually Exactly-Once in the real world?"*

#### The Principal Systems Response:
1. **Be Precise**: *"Kafka provides true Exactly-Once processing **within the closed loop of the Kafka ecosystem** (e.g., Kafka-to-Kafka streaming with `read_committed` and transactional producers)."*
2. **Acknowledge the Two-Generals / FLP Boundary**:
   - *"If your application writes to Kafka and an external RDBMS, or calls a third-party Stripe payment gateway, Kafka cannot coordinate an XA Two-Phase Commit across heterogeneous systems without catastrophic availability and latency degradation."*
3. **Provide the Production Solution**:
   - To achieve end-to-end exactly-once with external systems:
     - Combine Kafka with the **Transactional Outbox Pattern** (RDBMS local transaction + CDC).
     - Ensure all consumer side-effects are **idempotent** via unique constraint deduplication tables or Redis state locks.

### 7.2 Sizing and Tuning Transactions
When asked: *"What are the operational pitfalls of enabling transactions in high-throughput systems?"*

#### Key Trade-Offs to Highlight:
1. **Latency Overhead**: Each transaction requires multiple round-trips to the Transaction Coordinator (`AddPartitionsToTxn`, `PREPARE_COMMIT`, control markers). Micro-batching smaller than 100ms with transactions degrades throughput.
2. **LSO Stalls**: Long-running transactions delay all `read_committed` consumers on shared partitions. Keep transaction duration short ($\le 1\text{--}5\text{ seconds}$).
3. **Partition Allocation**: `__transaction_state` defaults to 50 partitions and replication factor 3. Ensure the cluster has at least 3 brokers to satisfy `min.insync.replicas = 2`.
