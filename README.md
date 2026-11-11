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

### 05. API Design, Gateway Patterns & Protocols
Enterprise RESTful resource modeling, RFC 9457 Problem Details, keyset vs. offset pagination, Stripe-style idempotency pipelines, Redis Lua sliding window rate limiters, BFF gateway patterns, gRPC/Protobuf HTTP/2 multiplexing, GraphQL DataLoader N+1 mitigation, and OpenAPI/CDCT contract governance.

#### 01. REST Standards & API Design
* [REST Resource Modeling, Richardson Maturity & RFC 9457](05_API_Design_and_Gateway_Patterns/01_REST_Standards_and_API_Design/01_rest_resource_modeling_and_rfc9457.md) — Pragmatic REST resource modeling, nouns vs. verbs, sub-resource state transitions, HTTP verbs safety vs. idempotency matrix, semantic HTTP status codes, standard RFC 9457 Problem Details for HTTP APIs (superseding RFC 7807), why pure HATEOAS fails in high-scale industry APIs, and Spring Boot 3 `ProblemDetail` integration.

#### 02. Advanced API Mechanics
* [API Pagination Architecture: Keyset vs. Offset](05_API_Design_and_Gateway_Patterns/02_Advanced_API_Mechanics/01_pagination_keyset_vs_offset.md) — B-Tree index scan physics of `OFFSET 1000000`, quadratic database degradation, pagination data drift anomalies, Keyset $O(\log N)$ seeks, mandatory tie-breaker rule (`ORDER BY created_at DESC, id DESC`), opaque Base64 URL-safe cursor tokens, and runnable verification suite.
* [Distributed Idempotency Pipelines & API Versioning](05_API_Design_and_Gateway_Patterns/02_Advanced_API_Mechanics/02_distributed_idempotency_and_versioning.md) — The 4-phase Stripe-style idempotency pipeline, SHA-256 payload fingerprinting to prevent parameter tampering, Redis distributed locks with exponential backoff & full jitter, and URL path vs. Custom Header vs. `Accept` content negotiation versioning strategies.

#### 03. Rate Limiting & Traffic Management
* [Rate Limiting Algorithms, Boundary Flaws & Redis Lua](05_API_Design_and_Gateway_Patterns/03_Rate_Limiting_and_Traffic_Management/01_rate_limiting_algorithms_and_redis_lua.md) — Comparative physics of 5 algorithms: Token Bucket, Leaky Bucket, Fixed Window, Sliding Window Log, and Sliding Window Counter. The $2\times$ boundary burst vulnerability of Fixed Window, Cloudflare's weighted Sliding Window Counter hybrid formula, atomic Redis Lua script, and fail-open vs. fail-closed availability trade-offs.
* [API Gateway Patterns, Throttling Headers & BFF Architecture](05_API_Design_and_Gateway_Patterns/03_Rate_Limiting_and_Traffic_Management/02_api_gateway_patterns_and_throttling.md) — Cross-cutting concerns at the network edge, standard throttling response contracts (`HTTP 429 Too Many Requests`, `Retry-After: 30`, `X-RateLimit-*`), Backends-for-Frontends (BFF) pattern separating Mobile vs. Web vs. 3rd-Party Partner traffic, edge circuit breakers, and runnable multi-channel gateway routing engine.

#### 04. Modern Protocols & API Contract Governance
* [Modern Protocols Matrix: REST vs. gRPC vs. GraphQL](05_API_Design_and_Gateway_Patterns/04_Modern_Protocols_and_Governance/01_protocol_matrix_rest_grpc_graphql.md) — Architectural decision matrix, gRPC & Protobuf internals, HTTP/2 binary framing & multiplexing over a single TCP socket, Proto3 immutable field tag numbers, 4 RPC models, the L4 vs. L7 gRPC load balancing dilemma, GraphQL schema execution, the $N+1$ query trap and Facebook DataLoader batching/caching solution, query depth/cost limits, and Automatic Persisted Queries (APQ).
* [API Contract Governance, OpenAPI & Consumer-Driven Contract Testing](05_API_Design_and_Gateway_Patterns/04_Modern_Protocols_and_Governance/02_api_contract_governance_and_openapi.md) — Contract-First design vs. Code-First anti-pattern, OpenAPI 3.0 specification architecture, code generation with openapi-generator, SemVer breaking vs. non-breaking change taxonomy, Postel's Law and Tolerant Reader pattern, automated CI/CD guardrails with Spectral & oasdiff, and Consumer-Driven Contract Testing (CDCT) with Pact.

