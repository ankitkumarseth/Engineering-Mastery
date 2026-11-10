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

### 02. Design Patterns, Microservices & Distributed Patterns
Enterprise design patterns in modern Java, domain-driven decomposition, and distributed consistency.

#### 01. GoF Design Patterns (Modern Java)
* [Creational Patterns](02_Design_Patterns_and_Microservices/01_GoF_Design_Patterns/01_creational_patterns.md) — Volatile Double-Checked Locking, Bill Pugh, Enum Singleton, Builder pattern vs. Java Records, functional factories.
* [Structural Patterns](02_Design_Patterns_and_Microservices/01_GoF_Design_Patterns/02_structural_patterns.md) — Dynamic Proxies vs. CGLIB, Spring AOP self-invocation traps, Decorator telemetry layering, and Adapter ACLs.
* [Behavioral Patterns](02_Design_Patterns_and_Microservices/01_GoF_Design_Patterns/03_behavioral_patterns.md) — Strategy via lambdas, Map-injection in Spring, in-process vs. distributed Observers, Template Callbacks, and Chain of Responsibility.

#### 02. Domain-Driven Design (DDD)
* [Strategic DDD](02_Design_Patterns_and_Microservices/02_Domain_Driven_Design/01_strategic_ddd.md) — Ubiquitous Language, Subdomains (Core vs. Supporting vs. Generic), Bounded Contexts, and Context Mapping (ACL, Shared Kernel).
* [Tactical DDD](02_Design_Patterns_and_Microservices/02_Domain_Driven_Design/02_tactical_ddd.md) — Entities vs. Value Objects, Aggregate Roots as transactional boundaries, and eradicating the Anemic Domain Model.

#### 03. Microservice & Distributed Architecture Patterns
* [Monolith Decomposition](02_Design_Patterns_and_Microservices/03_Microservice_Architecture/01_monolith_decomposition.md) — Strangler Fig migration pattern, Anti-Corruption Layer (ACL), and Database-per-Service vs. Shared Database.
* [Distributed Transactions & Sagas](02_Design_Patterns_and_Microservices/03_Microservice_Architecture/02_distributed_transactions_and_sagas.md) — Saga Orchestration vs. Choreography, Pivot transactions, Compensating actions, and Semantic Locking.
* [Event Sourcing & CQRS](02_Design_Patterns_and_Microservices/03_Microservice_Architecture/03_event_sourcing_and_cqrs.md) — The Dual-Write trap, Transactional Outbox Pattern with Debezium CDC, CQRS read projections, and Event Sourcing.
* [Cloud-Native Architecture & Resilience](02_Design_Patterns_and_Microservices/03_Microservice_Architecture/04_cloud_native_and_resilience.md) — Modernized 12-Factor App, Sidecar & Ambassador patterns, and W3C TraceContext context propagation with OpenTelemetry.

---

### 03. Java Deep Dive, Concurrency & JVM Mastery
Core JVM memory layout, high-performance concurrency, JMM mechanics, low-latency GC, and modern Java 17–21 features.

#### 01. Collections & Data Structures
* [Collections Framework Refresher & Python-to-Java Rosetta Stone](03_Java_Deep_Dive_and_Concurrency/01_Collections_and_Data_Structures/00_collections_framework_refresher.md) — Fundamental Collections hierarchy, List vs. Set vs. Map vs. Deque, Python-to-Java syntax cheat sheet, ArrayDeque stack/queue, sorting comparators, safe removal idioms, and runnable verification suite.
* [Collections Internals & Performance Trivia](03_Java_Deep_Dive_and_Concurrency/01_Collections_and_Data_Structures/01_collections_internals_and_trivia.md) — ArrayList 1.5x growth factor, CPU cache lines vs. LinkedList fragmentation, Fail-Fast `modCount` vs. Fail-Safe iterators, LinkedHashMap LRU cache, and Java 21 Sequenced Collections.
* [HashMap & ConcurrentHashMap Deep Dive](03_Java_Deep_Dive_and_Concurrency/01_Collections_and_Data_Structures/02_hashmap_and_concurrent_hashmap_deep_dive.md) — Bitwise hash spreading `h ^ (h >>> 16)`, power-of-two index masking `(n - 1) & hash`, Treeification threshold (8/64), Java 8 CAS + bin-head `synchronized` locking, and `CounterCell` striped sizing.

