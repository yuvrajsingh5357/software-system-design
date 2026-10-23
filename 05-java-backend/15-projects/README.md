# Project-Based Learning

Building toy applications is useless. To pass an SDE-2 interview, you must build systems that handle failure.

## Final Project: E-Commerce Backend

### Architecture
- **Tech:** Spring Boot, PostgreSQL, Redis, Kafka, Docker Compose.
- **Modules:** Order Service, Inventory Service, Notification Service.

### Requirements to implement
1. **API:** Create an Order API (`POST /orders`).
2. **Database:** Save Order to PostgreSQL. Use Optimistic Locking (`@Version`) to prevent double-updates.
3. **Caching:** Cache the product catalog in Redis. Use `@Cacheable`.
4. **Resiliency:** If the Inventory Service is slow, the Order API must not crash. Implement a Circuit Breaker (Resilience4j) or Timeout.
5. **Distributed Transaction:** Implement the Outbox Pattern. Save the Order and an `OrderCreatedEvent` in the same PostgreSQL transaction.
6. **Messaging:** A background scheduler reads the Outbox and pushes to Kafka.
7. **Idempotency:** The Notification Service listens to Kafka. It must check Redis or DB to ensure it doesn't send the same email twice if Kafka delivers the message twice.

### Interview Questions this project prepares you for:
- "How do you guarantee a database write and a Kafka publish both succeed?"
- "What happens to your system if Redis goes down completely?"
- "How do you prevent two users from buying the last item in stock?"
