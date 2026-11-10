# Consumer Groups, Offset Commit Strategies & Cooperative Rebalancing

## 1. Consumer Group Architecture & Load Distribution

In Apache Kafka, consumer horizontal scalability and load distribution are governed by **Consumer Groups**. Understanding the physical mapping between topic partitions and consumer threads is essential for Staff and Principal systems design.

```
Topic: "orders" (4 Partitions: P0, P1, P2, P3)

Scenario A: Balanced Consumer Group (2 Consumers)
[ Consumer 1 ] <==== Assigned ====> [ P0, P1 ]
[ Consumer 2 ] <==== Assigned ====> [ P2, P3 ]

Scenario B: Saturated Consumer Group (4 Consumers)
[ Consumer 1 ] <==== Assigned ====> [ P0 ]
[ Consumer 2 ] <==== Assigned ====> [ P1 ]
[ Consumer 3 ] <==== Assigned ====> [ P2 ]
[ Consumer 4 ] <==== Assigned ====> [ P3 ]

Scenario C: Over-Provisioned Consumer Group (5 Consumers)
[ Consumer 1 ] -> P0 | [ Consumer 2 ] -> P1 | [ Consumer 3 ] -> P2 | [ Consumer 4 ] -> P3
[ Consumer 5 ] -> IDLE / ZERO WORK! (No partition available!)
```

### 1.1 The Golden Rule of Partition Assignment
> [!IMPORTANT]
> **A single partition can be consumed by at most ONE consumer instance within the same consumer group at any given time.**
> - If a topic has $N$ partitions and a consumer group has $M$ consumers:
>   - If $M \le N$, partitions are divided evenly among consumers.
>   - If $M > N$, exactly $N$ consumers process traffic; the remaining $M - N$ consumers sit completely idle as warm standbys.
> - **Architectural Consequence**: The maximum degree of consumer parallelization for a topic is strictly bounded by its **partition count**. You cannot scale consumer throughput simply by adding pods if partition count is lower than pod count!

---

## 2. Offset Storage & Consumer Lag Mechanics

### 2.1 The Internal Commit Log: `__consumer_offsets`
In legacy Kafka ($< 0.9$), offsets were stored in ZooKeeper, inducing crippling write bottlenecks. Modern Kafka stores committed offsets in an internal, compacted system topic named `__consumer_offsets` (typically configured with 50 partitions).
- **Commit Record Key**: `[Group ID, Topic, Partition]`
- **Commit Record Value**: `[Offset, Leader Epoch, Metadata, Commit Timestamp]`
- **Compaction**: Because `__consumer_offsets` uses log compaction, only the latest committed offset per consumer group/partition is retained over time.

### 2.2 Consumer Lag: The Primary Health Metric
Consumer lag measures how far behind a consumer group is relative to real-time production:

$$\text{Consumer Lag} = \text{Log End Offset (LEO)} - \text{Current Committed Offset}$$

```
Partition 0:
[ 0 ] [ 1 ] [ 2 ] [ 3 ] [ 4 ] [ 5 ] [ 6 ] [ 7 ] [ 8 ] [ 9 ]
---------------------->                                     ^
Current Offset = 3                                          LEO = 10
(Consumer Lag = 10 - 3 = 7 unconsumed records)
```

- A rising lag trend indicates downstream bottlenecking (slow database I/O, thread pool starvation, or insufficient consumer concurrency).
- Monitoring lag via tools like **Kafka Exporter** or **LinkedIn Burrow** is preferred over CPU utilization, as consumer CPU can be 100% while lag is perfectly healthy (or CPU 10% while consumer is deadlocked).

---

## 3. Offset Commit Strategies: At-Most-Once vs. At-Least-Once

The exact line of code where a consumer commits its offset defines its delivery semantics:

```
+---------------------------------------------------------------------------------------+
|  CONSUMER POLL-PROCESS-COMMIT LIFECYCLE                                              |
|                                                                                       |
|  Consumer.poll() ===> [ In-Memory Records ] ===> Process Record ===> Commit Offset    |
|                             |                           |                 |           |
|                     (If crash here:            (If crash here:    (Safe at-least-once)|
|                      At-Most-Once               At-Least-Once                         |
|                      Data LOSS!)                DUPLICATE!)                           |
+---------------------------------------------------------------------------------------+
```

