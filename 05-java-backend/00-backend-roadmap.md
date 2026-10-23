# Java Backend Engineering Roadmap

This roadmap defines the learning progression for mastering Java Backend Engineering. It is designed to take you from core language fundamentals to designing, deploying, and debugging distributed microservices.

## Phase 1: Database Foundations
*Before writing backend code, you must understand how data is stored and retrieved.*
- [SQL Fundamentals & Queries](01-sql/01-sql-fundamentals-and-queries.md)
- [Indexes and Performance](01-sql/02-indexes-and-performance.md)
- [Transactions and Locks](01-sql/03-transactions-and-locks.md)
- [Advanced SQL Patterns (Pagination)](01-sql/04-advanced-sql-backend.md)

## Phase 2: Core Java & JVM Internals
*Understanding the engine that runs your code.*
- [Java Fundamentals & Platform Independence](02-core-java/01-java-fundamentals.md)
- [JDK, JRE, JVM Architecture](02-core-java/02-jdk-jre-jvm.md)
- [Java Program Execution](02-core-java/03-java-program-execution.md)
- [JVM Internals (ClassLoader, Memory, JIT)](02-core-java/04-jvm-internals.md)

## Phase 3: Object-Oriented Java
*Writing clean, reusable code.*
- [Classes and Objects](02-core-java/05-classes-and-objects.md)
- [Constructors](02-core-java/06-constructors.md)
- [Encapsulation](02-core-java/07-encapsulation.md)
- [Inheritance](02-core-java/08-inheritance.md)
- [Abstraction](02-core-java/09-abstraction.md)
- [Polymorphism (Overloading & Overriding)](02-core-java/10-polymorphism.md)

## Phase 4: Core APIs & Concurrency
*Handling data structures and parallel processing.*
- [Java Strings & Immutability](02-core-java/12-strings.md)
- [Exception Handling](02-core-java/13-exceptions.md)
- [Collections Framework](02-core-java/14-collections.md)
- [List, Set, Map Deep Dive](02-core-java/15-list-set-map.md)
- [HashMap Internals](02-core-java/16-hashmap-internals.md)
- [JVM Memory & GC](02-core-java/17-jvm-memory-and-gc.md)
- [Design Patterns in Java](02-core-java/18-design-patterns-backend.md)
- [Multithreading & Synchronization](02-core-java/22-multithreading.md)
- [Concurrency (Executors & Callable)](02-core-java/23-concurrency.md)

## Phase 5: Spring Core & Spring Boot
*The industry-standard framework for enterprise Java.*
- [Spring Core Fundamentals (IoC)](03-spring-core/01-spring-core-fundamentals.md)
- [Dependency Injection Deep Dive](03-spring-core/02-dependency-injection.md)
- [Bean Lifecycle and AOP](03-spring-core/03-bean-lifecycle-and-aop.md)
- [Spring Boot Architecture & Auto-Config](04-spring-boot/01-spring-boot-architecture.md)
- [DevTools & Actuator](04-spring-boot/04-devtools-and-actuator.md)

## Phase 6: Data Persistence (JPA/Hibernate)
*Mapping Objects to Relational Databases.*
- [JPA & Hibernate Fundamentals (Persistence Context)](05-jpa-hibernate/01-jpa-hibernate-fundamentals.md)
- [Relationships & Fetching Strategies](05-jpa-hibernate/02-relationships-fetching.md)
- [The N+1 Select Problem](05-jpa-hibernate/03-n-plus-one-problem.md)

## Phase 7: REST APIs & Security
*Exposing and protecting your data.*
- [HTTP & REST Fundamentals](06-rest-web/01-http-rest-fundamentals.md)
- [Spring Web Request Lifecycle (DispatcherServlet)](06-rest-web/02-spring-web-lifecycle.md)
- [Spring Security & Stateless JWTs](07-spring-security/01-security-and-jwt.md)

## Phase 8: Distributed Systems & Microservices
*Scaling the application.*
- [Microservices Architecture & Resilience Patterns](08-microservices/01-microservices-patterns.md)
- [Distributed Transactions (Saga & Outbox)](08-microservices/02-distributed-transactions.md)
- [Apache Kafka Architecture](09-apache-kafka/01-kafka-architecture.md)
- [Kafka Consumer Groups & Offsets](09-apache-kafka/02-consumer-groups-offsets.md)
- [Kafka Replication & Fault Tolerance](09-apache-kafka/03-replication-fault-tolerance.md)
- [Kafka vs RabbitMQ](09-apache-kafka/04-kafka-vs-rabbitmq.md)
- [RabbitMQ Fundamentals](10-rabbitmq/01-rabbitmq-fundamentals.md)
- [Redis Caching & Data Structures](11-redis/01-redis-fundamentals.md)

## Phase 9: Deployment, Operations & Testing
*Shipping and maintaining code in production.*
- [Docker Fundamentals for Java Backends](12-docker/01-docker-fundamentals.md)
- [Testing Strategies (Unit, Integration, Testcontainers)](16-testing/01-testing-strategies.md)
- [Observability (Logs, Metrics, Traces)](17-observability/01-observability-metrics.md)
- [Production Debugging (CPU, Memory, Deadlocks)](13-production-debugging/01-cpu-memory-debugging.md)

## Phase 10: Synthesis & System Design
*Putting it all together.*
- [Backend System Design Integration (Bridging HLD & Code)](14-backend-system-design-integration/01-implementation-bridge.md)
- [Practical Projects Progression](15-projects/01-project-roadmap.md)

---
## Advanced / Optional Technologies (Overview)
*These topics are context-dependent and represent specialized paths beyond core backend engineering.*
- **GraphQL / gRPC:** Alternatives to REST. Use gRPC for high-performance internal microservice communication. Use GraphQL when clients need to fetch sparse data from massive graphs.
- **NoSQL (MongoDB, Cassandra, DynamoDB):** Use when schema flexibility or massive horizontal write-scaling is required. Relational DBs are still preferred for strict transactional consistency.
- **Elasticsearch:** Required for full-text search capabilities and complex log aggregation (ELK Stack).
- **Kubernetes (K8s):** Container orchestration. Essential for managing hundreds of microservices, but often handled by DevOps teams.
- **CQRS & Event Sourcing:** Advanced architectural patterns for extreme read/write decoupling. Highly complex, use only when absolutely necessary.
