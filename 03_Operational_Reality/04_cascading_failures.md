# Cascading Failures

## Academic Definition
A **Cascading Failure** is a failure that grows over time as a result of a positive feedback loop. It occurs when a portion of a system fails, shifting its load to other parts of the system, which then causes those parts to become overloaded and fail in a domino effect until the entire cluster or service collapses.

## Primary Causes
1. **Resource Exhaustion:** Service A calls Service B. Service B gets slow. Service A waits, holding open threads/connections indefinitely until it runs out of memory or threads and crashes.
2. **Thundering Herd / Retry Storms:** A service blips briefly, and all clients retry simultaneously. This sudden spike of retries overwhelms the service instantly as it tries to recover, keeping it down.
3. **Death Marches (Load Redistribution):** Node 1 in a 3-node cluster crashes. Its traffic is redistributed to Nodes 2 and 3. The remaining nodes cannot handle the sudden 50% increase in load and crash. The system is unrecoverable until load is artificially halted.

## Prevention Mechanisms
1. **Circuit Breakers:** If Service B is failing or timing out, Service A uses a Circuit Breaker pattern to "trip" (stop sending requests entirely) for a set cooldown period. This prevents Service A from wasting resources waiting, and gives Service B breathing room to recover.
2. **Timeouts and Deadlines:** *Never* make an unbounded network call. Every RPC/HTTP call must have a strict timeout. Propagate deadlines across microservices (e.g., if the user times out after 2s, downstream services should cancel their work after 2s to save CPU).
3. **Bounded Queues:** Do not use infinite queues. If the queue is full, drop new messages immediately (Fail Fast).

## Capacity Planning (N+1 or N+2)
Design clusters to handle peak traffic even if 1 or 2 availability zones (AZs) or nodes go down. If you have 3 AZs and 1 goes down, the remaining 2 must have enough headroom to absorb the extra 33% of traffic without crossing their saturation limits.

## Principal Engineer Interview Context
Mention Circuit Breakers (Resilience4j / Hystrix) and Distributed Timeouts as mandatory architectural principles. In your Mastercard 3-AZ story, this is exactly what splitting liveness from readiness prevents (stopping the domino effect of rolling restarts).