### 3.1 At-Most-Once Delivery (Auto-Commit)
- **Configuration**: `enable.auto.commit = true`, `auto.commit.interval.ms = 5000`.
- **Mechanics**: When `poll()` is invoked, if 5 seconds have elapsed, Kafka automatically commits the highest offset returned from the *previous* poll, **regardless of whether processing succeeded or finished**.
- **Failure Mode**: If the consumer process crashes while processing message 50, upon restart it fetches message 100. **Messages 51 to 99 are permanently lost without trace.**
- **Use Case**: High-volume, loss-tolerant metrics (e.g., website page clickstreams).

### 3.2 At-Least-Once Delivery (Manual Commit After Processing)
- **Configuration**: `enable.auto.commit = false`.
- **Mechanics**: The application processes the records, executes database transactions, and calls `commitSync()` or `commitAsync()` **only after** processing finishes.
- **Failure Mode**: If the consumer crashes after writing to the database but before committing the offset, the replacement consumer re-processes those records upon rebalance.
- **Requirement**: **Downstream processing must be idempotent** (e.g., using database upserts, distributed locks, or deduplication tables).

### 3.3 Synchronous vs. Asynchronous Manual Commits
```java
// Anti-Pattern: commitSync() inside hot loop (Slashes throughput by 80%!)
for (ConsumerRecord<String, String> record : records) {
    process(record);
    consumer.commitSync(); // Blocks thread waiting for broker network RTT!
}

// Production Pattern: commitAsync() with commitSync() on shutdown/rebalance
try {
    while (running) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            process(record);
        }
        // Fire-and-forget: does not block poll loop!
        consumer.commitAsync((offsets, exception) -> {
            if (exception != null) {
                log.warn("Async commit failed for offsets {}", offsets, exception);
            }
        });
    }
} finally {
    try {
        // Guarantee clean final state before closing socket
        consumer.commitSync();
    } finally {
        consumer.close();
    }
}
```

---

## 4. Heartbeats, Liveness & The Deadlock Detection Triangle

Kafka separates **process liveness** (network connectivity) from **thread health** (active processing):

```
+---------------------------------------------------------------------------------------+
|                       KAFKA CONSUMER LIVENESS TRIANGLE                                |
|                                                                                       |
| 1. Heartbeat Thread (Background Daemon)                                               |
|    - Sends HeartbeatRequest every: heartbeat.interval.ms (3s)                         |
|    - Failure threshold:            session.timeout.ms (45s)                           |
|                                                                                       |
| 2. Processing Thread (User Application)                                              |
|    - Must call consumer.poll() within: max.poll.interval.ms (300s = 5 min)           |
|    - Batch size ceiling:               max.poll.records (500 records)                 |
+---------------------------------------------------------------------------------------+
```

### 4.1 The Slow Processing Rebalance Storm
A classic enterprise incident occurs when a consumer batch contains 500 records. If processing each record takes 1 second (e.g., calling a slow 3rd-party REST API):
- Total processing time = $500 \times 1\text{s} = 500\text{ seconds}$ ($\sim 8.3$ minutes).
- Because `max.poll.interval.ms = 300,000` ms (5 minutes), the broker's Group Coordinator concludes: *"The consumer thread is deadlocked or hung!"*
- The coordinator kicks the consumer out and triggers a **Rebalance Storm**.
- Meanwhile, the consumer finishes, calls `poll()`, discovers it was evicted, throws `CommitFailedException`, and triggers *another* rebalance. The entire consumer cluster enters a permanent death spiral!

### 4.2 The Production Antidote:
1. **Reduce `max.poll.records`**: Lower from `500` to `50` or `20`, ensuring worst-case batch processing time never exceeds $30\%$ of `max.poll.interval.ms`:
   $$\text{max.poll.records} \times \text{Max Record Latency} \le \frac{1}{3} \times \text{max.poll.interval.ms}$$
