# Redis Internals, Memory Physics & Redis Cluster Topology (16,384 Slots)

In high-concurrency architectures, Redis is the gold standard for in-memory key-value caching, distributed locks, session state stores, and real-time streaming counters. However, at the Staff and Principal Engineer level, treating Redis as an opaque black-box cache leads to severe production incidents: **unbounded memory leaks, fork-induced latency spikes, cluster slot misconfigurations, and cross-slot transaction errors**.

This playbook covers the mechanical physics of Redis: **why single-threaded execution outperforms multi-threaded architectures, internal memory structures, approximated LRU/LFU eviction math, and Redis Cluster's 16,384 hash slot sharding protocol**.

---

## 1. Why Is Redis Single-Threaded (And Why Is It So Fast?)

A common interview question explores how a single-threaded Redis process can process over 150,000 operations per second on modest hardware.

```
========================================================================================================================
                                     REDIS IN-MEMORY & EVENT LOOP PHYSICS
========================================================================================================================

  CONVENTIONAL MULTI-THREADED DB                    REDIS SINGLE-THREADED REACTOR (EPOLL)
  ┌─────────────────────────────────────────┐       ┌─────────────────────────────────────────┐
  │ Thread 1 ────> [ Mutex / Lock ] <──── T2│       │ 10,000 Sockets ──> epoll_wait() (Kernel)│
  │ Thread 3 ────> [ Buffer Latch ] <──── T4│       │                        │                │
  │ • Context switching overhead (CPU churn)│       │                        ▼                │
  │ • Lock contention & thread synchronization│       │             Single-Thread Event Loop    │
  │ • Cache line bouncing across CPU cores  │       │             • 100% Non-blocking RAM I/O │
  └─────────────────────────────────────────┘       │             • ZERO Lock Synchronization │
                                                    │             • Pure CPU L1/L2 Cache Hits │
                                                    └─────────────────────────────────────────┘
```

### 1.1 The Five Pillars of Redis Speed
1. **Pure In-Memory Operations**:
   - Memory access latency is $\sim 100\text{ nanoseconds}$, compared to NVMe SSD disk access ($\sim 100\text{ microseconds}$ $\to 1,000\times$ slower) and spinning magnetic disks ($\sim 10\text{ milliseconds}$ $\to 100,000\times$ slower).
   - Redis operations are bounded purely by RAM memory bandwidth and CPU clock cycles, never disk I/O.
2. **Zero Context Switching**:
   - Multi-threaded servers (e.g., standard Java thread pools) spend up to 20% of CPU time saving register states, flushing CPU pipelines, and performing thread context switches. Redis dedicates 100% of its CPU time to execution.
3. **Zero Lock Contention**:
   - Because commands are executed sequentially on a single thread, **Redis requires zero mutexes, zero spinlocks, and zero read-write condition variables**. Every command is inherently atomic.
4. **Non-Blocking I/O Multiplexing (`epoll` / `kqueue`)**:
   - Redis uses the operating system's event notification subsystem (`epoll` on Linux). A single thread monitors thousands of client TCP sockets simultaneously, reacting only when data is ready to be read.
5. **Threaded I/O (Redis 6.0+)**:
   - In modern high-throughput networks (10GbE / 40GbE), reading network packets and serializing/deserializing RESP protocol bytes became the primary CPU bottleneck.
   - Redis 6.0 introduced **I/O Threads**: Socket reads, command protocol parsing, and response writing are distributed across worker threads, while **command execution remains strictly single-threaded** to preserve lock-free atomicity.

---

## 2. Low-Level In-Memory Data Structures

Redis avoids generic standard library abstractions in favor of highly optimized, memory-contiguous C structures:

