# RabbitMQ Fundamentals and Architecture

## 1. What Is It?
RabbitMQ is a widely used open-source message broker. It implements the Advanced Message Queuing Protocol (AMQP). Unlike Kafka, which is a distributed append-only log, RabbitMQ is a traditional message queue designed for complex routing and task distribution.

## 2. Core Components
- **Producer:** Sends messages to RabbitMQ.
- **Exchange:** Receives messages from producers and pushes them to queues based on rules (bindings). Producers *never* send messages directly to a queue.
- **Queue:** A buffer that stores messages.
- **Consumer:** Reads messages from the queue.
- **Binding:** A link between an Exchange and a Queue, often involving a "Routing Key".

## 3. Types of Exchanges
1. **Direct Exchange:** Delivers messages to queues based on an exact match of the routing key. (e.g., `routing_key="pdf_generator"`).
2. **Topic Exchange:** Routes messages to one or many queues based on a wildcard match between the routing key and the binding pattern. (e.g., `routing_key="logs.error.auth"`, binding=`"logs.error.*"`).
3. **Fanout Exchange:** Ignores the routing key. Broadcasts the message to *all* queues bound to it. Useful for pub/sub (e.g., sending an event to 5 different microservices).
4. **Headers Exchange:** Uses message headers instead of the routing key for routing.

## 4. Message Reliability and Delivery Guarantees
- **Acknowledgements (Acks):** When a consumer receives a message, RabbitMQ doesn't instantly delete it. It waits for the consumer to send an `ack`. If the consumer crashes before sending the `ack`, RabbitMQ re-queues the message and delivers it to another consumer.
- **Publisher Confirms:** For producers to know a message wasn't lost in transit, the broker sends a confirmation back to the producer once it safely receives and routes the message.
- **Durability vs Persistence:** 
  - *Durable Queues:* Survive a broker restart.
  - *Persistent Messages:* Saved to disk so they survive a broker restart. (You need both to guarantee no data loss during a crash).
- **Prefetch Count:** Determines how many unacknowledged messages RabbitMQ will send to a single consumer at once. Crucial for load balancing. If `prefetch=1`, RabbitMQ won't give a consumer a new message until it processes the current one.

## 5. Advanced Features
- **TTL (Time-To-Live):** Messages can be set to expire after a certain time.
- **DLX (Dead Letter Exchange):** If a message is rejected, expires, or the queue is full, it can be automatically routed to a DLX. This is used for creating retry queues and error analysis queues.

## 6. Interview Questions
### Q. What is the difference between Kafka and RabbitMQ?
**Short Answer:** RabbitMQ is a smart-broker / dumb-consumer model. It pushes messages to consumers, tracks exactly who acknowledged what, and deletes messages once processed. It excels at complex routing. Kafka is a dumb-broker / smart-consumer model. It just appends messages to a log and keeps them forever (until retention expires). Consumers pull data and track their own offsets. Kafka excels at massive throughput and event streaming replay.

---

[← Previous: Kafka vs RabbitMQ](../09-apache-kafka/04-kafka-vs-rabbitmq.md) | [Next: Redis Caching & Data Structures →](../11-redis/01-redis-fundamentals.md)
