# Lecture 6: Consensus

As we saw in Lecture 5, **Total Order Broadcast** is incredibly powerful for database replication, but it usually relies on a single Leader node to sequence the messages. If that leader crashes, the system stops. 

To overcome this, systems historically relied on **Manual Failover**: a human operator gets notified of the crash and manually reconfigures the nodes to point to a new leader. While fine for planned maintenance, it is far too slow for unplanned outages. To make the system truly fault-tolerant, we need the nodes to automatically agree on who the *new* leader should be.

This requires **Consensus**—an automatic leader election.

## 6.1 Introduction to Consensus

The Consensus problem is defined as: *Several nodes want to come to an agreement about a single value. Once a decision is made, it is final.*

It has been mathematically proven that **Consensus and Total Order Broadcast are exactly equivalent**. If you can solve one, you can solve the other.
* **Famous Algorithms:** Paxos, Multi-Paxos, Raft, Zab (ZooKeeper).

### System Models and The FLP Impossibility Result
A famous theorem by Fischer, Lynch, and Paterson (FLP) proved that in a purely *asynchronous* system (where messages can be delayed infinitely), **no deterministic consensus algorithm is guaranteed to ever terminate**. 
* Because of this, practical consensus algorithms like Raft must assume a **Partially Synchronous** system. They rely on local clocks and timeouts to detect leader crashes and ensure the algorithm eventually makes progress.

Algorithms like Raft and Paxos assume a **Crash-Recovery** system model with fair-loss links. There are also consensus algorithms designed for a **Byzantine** system model (used in blockchains), but they are significantly more complex and less efficient than crash-recovery consensus.

### Leader Election and Split Brain
When a leader crashes, the remaining nodes must elect a new one. The biggest danger during an election is **Split Brain**—a scenario where two different nodes both believe they are the leader at the same time. If this happens, they will sequence updates differently, destroying the database's consistency.

```mermaid
graph TD
    subgraph Election Quorum for Node A (Majority)
    A((Node A))
    B((Node B))
    C((Node C))
    end
    
    subgraph Election Quorum for Node E (Majority)
    C
    D((Node D))
    E((Node E))
    end
    
    style C fill:#f8d7da,stroke:#dc3545,stroke-width:4px
    
    Note[Because Node C can only vote ONCE per term,<br/>it is mathematically impossible for both<br/>Node A and Node E to win the election.]
```

To prevent Split Brain, consensus algorithms use **Quorums**. A node can only become the leader if it receives votes from a *majority quorum* of the nodes in the cluster. Because any two majority quorums must overlap by at least one node, it is mathematically impossible for two nodes to win an election at the same time *in the same term*.

### Multiple Leaders Across Terms and Quorum Verification
While we can guarantee only one leader per term, we cannot prevent multiple leaders from existing across *different* terms due to network partitions. 

```mermaid
graph LR
    subgraph Partition 1
    Node1((Node 1<br/>Leader Term t))
    end
    
    subgraph Partition 2
    Node2((Node 2<br/>Follower))
    Node3((Node 3<br/>Follower))
    end
    
    Node1 -. "Network Partition" .- Node2
    Node1 -. "Network Partition" .- Node3
    
    Note[Node 2 and 3 elect a new leader in Term t+1.<br/>Node 1 still thinks it is the leader!]
```

Because of this, an old leader might still be blindly accepting requests, completely unaware that it has been usurped! To guarantee safety, **a leader must verify its quorum before delivering any message**. 

```mermaid
sequenceDiagram
    participant L as Leader
    participant F1 as Follower 1
    participant F2 as Follower 2
    
    Note over L,F2: Leader Election Phase (Term t)
    L->>F1: Shall I be leader?
    L->>F2: Shall I be leader?
    F1-->>L: Yes
    F2-->>L: Yes
    
    Note over L,F2: Message Delivery Phase
    L->>F1: Can we deliver message 'm'?
    L->>F2: Can we deliver message 'm'?
    F1-->>L: Okay
    F2-->>L: Okay
    L->>F1: Deliver 'm' please
    L->>F2: Deliver 'm' please
```
If a new leader had been elected, the old leader would find out during this second round-trip because at least one follower in the quorum would have already voted for the new term and would reject the old leader's request.

---

## 6.2 The Raft Consensus Algorithm

Raft is a consensus algorithm explicitly designed to provide a fault-tolerant **FIFO-Total Order Broadcast** out of the box.

### Node States
In Raft, a node is always in one of three states:
1. **Follower:** Passively receives log entries from the Leader.
2. **Candidate:** If a Follower stops hearing from the Leader (its randomized election timeout fires), it becomes a Candidate and requests votes from the cluster.
3. **Leader:** The node that won the election. It accepts writes from clients and replicates them to the Followers.

```mermaid
stateDiagram-v2
    [*] --> Follower: Starts up
    Follower --> Candidate: Suspects leader failure (timeout)
    Candidate --> Follower: Discovers a valid leader with a higher term
    Candidate --> Leader: Receives votes from a majority quorum
    Leader --> Follower: Discovers a new leader with a higher term
```