```
========================================================================================================================
                                     REDIS LOW-LEVEL DATA STRUCTURES
========================================================================================================================

  SDS (Simple Dynamic String)                       DICT (Progressive Rehashing)
  ┌─────────────────────────────────────────┐       ┌─────────────────────────────────────────┐
  │ [len: 4B][alloc: 4B][flags: 1B][buf...] │       │ ht[0] (Active Table)  ht[1] (New Table) │
  │ • O(1) strlen (No strlen() scan)        │       │ • Progressive Rehashing: Amortizes      │
  │ • Binary-safe (stores raw byte arrays)  │       │   rehash across 1,000,000 keys without  │
  │ • Prevents C buffer overflow            │       │   blocking the event loop!              │
  └─────────────────────────────────────────┘       └─────────────────────────────────────────┘

  QUICKLIST / LISTPACK                              SKIPLIST (Sorted Sets / ZSET)
  ┌─────────────────────────────────────────┐       ┌─────────────────────────────────────────┐
  │ Doubly-linked list of compact arrays    │       │ Multi-level linked list with            │
  │ • Solves pointer bloat (saves 80% RAM)  │       │ probabilistic forward pointers          │
  │ • Maximum CPU L1/L2 cache prefetching   │       │ • O(log N) search, insert, range query  │
  └─────────────────────────────────────────┘       └─────────────────────────────────────────┘
```

---

## 3. Memory Eviction Policies & Mathematical Models

When Redis reaches its `maxmemory` ceiling, it must evict keys to accommodate new writes.

### 3.1 The 8 Eviction Policies (`maxmemory-policy`)
* **`noeviction`** (Default): Returns out-of-memory errors on writes (`OOM command not allowed`).
* **`allkeys-lru`**: Evicts least recently used keys across all keys. (Best for general web caching).
* **`volatile-lru`**: Evicts LRU keys only among those configured with an expiration TTL.
* **`allkeys-lfu`**: Evicts least frequently used keys across all keys. (Protects against one-off access spikes).
* **`volatile-lfu`**: Evicts LFU keys only among those with a TTL.
* **`volatile-ttl`**: Evicts keys with the shortest remaining Time-To-Live.
* **`allkeys-random` / `volatile-random`**: Evicts random keys (rarely used).

---

### 3.2 Approximated LRU vs. True LRU
* **Why True LRU Fails in Redis**: A true LRU queue requires a doubly linked list linking all keys. Every `GET` or `SET` operation must unlink and move a node to the head of the list. This would consume **16 to 24 bytes of pointer overhead per key** (hundreds of megabytes of wasted RAM) and introduce memory allocator locks.
* **Redis Approximated LRU**:
  - Redis stores a **24-bit LRU clock** directly inside the 16-byte `robj` object header.
  - When memory is needed, Redis samples **5 random keys** (configured by `maxmemory-samples 5`), examines their 24-bit clock, and evicts the key with the oldest idle time.
  - *Mathematical Outcome*: Sampling 10 keys achieves **99.5% identical accuracy** to a true LRU list with **zero additional pointer memory**!

---

### 3.3 LFU: The Morris Logarithmic Counter
To prevent "cache pollution" (where a key accessed 10,000 times last week but never accessed today remains in cache under LRU), Redis implements **LFU**:
* An 8-bit counter tracks frequency, using a **Morris Logarithmic Counter**:
  $$\text{Increment Probability:} \quad P = \frac{1}{\text{counter} \times \text{lfu-log-factor} + 1}$$
* Counter values are decayed over time based on elapsed minutes (`lfu-decay-time`), ensuring dead keys drop in rank.

---

## 4. Redis Cluster Architecture: 16,384 Hash Slots

While **Redis Sentinel** provides master-replica automated failover, it is bounded by a **Single Master write ceiling** (all writes hit a single node).

To scale horizontally across hundreds of nodes, enterprise architectures use **Redis Cluster**—a decentralized, shared-nothing sharding architecture.