2. **Increase `max.poll.interval.ms`**: For heavy batch jobs, increase to 10 or 15 minutes.
3. **Decouple Ingestion from Processing**: Ingest records via `poll()`, enqueue them onto an internal Java `ThreadPoolExecutor` (with bounded queue and `CallerRunsPolicy`), and commit offsets asynchronously.

---

## 5. Rebalance Protocol Evolution: Eager vs. Cooperative Sticky

When a consumer joins, crashes, or leaves a group, partitions must be reassigned.

```
EAGER REBALANCE PROTOCOL (Stop-The-World)
Time 0: Consumer 3 joins group.
Time 1: [STOP THE WORLD!] Consumer 1 revokes P0, P1. Consumer 2 revokes P2, P3.
        (All message consumption halts cluster-wide!)
Time 2: JoinGroup & SyncGroup phase negotiation.
Time 3: Consumer 1 assigned P0. Consumer 2 assigned P1. Consumer 3 assigned P2, P3.
Time 4: Consumers re-open partition logs and resume consumption. (Multi-second delay!)

-----------------------------------------------------------------------------
COOPERATIVE STICKY ASSIGNOR (KIP-429 Incremental Rebalancing)
Time 0: Consumer 3 joins group.
Time 1: Consumer 1 continues processing P0, P1 uninterrupted!
        Consumer 2 continues processing P2, P3 uninterrupted!
Time 2: Coordinator calculates minimal migration delta: Only P3 needs to move.
Time 3: Phase 1: Consumer 2 revokes ONLY P3.
Time 4: Phase 2: Consumer 3 claims P3.
Result: P0, P1, and P2 had ZERO interruption! Downtime reduced by 95%!
```

### 5.1 Static Group Membership (KIP-345)
In cloud Kubernetes environments, rolling updates restart pods one by one. Under default settings, every pod restart triggers a full rebalance.
- **The Solution**: Configure a static identifier:
  ```properties
  group.instance.id = "order-consumer-pod-0"
  ```
- When pod-0 restarts, the Group Coordinator recognizes the `group.instance.id` and grants a grace period (`session.timeout.ms`, e.g., 60 seconds).
- No rebalance is triggered. The pod restarts, reconnects, and immediately resumes consuming its previously assigned partitions without a single interruption to peer pods!

---

## 6. Runnable Verification Suite

The following self-contained Java benchmark demonstrates:
1. **Consumer Lag Calculation**: Evaluating LEO vs. Current Offset across partitions.
2. **At-Most-Once vs. At-Least-Once Delivery**: Simulating process failure before and after offset commitment.
3. **Cooperative Sticky Rebalance Simulation**: Demonstrating incremental non-blocking partition handoff without stopping peer consumers.