### Log Consistency Checks
Raft divides time into **Terms** (an incrementing integer). Every leader election increments the Term.
To ensure no committed data is ever lost, a follower performs a strict **Log Consistency Check** before accepting a candidate's vote request or a leader's append request. The follower checks that its local log matches the sender's log. Specifically, they check that the *term* of the last log entry matches, and if so, that the lengths are consistent. If a follower's log is found to be inconsistent with a new leader, the follower truncates its local log to match the leader's exact history.

### The Full Raft Algorithm Pseudocode

Below is the complete pseudocode for the Raft algorithm, spanning node initialization, leader election, and log replication.

```python
# 1. Initialisation
on initialisation do:
    currentTerm := 0; votedFor := null
    log := []; commitLength := 0
    currentRole := follower; currentLeader := null
    votesReceived := {}; sentLength := []; ackedLength := []

on recovery from crash do:
    currentRole := follower; currentLeader := null
    votesReceived := {}; sentLength := []; ackedLength := []

on node nodeId suspects leader has failed, or on election timeout do:
    currentTerm := currentTerm + 1; currentRole := candidate
    votedFor := nodeId; votesReceived := {nodeId}; lastTerm := 0
    if log.length > 0 then lastTerm := log[log.length - 1].term
    msg := (VoteRequest, nodeId, currentTerm, log.length, lastTerm)
    for each node in nodes: send msg to node
    start election timer

# 2. Voting on a new leader
on receiving (VoteRequest, cId, cTerm, cLogLength, cLogTerm) at node nodeId do:
    if cTerm > currentTerm then:
        currentTerm := cTerm; currentRole := follower
        votedFor := null
    lastTerm := 0
    if log.length > 0 then lastTerm := log[log.length - 1].term
    logOk := (cLogTerm > lastTerm) or (cLogTerm == lastTerm and cLogLength >= log.length)
    if cTerm == currentTerm and logOk and votedFor in {cId, null} then:
        votedFor := cId
        send (VoteResponse, nodeId, currentTerm, true) to node cId
    else:
        send (VoteResponse, nodeId, currentTerm, false) to node cId

# 3. Collecting votes
on receiving (VoteResponse, voterId, term, granted) at nodeId do:
    if currentRole == candidate and term == currentTerm and granted then:
        votesReceived := votesReceived U {voterId}
        if len(votesReceived) >= (len(nodes) + 1) // 2:
            currentRole := leader; currentLeader := nodeId
            cancel election timer
            for each follower in nodes \ {nodeId} do:
                sentLength[follower] := log.length
                ackedLength[follower] := 0
                ReplicateLog(nodeId, follower)
    else if term > currentTerm then:
        currentTerm := term
        currentRole := follower
        votedFor := null
        cancel election timer

# 4. Broadcasting messages
on request to broadcast msg at node nodeId do:
    if currentRole == leader then:
        append the record (msg: msg, term: currentTerm) to log
        ackedLength[nodeId] := log.length
        for each follower in nodes \ {nodeId} do:
            ReplicateLog(nodeId, follower)
    else:
        forward the request to currentLeader via a FIFO link

periodically at node nodeId do:
    if currentRole == leader then:
        for each follower in nodes \ {nodeId} do:
            ReplicateLog(nodeId, follower)

# 5. Replicating from leader to followers
function ReplicateLog(leaderId, followerId):
    prefixLen := sentLength[followerId]
    suffix := [log[prefixLen], ..., log[log.length - 1]]
    prefixTerm := 0
    if prefixLen > 0 then prefixTerm := log[prefixLen - 1].term
    send (LogRequest, leaderId, currentTerm, prefixLen, prefixTerm, commitLength, suffix) to followerId

# 6. Followers receiving messages
on receiving (LogRequest, leaderId, term, prefixLen, prefixTerm, leaderCommit, suffix) at node nodeId do:
    if term > currentTerm then:
        currentTerm := term; votedFor := null
        cancel election timer
    if term == currentTerm then:
        currentRole := follower; currentLeader := leaderId
    
    logOk := (log.length >= prefixLen) and (prefixLen == 0 or log[prefixLen - 1].term == prefixTerm)
    if term == currentTerm and logOk then:
        AppendEntries(prefixLen, leaderCommit, suffix)
        ack := prefixLen + suffix.length
        send (LogResponse, nodeId, currentTerm, ack, true) to leaderId
    else:
        send (LogResponse, nodeId, currentTerm, 0, false) to leaderId

# 7. Updating followers' logs
function AppendEntries(prefixLen, leaderCommit, suffix):
    if suffix.length > 0 and log.length > prefixLen then:
        index := min(log.length, prefixLen + suffix.length) - 1
        if log[index].term != suffix[index - prefixLen].term then:
            log := [log[0], ..., log[prefixLen - 1]]
    if prefixLen + suffix.length > log.length then:
        for i := log.length - prefixLen to suffix.length - 1 do:
            append suffix[i] to log
    if leaderCommit > commitLength then:
        for i := commitLength to leaderCommit - 1 do:
            deliver log[i].msg to the application
        commitLength := leaderCommit

# 8. Leader receiving log acknowledgements
on receiving (LogResponse, follower, term, ack, success) at nodeId do:
    if term == currentTerm and currentRole == leader then:
        if success == true and ack >= ackedLength[follower] then:
            sentLength[follower] := ack
            ackedLength[follower] := ack
            CommitLogEntries()
        else if sentLength[follower] > 0 then:
            sentLength[follower] := sentLength[follower] - 1
            ReplicateLog(nodeId, follower)
    else if term > currentTerm then:
        currentTerm := term
        currentRole := follower
        votedFor := null
        cancel election timer

# 9. Leader committing log entries
function CommitLogEntries():
    minAcks := (len(nodes) + 1) // 2
    ready := {len in {1, ..., log.length} | acks(len) >= minAcks}
    if ready != {} and max(ready) > commitLength and log[max(ready) - 1].term == currentTerm then:
        for i := commitLength to max(ready) - 1 do:
            deliver log[i].msg to the application
        commitLength := max(ready)
```

