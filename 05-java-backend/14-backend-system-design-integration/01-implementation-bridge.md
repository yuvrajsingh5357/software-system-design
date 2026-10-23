# Backend System Design Integration

## 1. Bridging HLD to Implementation
High-Level Design (HLD) concepts must translate into actual Java/Spring code. This document bridges the gap.

## 2. Caching Implementation
- **HLD Concept:** Reduce database load and latency using an in-memory cache.
- **Implementation:** Redis + Spring Cache.
- **Code:** Add `@EnableCaching` to the main class. Annotate slow methods with `@Cacheable(value = "users", key = "#id")`. Spring automatically intercepts the method using AOP, checks Redis, and returns the cached value. Use `@CacheEvict` when updating the database to prevent stale data.

## 3. Message Queues (Asynchronous Processing)
- **HLD Concept:** Decouple services and handle traffic spikes using an event broker.
- **Implementation:** Apache Kafka + Spring Kafka.
- **Code:** Annotate a method with `@KafkaListener(topics = "orders", groupId = "email-service")`. Spring manages the consumer thread, polling, and deserialization. Set `acks=all` in `application.yml` for durability.

## 4. Distributed Transactions (Saga / Outbox)
- **HLD Concept:** Maintaining consistency across microservices without 2-Phase Commit.
- **Implementation:** Spring `@Transactional` + Transactional Outbox.
- **Code:** In a single `@Transactional` method, save the `Order` to the database AND save an `OrderCreatedEvent` to an `Outbox` table. A separate Spring `@Scheduled` job or Debezium (CDC) reads the Outbox table and publishes to Kafka.

## 5. Rate Limiting
- **HLD Concept:** Protect the system from DDoS or brute-force attacks.
- **Implementation:** Redis + Spring Cloud Gateway or Interceptors.
- **Code:** Use a Spring `HandlerInterceptor`. On every request, execute a Redis Lua script implementing the Token Bucket algorithm (`INCR` key based on IP Address). If the count exceeds the limit, return HTTP `429 Too Many Requests`.

## 6. Observability and Tracing
- **HLD Concept:** Tracing a single request across 5 different microservices.
- **Implementation:** Micrometer + Zipkin/Jaeger.
- **Code:** Include `spring-boot-starter-actuator` and `micrometer-tracing`. Spring automatically injects a `traceId` and `spanId` into every log statement and passes them in HTTP headers (`X-B3-TraceId`) to downstream services via `RestTemplate` or `WebClient`.

## 7. Database Connection Pooling
- **HLD Concept:** Creating TCP connections to the DB is expensive.
- **Implementation:** HikariCP.
- **Code:** Spring Boot uses HikariCP by default. Configure `spring.datasource.hikari.maximum-pool-size=10`. This maintains 10 open connections. If 11 requests arrive, the 11th waits until a connection is returned.

## 8. Interview Questions
### Q. How do you prevent dual-write failures when updating a database and publishing an event to Kafka?
**Short Answer:** I would use the Transactional Outbox pattern. If you simply call `repo.save()` and then `kafka.send()`, the DB might commit but the Kafka send might fail due to a network error, leaving the system inconsistent. By writing both the business entity and the event to the same database within a single `@Transactional` method, atomicity is guaranteed by the relational DB. A background process then safely reads the event table and publishes to Kafka.

---

[← Previous: Production Debugging (CPU, Memory, Deadlocks)](../13-production-debugging/01-cpu-memory-debugging.md) | [Next: Practical Projects Progression →](../15-projects/01-project-roadmap.md)