---

### 06. Apache Kafka & Event-Driven Architecture
Core event backbone for modern distributed systems: commit log storage internals, sparse offset indexing, OS zero-copy `sendfile` I/O, KRaft Raft consensus metadata quorum, idempotent and transactional Exactly-Once Semantics (EOS), consumer group cooperative rebalancing, Confluent Schema Registry with Apache Avro, Axon Framework event sourcing, and Kafka Streams/Connect CDC.

#### 01. Foundations & Storage Internals
* [Kafka Commit Log Topology & In-Sync Replicas (ISR) Mechanics](06_Apache_Kafka_and_Event_Streaming/01_Foundations_and_Storage_Internals/01_commit_log_topology_and_isr_mechanics.md) — The commit log abstraction vs. message queues, broker cluster roles, rack-aware follower fetching, In-Sync Replicas (ISR) quorum mechanics (`replica.lag.time.max.ms`), High Watermark (HW) vs. Log End Offset (LEO) consumer read barrier, MurmurHash2 deterministic partitioning, Sticky Partitioner, `min.insync.replicas`, and data loss under unclean leader elections.
* [Storage Internals, Segment Anatomy & OS Zero-Copy Architecture](06_Apache_Kafka_and_Event_Streaming/01_Foundations_and_Storage_Internals/02_storage_segments_and_zero_copy.md) — Segment rolling policies, file anatomy (`.log`, `.index`, `.timeindex`, `leader-epoch-checkpoint`), memory-mapped (`mmap`) sparse index binary search, sequential disk I/O physics, Linux kernel `sendfile(2)` eliminating CPU copies, and the 10GB Heap Rule.
* [Metadata Quorum (KRaft): Replacing ZooKeeper with Raft Consensus](06_Apache_Kafka_and_Event_Streaming/01_Foundations_and_Storage_Internals/03_kraft_metadata_quorum.md) — Why ZooKeeper was deprecated (KIP-500), dual-write metadata synchronization flaw, controller cold-start pauses, KRaft event-driven `@metadata-0` partition, Controller Quorum Raft consensus, warm standby state eliminating failover pauses (< 200ms failover), pull-based broker metadata streaming via `FetchSnapshot`, and zero-downtime migration.

#### 02. Producers, Consumers & Delivery Semantics
* [Producer Reliability, Idempotence & High-Throughput Batching](06_Apache_Kafka_and_Event_Streaming/02_Producers_Consumers_and_Delivery_Semantics/01_producer_reliability_and_idempotence.md) — The RecordAccumulator micro-batching architecture, Producer ID (PID) and sequence numbers (KIP-98), broker deduplication sliding window, `delivery.timeout.ms` budgeting, `batch.size` vs `linger.ms`, and compression codec trade-offs (Snappy vs LZ4 vs Zstandard vs GZIP).
* [Consumer Groups, Offset Commit Strategies & Cooperative Rebalancing](06_Apache_Kafka_and_Event_Streaming/02_Producers_Consumers_and_Delivery_Semantics/02_consumer_groups_and_rebalance_protocols.md) — 1:1 partition assignment rule, consumer lag monitoring, internal `__consumer_offsets` topic, At-Most-Once vs At-Least-Once delivery, asynchronous commit patterns, heartbeat thread vs processing thread deadlock detection (`max.poll.interval.ms`), and Eager Stop-the-World vs Cooperative Sticky Assignor (KIP-429).
* [Transactions & Exactly-Once Semantics (EOS) in Distributed Pipelines](06_Apache_Kafka_and_Event_Streaming/02_Producers_Consumers_and_Delivery_Semantics/03_transactions_and_exactly_once_semantics.md) — Atomic Consume-Transform-Produce pipelines, Transaction Coordinator and `__transaction_state` log, 6-phase 2PC protocol sequence, physical Commit/Abort markers, `isolation.level = read_committed`, Last Stable Offset (LSO) barrier, and transactional epoch zombie producer fencing.