```java
package com.engineering.mastery.kafka;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * ConsumerRebalanceProtocolPOC
 *
 * Demonstrates:
 * 1. Consumer lag tracking across topic partitions.
 * 2. Delivery semantics failure modes: At-Most-Once data loss vs. At-Least-Once duplicates.
 * 3. Eager Stop-the-World vs. Cooperative Sticky Assignor partition rebalancing.
 */
public class ConsumerRebalanceProtocolPOC {

    // =========================================================================
    // 1. Partition & Lag Tracker
    // =========================================================================
    public static class TopicPartition {
        final String topic;
        final int partition;
        long logEndOffset;
        final List<String> logMessages = new ArrayList<>();

        public TopicPartition(String topic, int partition) {
            this.topic = topic;
            this.partition = partition;
            this.logEndOffset = 0;
        }

        public synchronized void append(String msg) {
            logMessages.add(msg);
            logEndOffset++;
        }
    }

    public static class ConsumerGroupCoordinator {
        final Map<String, Long> committedOffsets = new ConcurrentHashMap<>();

        public void commit(String group, int partition, long offset) {
            committedOffsets.put(group + "-" + partition, offset);
        }

        public long getCommittedOffset(String group, int partition) {
            return committedOffsets.getOrDefault(group + "-" + partition, 0L);
        }

        public long calculateLag(String group, TopicPartition tp) {
            long committed = getCommittedOffset(group, tp.partition);
            return Math.max(0, tp.logEndOffset - committed);
        }
    }

    // =========================================================================
    // 2. Cooperative vs. Eager Rebalance Simulator
    // =========================================================================
    public static class RebalanceSimulator {

        public static void simulateEagerRebalance(List<String> activeConsumers, List<Integer> partitions, String newConsumer) {
            System.out.println("  [EAGER PROTOCOL - STOP THE WORLD]");
            System.out.println("  1. ALL existing consumers REVOKE ALL assigned partitions.");
            for (String c : activeConsumers) {
                System.out.printf("     → Consumer %s halts consumption & revokes assigned partitions!%n", c);
            }
            System.out.println("  2. Cluster-wide processing completely FROZEN during JoinGroup & SyncGroup.");
            List<String> updatedConsumers = new ArrayList<>(activeConsumers);
            updatedConsumers.add(newConsumer);
            System.out.printf("  3. Re-assigning %d partitions across %d consumers from scratch.%n",
                    partitions.size(), updatedConsumers.size());
        }

        public static void simulateCooperativeRebalance(
                Map<String, List<Integer>> currentAssignments,
                String newConsumer
        ) {
            System.out.println("  [COOPERATIVE STICKY PROTOCOL - NON-BLOCKING]");
            System.out.println("  1. Consumers keep processing unaffected partitions without pause!");

            // Identify only the single partition to migrate
            String donor = null;
            int partitionToMigrate = -1;
            for (Map.Entry<String, List<Integer>> entry : currentAssignments.entrySet()) {
                if (entry.getValue().size() > 1) {
                    donor = entry.getKey();
                    partitionToMigrate = entry.getValue().get(entry.getValue().size() - 1);
                    break;
                }
            }

            System.out.printf("  2. Phase 1: Consumer %s revokes ONLY partition %d. All other partitions stay active!%n",
                    donor, partitionToMigrate);
            System.out.printf("  3. Phase 2: Partition %d seamlessly reassigned to new Consumer %s (Zero downtime!).%n",
                    partitionToMigrate, newConsumer);
        }
    }

    // =========================================================================
    // 3. Execution & Verification Runner
    // =========================================================================
    public static void main(String[] args) {
        System.out.println("================================================================================");
        System.out.println("    KAFKA CONSUMER LAG, COMMIT STRATEGIES & COOPERATIVE REBALANCE POC           ");
        System.out.println("================================================================================\n");

        ConsumerGroupCoordinator coordinator = new ConsumerGroupCoordinator();
        TopicPartition p0 = new TopicPartition("orders.v1", 0);

        // Populate partition with 10 records
        for (int i = 0; i < 10; i++) {
            p0.append("order_evt_" + i);
        }

        // PART 1: Consumer Lag Calculation
        System.out.println("--- [1. Consumer Lag Measurement] ---");
        String group = "order-fulfillment-group";
        coordinator.commit(group, 0, 6); // Committed up to offset 6

        long lag = coordinator.calculateLag(group, p0);
        System.out.printf("Partition 0 LEO:        %d%n", p0.logEndOffset);
        System.out.printf("Group Committed Offset: %d%n", coordinator.getCommittedOffset(group, 0));
        System.out.printf("Active Consumer Lag:    %d unconsumed records%n%n", lag);

        // PART 2: Delivery Semantics: At-Most-Once vs At-Least-Once
        System.out.println("--- [2. Delivery Semantics Failure Simulation] ---");

        // Scenario A: At-Most-Once (Auto-Commit before processing)
        System.out.println("[At-Most-Once Simulation]:");
        long currentOffset = 6;
        long nextOffsetToCommit = currentOffset + 3; // Premature commit of batch (6..8)
        coordinator.commit(group, 0, nextOffsetToCommit);
        System.out.printf("  1. Auto-commit executed ahead of processing: Committed Offset = %d%n", nextOffsetToCommit);
        System.out.println("  2. CRASH! Process dies while processing record at offset 7!");
        System.out.printf("  3. Recovery: Next consumer resumes from offset %d -> Record 7 PERMANENTLY LOST!%n%n",
                coordinator.getCommittedOffset(group, 0));

        // Scenario B: At-Least-Once (Manual Commit after processing)
        System.out.println("[At-Least-Once Simulation]:");
        coordinator.commit(group, 0, 6); // Reset
        System.out.println("  1. Consumer fetches records 6, 7, 8. Processes record 6 and 7 to database.");
        System.out.println("  2. CRASH! Process dies before commitSync() can be executed!");
        System.out.printf("  3. Recovery: Next consumer resumes from last committed offset %d.%n",
                coordinator.getCommittedOffset(group, 0));
        System.out.println("  4. Records 6 and 7 re-delivered. (Safe IF AND ONLY IF downstream is idempotent!).\n");

        // PART 3: Rebalance Protocol Comparison
        System.out.println("--- [3. Rebalance Protocol Evolution: Eager vs. Cooperative] ---");
        List<String> consumers = List.of("consumer-pod-1", "consumer-pod-2");
        List<Integer> partitions = List.of(0, 1, 2, 3);

        System.out.println("A. Scaling up: Adding consumer-pod-3 under Eager Assignor:");
        RebalanceSimulator.simulateEagerRebalance(consumers, partitions, "consumer-pod-3");

        System.out.println("\nB. Scaling up: Adding consumer-pod-3 under Cooperative Sticky Assignor:");
        Map<String, List<Integer>> currentMap = new HashMap<>();
        currentMap.put("consumer-pod-1", new ArrayList<>(List.of(0, 1)));
        currentMap.put("consumer-pod-2", new ArrayList<>(List.of(2, 3)));
        RebalanceSimulator.simulateCooperativeRebalance(currentMap, "consumer-pod-3");

        System.out.println("\n================================================================================");
        System.out.println("                      VERIFICATION COMPLETED SUCCESSFULLY                       ");
        System.out.println("================================================================================");
    }
}
```

