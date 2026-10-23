# Databases: SQL, NoSQL, and NewSQL

This chapter covers how to choose, design, and scale databases for large systems — the most frequently discussed HLD topic.

## Table of Contents
- [SQL vs NoSQL Decision Framework](#sql-vs-nosql-decision-framework)
- [Relational Database Deep Dive](#relational-database-deep-dive)
- [NoSQL Database Types](#nosql-database-types)
- [Database Scaling Patterns](#database-scaling-patterns)
- [Database Sharding](#database-sharding)
- [NewSQL Databases](#newsql-databases)
- [Database Selection Guide](#database-selection-guide)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## SQL vs NoSQL Decision Framework

Choose SQL (Relational) when:
- Data has complex relationships (joins are common)
- ACID transactions are required (payments, inventory)
- Schema is well-defined and unlikely to change drastically
- You need rich query capabilities (aggregations, joins)
- Team is familiar with SQL

Choose NoSQL when:
- Massive scale (billions of records, millions of writes/sec)
- Schema is flexible/variable (documents, hierarchical data)
- Simple access patterns (lookup by key, range scan)
- Horizontal scaling from day one
- Specific data structures needed (graphs, time-series)

**Don't choose NoSQL because it's "modern"** — choose it when the trade-offs make sense.

---

## Relational Database Deep Dive

### ACID Properties

```
Atomicity   — Transaction is all-or-nothing
Consistency — DB moves from one valid state to another
Isolation   — Concurrent transactions don't interfere
Durability  — Committed transactions survive crashes
```

**Real example: Bank transfer**
```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 500 WHERE account_id = 'A';
  UPDATE accounts SET balance = balance + 500 WHERE account_id = 'B';
  -- If either fails, both roll back (Atomicity)
COMMIT;
```

### Indexes

```sql
-- B-Tree index (default) — fast for =, <, >, BETWEEN, ORDER BY
CREATE INDEX idx_user_email ON users(email);

-- Composite index — order matters!
CREATE INDEX idx_order_user_created ON orders(user_id, created_at);
-- Good for: WHERE user_id = ? AND created_at > ?
-- Bad for: WHERE created_at > ? (doesn't use index efficiently)

-- Partial index — index only matching rows
CREATE INDEX idx_active_users ON users(email) WHERE status = 'ACTIVE';

-- Hash index — fast for exact match only (no ranges)
-- Available in PostgreSQL, not MySQL InnoDB
```

**Index internals (B-Tree):**
```
All index entries in sorted order → binary search = O(log n)
Root node → branch nodes → leaf nodes → heap (actual row data)
Covering index: when all queried columns are in the index → no heap lookup needed
```

**When indexes hurt:**
- Write-heavy tables — every write must update all indexes
- Low-cardinality columns (e.g., boolean status) — full scan often faster
- Too many indexes → slow writes, large disk footprint

### Query Optimization

```sql
-- EXPLAIN shows query execution plan
EXPLAIN SELECT * FROM orders 
WHERE user_id = 123 AND status = 'PENDING'
ORDER BY created_at DESC 
LIMIT 10;

-- Look for:
-- Seq Scan → no index used (bad for large tables)
-- Index Scan → good
-- Index Only Scan → covering index (best)
-- Hash Join vs Nested Loop vs Merge Join
```

**Common query optimization techniques:**
1. Add missing indexes
2. Use covering indexes to avoid table lookups
3. Avoid `SELECT *` — fetch only needed columns
4. Use `LIMIT` for pagination
5. Avoid `DISTINCT` and `GROUP BY` on large datasets without indexes
6. Use connection pooling (PgBouncer, HikariCP)

### Replication

```
Master-Replica (Leader-Follower):
  - All writes → master
  - Reads can go to replicas
  - Replication lag: replicas may be seconds behind
  - Failover: promote replica to master on master failure

Multi-Master:
  - Writes to any node
  - Conflict resolution needed
  - Used in Galera Cluster (MySQL), CockroachDB

Read replicas scaling:
  100K reads/sec → one master + 10 replicas (10K reads each)
  Need: replication factor + load balancer in front of replicas
```

---

## NoSQL Database Types

### 1. Key-Value Store

**Examples:** Redis, DynamoDB, Memcached, Amazon ElastiCache

**Structure:** `key → value` (value is opaque blob or rich type in Redis)

**Redis data types:**
```
String    → GET/SET key
Hash      → HGET/HSET (user profile fields)
List      → LPUSH/RPOP (message queue, activity feed)
Set       → SADD/SMEMBERS (unique visitors, tags)
Sorted Set → ZADD/ZRANGE (leaderboard, top-K)
HyperLogLog → PFADD/PFCOUNT (approximate unique count)
```

**Use cases:**
- Session storage (`session_id → user_data`)
- Caching (DB query results)
- Rate limiting (counter per user per second)
- Pub/Sub messaging
- Distributed locks (`SET key NX PX 10000`)

---

### 2. Document Store

**Examples:** MongoDB, Firestore, CouchDB, DynamoDB (document mode)

**Structure:** Collections of JSON-like documents with nested fields.

```json
{
  "_id": "order-123",
  "userId": "user-456",
  "items": [
    {"productId": "prod-1", "qty": 2, "price": 29.99},
    {"productId": "prod-2", "qty": 1, "price": 9.99}
  ],
  "totalAmount": 69.97,
  "status": "SHIPPED",
  "shippingAddress": {
    "street": "123 Main St",
    "city": "Mumbai",
    "pincode": "400001"
  }
}
```

**When to use:**
- Hierarchical data that belongs together (order + items)
- Flexible schema (user profiles with optional fields)
- Rapid development with evolving schema

**When NOT to use:**
- Frequent cross-document joins
- Strong transactional consistency across documents

---

### 3. Column-Family Store

**Examples:** Apache Cassandra, HBase, ScyllaDB

**Structure:** Rows × columns, but sparse (each row can have different columns). Designed for massive write throughput.

**Cassandra data model:**
```cql
-- Design your schema around your query patterns!
-- Bad: design table, then write queries
-- Good: know query, then design table

-- Query: "Get all messages for user X, ordered by time"
CREATE TABLE messages_by_user (
    user_id UUID,
    message_time TIMESTAMP,
    message_id UUID,
    content TEXT,
    sender_id UUID,
    PRIMARY KEY (user_id, message_time)  -- partition key + clustering key
) WITH CLUSTERING ORDER BY (message_time DESC);

-- Partition key (user_id): determines which node stores the data
-- Clustering key (message_time): ordering within partition
```

**Cassandra write path:**
1. Write to commit log (WAL) → durable immediately
2. Write to MemTable (in-memory) → fast
3. Periodically flush MemTable → SSTable on disk
4. Compaction: merge SSTables → remove tombstones (deleted records)

**CAP position:** AP (eventual consistency) — designed for availability and partition tolerance.

**Use cases:**
- Time-series data (IoT metrics, logs)
- Messages/chat (partitioned by conversation)
- Activity feeds (partitioned by user)
- Writes: 100K+/sec per node

---

### 4. Graph Database

**Examples:** Neo4j, Amazon Neptune, JanusGraph

**Structure:** Nodes + Edges with properties.

```
Use when: relationships ARE the data.
  - Social network (FRIEND, FOLLOWS relationships)
  - Recommendation engine (users who bought X also bought Y)
  - Fraud detection (transaction graphs)
  - Knowledge graphs

Cypher query (Neo4j):
MATCH (user:User {id: '123'})-[:FOLLOWS]->(friend)-[:FOLLOWS]->(fofriend)
WHERE NOT (user)-[:FOLLOWS]->(fofriend)  -- not already following
RETURN fofriend.name, COUNT(*) AS mutualFriends
ORDER BY mutualFriends DESC LIMIT 10
-- "People you may know" recommendation
```

---

## Database Scaling Patterns

### Connection Pooling

```
App servers → Connection Pool → Database
              (PgBouncer for Postgres, HikariCP for Java)

Without pool: each request creates/destroys DB connection (~50ms overhead)
With pool: connections pre-established; borrow/return from pool (~1ms overhead)

Config:
  pool_size: 10-50 per app server (tune to DB max_connections)
  max_connections in DB = num_app_servers × pool_size
```

### Read Replicas

```
Write → Master
Reads → Round-robin across replicas

Replication lag: typically 10-100ms (can be seconds under load)
Use case for eventual consistency: social feeds, product catalog
Use master for read: immediately after write (read-your-writes)
```

### Caching Layer (Write-Around / Write-Through / Write-Back)

```
Write-Through: write to cache + DB simultaneously (consistent, slower write)
Write-Around:  write only to DB; cache populated on read miss (cache may be stale)
Write-Back:    write to cache; async write to DB (fast, risk of data loss)

Most common: Write-Around for reads, separate invalidation on writes
```

---

## Database Sharding

**Sharding** = horizontal partitioning — split data across multiple database instances.

### Sharding Strategies

#### Range Sharding
```
Shard 1: user_id 1 – 1,000,000
Shard 2: user_id 1,000,001 – 2,000,000

Problem: hotspot if recent users are more active (all traffic to last shard)
Good for: time-series data sharded by date
```

#### Hash Sharding
```
shard = hash(user_id) % num_shards

Problem: resharding (adding shards) requires moving all data
Good for: uniform distribution
```

#### Consistent Hashing
```
Virtual ring of 360 degrees (or 2^32 slots)
Each shard owns a range of the ring
New shard added: takes from neighbors only, minimizes data movement
Used by: Cassandra, DynamoDB, Amazon Dynamo

   0°
   ↑  Shard 1 (0-90°)
90°    Shard 2 (90-180°)
   ↓  Shard 3 (180-270°)
270°   Shard 4 (270-360°)

key → hash → position on ring → clockwise to nearest shard
```

### Sharding Challenges

| Challenge | Problem | Solution |
|-----------|---------|---------|
| Cross-shard joins | Users on shard 1, orders on shard 2 | Denormalize data, or aggregate in application |
| Cross-shard transactions | ACID across shards | 2PC (slow), Saga pattern, or design to avoid |
| Hotspot | One shard gets all traffic | Better shard key, consistent hashing |
| Resharding | Adding shards moves data | Consistent hashing reduces movement |
| Global unique ID | Auto-increment doesn't work across shards | Snowflake ID, UUID |

---

## NewSQL Databases

**NewSQL** combines SQL ACID semantics with NoSQL horizontal scalability.

| Database | Notes |
|----------|-------|
| **Google Spanner** | Globally distributed, external consistency, uses TrueTime API |
| **CockroachDB** | Open-source Spanner-like, PostreSQL wire protocol |
| **YugabyteDB** | PostgreSQL-compatible, distributed |
| **TiDB** | MySQL-compatible, Raft consensus for replication |

**Use when:** You need PostgreSQL semantics at Cassandra scale. Expensive — used at Google, Stripe, etc.

---

## Database Selection Guide

| Use Case | Recommended DB | Why |
|----------|---------------|-----|
| Financial transactions | PostgreSQL | ACID, strong consistency |
| User profiles | MongoDB or PostgreSQL | Flexible schema or relational |
| Session storage | Redis | Fast key-value, TTL support |
| Activity feed | Cassandra | Write-heavy, time-ordered, partition by user |
| Search | Elasticsearch | Full-text, faceted search |
| Analytics | Redshift, BigQuery | Columnar, optimized for aggregations |
| Time-series (IoT) | TimescaleDB, InfluxDB | Optimized for time-range queries |
| Social graph | Neo4j | Native graph traversal |
| Object storage | S3 | Blob storage for images, videos |
| Leaderboard | Redis Sorted Set | O(log n) rank queries |
| Distributed lock | Redis SETNX or ZooKeeper | Atomic lock with TTL |

---

## Interview Questions

### Q: How would you shard a social network's user database?

**Shard key: user_id** (consistent hashing)
- Why user_id: most queries are by user
- Consistent hashing: minimizes resharding overhead

**Challenge:** "Find all friends of user X" — friends table is also sharded. Solution: store friendship bidirectionally (on both user's shards) or use a separate friendship graph service.

**Challenge:** Famous users (Cristiano Ronaldo with 500M followers) — hotspot. Solution: Separate "celebrity" tier, or fan-out on write to pre-computed feeds.

### Q: When to use denormalization in a SQL schema?

Denormalization adds redundancy to improve read performance:
- When joins are too expensive (frequent, large tables)
- When data is read much more than written
- When consistency of denormalized data can be managed

```sql
-- Instead of joining users + orders every time:
-- Add user_name, user_email to the orders table (redundant but fast reads)
ALTER TABLE orders ADD COLUMN user_name VARCHAR(255);
-- Now: SELECT * FROM orders WHERE order_id = ? -- no join needed
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | ACID: A | Atomicity — all or nothing |
| 2 | ACID: C | Consistency — valid state to valid state |
| 3 | ACID: I | Isolation — concurrent transactions don't interfere |
| 4 | ACID: D | Durability — committed data survives crashes |
| 5 | B-Tree index | O(log n) for =, <, >, BETWEEN, ORDER BY |
| 6 | Composite index order | Match query's WHERE clause prefix |
| 7 | Cassandra primary key | (partition key, clustering key) — partition = node, cluster = sort |
| 8 | Cassandra write path | Commit log → MemTable → SSTable |
| 9 | Consistent hashing | Virtual ring; new node takes from neighbors only |
| 10 | Range sharding risk | Hotspot on recent data |
| 11 | Hash sharding risk | Resharding = move all data |
| 12 | Redis sorted set use | Leaderboard; O(log n) rank |
| 13 | Read replica replication lag | Typically 10-100ms; seconds under load |
| 14 | NewSQL | ACID at NoSQL scale; Spanner, CockroachDB |
| 15 | Cross-shard joins | Avoid by denormalization or application-level joins |

---

*Next: [Caching](03-caching.md)*
