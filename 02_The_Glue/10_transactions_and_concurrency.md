# 10. Transactions & Concurrency (DDIA)

**Goal:** Ensure that in a chaotic world of concurrent requests, hardware failures, and network drops, the database remains in a consistent state.

## 1. ACID Properties (Simplified)
*   **Atomicity:** "All or Nothing." If a transaction fails halfway, any changes made are safely rolled back. (Prevents half-finished states).
*   **Consistency:** The application's rules are enforced (e.g., account balance cannot go below zero). 
*   **Isolation:** Concurrent transactions don't step on each other's toes. (This is the hardest one to achieve at scale).
*   **Durability:** Once a transaction commits, it is written to non-volatile memory (disk) and will survive a power loss.

## 2. Isolation Levels (From Weakest to Strongest)
When multiple users edit data at the same time, weird things (Race Conditions) happen. Isolation levels define how much "weirdness" you tolerate for the sake of speed.

1.  **Read Uncommitted:** You can read data that another transaction hasn't saved yet (Dirty Read). *Terrible idea.*
2.  **Read Committed (Default for many DBs like Postgres/Oracle):** You only see saved data. Solves Dirty Reads, but allows "Non-repeatable reads" (reading the same row twice in a transaction gives different results if someone else committed an update).
3.  **Repeatable Read / Snapshot Isolation:** A transaction takes a "snapshot" of the DB. You see the data exactly as it was when you started, regardless of what others commit. (Solves most concurrency issues).
4.  **Serializable:** The strongest. It executes transactions as if they were running one-by-one in sequence. *Very safe, but incredibly slow.*

## 3. Distributed Transactions (Microservices)
How do you do a transaction when Service A (Order) and Service B (Payment) have different databases?

### A. Two-Phase Commit (2PC) - *The Legacy Way*
- Uses a central "Coordinator".
- **Phase 1:** Coordinator asks all DBs "Are you ready to commit?" DBs lock their rows and say "Yes".
- **Phase 2:** Coordinator says "Commit!"
- **Why it's bad:** It's blocking. If the Coordinator crashes in Phase 1, the DB rows stay locked forever. Terrible for microservices.

### B. The Saga Pattern - *The Cloud-Native Way*
- Break the distributed transaction into a sequence of local transactions.
- If Step 3 fails, you don't "rollback" the database. Instead, you run **Compensating Transactions** (Step 2_Undo, Step 1_Undo).
- **Choreography:** Services publish events to Kafka. Other services listen and react. (Good for simple flows).
- **Orchestration:** A central service (like Camunda or AWS Step Functions) commands other services what to do. (Good for complex flows).

## Interview Cheat Sheet 💡
*   **"How do you prevent two users from booking the same seat?"** -> Optimistic Locking (version column in DB) or pessimistic locking (`SELECT FOR UPDATE`).
*   **"How do you handle transactions across two microservices?"** -> Strongly argue against 2PC. Propose the Saga Pattern (Event-Driven architecture).