#### 03. Schema Registry, Contracts & Event Modeling
* [Schema Registry Architecture & Apache Avro Serialization](06_Apache_Kafka_and_Event_Streaming/03_Schema_Registry_and_Event_Modeling/01_schema_registry_and_avro_serialization.md) — Centralized schema management, Subject Naming Strategies (TopicNameStrategy vs RecordNameStrategy), the 5-byte Confluent wire protocol (`[0x00][Schema ID: 4B][Binary]`), zero-roundtrip client-side LRU schema caching, and wire footprint savings vs JSON.
* [Schema Evolution, Compatibility Modes & Contract Guardrails](06_Apache_Kafka_and_Event_Streaming/03_Schema_Registry_and_Event_Modeling/02_schema_evolution_and_compatibility.md) — Compatibility modes matrix (BACKWARD, FORWARD, FULL, TRANSITIVE), default value rules, safe field renaming with Avro `aliases`, enum evolution traps, and automated Maven/Gradle CI/CD compatibility guardrails.
* [Log Compaction & Enterprise Event Sourcing with Axon Framework and Apache Avro](06_Apache_Kafka_and_Event_Streaming/03_Schema_Registry_and_Event_Modeling/03_log_compaction_and_axon_event_sourcing.md) — Kafka log compaction mechanics, clean vs dirty sections, tombstone record deletion window (`delete.retention.ms`), and the Tier-0 Mastercard architecture: Axon Framework Aggregate Root lifecycle (`@AggregateVersion`), mapping `DomainEventMessage` to Kafka with Axon headers, custom Avro serialization via Schema Registry, `TrackingEventProcessor` CQRS read projections, and Aggregate Snapshotting (`EventCountSnapshotTrigger`).

#### 04. Stream Processing, Connect & DDIA Concepts
* [Kafka Streams, Windowing Mechanics & Embedded RocksDB State Stores](06_Apache_Kafka_and_Event_Streaming/04_Stream_Processing_Connect_and_DDIA/01_kafka_streams_and_windowing.md) — Stream-table duality (`KStream` vs `KTable` vs `GlobalKTable`), embedded off-heap RocksDB state stores, changelog topics and standby replicas, windowing taxonomy (Tumbling, Hopping, Sliding, Session), event-time processing with grace periods, and stream join co-partitioning rules.
* [Kafka Connect, Debezium CDC & DDIA Distributed Log Architecture](06_Apache_Kafka_and_Event_Streaming/04_Stream_Processing_Connect_and_DDIA/02_kafka_connect_cdc_and_ddia_comparisons.md) — Distributed Kafka Connect cluster topology, Transactional Outbox pattern with Debezium CDC, tailing database WAL/Redo logs with zero dual-writes, Martin Kleppmann's DDIA foundations (Destructive Queues vs Append-Only Logs, Competing Consumers vs Partition Ordering), and Lambda vs Kappa Architecture.

---

### 07. Storage Engines, Database Architecture & DDIA Internals
Deep relational mechanics (Oracle B-Trees, CBO execution plans, partitioning, and Active-Active Data Guard vs GoldenGate), the NoSQL polyglot persistence taxonomy, storage engine internals (B-Trees vs LSM-Trees, Bloom filters, and compaction), distributed caching (Redis epoll, memory eviction, and cluster topology), and Martin Kleppmann's DDIA transaction and partitioning foundations.

#### 01. Relational Architecture & Oracle Internals
* [B-Tree Index Internals, Clustering Factor & CBO Execution Plans](07_Storage_Engines_and_Database_Architecture/01_Relational_Architecture_and_Oracle_Internals/01_btree_indexes_and_cbo_execution_plans.md) — B-Tree block anatomy (Root, Branch, Leaf doubly linked chain `PREV_LEAF`/`NEXT_LEAF`), Clustering Factor (CF) crossover point ($\text{Blocks} \le CF \le \text{Rows}$), Composite index leading column rule, Reverse-Key indexes mitigating sequence insert latch contention, Cost-Based Optimizer (CBO) histograms, bind variable peeking traps, and reading EXPLAIN PLAN (`access` vs `filter` predicates).
* [Partitioning Strategies, Oracle RAC, Data Guard & GoldenGate CDC](07_Storage_Engines_and_Database_Architecture/01_Relational_Architecture_and_Oracle_Internals/02_partitioning_strategies_and_oracle_ha_dr.md) — Partitioning taxonomy (Range, List, Hash, Composite Range-Hash), Local Equipartitioned vs Global Indexes (`UPDATE GLOBAL INDEXES`), Static vs Dynamic Partition Pruning, Enterprise HA/DR (Oracle RAC shared-everything cache fusion vs Data Guard physical block-for-block redo shipping vs GoldenGate logical CDC), and the Tier-0 Mastercard 3-AZ Active-Active architecture (why GoldenGate was abandoned due to CDR conflict hell and replication lag spikes in favor of Data Guard with consistent hash ingress routing).

