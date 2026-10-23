# Java Backend Interview Master Index

This index categorizes the critical interview questions documented across the repository. Follow the links to find the detailed answers, follow-up chains, and production contexts.

## Basic (SDE-1)
*Focus: Language fundamentals, basic Spring, HTTP, and simple DB concepts.*
- **Core Java:** What is the difference between a class and an object? ([Link](02-core-java/05-classes-and-objects.md))
- **Core Java:** What is the difference between String, StringBuilder, and StringBuffer? ([Link](02-core-java/12-strings.md))
- **Core Java:** What is the difference between Fail-Fast and Fail-Safe iterators? ([Link](02-core-java/14-collections.md))
- **Spring Core:** What is Dependency Injection and why use it? ([Link](03-spring-core/02-dependency-injection.md))
- **Spring Boot:** What is the difference between `@Controller` and `@RestController`? ([Link](04-spring-boot/01-spring-boot-architecture.md))
- **REST:** What is idempotency in REST? ([Link](06-rest-web/01-http-rest-fundamentals.md))
- **SQL:** What is the difference between WHERE and HAVING? ([Link](01-sql/01-sql-fundamentals-and-queries.md))

## Intermediate (SDE-2)
*Focus: Internal mechanics, JVM behavior, advanced Spring, and caching.*
- **Core Java:** How does a HashMap resolve collisions and what is Treeification? ([Link](02-core-java/16-hashmap-internals.md))
- **Core Java:** How does the JVM calculate Memory limits inside Docker containers? ([Link](12-docker/01-docker-fundamentals.md))
- **Spring Core:** How does Spring resolve Circular Dependencies? ([Link](03-spring-core/03-bean-lifecycle-and-aop.md))
- **Spring Boot:** How does Auto-Configuration actually work under the hood? ([Link](04-spring-boot/01-spring-boot-architecture.md))
- **JPA/Hibernate:** What causes a LazyInitializationException? ([Link](05-jpa-hibernate/02-relationships-fetching.md))
- **SQL:** What happens if you index a boolean column? ([Link](01-sql/02-indexes-and-performance.md))
- **Security:** How do stateless JWTs work and why do we need Refresh tokens? ([Link](07-spring-security/01-security-and-jwt.md))

## Advanced / System Design Integration
*Focus: Distributed systems, resilience, concurrency, and architecture trade-offs.*
- **Concurrency:** How do you implement a distributed lock across microservices? ([Link](11-redis/01-redis-fundamentals.md))
- **Spring AOP:** Why does `@Transactional` fail to rollback on internal method calls? ([Link](03-spring-core/03-bean-lifecycle-and-aop.md))
- **Microservices:** What is the Outbox Pattern and why is it needed? ([Link](08-microservices/02-distributed-transactions.md))
- **Kafka:** How does Kafka guarantee message ordering? ([Link](09-apache-kafka/01-kafka-architecture.md))
- **Kafka:** How does Kafka prevent data loss if a broker crashes? ([Link](09-apache-kafka/03-replication-fault-tolerance.md))
- **Messaging:** When would you choose RabbitMQ over Kafka? ([Link](09-apache-kafka/04-kafka-vs-rabbitmq.md))
- **Performance:** How do you handle database pagination for millions of records? ([Link](01-sql/04-advanced-sql-backend.md))

## Production Debugging
*Focus: Real-world operational scenarios.*
- **Memory:** How do you diagnose a JVM memory leak? ([Link](13-production-debugging/01-cpu-memory-debugging.md))
- **Database:** How do you fix an N+1 query problem degrading API performance? ([Link](05-jpa-hibernate/03-n-plus-one-problem.md))
- **Microservices:** How do you trace a failure across 5 different services? ([Link](17-observability/01-observability-metrics.md))
