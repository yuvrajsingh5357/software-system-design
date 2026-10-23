# Kafka Replication and Fault Tolerance

## 1. What Is It?
Fault tolerance means the system keeps working without losing data even when hardware parts fail. Kafka is specifically designed to survive broker failures, crashes, network issues, and server loss without stopping message flow.

## 2. Message Durability
Message durability means Kafka never loses your data.
- **Written to Disk:** Kafka never stores messages merely in memory. It writes every message directly to disk in an append-only log file.
- **Immutable:** Once written, messages are never modified or overwritten. This prevents corruption.

## 3. Partition Replication Across Brokers
Kafka achieves high availability by keeping multiple copies of data across different brokers.
- Each partition has **one leader** and **multiple followers** (replicas).
- The **Leader** handles all read and write requests for that partition.
- The **Followers** continuously copy data from the leader in real-time.
- If the leader broker crashes, one of the followers is automatically and instantly promoted to be the new leader, preventing downtime.

## 4. In-Sync Replicas (ISR)
- Kafka tracks a list called In-Sync Replicas (ISR). These are followers that have fully caught up with the leader's data.
- Only members of the ISR list are eligible to become a new leader if the current leader fails. This ensures the new leader will not be missing recent messages.

## 5. Acknowledgment Levels (acks setting)
The producer controls the durability/performance trade-off using the `acks` configuration when sending a message:
- **`acks = 0` (Send and Forget):** Producer doesn't wait for any confirmation. Fastest, but highly unsafe. If the broker crashes immediately, the message is lost.
- **`acks = 1`:** Producer waits for the Leader broker to write the message to its disk and confirm. Safe against producer errors, but if the Leader crashes before followers replicate it, data is lost.
- **`acks = all` (Strongest Durability):** Producer waits for the Leader AND all ISR followers to safely store the message. This guarantees that as long as at least one replica survives, the message will not be lost. This is standard for production environments handling financial or critical data.

## 6. KRaft Metadata Replication
In modern Kafka versions, ZooKeeper has been removed. Metadata (which broker holds which partition, who the leaders are) is managed using **KRaft mode**.
- Metadata is replicated across multiple internal controller nodes.
- This makes Kafka's cluster management even faster, more durable, and completely self-contained.

## 7. Interview Questions

### Q. How does Kafka prevent data loss if a broker goes down?
#### Short Interview Answer
Kafka relies on partition replication and the `acks=all` setting. Data is written to a Leader partition and simultaneously copied to Follower partitions on other brokers (In-Sync Replicas). The producer only receives an acknowledgment after all ISR nodes have safely written the data to disk. If the Leader broker fails, Kafka automatically promotes a fully synchronized Follower to become the new Leader.

---

[← Previous: Kafka Consumer Groups & Offsets](02-consumer-groups-offsets.md) | [Next: Kafka vs RabbitMQ →](04-kafka-vs-rabbitmq.md)