#### 02. NoSQL Taxonomy & Storage Engine Internals (DDIA)
* [Polyglot Persistence Matrix & Document Data Modeling](07_Storage_Engines_and_Database_Architecture/02_NoSQL_Taxonomy_and_Storage_Engine_Internals/01_nosql_taxonomy_and_document_modeling.md) — Decision matrix across 6 database categories (Relational, Document, Wide-Column, Key-Value, Graph, Vector), Document modeling trade-offs (Embedding vs Referencing), MongoDB WiredTiger B-Tree storage engine, memory cache eviction (50% RAM - 1GB), Write Concern (`w: "majority"`, `j: true`), Read Concern (`"majority"`), and the 16MB BSON document hard limit trap.
* [Storage Engine Deep Dive: B-Trees vs. LSM-Trees & Wide-Column Modeling](07_Storage_Engines_and_Database_Architecture/02_NoSQL_Taxonomy_and_Storage_Engine_Internals/02_storage_engine_internals_btree_vs_lsm.md) — B-Trees (in-place page mutation, write amplification, doublewrite buffer torn page defense) vs LSM-Trees (append-only sequential write path, in-memory Memtable, sequential commit log, immutable disk SSTables), Size-Tiered (STCS) vs Leveled Compaction (LCS), Bloom Filter mathematics ($m = -\frac{n \ln p}{(\ln 2)^2}$ eliminating wasted disk reads), Cassandra/DynamoDB wide-column modeling (Partition key Murmur3 token ring vs Clustering key physical sort order), and tombstone scan timeouts (`gc_grace_seconds`).

#### 03. Distributed Caching Architecture & Redis
* [Distributed Caching Topologies, Patterns & Failure Modes](07_Storage_Engines_and_Database_Architecture/03_Distributed_Caching_and_Redis/01_caching_patterns_and_failure_modes.md) — Caching topologies (Cache-Aside, Write-Through, Write-Behind, Refresh-Ahead), why DELETE on write is superior to UPDATE, Cache Stampede (thundering herd / dog-piling) mutex lock defense vs XFetch Probabilistic Early Expiration Algorithm ($\Delta \cdot \beta \cdot (-\ln(\text{rand}())) > \text{Expiry} - \text{Now}$), Cache Avalanche TTL uniform random jitter, and Cache Penetration Bloom filter defense.
* [Redis Internals, Single-Threaded Reactor & Cluster Topologies](07_Storage_Engines_and_Database_Architecture/03_Distributed_Caching_and_Redis/02_redis_internals_and_cluster_topologies.md) — Why Redis is single-threaded and fast (100ns RAM latency, zero mutex lock contention, Linux `epoll` I/O multiplexing, CPU cache locality, threaded I/O for socket read/write), internal data structures (SDS, Dict progressive rehashing, Quicklist/Listpack, Skiplist `zset`), memory eviction policies (Approximated LRU 24-bit clock 5-sample check, LFU Morris logarithmic counter), and Redis Sentinel (single-master failover) vs Redis Cluster (16,384 virtual hash slots, CRC16 routing, `{hash_tag}` co-location, `MOVED` vs `ASK` client redirections).

