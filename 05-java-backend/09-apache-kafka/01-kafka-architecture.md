# Kafka Architecture

## 1. What Is It?
Apache Kafka is a highly scalable, distributed event streaming platform. It is designed to handle massive amounts of data with very high throughput, low latency, and strong fault tolerance.

## 2. Core Concepts

### A. Topic
- A named channel where Kafka stores messages.
- A producer never sends data directly to a broker; it always sends data into a topic.
- A topic is like a category or folder for related messages (e.g., `orders`, `payments`, `notifications`).

### B. Partitions
- A Kafka Topic is divided into smaller units called partitions.
- A partition is an **ordered message log** where messages are stored in the exact order they arrive.
- **Why Partitions?** Kafka uses multiple partitions to achieve high performance, scalability, and parallel message processing.

### C. Broker
- A Kafka Broker is a server inside Apache Kafka that receives messages from producers, stores those messages in partitions on disk, and delivers them to consumers when requested.
- It is the main component that keeps Kafka running.

### D. Cluster
- A Kafka Cluster is a group of Kafka Brokers working together.
- Instead of running Kafka on one machine, a cluster uses multiple brokers to handle more data and serve more clients.
- If one broker fails, the cluster keeps running.

## 3. How Does Kafka Achieve High Throughput and Low Latency?
Kafka's performance comes from specific architectural decisions:
- **Sequential I/O:** Kafka writes data sequentially to disk. Sequential disk writes are extremely fast (sometimes faster than random memory writes).
- **Zero-Copy:** Kafka sends data from disk directly to the network socket, bypassing the application layer and avoiding extra copying in memory.
- **Page Cache:** Kafka relies heavily on the OS page cache. Most reads come directly from memory, not the physical disk.
- **Batching:** Producers and consumers send or read multiple messages in a single request, reducing network overhead.
- **Lightweight Protocol:** Kafka uses a highly optimized binary protocol over TCP.

## 4. Kafka Producer
An application that generates new data and publishes it to Kafka.
- Example: An Order Service that publishes an "OrderCreated" event to the `orders` topic.
- The producer's job is simple: it creates the message and sends it.
- **Message Routing:** A producer decides which partition a message goes to. It can be round-robin (if no key is provided), or hashed by a specific key (e.g., `order_id` ensures all events for that order go to the exact same partition, preserving strict ordering for that order).

## 5. Kafka Consumer
An application that connects to Kafka, reads messages from a topic, and processes them.
- Example: A Notification Service reading the `orders` topic to send confirmation emails.

## 6. Interview Questions

### Q. How does Kafka guarantee message ordering?
#### Short Interview Answer
Kafka only guarantees ordering **within a single partition**, not across the entire topic. If strict ordering is required for a specific entity (like state changes for an order), the producer must assign a "Partition Key" (like the Order ID) to the message. Kafka hashes this key to ensure all messages with that key are routed to the exact same partition, thereby preserving their exact sequence.

---

[← Previous: Distributed Transactions (Saga & Outbox)](../08-microservices/02-distributed-transactions.md) | [Next: Kafka Consumer Groups & Offsets →](02-consumer-groups-offsets.md)