```
========================================================================================================================
                                     REDIS CLUSTER 16,384 HASH SLOT TOPOLOGY
========================================================================================================================

                                     Client Key: "user:101"
                                                │
                                                ▼
                                     CRC16("user:101") % 16384 = Slot 2450
                                                │
                 ┌──────────────────────────────┼──────────────────────────────┐
                 │                              │                              │
                 ▼ Slot [0..5460]               ▼ Slot [5461..10922]           ▼ Slot [10923..16383]
        ┌──────────────────┐           ┌──────────────────┐           ┌──────────────────┐
        │  MASTER NODE A   │           │  MASTER NODE B   │           │  MASTER NODE C   │
        │  Slot: 2450 ✓    │           │  Slot: 7120      │           │  Slot: 14200     │
        └────────┬─────────┘           └────────┬─────────┘           └────────┬─────────┘
                 │ Async Replica                │ Async Replica                │ Async Replica
                 ▼                              ▼                              ▼
        ┌──────────────────┐           ┌──────────────────┐           ┌──────────────────┐
        │  Replica Node A1 │           │  Replica Node B1 │           │  Replica Node C1 │
        └──────────────────┘           └──────────────────┘           └──────────────────┘
        
        • Gossip Protocol: All Masters exchange PING/PONG heartbeats on Port 16379 (Port + 10,000).
        • Zero Central Proxy: Smart clients cache slot mapping; route directly to target Master!
```

### 4.1 The Hash Slot Mathematics
* Every key is deterministically mapped to one of **16,384 logical hash slots**:
  $$\text{Hash Slot} = \text{CRC16}(\text{Key}) \pmod{16384}$$
* **Why Exactly 16,384 Slots?**
  1. *Heartbeat Overhead*: Redis Cluster nodes continuously exchange gossip ping/pong packets. Heartbeat packets include a bitmap of slot assignments. 16,384 slots require exactly **2KB of memory** ($16,384 / 8 = 2,048\text{ bytes}$). If Redis used 65,536 slots, heartbeat payloads would expand to 8KB, creating massive network bandwidth saturation in 1,000-node clusters.
  2. *Cluster Scale*: 16,384 slots provides ample granularity to rebalance across up to 1,000 master nodes (averaging ~16 slots per node).

---

### 4.2 Hash Tags `{...}`: Multi-Key Atomicity
In a clustered environment, multi-key operations (`MGET`, `MSET`, `EVAL` Lua scripts, Transactions) fail with a `CROSSSLOT Keys in request don't hash to the same slot` error if the keys reside on different nodes.

* **The Hash Tag Solution**:
  If a key contains `{...}`, **only the text inside the curly braces is hashed**:
  - `user:{cust_101}:profile` $\to \text{CRC16}("cust\_101") \pmod{16384} \to \text{Slot } 5120$
  - `user:{cust_101}:orders` $\to \text{CRC16}("cust\_101") \pmod{16384} \to \text{Slot } 5120$
  - `user:{cust_101}:balance` $\to \text{CRC16}("cust\_101") \pmod{16384} \to \text{Slot } 5120$
* Both keys hash to the **exact same slot** and reside on the **same physical master node**, enabling atomic multi-key operations!

---

### 4.3 Smart Client Routing & Redirection Protocol
Redis Cluster does not use an intermediate reverse proxy (like Envoy or HAProxy), which would add a network hop. Instead, clients are **Smart Clients** (e.g., Jedis, Lettuce):
1. Upon connecting, the client issues `CLUSTER SLOTS` to download the complete slot-to-node routing table and caches it in memory.
2. The client calculates $\text{CRC16}(\text{key}) \pmod{16384}$ and opens a direct TCP connection to the correct master node.
3. **Redirection Protocols**:
   - **`MOVED <slot> <ip:port>`**: Returned if a slot has permanently migrated to a new node. The client updates its local routing table and re-issues the command.
   - **`ASK <slot> <ip:port>`**: Returned if a slot is currently mid-migration. The client sends an `ASKING` command followed by the query to the target node **without** updating its local slot cache.

---

## 5. Standalone Java 21 Verification POC: Redis Cluster Slot Router & CRC16 Engine