#### 04. DDIA Foundations: Transactions, Concurrency & Partitioning
* [ACID vs. BASE, Transaction Isolation Anomalies & MVCC Mechanics](07_Storage_Engines_and_Database_Architecture/04_DDIA_Foundations_Transactions_and_Partitioning/01_acid_isolation_levels_and_mvcc.md) — ACID vs BASE, Daniel Abadi's PACELC theorem, Quorum mathematics ($R + W > N$ strong consistency via Pigeonhole Principle vs $R + W \le N$ eventual consistency), ANSI SQL isolation levels vs DDIA anomalies (Dirty Reads, Non-Repeatable Reads, Phantom Reads, and Write Skew), Write Skew invariant breaches under Snapshot Isolation (why row locks fail) and Serializable Snapshot Isolation (SSI) defense, and MVCC mechanics (Oracle Undo Rollback Segments `ORA-01555` vs PostgreSQL Heap Tuple Versioning `xmin`/`xmax` and autovacuum bloat).
* [Distributed Transactions (2PC vs. Sagas), XA Protocols & Data Partitioning (Consistent Hashing)](07_Storage_Engines_and_Database_Architecture/04_DDIA_Foundations_Transactions_and_Partitioning/02_distributed_transactions_and_partitioning.md) — Atomic commitment problem, Two-Phase Commit (2PC) protocol sequence (Prepare irrevocable promise vs Commit), the In-Doubt zombie state and indefinite lock blocking failure mode, XA transaction overhead, why modern microservices replace 2PC with the Saga Pattern (Orchestration vs Choreography, semantic compensating transactions, Pivot transactions, semantic locks), the Naive Hash Modulo disaster ($\frac{N-1}{N}$ keys relocated), Consistent Hashing Ring (Ketama Algorithm $[0..2^{32}-1]$ limiting churn to $\frac{1}{N+1}$), and Virtual Nodes (Vnodes) eliminating data skew and enabling parallel rebalancing.

---

### 08. AWS Cloud Architecture & Resiliency (SAP-C02)
Enterprise cloud architecture and multi-region resilience based on the AWS Certified Solutions Architect - Professional (SAP-C02) blueprint: multi-account governance (Control Tower, SCP inheritance), hybrid networking (Transit Gateway, Direct Connect, Route 53 Resolver), serverless & event-driven compute (Lambda lifecycle, Provisioned Concurrency, EventBridge API Destinations), scalable cloud persistence (DynamoDB Single-Table, Aurora 6-way multi-AZ storage), disaster recovery (4 DR models, Route 53 ARC, WORM Vault Lock), cryptography (KMS envelope encryption, multi-region CMKs), FinOps optimization, 7 Rs migration (AWS MGN, DMS + SCT, Snowball Edge), and canonical exam scenario walkthroughs.

#### 01. Design for Organizational Complexity & Hybrid Networking (Domain 1)
* [Multi-Account Governance, SCP Evaluation & Enterprise Identity Federation](08_AWS_Cloud_Architecture_and_Resiliency/01_Design_for_Organizational_Complexity/01_multi_account_governance_and_identity.md) — Management account isolation rules, SCP evaluation hierarchy and explicit deny precedence, region restriction guardrails with global service exemptions, AWS Control Tower preventive vs detective controls, IAM Identity Center (SSO) SAML/OIDC federation with automated role provisioning, cross-account STS AssumeRole with ExternalId Confused Deputy defense, and Directory Service decision matrix (Managed AD vs AD Connector vs Simple AD).
* [Complex Network Topologies, Hybrid Direct Connect & Route 53 Resolver](08_AWS_Cloud_Architecture_and_Resiliency/01_Design_for_Organizational_Complexity/02_complex_networking_and_hybrid_connectivity.md) — Transit Gateway (TGW) hub-and-spoke route domains, inter-region peering, Centralized Inspection VPC with Gateway Load Balancer (GWLB) GENEVE encapsulation (Port 6081), the mandatory TGW Appliance Mode symmetric routing defense, Direct Connect (Dedicated vs Hosted, Private/Transit VIFs, DXGW, MACsec L2 encryption, IPsec VPN over DX), and Route 53 Resolver hybrid split-horizon DNS (Inbound vs Outbound endpoints shared via RAM).

