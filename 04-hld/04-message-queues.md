# Message Queues and Event-Driven Architecture

Message queues enable asynchronous, decoupled, and resilient distributed systems. This chapter covers the what, why, and how — including Kafka deep dive.

## Table of Contents
- [Why Message Queues?](#why-message-queues)
- [Core Concepts](#core-concepts)
- [Apache Kafka Deep Dive](#apache-kafka-deep-dive)
- [RabbitMQ vs Kafka](#rabbitmq-vs-kafka)
- [Event-Driven Architecture Patterns](#event-driven-architecture-patterns)
- [When to Use Message Queues](#when-to-use-message-queues)
- [Common Patterns and Problems](#common-patterns-and-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Why Message Queues?

**Problem:** Service A directly calls Service B. Issues:
1. **Tight coupling** — if B is down, A fails
2. **Speed mismatch** — A produces faster than B can process
3. **No buffering** — traffic spike → B overwhelmed
4. **No retry** — if B fails processing, request is lost

**Solution: Message Queue as intermediary**

```
Without MQ:
  OrderService → [HTTP] → InventoryService → [HTTP] → NotificationService
  If NotificationService is down → order fails!

With MQ:
  OrderService → [MQ] → InventoryService (async)
                → [MQ] → NotificationService (async)
  NotificationService down? Messages queue up, retry when it recovers.
```

**Benefits:**
- **Decoupling:** Producer and consumer don't know each other
- **Buffering:** Queue absorbs traffic spikes
- **Async processing:** Producer doesn't wait for consumer
- **Reliability:** Messages persisted — survived consumer crashes
- **Fan-out:** One message → many consumers

---

## Core Concepts

```
Producer → publishes messages to a topic/queue
Consumer → subscribes and processes messages
Broker   → stores and delivers messages (Kafka server, RabbitMQ node)
Topic    → named channel (like a table name)
Partition→ ordered, immutable log within a topic
Offset   → position of a message within a partition
Consumer Group → multiple consumers sharing work on a topic
```

**Delivery semantics:**

| Guarantee | What it means | Implementation |
|-----------|--------------|----------------|
| At-most-once | May lose messages | Don't retry; ack before processing |
| At-least-once | May duplicate | Retry on failure; ack after processing |
| Exactly-once | No loss, no duplicate | Transactional producers + idempotent consumers |

---

## Apache Kafka Deep Dive

### Architecture

```
Kafka Cluster:
  3+ Brokers (servers)
    ↓
  Topics (partitioned, replicated)
    ↓
  Partitions (ordered log of messages)

Example: Topic "orders" with 3 partitions, replication factor 3:

Broker 1: Partition 0 (Leader), Partition 2 (Replica)
Broker 2: Partition 1 (Leader), Partition 0 (Replica)
Broker 3: Partition 2 (Leader), Partition 1 (Replica)

Each partition is a leader on one broker, replicated to ISR (In-Sync Replicas) on others.
```

### Producer

```
Message → Producer → Serializer → Partitioner → Buffer → Broker

Partitioning:
  Round-robin if no key
  hash(key) % numPartitions if key specified
  All messages with same key → same partition (ordering guaranteed)
  
Batching:
  Producer batches messages (linger.ms, batch.size) for throughput
  
Acknowledgment (acks):
  acks=0  → fire and forget (max throughput, may lose messages)
  acks=1  → leader acks (may lose if leader fails before replication)
  acks=all→ all ISRs ack (max durability, slower)
```

### Consumer

```
Consumer Group:
  Topic: 3 partitions
  Consumer Group "order-processor" with 3 consumers:
    Consumer 1 → Partition 0
    Consumer 2 → Partition 1
    Consumer 3 → Partition 2
  
  Add 4th consumer → one idle (consumers can't exceed partitions)
  
Offset management:
  Kafka stores committed offsets in __consumer_offsets topic
  
  auto.commit.enable=true  → offset committed every 5 seconds (can lose messages)
  manual commit (recommended):
    try {
      process(records)          // process first
      consumer.commitSync()     // then commit
    } catch (Exception e) {
      // message will be reprocessed on restart (at-least-once)
    }
```

### Kafka as an Event Log

**Kafka's key insight:** A topic is an **immutable, append-only log**.

```
Time →
[Offset 0] [Offset 1] [Offset 2] [Offset 3] [Offset 4] ...

Each consumer group has its own offset → read independently.
New consumer group can read from offset 0 → replay all events!

Use cases:
  - Event sourcing: store all events, replay to rebuild state
  - Audit log: all changes in permanent log
  - Multiple consumers: analytics + notifications + search indexing from same stream
```

### Retention

```
By time: delete.retention.ms = 604800000 (7 days default)
By size: log.retention.bytes = 1073741824 (1 GB)
Compaction: keep only latest value per key (useful for state snapshots)
```

### Kafka vs Traditional Queue

| | Kafka | Traditional MQ (RabbitMQ) |
|-|-------|--------------------------|
| Model | Log-based | Queue-based |
| After consumption | Message retained (with TTL) | Message deleted |
| Consumer groups | Multiple, independent | Competing consumers on same queue |
| Ordering | Per partition | Per queue |
| Throughput | Very high (millions/sec) | High (100K/sec) |
| Replay | Yes | No |
| Best for | Streaming, event sourcing | Task queues, RPC |

---

## RabbitMQ vs Kafka

| Feature | RabbitMQ | Kafka |
|---------|---------|-------|
| Model | Push (broker pushes to consumer) | Pull (consumer polls broker) |
| Message deletion | On acknowledgment | After TTL/compaction |
| Ordering | Per queue | Per partition |
| Throughput | ~100K msgs/sec | ~1M msgs/sec per partition |
| Replay | No | Yes |
| Routing | Exchanges + routing keys | Topic + partition |
| Use case | Task queues, RPC, complex routing | Event streaming, log aggregation |
| Learning curve | Lower | Higher |

**Use RabbitMQ when:**
- Complex routing (topic exchanges, fanout, direct)
- Short-lived tasks (job queues)
- Low message volume

**Use Kafka when:**
- High throughput (millions/sec)
- Multiple independent consumers of same data
- Event sourcing / audit log
- Data pipeline (Kafka Connect, Kafka Streams)

---

## Event-Driven Architecture Patterns

### Saga Pattern (Distributed Transactions)

**Problem:** Microservices transaction spans multiple services. No distributed ACID transactions.

**Solution:** Saga — sequence of local transactions with compensating transactions on failure.

```
Order Saga:
  1. OrderService: create order (PENDING)
  2. InventoryService: reserve items
     → success → continue
     → fail → compensate: cancel order (CANCELLED)
  3. PaymentService: charge customer
     → success → continue
     → fail → compensate: release inventory, cancel order
  4. ShipmentService: schedule delivery
     → success → update order (CONFIRMED)
     → fail → compensate: refund, release inventory, cancel order

Implementation:
  Choreography: each service publishes event → next service listens (simpler)
  Orchestration: central Saga orchestrator directs sequence (easier to track)
```

### Event Sourcing

**Idea:** Store events, not state. Replay events to rebuild state.

```
Instead of:
  users table: {id, email, balance}
  UPDATE users SET balance = 900 WHERE id = 1

Store events:
  UserCreated {id:1, email:"a@b.com", balance: 1000}
  MoneyWithdrawn {userId:1, amount: 100}
  MoneyDeposited {userId:1, amount: 500}

Current state = replay of all events
Benefit: full audit trail; time travel; rebuild any projection
```

### CQRS (Command Query Responsibility Segregation)

**Idea:** Separate read and write models.

```
Write side: Command → Command Handler → Domain Model → Event Store
Read side:  Query → Read Model (denormalized, optimized for reads) → Response

Event Store → Project events → Read Model

Example: E-commerce
  Write model: Orders, Products (normalized, consistent)
  Read model: ProductSearchIndex (Elasticsearch), UserOrderHistory (DynamoDB)
  Events: ProductUpdated → update search index
```

---

## When to Use Message Queues

✅ **Use a queue when:**
- Processing takes too long to do synchronously (video encoding, report generation)
- You need buffering for traffic spikes
- Multiple systems need the same event (fan-out)
- You need retry/dead-letter handling
- Decoupling is more important than synchronous response

❌ **Don't use a queue when:**
- You need a synchronous response to return to the user
- Simple request-response is sufficient (don't over-engineer)
- ACID transactions across services (use Saga or design around it)

---

## Common Patterns and Problems

### Dead Letter Queue (DLQ)

```
Failed messages (after N retries) → Dead Letter Queue
Operator examines DLQ → fix bug → replay messages

All production Kafka/RabbitMQ systems MUST have DLQ strategy.
```

### Exactly-Once Semantics in Kafka

```
Producer side: idempotent producer (enable.idempotence=true)
  Each message gets sequence number → broker deduplicates

Consumer side: transactional API
  Kafka transaction: read → process → write to output topic → commit offset
  All atomic — if any fails, entire transaction rolls back

Cost: ~3× throughput reduction vs at-least-once
Use only when duplicate processing is truly unacceptable (payments)
```

### Message Ordering

```
Kafka guarantees: ordering within a partition
  → messages with same key → same partition → same order

If you need global ordering: use single partition (limits throughput to ~100MB/s)
If you need per-user ordering: key by userId → all user's messages in order

RabbitMQ: per-queue ordering; if one consumer fails and another retries → out of order
```

---

## Interview Questions

### Q: How does Kafka achieve high throughput?

1. **Sequential disk writes:** Kafka writes to disk sequentially (append-only). Sequential I/O is 100× faster than random I/O (100 MB/s vs 1 MB/s).
2. **Zero-copy:** Kafka uses `sendfile()` syscall — OS copies from disk to network buffer without going through userspace.
3. **Batching:** Producers batch messages; consumers fetch batches — amortizes network round-trip overhead.
4. **Compression:** Messages compressed (LZ4/Snappy) in batches.
5. **Partitioning:** Multiple partitions = parallel writes and reads.

### Q: How would you handle duplicate messages?

**At-least-once delivery is the standard.** Your consumer must be idempotent:
1. **Idempotency key:** Include a unique ID in the message. Consumer checks if already processed.
2. **Database upsert:** `INSERT ... ON CONFLICT DO NOTHING`
3. **Idempotent operations:** Some operations are naturally idempotent (setting a field to a value, vs incrementing it).

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | MQ benefits | Decoupling, buffering, async, fan-out, retry |
| 2 | At-least-once | Retry on failure; ack after processing; may duplicate |
| 3 | Exactly-once | Idempotent producer + transactional consumer; ~3× slower |
| 4 | Kafka partition | Ordered, immutable log; messages with same key → same partition |
| 5 | Kafka consumer group | Each partition → one consumer; consumer count ≤ partition count |
| 6 | Kafka offset commit | Commit after processing (at-least-once); never before |
| 7 | Kafka vs RabbitMQ | Kafka: streaming, log, replay; RabbitMQ: task queues, routing |
| 8 | acks=all | All ISRs ack; max durability |
| 9 | Kafka throughput | Sequential writes, zero-copy, batching, compression |
| 10 | Saga pattern | Sequence of local txns + compensating txns on failure |
| 11 | Choreography | Services emit events; no central orchestrator |
| 12 | Orchestration | Central saga orchestrator directs services |
| 13 | CQRS | Separate read and write models; optimize independently |
| 14 | Dead Letter Queue | Failed messages after N retries; investigate and replay |
| 15 | Event sourcing | Store events not state; replay to rebuild; full audit trail |

---

*Next: [API Design and REST](05-api-design-and-rest.md)*
