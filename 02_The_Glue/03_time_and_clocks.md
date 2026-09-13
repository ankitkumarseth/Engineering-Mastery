# Lecture 3: Time, Clocks, and Ordering of Events

Time is critical in distributed systems for timeouts, scheduling, and determining the order of events across multiple nodes. However, measuring time consistently across a network is incredibly difficult.

## 3.1 Physical Clocks

Physical clocks measure elapsed seconds. 
* **Quartz Clocks:** Cheap and ubiquitous, but they suffer from **clock drift** (running slightly fast or slow due to temperature or manufacturing imperfections). A typical quartz clock drifts by about 1 second every 12 days (~1 ppm).
* **Atomic Clocks:** Extremely accurate, measuring the resonant frequency of atoms (like Caesium-133). Very expensive and bulky.
* **GPS Clocks:** Satellites broadcast highly accurate atomic time, but require a clear line of sight to the sky (meaning data centers need roof antennas).

### Coordinated Universal Time (UTC) and Leap Seconds
Because the Earth's rotation is not perfectly constant, astronomical time drifts away from atomic time. To keep them aligned, **leap seconds** are occasionally inserted into UTC. 
* A minute might occasionally have 61 seconds (e.g., `23:59:60`).
* Software that relies on sub-second precision often crashes or enters livelocks during leap seconds (e.g., the 2012 Reddit/Linux crash).
* **Smearing:** To avoid crashes, companies like Google "smear" the leap second by running their clocks slightly slower over a 24-hour period, hiding the leap second from applications.

### Timestamp Representations
> [!NOTE]
> Two common representations of timestamps are used in computing:
> * **Unix time:** The number of seconds since 1 January 1970 00:00:00 UTC (the "epoch"), excluding leap seconds.
> * **ISO 8601:** Year, month, day, hour, minute, second, and timezone offset relative to UTC (e.g., `2021-11-09T09:50:17+00:00`).

**Exercise 6.** Describe some problems that may arise from leap second smearing.

---

## 3.2 Clock Synchronization and Monotonic Clocks

Because quartz clocks drift, computers must periodically sync their clocks with highly accurate servers over the network using the **Network Time Protocol (NTP)**.

### NTP Strata Hierarchy and Statistical Filtering
NTP servers are arranged into a hierarchy of **strata**:
* **Stratum 0:** Atomic clock or GPS receiver.
* **Stratum 1:** Servers synced directly with a stratum 0 device.
* **Stratum 2:** Servers that sync with stratum 1, etc.

NTP clients contact multiple servers, discard outliers, and average the rest. They make multiple requests to the same server and use statistics to reduce random error due to variations in network latency.

### How NTP Estimates Clock Skew
NTP attempts to calculate the difference (skew) between the client's clock and the server's clock by measuring the round-trip network time. 

```mermaid
sequenceDiagram
    participant Client
    participant Server
    
    Client->>Server: Request (t1)
    Note over Server: Receives at t2
    Note over Server: Processes
    Server->>Client: Response (t1, t2, t3)
    Note over Client: Receives at t4
```

The round-trip network delay $\delta$ and estimated clock skew $\theta$ are calculated as:
* **Round-trip delay:** $\delta = (t_4 - t_1) - (t_3 - t_2)$
* **Estimated clock skew:** $\theta = \frac{t_3 + \delta/2 - t_4}{1} = \frac{t_2 - t_1 + t_3 - t_4}{2}$

*Assumption:* It assumes network latency is perfectly symmetric (takes the same amount of time to travel there as it does to come back).
*Problem:* If network congestion causes asymmetric latency, the NTP time estimation will be inherently inaccurate.

**Exercise 7.** What is the maximum possible error in the NTP client's estimate of skew with regard to one particular server, assuming that both nodes correctly follow the protocol?

### Slewing vs Stepping
If a computer's clock is out of sync, how does NTP fix it?
* **Slewing:** If the error is small (< 125ms), NTP slightly speeds up or slows down the local clock until it catches up.
* **Stepping:** If the error is large, NTP violently overwrites the local clock with the server's time. *The clock will visibly jump forwards or backwards.*

If the skew is very large ($\ge$ 1,000 s), NTP may panic and do nothing, requiring operator intervention.

### Two Types of Clocks in Programming
Because of NTP stepping, developers must be extremely careful which clock they use:

1. **Time-of-Day Clocks (Real-time):**
   * Returns the time since an epoch (e.g., `System.currentTimeMillis()` in Java).
   * Can be stepped backwards by NTP.
   * **DANGER:** Never use this to measure elapsed duration, as you might get a negative time difference!

2. **Monotonic Clocks:**
   * Returns time since an arbitrary point (e.g., when the machine booted).
   * Guaranteed to always move forward. Cannot jump backwards.
   * Only useful for measuring elapsed duration on a *single node*. Meaningless when compared across different nodes. (e.g., `System.nanoTime()` in Java).

```java
// BAD: Time-of-day clock (may jump backwards)
long startTime = System.currentTimeMillis();
doSomething();
long endTime = System.currentTimeMillis();
long elapsedMillis = endTime - startTime; // Could be negative!

// GOOD: Monotonic clock (always moves forward)
long startTime = System.nanoTime();
doSomething();
long endTime = System.nanoTime();
long elapsedNanos = endTime - startTime; // Guaranteed >= 0
```

---

## 3.3 Causality and Happens-Before

Since physical clocks are unreliable across a network, physical timestamps can be inconsistent with expected causality. For example, even with synced clocks, if A sends $m_1$ at $t_1$ and B replies with $m_2$ at $t_2$, it's possible that $t_2 < t_1$ if A's clock is slightly ahead of B's.

```mermaid
sequenceDiagram
    participant A as User A
    participant B as User B
    participant C as User C
    
    A->>B: m1 (t1)
    A->>C: m1 (t1)
    B->>C: m2 (t2)
    B->>A: m2 (t2)
    Note over A,C: m2 is a reply to m1, but we could have t2 < t1! Timestamp order is inconsistent with expected order.
```

We need a mathematical way to reason about the order of events without relying on physical time. We borrow the concept of **causality** from physics.

> [!NOTE]
> If Event A sends a message that causes Event B to happen, then A must have occurred before B. It is impossible for an effect to precede its cause.

### The Happens-Before Relation ($\to$)

We define the "happens-before" relation mathematically. Event `a` happens-before event `b` (written `a → b`) if:
1. `a` and `b` occurred on the same node, and `a` executed first.
2. `a` is the sending of a message, and `b` is the receipt of that *same* message.
3. Transitivity: If `a → c` and `c → b`, then `a → b`.

```mermaid
sequenceDiagram
    participant A
    participant B
    participant C
    
    Note over A: Event a
    Note over A: Event b
    Note over B: Event c
    A->>B: m1
    Note over B: Event d (receives m1)
    Note over A: Event e
    B->>C: m2
    Note over C: Event f (receives m2)
    
    Note over A,C: a → b, c → d, e → f (node execution order)<br/>b → c (m1), d → f (m2)<br/>a → f, b → f (transitivity)
```

### Concurrent Events ($\parallel$)
The happens-before relation is a **partial order**. It does not order *everything*.
If two events happen on different nodes, and no messages are exchanged between them, we say they are **concurrent** (written `a || b`). 

* Note: "Concurrent" in distributed systems does *not* mean they happened at the exact same physical time. It simply means they are completely independent and neither one influenced the other.

**Exercise 8.** Prove that the happens-before relation is a strict partial order.

**Exercise 9.** Show that for any two events $a$ and $b$, exactly one of the three following statements must be true: either $a \to b$, or $b \to a$, or $a \parallel b$.