> [!IMPORTANT]
> **Staff Insight: Raft Log Invariants & Leader Completeness**
> 1. **Leader Completeness**: Raft guarantees that if a log entry is committed in a given term, that entry will be present in the logs of the leaders for all higher terms.
> 2. **Election Restriction Rule**: A voter denies its vote if the candidate’s log is less up-to-date than its own (comparing `lastLogTerm` first, then `log.length`). Node A (holding older Term 1 entries) can never win an election over Node B or C holding Term 2/3 entries.
> 3. **Log Reconciliation**: A newly elected leader never truncates its own log; it forces followers to match its log by decrementing `nextIndex[follower]` until a common ancestor is found, overwriting any uncommitted conflicting follower entries.

---

## Staff / Principal Interview Strategy: Consensus vs. Two-Phase Commit (2PC)

The difference between **Consensus (Raft/Paxos)** and **Atomic Commitment (2PC)** is one of the most frequently tested concepts in Staff and Principal distributed systems rounds:

```mermaid
flowchart TD
    subgraph Consensus ["Consensus (Raft / Paxos)"]
        direction TB
        C1["Goal: Replicate same state machine across identical nodes"]
        C2["Agreement: Requires STRICT MAJORITY (⌊N/2⌋ + 1)"]
        C3["Fault Tolerance: Tolerates minority node crashes cleanly"]
        C4["Used In: etcd, ZooKeeper, CockroachDB ranges, Spanner groups"]
    end

    subgraph TwoPC ["Atomic Commitment (Two-Phase Commit - 2PC)"]
        direction TB
        T1["Goal: Coordinate all-or-nothing transactions across DIFFERENT shards"]
        T2["Agreement: Requires 100% UNANIMOUS consent from all participants"]
        T3["Fault Tolerance: FRAGILE. Blocks indefinitely if coordinator crashes mid-commit"]
        T4["Used In: Distributed XA transactions, cross-shard RDBMS ACID"]
    end
```

### 1. The Definitive Distinction
| Dimension | **Consensus (Raft / Paxos)** | **Two-Phase Commit (2PC)** |
| :--- | :--- | :--- |
| **Problem Solved** | State Machine Replication (SMR) among homogeneous replicas. | Distributed Transaction Atomicity across heterogeneous systems. |
| **Quorum Needed** | **Majority ($\lfloor N/2 \rfloor + 1$)**. | **Unanimous (100% of participants)**. |
| **Crash Behavior** | Continuous progress if minority fails. | **Blocks indefinitely** holding locks if coordinator crashes in Phase 2. |
| **Can a node vote NO?**| No (unless invalid term/log). | Yes (if constraint violated, aborts entire transaction). |

### 2. How Modern Distributed Databases Combine Both (The Spanner Pattern)
Google Spanner and CockroachDB combine both algorithms to achieve high-performance distributed transactions:
* They use **Two-Phase Commit (2PC)** across different table partitions to commit multi-shard transactions.
* **The Genius Fix for 2PC Blocking**: Each participant in the 2PC transaction is *not* a single physical server; it is a **Paxos/Raft Consensus Group**!
* If the primary coordinator crashes mid-transaction, its Paxos group elects a new leader in milliseconds. The new leader recovers the coordinator's state and completes the 2PC protocol—**completely eliminating the 2PC blocking vulnerability**!

### 3. The FLP Impossibility Result (Fischer, Lynch, Paterson)
* **The Theorem**: In an asynchronous network, no deterministic consensus protocol can guarantee both **Safety** (never making a wrong decision) and **Liveness** (always reaching a decision) if even a single node can fail.
* **How Real Systems Bypass FLP**: Real systems (Raft, Paxos) choose **Partial Synchrony**. They prioritize 100% Safety (never committing divergent data) and use randomized election timeouts to guarantee Liveness with high probability.