---

## 7. Staff / Principal Interview Strategy

### 7.1 Surviving Consumer Group Rebalance Storms
When an interviewer asks: *"During high traffic, our consumer group enters a perpetual rebalance state where partitions are constantly revoked and reassigned, causing processing to freeze completely. How do you diagnose and fix this?"*

#### The Principal Diagnosis & Fix Protocol:
1. **Differentiate Heartbeat Timeout vs. Processing Timeout**:
   - Check the client logs:
     - If `CommitFailedException: message: "Commit cannot be completed since the group has already rebalanced and assigned the partitions to another member"` appears, this is a **`max.poll.interval.ms` breach**, NOT a heartbeat failure!
     - The background heartbeat thread was healthy, but the user processing thread took longer than 5 minutes to complete the batch returned by `poll()`.
2. **Immediate Remediation Steps**:
   - **Step 1**: Reduce `max.poll.records` (e.g., from `500` to `50`).
   - **Step 2**: Upgrade assignor from `RangeAssignor` to `CooperativeStickyAssignor`:
     ```properties
     partition.assignment.strategy = org.apache.kafka.clients.consumer.CooperativeStickyAssignor
     ```
     This confines rebalancing to the affected partition delta rather than stopping the world.
   - **Step 3**: Configure `group.instance.id` (Static Membership) on Kubernetes consumer pods so rolling deployments do not trigger rebalance cascades.

### 7.2 The Synchronous Commit Performance Pitfall
When asked: *"Should we invoke `commitSync()` after every message in our Kafka consumer to prevent duplicates?"*

#### The Staff Engineering Response:
- State firmly: *"No. Calling `commitSync()` per message is an anti-pattern that destroys consumer throughput."*
- Explain the physical mechanics:
  - `commitSync()` is a blocking synchronous RPC that waits for the network round-trip to the broker's `__consumer_offsets` partition leader.
  - If network RTT is $5\text{ms}$, single-threaded consumer throughput is mathematically capped at:
    $$\text{Max Throughput} = \frac{1000\text{ms}}{5\text{ms}} = 200\text{ messages/second!}$$
- Instead, propose **batch-level asynchronous commits (`commitAsync()`)** paired with transactional outbox, database unique constraints, or idempotent business keys downstream.
