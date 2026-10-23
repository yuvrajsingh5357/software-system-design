# Distributed Transactions

## 1. What is it?
Executing a transaction that spans multiple independent databases or microservices.

## 2. Why do we need it?
In a monolith, `@Transactional` handles everything (ACID). In microservices, the Order Service has its DB, and the Inventory Service has its DB. 
*(Note: See the [HLD Section](../../04-hld/06-microservices-and-service-mesh.md) for distributed system theory. Below focuses on backend implementation).*

## 3. The Problem
Order is created $\rightarrow$ Inventory is reduced. 
If Inventory reduction fails, how do we rollback the Order? We cannot simply "rollback" an HTTP call.

## 4. Solutions (Implementation)

### A. Two-Phase Commit (2PC)
- **Mechanism:** A coordinator asks all services to "Prepare". If all say yes, it sends "Commit".
- **Backend implementation:** Java Transaction API (JTA) / XA Transactions.
- **Why it's bad:** Blocking, slow, not supported by many modern NoSQL DBs. Rarely used in microservices.

### B. The Saga Pattern (Choreography)
- **Mechanism:** Each service executes a local transaction and publishes an event (e.g., to Kafka). If a downstream service fails, it publishes a failure event, triggering **Compensating Transactions** upstream.
- **Example:**
  1. Order Service creates order (Status: PENDING), emits `OrderCreatedEvent`.
  2. Inventory Service listens, reserves stock. If fails, emits `InventoryFailedEvent`.
  3. Order Service listens to `InventoryFailedEvent`, changes order status to CANCELLED.

### C. The Outbox Pattern
- **Problem:** Writing to DB and publishing to Kafka is not atomic. What if DB commits but Kafka is down?
- **Solution:** 
  1. Save business data AND the event data in the *same* database transaction (using an `outbox` table).
  2. A separate background process (e.g., Debezium or a Spring Scheduled task) polls the `outbox` table and pushes to Kafka.

## 5. Interview Questions

### SDE-2 / System Design Integration
**Q: How do you guarantee exactly-once processing in a Saga?**
**Detailed Answer:** You can't natively. You guarantee *at-least-once* delivery via Kafka + Outbox. To simulate exactly-once, the consumer (e.g., Inventory Service) MUST be **Idempotent**.
**Implementation:** Use a unique `eventId` or `orderId`. Before processing, attempt to insert this ID into a `processed_events` table (or Redis) with a UNIQUE constraint. If it throws a constraint violation, silently acknowledge the message and ignore it.

---

[← Previous: Microservices Architecture & Resilience Patterns](01-microservices-patterns.md) | [Next: Apache Kafka Architecture →](../09-apache-kafka/01-kafka-architecture.md)
