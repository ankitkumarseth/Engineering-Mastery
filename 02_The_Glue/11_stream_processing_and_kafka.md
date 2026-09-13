# 11. Stream Processing & Kafka (DDIA)

**Goal:** Moving away from batch processing (running a job once a night) to stream processing (reacting to events in near real-time).

## 1. Message Brokers vs Event Logs

### Standard Message Broker (RabbitMQ / ActiveMQ)
- **Model:** "Queue" model. 
- **Behavior:** Once a consumer reads a message and acknowledges it, the broker deletes it. 
- **Use Case:** Task queues (e.g., sending an email, processing a background job).

### Log-Based Broker (Apache Kafka)
- **Model:** Append-only Immutable Log.
- **Behavior:** Messages are written to a log on disk and stay there for a retention period (e.g., 7 days). Consumers read from the log using an "Offset" (bookmark).
- **Use Case:** High-throughput streaming, Event Sourcing, allowing multiple different systems to read the exact same data at their own pace.

## 2. Kafka Architecture (The Core Concepts)
*   **Topic:** The category/feed name.
*   **Partition:** Topics are split into partitions. This is the secret to Kafka's scalability. Each partition is hosted on a different broker (server).
*   **Ordering Guarantee:** Kafka only guarantees strict ordering **within a single partition**. If you need events for "Customer A" to be processed in order, you must use Customer A's ID as the Partition Key.
*   **Consumer Groups:** A group of consumers working together to read a topic. Kafka assigns each partition to exactly one consumer in the group. If you have 5 partitions, you can have a maximum of 5 consumers working in parallel.

## 3. Delivery Semantics (The Hardest Problem)
What happens if a consumer reads a message, processes it, but crashes before saving its offset?

1.  **At-Most-Once:** Save the offset *before* processing. If you crash, you never read it again. (Message loss).
2.  **At-Least-Once (Most Common):** Process the message, *then* save the offset. If you crash, the next consumer reads it again. (Causes duplicates).
3.  **Exactly-Once:** Incredibly difficult. Requires Kafka's transaction feature and idempotent consumers (consumers that can safely process duplicates without messing up the database).

## 4. The Outbox Pattern (Crucial Interview Pattern)
**The Problem:** You need to save data to your Oracle DB *and* publish a message to Kafka. If the DB saves, but Kafka goes down, your system is inconsistent. (Dual-write problem).

**The Solution (Outbox):** 
1. Save your data AND save the event into an `OUTBOX` table in the *same* database transaction.
2. A separate background worker (or Debezium/Change-Data-Capture) reads the `OUTBOX` table and reliably publishes it to Kafka.

## Interview Cheat Sheet 💡
*   **"How do we process 1 million events per second?"** -> Kafka partitioned across multiple brokers, consumed by a Consumer Group scaled to match the number of partitions.
*   **"How do we ensure events aren't processed out of order?"** -> Use a reliable Partition Key (like TransactionID or UserID) so related events go to the exact same partition.
*   **"How do we handle duplicate messages from Kafka?"** -> Make the consumer **Idempotent**. (e.g., Check if `event_id` exists in the DB before processing, or use `UPSERT` statements).
