# Engineering Mastery: Staff & Principal Technical Dossier

A comprehensive, production-grade curriculum and reference handbook for mastering software engineering, distributed systems, and technical architecture at the **Senior Tech Lead, Staff, and Principal Engineer** level.

Structured from first principles to distributed scale and operational reliability.

---

## 📚 Master Curriculum

### 01. System Design & Engineering Principles
Comprehensive architecture frameworks, fundamental building blocks, distributed systems theory, and operational reliability.

#### 00. The Framework
* [The System Design Interview Playbook: Staff & Principal Guide](01_System_Design_and_Principles/00_The_Framework/the_system_design_playbook.md) — 45-minute whiteboard cadence, back-of-the-envelope math, latency numbers every engineer should know, and failure defense.

#### 01. The Lego Blocks
* [Load Balancers](01_System_Design_and_Principles/01_The_Lego_Blocks/01_Load_Balancers/load_balancers.md) — L4 vs. L7, GSLB, routing algorithms, and SSL offloading.
* [Caching](01_System_Design_and_Principles/01_The_Lego_Blocks/02_Caching/caching.md) — Hardware, OS page cache, caching strategies, and eviction policies.
* [CDNs](01_System_Design_and_Principles/01_The_Lego_Blocks/03_CDNs/cdn.md) — Edge delivery, Anycast DNS, push vs. pull models.
* [API Gateways](01_System_Design_and_Principles/01_The_Lego_Blocks/04_API_Gateways/api_gateways.md) — Reverse proxy, auth, throttling, service discovery, and protocol translation.
* [Network Protocols](01_System_Design_and_Principles/01_The_Lego_Blocks/05_Network_Protocols/network_protocols.md) — TCP/UDP, TLS 1.3, HTTP/2 & HTTP/3 (QUIC), REST vs. GraphQL vs. gRPC.

#### 02. The Glue (Distributed Systems Theory)
* [Foundations & RPC](01_System_Design_and_Principles/02_The_Glue/01_foundations_and_rpc.md) — Message passing, location transparency, IDLs.
* [System Models & Faults](01_System_Design_and_Principles/02_The_Glue/02_system_models_and_faults.md) — Crash-stop, crash-recovery, Byzantine faults, synchronous vs. asynchronous networks.
* [Time & Clocks](01_System_Design_and_Principles/02_The_Glue/03_time_and_clocks.md) — Physical clocks, NTP, clock drift, Lamport timestamps, Vector clocks.
* [Broadcast](01_System_Design_and_Principles/02_The_Glue/04_broadcast.md) — FIFO, Causal, and Total Order Broadcast protocols.
* [Replication](01_System_Design_and_Principles/02_The_Glue/05_replication.md) — Single-leader, multi-leader, quorum replication ($W + R > N$).
* [Consensus](01_System_Design_and_Principles/02_The_Glue/06_consensus.md) — The FLP Impossibility Result, Two-Phase Commit (2PC), Raft, and Paxos.
* [Replica Consistency](01_System_Design_and_Principles/02_The_Glue/07_replica_consistency.md) — Linearizability vs. Serializability, Read-after-write, Monotonic reads.
* [Case Studies](01_System_Design_and_Principles/02_The_Glue/08_case_studies.md) — Google Spanner (TrueTime), Apache ZooKeeper (ZAB).

#### 03. Operational Reality (Google SRE Standard)
* [SLOs & Error Budgets](01_System_Design_and_Principles/03_Operational_Reality/01_slos_and_error_budgets.md) — SLI math, downtime by nines, multi-window multi-burn-rate alerting, and error budget governance.
* [The 4 Golden Signals & Observability](01_System_Design_and_Principles/03_Operational_Reality/02_the_4_golden_signals.md) — Latency, Traffic, Errors, Saturation; Tail-at-scale amplification math; Google Signals vs. RED vs. USE; PromQL queries.
* [Handling Overload & Graceful Degradation](01_System_Design_and_Principles/03_Operational_Reality/03_handling_overload.md) — Little's Law, LIFO vs. FIFO queuing under load, CoDel load shedding, AWS Full Jitter backoff, and Resilience4j.
* [Cascading Failures & Resiliency](01_System_Design_and_Principles/03_Operational_Reality/04_cascading_failures.md) — Circuit breakers, distributed deadline propagation, bulkheading, and Kubernetes Liveness vs. Readiness probe traps.

---

### 🚀 Upcoming Modules
* **02. Design Patterns, Microservices & Distributed Patterns** (`CORE.4`)
* **03. Java Deep Dive & Concurrency** (`CORE.5` & `CORE.6`)
* **04. Spring Framework Ecosystem** (`CORE.7`)
* **05. API Design & Gateway Patterns** (`CORE.8`)
* **06. Apache Kafka & Event-Driven Architecture** (`CORE.9`)
* **07. Storage Engines & Databases (Oracle, MongoDB, Distributed DBs)** (`CORE.10`)
* **08. AWS Cloud Architecture & Resiliency** (`CORE.11`)
* **09. Security Architecture & Zero Trust** (`CORE.12`)
* **10. Agentic AI, LLMs & RAG Architecture** (`CORE.13`)

---

## 📖 Primary References
* Martin Kleppmann, *Distributed Systems* Lecture Series, Cambridge University
* Google Site Reliability Engineering (SRE) Books (*Site Reliability Engineering* & *The Site Reliability Workbook*)
* Alex Xu, *System Design Interview* (Volumes 1 & 2)
* Martin Kleppmann, *Designing Data-Intensive Applications* (DDIA)
* [ByteByteGo Architecture Series](https://www.youtube.com/@ByteByteGo)