#### 02. Design for New Solutions & Serverless (Domain 2)
* [Serverless Compute, Container Orchestration & Event-Driven Architecture](08_AWS_Cloud_Architecture_and_Resiliency/02_Design_for_New_Solutions_and_Serverless/01_serverless_compute_containers_and_event_driven.md) — Firecracker microVM lifecycle (Init, Invoke, Shutdown), static context caching, Provisioned Concurrency cold start elimination, Hyperplane VPC networking (< 1ms delay), Reserved Concurrency database protection, ECS on Fargate Capacity Providers vs EKS Karpenter just-in-time direct EC2 auto-scaling (< 45s), Amazon EventBridge content-based pattern matching, API Destinations zero-code webhooks, Archive & Replay, and Amazon MSK with KRaft and S3 Tiered Storage.
* [Edge Routing, CloudFront Failover & Scalable Cloud Persistence](08_AWS_Cloud_Architecture_and_Resiliency/02_Design_for_New_Solutions_and_Serverless/02_edge_services_and_scalable_cloud_persistence.md) — API Gateway HTTP APIs vs REST APIs vs WebSocket APIs, CloudFront Origin Groups automated cross-region failover, CloudFront Functions (< 1ms JS at edge) vs Lambda@Edge, DynamoDB Single-Table Design modeling, GSI vs LSI (avoiding the 10GB poison pill), DynamoDB Global Tables active-active Last-Writer-Wins (LWW) replication, Amazon Aurora 6-way multi-AZ storage (4/6 write quorum, 3/6 read quorum, self-healing), Aurora Serverless v2 instant ACU scaling, Aurora Global Database sub-second cross-region replication, and ElastiCache Redis Cluster mode.

#### 03. Continuous Improvement, Multi-Region DR & FinOps (Domain 3)
* [Disaster Recovery Architecture, Multi-Region Resilience & Route 53 ARC](08_AWS_Cloud_Architecture_and_Resiliency/03_Continuous_Improvement_Multi_Region_DR_and_FinOps/01_disaster_recovery_and_multi_region_resilience.md) — The 4 DR models (Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active) with RPO/RTO trade-offs, Route 53 Application Recovery Controller (ARC) cell-based routing controls and readiness checks bypassing DNS caching thrashing, and immutable ransomware defense via AWS Backup Vault Lock (Compliance Mode WORM protection).
* [Cloud Cryptography, Observability & FinOps Cost Optimization](08_AWS_Cloud_Architecture_and_Resiliency/03_Continuous_Improvement_Multi_Region_DR_and_FinOps/02_cloud_security_observability_and_finops.md) — AWS KMS Envelope Encryption protocol (overcoming the 4KB limit and API throttling; immediate in-memory PDK wipe), Multi-Region CMKs (`mrk-*`), AWS Secrets Manager 4-step dual-user rotation, CloudWatch Cross-Account Observability, X-Ray distributed trace context propagation, Amazon GuardDuty zero-agent ML threat detection, AWS WAF rate-based rules, Compute Savings Plans vs EC2 Savings Plans vs RIs, Spot ASG `price-capacity-optimized` allocation with 2-minute termination notice handling, S3 Intelligent-Tiering zero-retrieval-fee cycles, and runnable verification POC.

#### 04. Workload Migration & Hybrid Modernization (Domain 4)
* [Cloud Migration Strategies (7 Rs), AWS MGN, DMS & Hybrid Storage Architecture](08_AWS_Cloud_Architecture_and_Resiliency/04_Workload_Migration_and_Hybrid_Modernization/01_workload_migration_data_transfer_and_hybrid_storage.md) — The 7 Rs framework, AWS Application Migration Service (MGN) agent-based block-level continuous replication and zero-downtime cutover, AWS DMS (Full Load + CDC) and AWS SCT for heterogeneous database migration, AWS DataSync protocol acceleration vs AWS Snow Family (Snowcone, Snowball Edge, Snowmobile) bandwidth vs shipping tipping point calculation, AWS Storage Gateway (S3 File, FSx File, Volume Cached/Stored, Tape VTL), and Amazon FSx (FSx for NetApp ONTAP SnapMirror & FSx for Windows File Server multi-AZ).

#### 05. Exam Practice & Scenario Walkthroughs
* [AWS SAP-C02 High-Yield Exam Scenario & Distractor Breakdown Dossier](08_AWS_Cloud_Architecture_and_Resiliency/05_Exam_Practice_and_Scenario_Walkthroughs/01_sap_c02_high_yield_scenario_dossier.md) — In-depth architectural analysis of 10 canonical multi-tier exam scenario archetypes: multi-account egress DPI with GWLB and Appliance Mode, heterogeneous Oracle modernization, hybrid cross-cloud DNS, active-active multi-region failover, WORM compliance with Object Lock, serverless cold start mitigation, offline PB-scale migration, cross-account secret rotation, EventBridge API destinations, and multi-protocol storage migration. Includes flawed dump answer traps, community consensus, exhaustive distractor deconstruction, and keyword triggers.