#### 02. Threading Basics & Lifecycle
* [Threading Fundamentals & Thread Coordination](03_Java_Deep_Dive_and_Concurrency/02_Threading_Basics_and_Lifecycle/01_threading_fundamentals_and_coordination.md) — Thread vs. Runnable vs. Callable, 6 JVM Thread States, `wait()`/`notify()` in `while` loops, Lost Wakeup race conditions, cooperative interruption flags, and runnable BoundedBlockingQueue.

#### 03. Concurrency, JMM & Locks
* [Java Memory Model (JMM), Volatile & Atomics](03_Java_Deep_Dive_and_Concurrency/03_Concurrency_JMM_and_Locks/01_jmm_volatile_and_atomics.md) — MESI cache coherence protocol, Store Buffers, Happens-Before guarantees, `volatile` memory barriers, DCL singleton bug, lock-free CAS, `LongAdder` cell striping, and ABA avoidance with `AtomicStampedReference`.
* [Locks, Conditions & AbstractQueuedSynchronizer (AQS)](03_Java_Deep_Dive_and_Concurrency/03_Concurrency_JMM_and_Locks/02_locks_conditions_and_aqs.md) — Biased/Lightweight/Heavyweight monitor escalation, ReentrantLock non-fair thread barging, AQS state & CLH FIFO queue mechanics, Condition variables, and StampedLock optimistic reads.
* [ThreadPoolExecutor & Asynchronous Pipelines](03_Java_Deep_Dive_and_Concurrency/03_Concurrency_JMM_and_Locks/03_threadpools_and_async_orchestration.md) — ThreadPoolExecutor lifecycle, unbounded queue OOM trap, `CallerRunsPolicy` backpressure, thread pool sizing math (Little's Law), and `CompletableFuture` async orchestration.
* [Virtual Threads & Project Loom](03_Java_Deep_Dive_and_Concurrency/03_Concurrency_JMM_and_Locks/04_virtual_threads_project_loom.md) — M:N Loom carrier thread architecture, Continuation yield mechanics, the `synchronized` carrier thread pinning trap and `ReentrantLock` antidote, and why pooling virtual threads is an anti-pattern.

#### 04. JVM Internals & Garbage Collection
* [JVM Memory Layout, Compressed OOPs & JIT Compilation](03_Java_Deep_Dive_and_Concurrency/04_JVM_Internals_and_GC/01_jvm_memory_layout_and_jit.md) — Stack Frames, LVT, Operand Stack, Metaspace vs. Heap, 64-bit Object Header Layout (JOL), the 32GB Compressed OOPs cliff, Tiered Compilation (Tiers 0–4), Escape Analysis, Scalar Replacement, and `ClassNotFoundException` vs. `NoClassDefFoundError`.
* [Garbage Collection: G1GC vs. Generational ZGC Architecture](03_Java_Deep_Dive_and_Concurrency/04_JVM_Internals_and_GC/02_garbage_collection_g1_and_zgc.md) — Weak Generational Hypothesis, G1GC 2048 regions, RSet Card Table, SATB concurrent marking, pause-time goals vs. Evacuation Failure, ZGC Colored Pointers, Self-Healing Load Barriers, and Java 21 Generational ZGC.

#### 05. Modern Java & Microbenchmarking
* [Modern Java: Records, Sealed Types & Pattern Matching](03_Java_Deep_Dive_and_Concurrency/05_Modern_Java_and_JMH/01_modern_java_records_and_pattern_matching.md) — Java 17/21 Records, Compact Constructors, serialization immunity, Sealed Hierarchies (Sum Types) + Records (Product Types) = Algebraic Data Types, and Java 21 Record Pattern Deconstruction without `default`.
* [JMH Microbenchmarking Mastery & JIT Profiling](03_Java_Deep_Dive_and_Concurrency/05_Modern_Java_and_JMH/02_jmh_microbenchmarking_mastery.md) — Why `System.currentTimeMillis()` is flawed, Dead Code Elimination, Constant Folding, `@Warmup`, `@Measurement`, `@Fork` process isolation, `Blackhole.consume()`, and concurrency throughput comparisons.

---

### 04. Spring Framework Ecosystem: Boot 3, Cloud, Security & Streaming
Modern Spring Boot 3 internals, bean lifecycle, reactive gateways, Resilience4j fault isolation, stateless OAuth2 Resource Server, and high-performance JPA persistence.

#### 01. Spring Core Internals & Bean Lifecycle
* [Spring Bean Lifecycle, Container Internals & Scopes Mastery](04_Spring_Framework_Ecosystem/01_Spring_Core_and_Internals/01_bean_lifecycle_and_container_internals.md) — BeanFactory vs. ApplicationContext, BeanDefinition, Aware hooks, BeanPostProcessor (BPP) vs. BeanFactoryPostProcessor (BFPP), the prototype-in-singleton injection trap (ObjectProvider, @Lookup), and 3-level singleton cache circular dependency resolution.
* [Spring AOP Mechanics, Dynamic Proxies & @Transactional Internals](04_Spring_Framework_Ecosystem/01_Spring_Core_and_Internals/02_spring_aop_proxies_and_transactional.md) — JDK Dynamic Proxies vs. CGLIB bytecode subclassing, why Spring Boot 3 defaults to `proxyTargetClass=true`, the @Transactional self-invocation trap and 3 architectural antidotes, propagation levels (REQUIRED vs. REQUIRES_NEW), HikariCP connection starvation deadlocks, and `readOnly=true` dirty-checking optimization.

#### 02. Spring Boot 3 Architecture & Production Readiness
* [Spring Boot 3 Auto-Configuration, Custom Starters & GraalVM AOT](04_Spring_Framework_Ecosystem/02_Spring_Boot_3_and_Observability/01_autoconfiguration_starters_and_graalvm_aot.md) — @EnableAutoConfiguration mechanics, replacing `spring.factories` with `AutoConfiguration.imports`, @Conditional evaluation pipeline, anatomy of a production custom starter, Jakarta EE 10 migration, declarative HTTP Interfaces (@HttpExchange), and GraalVM Ahead-Of-Time (AOT) reachability metadata.
* [Spring Boot 3 Production Observability, Micrometer & Telemetry](04_Spring_Framework_Ecosystem/02_Spring_Boot_3_and_Observability/02_production_observability_actuator_micrometer.md) — Deprecating Spring Cloud Sleuth for Micrometer Tracing, unified Observation API (metrics + traces in a single call), W3C Trace Context propagation, Java 21 Virtual Threads & MDC leak prevention, Kubernetes Liveness vs. Readiness cascade failure traps, and Actuator zero-trust security lockdown.

#### 03. Spring Security 6 & OAuth2 Architecture
* [Spring Security 6 Architecture, SecurityFilterChain & OAuth2 JWT Mastery](04_Spring_Framework_Ecosystem/03_Spring_Security_6_and_OAuth2/01_security_filter_chain_and_oauth2_jwt.md) — DelegatingFilterProxy to FilterChainProxy dispatch sequence, removing WebSecurityConfigurerAdapter for component-based SecurityFilterChain beans, SecurityContextHolder thread-local strategies, stateless OAuth2 Resource Server, NimbusJwtDecoder JWK Set caching, and custom Keycloak/Okta claim-to-authority converters.
* [Transport & Application Security: mTLS, CORS Mechanics, CSRF & Method Security](04_Spring_Framework_Ecosystem/03_Spring_Security_6_and_OAuth2/02_application_security_mtls_cors_method_security.md) — Mutual TLS (mTLS) zero-trust encryption with Spring Boot 3.1+ SSL bundles, X.509 client certificate authentication, the CORS-vs-Security filter ordering trap, when to enable vs. disable CSRF in stateless REST APIs, and fine-grained method-level security with @EnableMethodSecurity and SpEL.

#### 04. Spring Cloud, Stream & Enterprise Integration
* [Spring Cloud Gateway, Resilience4j Circuit Breakers & Service Discovery](04_Spring_Framework_Ecosystem/04_Spring_Cloud_and_Enterprise_Integration/01_spring_cloud_gateway_and_resilience4j.md) — Reactive Netty event loop vs. blocking Zuul, route predicates & gateway filters, multi-tenant Redis Token Bucket rate limiting, Resilience4j Circuit Breaker 3-state machine (Closed, Open, Half-Open), TimeLimiter, Bulkhead, and why Kubernetes native CoreDNS renders Netflix Eureka obsolete.
* [Spring Cloud Stream, Kafka Binders & Enterprise Integration Patterns (EIP)](04_Spring_Framework_Ecosystem/04_Spring_Cloud_and_Enterprise_Integration/02_spring_cloud_stream_and_integration_patterns.md) — Deprecating @StreamListener for pure Java functional programming (Supplier, Function, Consumer), deterministic dynamic binding conventions, Kafka consumer group partition load balancing, poison pill isolation via Dead Letter Queues (DLQ), ErrorHandlingDeserializer, and Spring Integration EIP Java DSL flows.

#### 05. Spring Data JPA & Persistence Performance
* [Spring Data JPA, The N+1 Problem & High-Performance Persistence](04_Spring_Framework_Ecosystem/05_Spring_Data_JPA_and_Persistence_Tuning/01_jpa_n_plus_one_and_performance_tuning.md) — Zero-allocation class-based DTO constructor projections, dynamic Specifications, the N+1 query problem, remediation via JOIN FETCH, MultipleBagFetchException, dynamic @EntityGraph, global batch fetching via `hibernate.default_batch_fetch_size`, persistence context first-level cache lifecycle, and JDBC write batching.
* [Open-Session-In-View (OSIV) Anti-Pattern & HikariCP Pool Sizing](04_Spring_Framework_Ecosystem/05_Spring_Data_JPA_and_Persistence_Tuning/02_osiv_anti_pattern_and_hikaricp_tuning.md) — The catastrophic latency amplification of `spring.jpa.open-in-view=true`, connection hold duration during slow JSON rendering/network I/O, safely disabling OSIV, managing LazyInitializationException, Brett Wooldridge's HikariCP pool sizing formula ($Core \times 2 + Spindle$), Little's Law capacity planning ($L = \lambda \times W$), and leak detection thresholds.

---

### 🚀 Upcoming Modules
* **05. API Design & Gateway Patterns** (`CORE.7`)
* **06. Apache Kafka & Event-Driven Architecture** (`CORE.8`)
* **07. Storage Engines & Databases (Oracle, MongoDB, Redis, DDIA)** (`CORE.9`)
* **08. AWS Cloud Architecture & Resiliency (SAP-C02)** (`CORE.10`)
* **09. Security Architecture & Zero Trust** (`CORE.11`)
* **10. Agentic AI, Multi-Agent Systems & LangGraph** (`CORE.12`)

---

## 📖 Primary References
* Martin Kleppmann, *Distributed Systems* Lecture Series, Cambridge University
* Google Site Reliability Engineering (SRE) Books (*Site Reliability Engineering* & *The Site Reliability Workbook*)
* Alex Xu, *System Design Interview* (Volumes 1 & 2)
* Martin Kleppmann, *Designing Data-Intensive Applications* (DDIA)
* [ByteByteGo Architecture Series](https://www.youtube.com/@ByteByteGo)
