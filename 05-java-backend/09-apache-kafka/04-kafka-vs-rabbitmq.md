# Kafka vs RabbitMQ

## 1. Architectural Difference
- **RabbitMQ (Smart Broker, Dumb Consumer):** The broker tracks message state. It routes messages to specific queues based on rules. Once a consumer acknowledges a message, the broker deletes it.
- **Kafka (Dumb Broker, Smart Consumer):** The broker is simply a distributed append-only log. It does not track which consumer read what. The consumer tracks its own "offset". Kafka does not delete messages on read; it deletes them based on a time or size retention policy.

## 2. Delivery Model
- **RabbitMQ:** Push model (mostly). Excellent for complex routing (e.g., Direct, Topic, Fanout exchanges).
- **Kafka:** Pull model. The consumer polls the broker for a batch of messages.

## 3. Why choose Kafka? (Use Cases)
- **High Throughput:** Kafka can handle millions of messages per second.
- **Event Sourcing / Replay:** Because Kafka retains messages after they are read, a new consumer can start from offset 0 and replay the entire history of the system.
- **Stream Processing:** Aggregating data over time windows (e.g., count logins per 5 minutes).

## 4. Why choose RabbitMQ? (Use Cases)
- **Complex Routing:** Send message to Queue A if header=X, Queue B if header=Y.
- **Task Queues:** RabbitMQ excels at distributing tasks to a pool of competing consumers where processing time varies heavily. If one task takes 5 minutes, RabbitMQ gives the next task to an idle consumer. Kafka struggles here because partitions are locked to a specific consumer.

## 5. Interview Strategy
Do NOT say one is better than the other.
*SDE-2 Answer:* "I would use Kafka for the event-driven backbone of the architecture where I need replayability, audit logs, and massive throughput (like tracking clickstreams). I would use RabbitMQ for specific asynchronous task execution queues where I need fine-grained routing and individual message retries without blocking the queue."

---

[← Previous: Kafka Replication & Fault Tolerance](03-replication-fault-tolerance.md) | [Next: RabbitMQ Fundamentals →](../10-rabbitmq/01-rabbitmq-fundamentals.md)