---

### 09. Security Architecture & Zero Trust (CORE.11)
Zero-Trust architecture, identity federation, cryptographic engineering, OWASP API security, and enterprise CI/CD pipeline defenses: OAuth 2.0 & OIDC token flows, PKCE, token exchange (RFC 8693), JWT internals and stateless revocation, RBAC vs ABAC, decoupled policy engines (OPA Rego, Cedar), applied cryptography (AES-256-GCM AEAD, envelope encryption, password hashing with Argon2id), NIST SP 800-207 Zero Trust, TLS 1.3 & mTLS, SPIFFE/SPIRE workload identities, dynamic secrets lifecycles (HashiCorp Vault, AWS Secrets Manager), OWASP API Top 10 (BOLA/IDOR contextual isolation, SSRF & IMDSv2 defense, CORS vs CSRF), and enterprise CI/CD security pipelines (AST-based SAST via SonarQube & Checkmarx, SCA dependency vulnerability analysis & open-source copyleft license governance via Black Duck, SBOM generation, and Cosign container signing).

#### 00. Security & Identity Foundations
* [Security Architecture & Identity Foundations: Core Concepts Primer](09_Security_Architecture_and_Zero_Trust/00_Security_and_Identity_Foundations/01_security_and_identity_fundamentals.md) — First-principles mental models: Authentication (AuthN) vs Authorization (AuthZ), evolution from HTTP Basic Auth and Session Cookies to SAML 2.0 (Enterprise XML) and OAuth 2.0 / OpenID Connect (JSON/REST), token paradigms (By-Value JWT vs By-Reference Opaque UUID, and the Phantom Token Pattern), transport security (One-Way TLS 1.3 vs Mutual TLS / mTLS at Layer 4), NIST SP 800-207 Zero Trust Architecture (Never Trust, Always Verify, Assume Breach), and Staff/Principal architect decision matrices.

#### 01. Identity, Authentication & Token Architecture (OAuth2 & OIDC)
* [OAuth 2.0, OpenID Connect & Machine-to-Machine Token Architecture](09_Security_Architecture_and_Zero_Trust/01_Identity_Authentication_and_OAuth2/01_oauth2_oidc_and_m2m_token_architecture.md) — Authentication vs Authorization, OIDC ID Token vs OAuth2 Access Token audiences, Authorization Code Flow with PKCE (RFC 7636 `code_verifier` / `code_challenge` S256 method eliminating client secret in browser/mobile), M2M Client Credentials (RFC 6749) and the "Lost User Context" dilemma, RFC 8693 OAuth 2.0 Token Exchange (`sub` + `act` propagation across microservices), automated zero-downtime dual-secret rotation, and enterprise SSO federation (SAML 2.0 XML signatures vs OIDC JWTs).
* [JWT Internals, Stateless Revocation Paradigms & RBAC vs. ABAC Authorization](09_Security_Architecture_and_Zero_Trust/01_Identity_Authentication_and_OAuth2/02_jwt_internals_revocation_and_rbac_abac.md) — JWT anatomy, RS256 asymmetric signing vs HS256 symmetric forgery risks, JSON Web Key Set (JWKS) dynamic rotation (`/.well-known/jwks.json` caching and kid lookup), the Stateless Revocation Dilemma (short-lived tokens + Refresh Token Rotation with reuse breach detection vs distributed Redis JTI denylist with TTL cleanup), RBAC role explosion antipattern vs ABAC 4-dimensional dynamic policy evaluation (Subject, Resource, Action, Environment), and decoupled policy engines (Open Policy Agent Rego and AWS Verified Permissions Cedar).

