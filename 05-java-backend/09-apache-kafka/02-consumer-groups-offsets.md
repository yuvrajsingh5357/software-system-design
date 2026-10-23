# Kafka Consumer Groups and Offsets

## 1. What Are Consumer Groups?
A Consumer Group is a set of consumers that work together to read messages from a Kafka topic in parallel.

## 2. How Do They Work?
- When multiple consumers belong to the **same group**, Kafka divides the topic's partitions among them.
- **Rule:** A single partition can be consumed by exactly **ONE** consumer in a group at a time.
- Each consumer gets only its share of the partitions.
- This allows Kafka to process messages faster, in parallel, and without duplicating data processing inside the same group.

### Scaling Example:
If a topic has 4 partitions:
- **1 Consumer in group:** It reads from all 4 partitions.
- **4 Consumers in group:** Each consumer reads from exactly 1 partition. This is the maximum parallel throughput.
- **5 Consumers in group:** 4 consumers read from 1 partition each, and the 5th consumer sits idle.

## 3. Independent Processing (Different Groups)
If you have an `Order Service` generating messages, and you need to send an Email AND update an Analytics Dashboard:
- The Email Service should be in `ConsumerGroup-Email`.
- The Analytics Service should be in `ConsumerGroup-Analytics`.
- Kafka will deliver every message to *both* groups independently. Each group tracks its own position in the log.

## 4. What Are Offsets?
- An offset is a unique, incremental ID number assigned to each message inside a partition.
- It acts like a bookmark.
- As a consumer reads messages, it periodically sends its current offset back to Kafka (or stores it). This is called **Committing the Offset**.
- If a consumer crashes and restarts, it looks up its last committed offset and resumes reading from exactly where it left off, preventing data loss.

## 5. Interview Questions

### Q. What happens if a consumer crashes while processing a message?
#### Short Interview Answer
It depends on when the offset is committed. 
- If the consumer uses **"At-most-once"** (commits the offset immediately upon receiving the message before processing), the message will be lost because Kafka thinks it was successfully processed.
- If the consumer uses **"At-least-once"** (processes the message, saves data to a DB, and *then* commits the offset), the crash prevents the commit. When the consumer restarts, it will read the same message again. This guarantees no data loss but requires the consumer to be **idempotent** to handle duplicate processing safely.

---

[← Previous: Apache Kafka Architecture](01-kafka-architecture.md) | [Next: Kafka Replication & Fault Tolerance →](03-replication-fault-tolerance.md)
