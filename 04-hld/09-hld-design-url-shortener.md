# HLD Case Study: Design a URL Shortener (like bit.ly)

This classic system design question tests capacity estimation, database design, caching, and distributed ID generation.

## Table of Contents
- [Requirements and Clarifications](#requirements-and-clarifications)
- [Capacity Estimation](#capacity-estimation)
- [API Design](#api-design)
- [Data Model](#data-model)
- [High-Level Architecture](#high-level-architecture)
- [Deep Dive: ID Generation](#deep-dive-id-generation)
- [Deep Dive: Caching Strategy](#deep-dive-caching-strategy)
- [Deep Dive: Analytics](#deep-dive-analytics)
- [Scalability Analysis](#scalability-analysis)
- [Interview Tips](#interview-tips)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Requirements and Clarifications

**Ask these questions:**

1. "Is this a public service or internal?" → public (affects auth requirements)
2. "What's the expected QPS?" → 1000 writes/sec, 100K reads/sec (read-heavy)
3. "Should short URLs expire?" → Yes, configurable, default no expiry
4. "Do we need analytics (click tracking)?" → Yes, basic (total clicks, country, browser)
5. "Should users be able to create custom aliases?" → Nice to have

**Functional Requirements:**
1. Given a long URL, generate a short URL (7-8 characters)
2. Redirect short URL → long URL
3. URLs optionally expire
4. Analytics: click count per URL

**Non-Functional Requirements:**
- High availability: 99.99%
- Low latency for redirects: <50ms
- Scale: 100K reads/sec, 1000 writes/sec
- Durability: URLs stored permanently (unless expired)

---

## Capacity Estimation

```
Writes:
  1000 URL shortenings/sec
  1000 × 86,400 = 86.4M URLs/day
  86.4M × 365 = ~31.5B URLs/year
  
  Storage per record:
    shortCode: 8 bytes
    longUrl: 200 bytes (average)
    userId: 8 bytes
    createdAt: 8 bytes
    expiresAt: 8 bytes
    clickCount: 8 bytes
    Total: ~250 bytes
  
  5-year storage: 31.5B × 5 × 250 bytes = ~40 TB

Reads:
  100K redirect requests/sec
  Read/Write ratio = 100:1 (very read-heavy)
  
  With cache (90% hit rate): 10K requests/sec to DB

Bandwidth:
  Read: 100K × 200 bytes = 20 MB/s
  Write: 1000 × 250 bytes = 250 KB/s
```

---

## API Design

```
POST /api/urls
Request:
  {
    "longUrl": "https://www.example.com/very-long-path?with=query&params=many",
    "customAlias": "mylink",      // optional
    "expiresAt": "2025-12-31"     // optional
  }
Response: 201 Created
  {
    "shortUrl": "https://bit.ly/abc1234",
    "shortCode": "abc1234",
    "longUrl": "https://www.example.com/...",
    "expiresAt": "2025-12-31"
  }

GET /{shortCode}
Response: 302 Found
  Location: https://www.example.com/very-long-path?...
  
  (302 = temporary redirect — client doesn't cache, analytics more accurate)
  (301 = permanent — browser caches, reduces server load but breaks analytics)

GET /api/urls/{shortCode}/analytics
Response: 200 OK
  {
    "shortCode": "abc1234",
    "totalClicks": 15234,
    "clicksByCountry": {"IN": 5000, "US": 4000, ...},
    "clicksByDay": [{"date": "2024-01-01", "count": 500}, ...]
  }

DELETE /api/urls/{shortCode}
Response: 204 No Content

Rate Limiting:
  Anonymous: 10 URLs/day
  Authenticated: 1000 URLs/day
  Header: X-RateLimit-Remaining: 999
```

---

## Data Model

### Primary URL table

```sql
CREATE TABLE urls (
    short_code    VARCHAR(10)   PRIMARY KEY,
    long_url      TEXT          NOT NULL,
    user_id       BIGINT,
    created_at    TIMESTAMP     DEFAULT NOW(),
    expires_at    TIMESTAMP,
    click_count   BIGINT        DEFAULT 0,
    is_active     BOOLEAN       DEFAULT TRUE
);

CREATE INDEX idx_urls_user_id ON urls(user_id);
CREATE INDEX idx_urls_expires_at ON urls(expires_at) WHERE expires_at IS NOT NULL;
```

### Click Analytics Table (Cassandra or ClickHouse)

```
-- For analytics: use time-series database, not OLTP
-- Cassandra: write-optimized, partition by shortCode

CREATE TABLE click_events (
    short_code   TEXT,
    clicked_at   TIMESTAMP,
    click_id     UUID,
    country      TEXT,
    browser      TEXT,
    device       TEXT,
    ip_address   INET,
    PRIMARY KEY (short_code, clicked_at, click_id)
) WITH CLUSTERING ORDER BY (clicked_at DESC);

-- Query last 100 clicks for a URL:
SELECT * FROM click_events WHERE short_code = 'abc1234' LIMIT 100;
```

### Database Choice

| Data | Database | Why |
|------|---------|-----|
| URLs (shortCode → longURL) | PostgreSQL or DynamoDB | ACID, simple key lookup |
| Sessions/cache | Redis | Fast key-value, TTL |
| Analytics events | Cassandra or ClickHouse | Write-heavy, time-series |
| User accounts | PostgreSQL | Relational, ACID |

---

## High-Level Architecture

```
                    ┌─────────────────────────────────────┐
                    │           DNS / Load Balancer        │
                    └─────────────┬─────────────┬─────────┘
                                  │             │
                    ┌─────────────▼──────┐  ┌───▼─────────────────┐
                    │   Redirect Service  │  │   Write Service      │
                    │   (read-optimized)  │  │   (URL shortening)   │
                    └─────────────┬───────┘  └───┬─────────────────┘
                                  │              │
                    ┌─────────────▼──────┐       │ID from:
                    │   Redis Cache       │       │Snowflake / Base62
                    │   shortCode→longURL │       │
                    └─────────────┬───────┘  ┌───▼─────────────────┐
                                  │ (miss)   │   PostgreSQL         │
                    ┌─────────────▼──────────▼─────────────────────┐
                    │               PostgreSQL (Primary)             │
                    │               + Read Replicas                  │
                    └─────────────────────────────────────────────┘
                                  │ (async click events)
                    ┌─────────────▼──────────────────────────────┐
                    │        Kafka → Cassandra (Analytics)        │
                    └────────────────────────────────────────────┘
```

**Request flow (redirect):**
1. User hits `bit.ly/abc1234`
2. DNS → Load Balancer → Redirect Service
3. Redirect Service: check Redis cache
4. Cache hit (90%): return 302 with `Location: longUrl`
5. Cache miss: query PostgreSQL, populate cache, return 302
6. Async: publish click event to Kafka → Cassandra stores analytics

---

## Deep Dive: ID Generation

How to generate the short code (`abc1234`)?

### Option 1: MD5 / Hash

```
hash = MD5(longUrl).substring(0, 7)  // 7 chars from MD5
Problem:
  - Same URL → same hash (good for dedup, bad if you want multiple short URLs per long URL)
  - Hash collision (different URLs → same hash) → must handle
  - MD5 output is hex (0-9, a-f) → 7 chars = 16^7 = 268M → not enough for billions
```

### Option 2: Auto-increment + Base62 encoding ← Recommended

```
Generate unique numeric ID → encode in base62

Base62 alphabet: 0-9a-zA-Z (62 characters)
7 characters → 62^7 = 3.5 trillion unique codes → sufficient for decades

ID = auto-increment or Snowflake ID
short_code = base62Encode(id)

Example:
  id = 12345678
  12345678 % 62 = 38 → 'c'  (0..9=0-9, 10..35=a-z, 36..61=A-Z)
  12345678 / 62 = 199123
  ... recurse until 0
  short_code = "dnh75C"
```

```java
// Base62 encoding
private static final String BASE62 = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

public String encode(long id) {
    StringBuilder sb = new StringBuilder();
    while (id > 0) {
        sb.append(BASE62.charAt((int)(id % 62)));
        id /= 62;
    }
    return sb.reverse().toString();
}

public long decode(String code) {
    long id = 0;
    for (char c : code.toCharArray()) {
        id = id * 62 + BASE62.indexOf(c);
    }
    return id;
}
```

### Option 3: Snowflake ID

```
64-bit integer:
  1 bit:  sign bit (0)
  41 bits: timestamp (milliseconds since epoch) → 69 years
  10 bits: machine/datacenter ID → 1024 machines
  12 bits: sequence number → 4096 IDs/ms per machine

Total: 4096 × 1024 machines × 1000ms = ~4 billion IDs/second

Benefits:
  - Sortable (time-ordered)
  - No coordination needed (machine ID embedded)
  - No DB auto-increment bottleneck
```

---

## Deep Dive: Caching Strategy

```
Cache-aside with TTL:
  key:   "url:{shortCode}"
  value: longUrl
  TTL:   24 hours (most URLs are hot for a day)

Cache sizing:
  100K reads/sec × 90% cache hit = 90K reads from cache
  10K cache entries × 200 bytes = 2MB → tiny! Redis handles millions of entries

Eviction: LRU (popular URLs stay; cold URLs evicted)

Cache warming: no need — cache fills naturally on first access

On URL expiry: 
  Set Redis TTL = min(URL expiry, 24 hours)
  Or: on access, check DB expiry and return 410 Gone if expired
```

---

## Deep Dive: Analytics

**Challenge:** 100K redirect requests/sec means 100K click events/sec. Direct DB writes would kill the database.

**Solution: Async pipeline**

```
Redirect Service
  → Log click event to local buffer
  → Flush to Kafka every 100ms (batch)
  → Kafka "click-events" topic
  → Consumer group: ClickEventProcessor
  → Write to Cassandra (time-series)
  → Daily aggregation job: Cassandra → ClickHouse for dashboards

Real-time count:
  Redis INCR "clicks:{shortCode}" on each redirect
  Periodic background job: sync Redis count to PostgreSQL
```

---

## Scalability Analysis

| Component | Current | At 10× scale |
|-----------|---------|-------------|
| Redirect Service | 3 instances | 30 instances (stateless, auto-scale) |
| Write Service | 2 instances | 10 instances (stateless) |
| PostgreSQL | 1 primary, 2 replicas | Read replicas + shard by shortCode prefix |
| Redis | Single node | Redis Cluster (16384 slots) |
| Kafka | 3 brokers | 10 brokers, more partitions |
| Cassandra | 3 nodes | 10 nodes (linear scale) |

---

## Interview Tips

1. **Start with requirements** — "Is this read-heavy?" (yes, 100:1). "Do we need analytics?" affects architecture significantly.

2. **Capacity estimation early** — "100K reads/sec tells me caching is critical."

3. **ID generation depth** — candidates who explain Base62 + Snowflake differentiate themselves.

4. **302 vs 301 redirect** — "302 is preferred for analytics (browser doesn't cache) even though it's slightly slower."

5. **Analytics as a separate concern** — "I'd use a separate analytics pipeline (Kafka + Cassandra) to avoid impacting the read path."

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Read/write ratio | ~100:1 (very read-heavy) |
| 2 | Short code length | 7 chars in base62 = 3.5 trillion unique codes |
| 3 | Base62 alphabet | 0-9a-zA-Z (62 chars) |
| 4 | Snowflake ID bits | 41 timestamp + 10 machine + 12 sequence |
| 5 | Redirect type | 302 (analytics) vs 301 (browser caches, less load) |
| 6 | Cache strategy | Cache-aside; key = "url:{shortCode}"; TTL = 24h |
| 7 | Cache hit target | 90%+ with LRU |
| 8 | Analytics approach | Kafka → Cassandra (avoid direct DB writes at 100K/sec) |
| 9 | Click count real-time | Redis INCR → periodic sync to PostgreSQL |
| 10 | URL expiry | Cassandra TTL or check in redirect service |
| 11 | Rate limiting | Redis counter per user/IP with sliding window |
| 12 | Storage 5 years | ~40 TB for billions of URLs |
| 13 | DB choice: URLs | PostgreSQL (ACID) or DynamoDB (scale) |
| 14 | Horizontal scaling | Redirect service is stateless → easy scale-out |
| 15 | ID collision prevention | Auto-increment ID → no collision by definition |

---

*Next: [HLD Case Study — Design Twitter/Feed System](10-hld-design-twitter.md)*