#### 02. Applied Cryptography, Zero Trust & Secrets Management
* [Applied Cryptography, AEAD Ciphers & Envelope Encryption](09_Security_Architecture_and_Zero_Trust/02_Applied_Cryptography_and_Zero_Trust/01_applied_cryptography_and_envelope_encryption.md) — Symmetric vs Asymmetric cryptography performance and use cases, AES-256-GCM Authenticated Encryption with Associated Data (AEAD) with 128-bit authentication tag and AAD context binding, the fatal GCM nonce/IV reuse catastrophe (XOR plaintext leak & GHASH subkey extraction), HSM-backed Envelope Encryption (KEK/CMK vs DEK with immediate in-memory PDK wipe), password hashing vulnerabilities (why raw SHA-256 is cracked at 150B hashes/sec on GPUs), and memory-hard hashing standards (Argon2id RFC 9106 and BCrypt).
* [Zero Trust Architecture, Mutual TLS (mTLS) & Secrets Lifecycle Management](09_Security_Architecture_and_Zero_Trust/02_Applied_Cryptography_and_Zero_Trust/02_zero_trust_mtls_and_secrets_lifecycle.md) — NIST SP 800-207 Zero Trust Architecture (7 core tenets, hostile internal perimeter), TLS 1.3 handshake mechanics (RFC 8446 1-RTT, 0-RTT resumption, PFS via Ephemeral Diffie-Hellman), Mutual TLS (mTLS) bidirectional X.509 certificate verification at Layer 4, SPIFFE / SPIRE workload identities and automated short-lived X.509 SVIDs (1–24 hr validity) eliminating CRL/OCSP bottlenecks, dynamic secrets lifecycles (HashiCorp Vault dynamic DB engine with 1-hr leases, heartbeat renewal, and automated drop on expiry), and Vault vs AWS Secrets Manager architecture comparison.

#### 03. Application Security (OWASP API Top 10) & CI/CD Pipeline Defenses
* [OWASP API Top 10 Deep Dive: BOLA, SSRF, Injection & CORS/CSRF Realities](09_Security_Architecture_and_Zero_Trust/03_AppSec_OWASP_Top_10_and_CICD_Pipeline/01_owasp_api_top_10_defenses_bola_ssrf.md) — OWASP API #1: Broken Object Level Authorization (BOLA / IDOR) enumeration vectors, UUIDv4 vs predictable IDs, contextual tenant-aware SQL queries (`WHERE id = :id AND tenant_id = :jwtTenantId AND owner_id = :jwtUserId`), OWASP API #7: Server-Side Request Forgery (SSRF) and the Capital One cloud metadata breach (`169.254.169.254`), AWS IMDSv2 defense (PUT session token challenge + hop limit 1 blocking proxy traversal), parameterized SQL injection prevention, CORS preflight semantics (`OPTIONS`, `Access-Control-Allow-*`), and why CSRF is irrelevant for stateless `Authorization: Bearer` APIs.
* [Enterprise CI/CD Security: SAST, SCA, License Compliance & Supply Chain Security](09_Security_Architecture_and_Zero_Trust/03_AppSec_OWASP_Top_10_and_CICD_Pipeline/02_enterprise_cicd_security_sast_sca.md) — Static Application Security Testing (SAST) with SonarQube & Checkmarx (AST parsing, control flow graphs, taint analysis tracking untrusted sources to sensitive sinks, PR quality gate thresholds), Software Composition Analysis (SCA) with Black Duck & Snyk (transitive dependency vulnerability scanning for Log4Shell/Spring4Shell, CVSS v3/v4 score blocking), open-source license governance (permissive MIT/Apache 2.0 vs viral strong copyleft GPL/AGPL v3 source code disclosure risks), software supply chain defenses (CycloneDX/SPDX SBOM generation, Sigstore Cosign container image signing verified by Kyverno admission controllers), and comprehensive Java 21 security verification POC.

---

### 🚀 Upcoming Modules
* **10. Agentic AI, Multi-Agent Systems & LangGraph** (`CORE.12`)

---

## 📖 Primary References
* Martin Kleppmann, *Distributed Systems* Lecture Series, Cambridge University
* Google Site Reliability Engineering (SRE) Books (*Site Reliability Engineering* & *The Site Reliability Workbook*)
* Alex Xu, *System Design Interview* (Volumes 1 & 2)
* Martin Kleppmann, *Designing Data-Intensive Applications* (DDIA)
* [ByteByteGo Architecture Series](https://www.youtube.com/@ByteByteGo)
