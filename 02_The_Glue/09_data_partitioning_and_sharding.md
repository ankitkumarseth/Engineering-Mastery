# 09. Data Partitioning & Sharding (DDIA)

**Goal:** Break a massive database into smaller chunks (partitions/shards) so it can scale horizontally across multiple machines.

## 1. Why Partition?
Replication (copying data) solves *read* scaling and fault tolerance. 
Partitioning (splitting data) solves **write** scaling and storage capacity limits.
*Every partition acts as its own mini-database.*

## 2. Partitioning Strategies

### A. Key Range Partitioning
- **How it works:** Assign a continuous range of keys to each partition (e.g., A-F goes to Node 1, G-M to Node 2).
- **Pros:** Excellent for range queries (e.g., "Get all records between 2024 and 2025").
- **Cons:** High risk of **Hotspots**. If you partition by date, today's partition gets 100% of the write traffic, while others sit idle.

### B. Hash Partitioning
- **How it works:** Pass the key through a hash function, and assign the hash to a partition (e.g., `hash(key) % num_nodes`).
- **Pros:** Evenly distributes writes! Destroys hotspots.
- **Cons:** You lose the ability to do efficient range queries. You have to query *every* node and combine the results (Scatter/Gather).

### C. Compound / Composite Primary Key (The Sweet Spot)
- Used in databases like Cassandra and DynamoDB.
- A key has two parts: `(PartitionKey, SortKey)`.
- The `PartitionKey` is hashed to find the node. The `SortKey` is used to keep data sorted locally on that node for efficient range queries.

## 3. Rebalancing (What happens when you add a node?)
If your hash function is `hash(key) % 10` and you add a node, it becomes `% 11`. **Every single key** evaluates to a new number, meaning you have to move 100% of your data. This is catastrophic.

**The Solution: Consistent Hashing**
- Instead of `% N`, visualize a ring of hashes from `0` to `MAX`. 
- Nodes are placed on the ring. Data keys are placed on the ring. A key belongs to the *next node* moving clockwise.
- **Why it matters in interviews:** If you add or remove a node, only the data mapped to that *specific* segment of the ring moves. The rest of the database is unaffected. (Crucial for distributed caching like Redis or Memcached).

## 4. Secondary Indexes in Partitioned DBs
- **Document-partitioned (Local Index):** Each partition maintains its own secondary index. (Pro: Fast writes. Con: Read queries must be broadcasted to all partitions - Scatter/Gather).
- **Term-partitioned (Global Index):** A separate partition just for the index. (Pro: Fast reads. Con: Writes are slower because they require updating a different partition, often requiring distributed transactions).

## Interview Cheat Sheet 💡
*   **"Our DB is full, but we have tons of sequential reads."** -> Propose Key-Range partitioning, but warn about write hotspots.
*   **"Our DB is full, and we need massive write throughput."** -> Propose Hash partitioning.
*   **"We need to add nodes dynamically without downtime."** -> Mention Consistent Hashing.