The following self-contained Java program simulates:
1. **CRC16 Hash Slot Calculation**: Standard CCITT-16 algorithm mapping keys to `[0..16383]`.
2. **Hash Tag `{...}` Parser**: Extracting co-located hash tags for atomic multi-key operations.
3. **Smart Client Routing Table**: Simulating a 3-shard cluster with automated `MOVED` redirection handling.

```java
package com.engineering.mastery.database;

import java.nio.charset.StandardCharsets;
import java.util.*;

/**
 * RedisClusterHashSlotRouterPOC
 *
 * Simulates:
 * 1. Redis CRC16 Hash Slot calculation across 16,384 slots.
 * 2. Hash Tag {...} co-location extraction.
 * 3. Smart Client routing cache & MOVED redirection handling.
 */
public class RedisClusterHashSlotRouterPOC {

    public static final int TOTAL_SLOTS = 16384;

    // =========================================================================
    // 1. Redis CRC16 Algorithm (XMODEM CCITT-16)
    // =========================================================================
    public static class RedisCrc16 {
        private static final int[] CRC16_LOOKUP_TABLE = new int[256];

        static {
            for (int i = 0; i < 256; i++) {
                int curr = i << 8;
                for (int j = 0; j < 8; j++) {
                    if ((curr & 0x8000) != 0) {
                        curr = ((curr << 1) ^ 0x1021) & 0xFFFF;
                    } else {
                        curr = (curr << 1) & 0xFFFF;
                    }
                }
                CRC16_LOOKUP_TABLE[i] = curr;
            }
        }

        public static int getSlot(String key) {
            String tag = extractHashTag(key);
            byte[] bytes = tag.getBytes(StandardCharsets.UTF_8);
            int crc = 0x0000;
            for (byte b : bytes) {
                int index = ((crc >> 8) ^ (b & 0xFF)) & 0xFF;
                crc = ((crc << 8) ^ CRC16_LOOKUP_TABLE[index]) & 0xFFFF;
            }
            return crc % TOTAL_SLOTS;
        }

        private static String extractHashTag(String key) {
            int start = key.indexOf('{');
            if (start != -1) {
                int end = key.indexOf('}', start + 1);
                if (end != -1 && end > start + 1) {
                    return key.substring(start + 1, end);
                }
            }
            return key;
        }
    }

    // =========================================================================
    // 2. Simulated Redis Cluster Nodes & Smart Client
    // =========================================================================
    public record ClusterNode(String nodeName, String address, int startSlot, int endSlot) {
        public boolean ownsSlot(int slot) {
            return slot >= startSlot && slot <= endSlot;
        }
    }

    public static class SmartRedisClusterClient {
        private final List<ClusterNode> knownNodes = new ArrayList<>();
        private final Map<Integer, ClusterNode> slotCache = new HashMap<>();

        public SmartRedisClusterClient(List<ClusterNode> nodes) {
            this.knownNodes.addAll(nodes);
            refreshSlotCache();
        }

        public void refreshSlotCache() {
            slotCache.clear();
            for (ClusterNode node : knownNodes) {
                for (int s = node.startSlot(); s <= node.endSlot(); s++) {
                    slotCache.put(s, node);
                }
            }
        }

        public String executeGet(String key) {
            int slot = RedisCrc16.getSlot(key);
            ClusterNode target = slotCache.get(slot);

            // Verify if target node owns the slot (simulating server check)
            if (target == null || !target.ownsSlot(slot)) {
                // Simulates receiving MOVED redirection from Redis server
                ClusterNode actualOwner = findOwner(slot);
                System.out.printf(Locale.US, "  [Smart Client] MOVED %d %s (Updating slot cache!)%n",
                        slot, actualOwner.address());
                slotCache.put(slot, actualOwner);
                target = actualOwner;
            }

            return String.format(Locale.US, "[SUCCESS] Fetched '%s' from %s (Slot %d)",
                    key, target.nodeName(), slot);
        }

        private ClusterNode findOwner(int slot) {
            for (ClusterNode n : knownNodes) {
                if (n.ownsSlot(slot)) return n;
            }
            throw new IllegalStateException("Slot unassigned: " + slot);
        }
    }

    // =========================================================================
    // 3. Main Benchmark Runner
    // =========================================================================
    public static void main(String[] args) {
        System.out.println("================================================================================");
        System.out.println("     REDIS CLUSTER 16,384 HASH SLOT ROUTER & SMART CLIENT POC                   ");
        System.out.println("================================================================================\n");

        // 3 Master Shards dividing 16,384 slots
        ClusterNode nodeA = new ClusterNode("Master-A", "10.0.1.10:6379", 0, 5460);
        ClusterNode nodeB = new ClusterNode("Master-B", "10.0.1.11:6379", 5461, 10922);
        ClusterNode nodeC = new ClusterNode("Master-C", "10.0.1.12:6379", 10923, 16383);

        List<ClusterNode> cluster = List.of(nodeA, nodeB, nodeC);
        SmartRedisClusterClient client = new SmartRedisClusterClient(cluster);

        System.out.println("--- [1. Verifying CRC16 Slot Distribution] ---");
        String[] testKeys = {
                "user:1001",
                "orders:9942",
                "inventory:item_55",
                "payment:card_88124"
        };

        for (String k : testKeys) {
            int slot = RedisCrc16.getSlot(k);
            System.out.printf(Locale.US, "Key: '%-20s' -> Slot: %5d -> Managed by: %s%n",
                    k, slot, (slot <= 5460 ? "Master-A" : (slot <= 10922 ? "Master-B" : "Master-C")));
        }

        System.out.println("\n--- [2. Verifying Hash Tag {...} Slot Co-Location] ---");
        String tagKey1 = "{account:42}.profile";
        String tagKey2 = "{account:42}.balance";
        String tagKey3 = "{account:42}.transactions";

        int slot1 = RedisCrc16.getSlot(tagKey1);
        int slot2 = RedisCrc16.getSlot(tagKey2);
        int slot3 = RedisCrc16.getSlot(tagKey3);

        System.out.printf(Locale.US, "Key: '%-25s' -> Slot: %d%n", tagKey1, slot1);
        System.out.printf(Locale.US, "Key: '%-25s' -> Slot: %d%n", tagKey2, slot2);
        System.out.printf(Locale.US, "Key: '%-25s' -> Slot: %d%n", tagKey3, slot3);

        if (slot1 == slot2 && slot2 == slot3) {
            System.out.println("✓ SUCCESS: Hash tags guarantee identical hash slots! Safe for atomic MGET/EVAL transactions.\n");
        } else {
            throw new AssertionError("Hash tags failed to co-locate slots!");
        }

        System.out.println("--- [3. Simulating Smart Client Request Execution] ---");
        for (String k : testKeys) {
            String result = client.executeGet(k);
            System.out.println(result);
        }

        System.out.println("\n================================================================================");
        System.out.println("                      VERIFICATION COMPLETED SUCCESSFULLY                       ");
        System.out.println("================================================================================");
    }
}
```

---

## 6. Principal Engineer Architectural Takeaways

1. **Redis Scalability Is Network and Slot Driven**: Single-threaded command execution ensures zero mutex overhead. Scale writes horizontally with Redis Cluster (16,384 slots) rather than scaling vertically to huge memory sizes.
2. **Use Hash Tags for Co-Located Multi-Key Transactions**: To perform atomic batch queries or transactions in Redis Cluster, wrap common entity keys in `{entity_id}` to prevent `CROSSSLOT` failures.
3. **Smart Clients Eliminate Centralized Reverse Proxies**: Always use cluster-aware clients (Lettuce, Jedis) that cache slot allocations locally and follow `MOVED` redirections directly to avoid proxy bottlenecks.
