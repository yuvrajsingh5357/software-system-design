# Microservices Architecture and Patterns

## 1. Monolith vs Microservices
- **Monolith:** All business logic, UI, and database access are packaged into a single deployable unit (e.g., a single `.jar` or `.war` file). Easy to develop and test initially, but becomes hard to scale, deploy, and understand as the team grows.
- **Microservices:** The application is broken down into small, loosely coupled, independently deployable services organized around business capabilities (Bounded Contexts).

## 2. Distributed System Challenges
When you split a monolith, you introduce the network. The network is unreliable.
- **Latency:** Calling a method in a monolith takes nanoseconds. Calling a microservice over HTTP takes milliseconds.
- **Failure:** The target service might be down, slow, or returning errors.
- **Data Consistency:** You no longer have a single database with ACID transactions spanning all tables.

## 3. Resilience Patterns
- **Timeouts:** Never wait indefinitely for a response. Fail fast.
- **Retries:** Retry a failed request if it's transient (e.g., network blip). Use *Exponential Backoff*.
- **Circuit Breaker:** If a downstream service is failing continuously, "trip" the circuit. Stop sending requests to it for a while to let it recover, and return a fallback response immediately. (e.g., Resilience4j).
- **Bulkhead:** Isolate resources (like thread pools) so a failure in one part of the system doesn't bring down the entire system.

## 4. Communication: Sync vs Async
- **Synchronous (REST/gRPC):** Service A calls Service B and waits for the response. Creates tight temporal coupling. If B is down, A fails.
- **Asynchronous (Kafka/RabbitMQ):** Service A publishes an event. Service B consumes it later. They don't need to be online at the same time. Highly scalable and decoupled.

## 5. Distributed Transactions
In microservices, the "Database per Service" pattern means you cannot use simple database transactions.
- **Saga Pattern:** A sequence of local transactions. Each local transaction updates the database and publishes an event to trigger the next local transaction in the saga. If a step fails, the saga executes *compensating transactions* to undo the previous steps.
- **Outbox Pattern:** Solving the "Dual Write Problem". How do you atomically save to a DB and publish to Kafka? You save the business entity AND an "event" to an `Outbox` table in the *same* local DB transaction. A separate background process reads the Outbox table and reliably publishes to Kafka.

## 6. Interview Questions
### Q. What is an API Gateway?
**Short Answer:** It is a single entry point for all clients. It handles cross-cutting concerns like routing, load balancing, authentication, rate limiting, and SSL termination, shielding the internal microservice architecture from the outside world.

### Q. What is the CAP Theorem?
**Short Answer:** It states that a distributed data store can only simultaneously provide two out of three guarantees: Consistency (every read receives the most recent write), Availability (every request receives a response), and Partition Tolerance (the system operates despite network failures). Because network partitions (P) are unavoidable, you must trade off between C and A.

---

[← Previous: Spring Security & Stateless JWTs](../07-spring-security/01-security-and-jwt.md) | [Next: Distributed Transactions (Saga & Outbox) →](02-distributed-transactions.md)
