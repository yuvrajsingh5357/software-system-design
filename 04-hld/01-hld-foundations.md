# HLD Foundations: Scalability and Core Concepts

This chapter lays the groundwork for all system design discussions. These concepts appear in every HLD interview — you must be able to discuss them fluently.

## Table of Contents
- [The System Design Interview Framework](#the-system-design-interview-framework)
- [Horizontal vs Vertical Scaling](#horizontal-vs-vertical-scaling)
- [CAP Theorem](#cap-theorem)
- [PACELC Theorem](#pacelc-theorem)
- [Latency, Throughput, and Availability](#latency-throughput-and-availability)
- [SLAs, SLOs, and SLIs](#slas-slos-and-slis)
- [Back-of-Envelope Estimation](#back-of-envelope-estimation)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## The System Design Interview Framework

Use the **RADIO framework** (Requirements, API, Data model, Infrastructure/architecture, Optimization):

```
Step 1: Clarify Requirements (5 min)
   - Functional: What does the system do?
   - Non-functional: Scale, availability, latency, consistency requirements
   - Constraints: Users, QPS, data size, budget

Step 2: Capacity Estimation (3-5 min)
   - Daily Active Users (DAU) → QPS
   - Storage requirements
   - Bandwidth

Step 3: API Design (2-3 min)
   - Core REST endpoints
   - Request/response formats

Step 4: Data Model (3-5 min)
   - Key entities and relationships
   - Database choice (SQL vs NoSQL) and justification

Step 5: High-Level Architecture (10-15 min)
   - Draw the system
   - Walk through request flow
   - Identify components: load balancer, services, databases, caches, CDN

Step 6: Deep Dive (10-15 min)
   - Interviewer picks one area: database design, caching, consistency
   - Address bottlenecks proactively

Step 7: Wrap-up (2 min)
   - Monitoring, alerting, on-call
   - Future improvements
```

---

## Horizontal vs Vertical Scaling

| | Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
|-|-----------------------------|-------------------------------|
| What | Bigger machine (more CPU, RAM) | More machines |
| Limit | Hardware limit — eventually impossible | No hard limit |
| Cost | Exponential cost above a point | Linear cost |
| Downtime | Requires restart | Can be done live (rolling deploy) |
| Failure | Single point of failure | N-1 redundancy |
| Complexity | Low — one machine | High — need load balancer, coordination |
| Best for | Databases (initially), stateful services | Stateless services, microservices |

**Common interview trap:** "Can't we just get a bigger server?" Yes, up to a point — but at scale you MUST scale out. Vertical scaling has a hard limit and doesn't give you redundancy.

---

## CAP Theorem

**In the presence of a network partition, a distributed system can provide at most 2 of 3 guarantees:**

```
C — Consistency: Every read returns the most recent write
A — Availability: Every request receives a response (not necessarily consistent)
P — Partition Tolerance: System operates despite network partitions

In a distributed system, P is not optional (network partitions WILL happen).
Therefore: choose C or A when partition occurs.

CP system: Returns error if data may be stale
AP system: Returns possibly stale data rather than error
```

**Database trade-off table:**

| System | Type | When to use |
|--------|------|-------------|
| MySQL/PostgreSQL | CP | Financial transactions, inventory, reservations |
| Cassandra | AP | User activity logs, time-series, notifications |
| MongoDB | CP (default) | General purpose documents |
| DynamoDB | Configurable | Reads: eventual/strong; writes: conditional |
| Redis (cluster) | AP | Caching, session store |
| ZooKeeper | CP | Distributed coordination, leader election |
| HBase | CP | Real-time read/write on big data |

**Interview usage:**

> "For the payment service, I'll use PostgreSQL with strong consistency (CP) — we cannot have a user charged twice. For the activity feed, I'll use Cassandra (AP) — eventual consistency is acceptable; a user can see their post appear a few seconds late."

---

## PACELC Theorem

CAP only addresses partitions. PACELC extends it:

```
If Partition:   choose between Availability and Consistency
Else (normal):  choose between Latency and Consistency
```

**Why it matters:** Even without partitions, distributed systems face a latency vs consistency trade-off:
- Strong consistency requires coordination (quorum, 2PC) → higher latency
- Eventual consistency skips coordination → lower latency

| System | Partition choice | Normal choice |
|--------|-----------------|---------------|
| DynamoDB | PA | EL (eventual) |
| Cassandra | PA | EL |
| MySQL | PC | EC (consistent, higher latency) |
| Spanner | PC | EC |
| HBase | PC | EC |

---

## Latency, Throughput, and Availability

### Latency Numbers (commit to memory)

```
L1 cache reference:               ~1 ns
Branch mispredict:                ~5 ns
L2 cache reference:               ~7 ns
Mutex lock/unlock:                ~100 ns
Memory access (RAM):              ~100 ns
Compress 1KB with Zippy:          ~10 μs
Send 2KB over 1 Gbps network:     ~20 μs
SSD random read:                  ~100 μs  (0.1 ms)
Read 1 MB sequentially from RAM:  ~250 μs
Round trip within datacenter:     ~500 μs
HDD seek + read:                  ~10 ms
Read 1 MB from SSD:               ~1 ms
Packet SFO → NYC → SFO:           ~150 ms
```

**Rules of thumb:**
- SSD >> HDD for latency (~100× faster)
- Network within datacenter ~0.5ms; cross-region ~150ms
- RAM >> SSD >> HDD

### Throughput

**Little's Law:** `L = λW` — Average number in system = arrival rate × average time spent

```
If a system processes 1000 requests and each takes 1 second:
L = 1000 req in flight = 1000 req/sec × 1 sec
```

### Availability

```
Availability (%) = Uptime / (Uptime + Downtime) × 100

Nines of availability:
  99%     = 3.65 days/year downtime
  99.9%   = 8.76 hours/year downtime
  99.99%  = 52.6 minutes/year downtime
  99.999% = 5.26 minutes/year downtime

Achieving 99.99%:
  - Active-passive failover (< 1 min failover)
  - Database replication
  - Multi-AZ deployment
  - Health checks + auto-restart

Achieving 99.999%:
  - Multi-region active-active
  - Zero-downtime deployments
  - Chaos engineering
```

---

## SLAs, SLOs, and SLIs

| Term | What it is | Example |
|------|-----------|---------|
| **SLI** (Service Level Indicator) | A metric you measure | Request success rate, P99 latency |
| **SLO** (Service Level Objective) | Internal target for an SLI | 99.9% requests < 200ms, 99.95% success rate |
| **SLA** (Service Level Agreement) | External contract with penalty | 99.9% availability or 10% discount |

**Typical SLIs to track:**
- **Availability:** `successful_requests / total_requests`
- **Latency:** P50, P95, P99 response time
- **Error rate:** `5xx responses / total responses`
- **Throughput:** requests/second

**Error budget:** `1 - SLO`. If SLO is 99.9%, error budget = 0.1% = 43.8 minutes/month. If you burn through it, freeze new deployments.

---

## Back-of-Envelope Estimation

**Master these numbers:**

```
1 million  = 10^6
1 billion  = 10^9
1 trillion = 10^12

Data sizes:
  1 char = 1 byte
  1 int  = 4 bytes
  UUID   = 16 bytes
  Image  = ~200 KB
  Video  = ~1 GB/hour

QPS Estimation:
  DAU × actions_per_day ÷ 86,400 seconds = QPS
  
  Example: Twitter
    300M DAU × 3 tweets/day = 900M tweets/day
    900M / 86,400 ≈ 10,000 writes/sec
    Read-to-write = 100:1 → 1,000,000 reads/sec
```

**Example: Design YouTube storage**
```
DAU: 100M users
Videos uploaded per day: 500 hours/minute = 30,000 hours/day
Average video: 1 GB/hour = 30 TB/day = ~1 PB/month
With 3x replication: ~3 PB/month
With encoding (360p, 720p, 1080p): ~5x expansion = ~15 PB/month
```

**Storage per user record:**
```
User profile: 1 KB
Product catalog: 10 bytes per item, 100M products = 1 GB
Tweets (280 chars): 500 bytes each
```

---

## Interview Questions

### Q: If CAP says only 2 of 3, why do databases claim to offer all three?

Network partitions are rare but inevitable. CAP says: when a partition happens, choose C or A. During normal operation (no partition), a well-designed system can offer consistency AND availability. The catch is only when partitions occur.

Also, "consistency" in CAP is linearizability — the strictest form. Databases offer weaker consistency levels (read-your-writes, monotonic reads) that may feel like "consistency" in practice without the full penalty.

### Q: When would you choose eventual consistency?

- User-facing features where staleness for a few seconds is acceptable: social feeds, notification counts, view counts, recommendations
- High write throughput requirements where strong consistency would be a bottleneck
- Global replication where synchronous writes across regions are too slow
- Examples: Instagram feeds, YouTube view counts, Amazon product recommendations

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | RADIO framework | Requirements → API → Data → Infrastructure → Optimization |
| 2 | Vertical scaling limit | Hardware limit; single point of failure |
| 3 | Horizontal scaling | Stateless services scale easily; stateful needs coordination |
| 4 | CAP: P is not optional | Network partitions happen; choose C or A |
| 5 | CP systems | MySQL, PostgreSQL, ZooKeeper — use for financial data |
| 6 | AP systems | Cassandra, DynamoDB (eventual) — use for social feeds |
| 7 | SSD vs HDD latency | SSD ~0.1ms; HDD ~10ms; 100× difference |
| 8 | Datacenter round trip | ~0.5ms; cross-region ~150ms |
| 9 | 99.99% availability | 52.6 min downtime/year; needs multi-AZ failover |
| 10 | SLI | Measurement: success rate, P99 latency |
| 11 | SLO | Internal target: 99.9% requests < 200ms |
| 12 | SLA | External contract with penalty |
| 13 | Error budget | 1 - SLO; burn through it → freeze deployments |
| 14 | QPS formula | DAU × actions/day ÷ 86,400 |
| 15 | PACELC | Adds: even without partition → latency vs consistency trade-off |

---

*Next: [Databases: SQL, NoSQL, and NewSQL](02-databases-sql-nosql-newsql.md)*
