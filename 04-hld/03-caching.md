# Caching

Caching is the single most impactful performance optimization in distributed systems. This chapter covers where to cache, how to cache, and how to keep caches consistent.

## Table of Contents
- [Why Cache?](#why-cache)
- [Cache Hierarchy](#cache-hierarchy)
- [Cache Writing Policies](#cache-writing-policies)
- [Cache Eviction Policies](#cache-eviction-policies)
- [Cache Invalidation](#cache-invalidation)
- [Cache Problems and Solutions](#cache-problems-and-solutions)
- [Redis Deep Dive](#redis-deep-dive)
- [CDN Caching](#cdn-caching)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Why Cache?

```
Without cache:  Request → App Server → Database (10ms)
With cache:     Request → App Server → Cache (0.1ms) → 100× faster

Real impact at scale:
  100K requests/sec × 10ms each = 1000 concurrent DB queries needed
  With 95% cache hit rate:
  100K × 5% = 5K requests/sec to DB → 95% reduction
```

**Cache is effective because of:**
- **Temporal locality:** Recently accessed data is likely accessed again soon
- **Pareto principle:** 20% of data serves 80% of requests
- **Computed results:** Expensive aggregations cached, not recomputed

---

## Cache Hierarchy

```
Browser Cache         → CSS, JS, images (days/weeks)
CDN Edge Cache        → Static assets near user (minutes/hours)
API Gateway Cache     → Common API responses (seconds/minutes)
Application Cache     → Service-level cache (Guava Cache, Caffeine)
Distributed Cache     → Redis, Memcached (seconds/hours)
Database Cache        → InnoDB buffer pool, PostgreSQL shared_buffers
```

**Rule of thumb:** Cache as close to the user as possible.

### When to Add Each Layer

| Layer | Add when | Cache size |
|-------|---------|-----------|
| Browser | Static assets, SPA | User's device storage |
| CDN | Images, videos, public API | Globally distributed |
| API Gateway | Identical unauthenticated requests | Configurable per endpoint |
| Application | Computed objects, frequent DB reads | JVM heap / machine RAM |
| Redis | Cross-service shared state, sessions | Dedicated cache cluster |

---

## Cache Writing Policies

### Cache-Aside (Lazy Loading) ← Most Common

```
Read:
  1. Check cache
  2. On miss: read from DB, populate cache, return

Write:
  1. Write to DB
  2. Invalidate or update cache key

Code:
  User user = cache.get(userId);
  if (user == null) {
      user = db.findById(userId); // cache miss
      cache.set(userId, user, TTL);
  }
  return user;
```

**Pros:** Only requested data is cached; cache starts empty (no startup penalty)
**Cons:** Cache miss penalty (extra DB call); stale data between write and invalidation

### Write-Through

```
Write:
  1. Write to cache
  2. Write to DB (synchronously)
  
Read: always hits cache (if not evicted)
```

**Pros:** Cache always has latest data; no cache miss on read
**Cons:** Write latency doubled; cache fills with data that may not be read

### Write-Back (Write-Behind)

```
Write:
  1. Write to cache
  2. Asynchronously flush to DB (queue/background job)

Read: from cache
```

**Pros:** Very fast writes; DB batching possible
**Cons:** Risk of data loss if cache crashes before flush; complex

### Write-Around

```
Write:
  1. Write directly to DB (skip cache)
  
Read: cache-aside pattern

Use when: write-once-read-rarely (logs, events)
```

---

## Cache Eviction Policies

When cache is full, which entry to evict?

| Policy | How it works | Best for |
|--------|-------------|---------|
| **LRU** (Least Recently Used) | Evict least recently accessed | General purpose — most common |
| **LFU** (Least Frequently Used) | Evict least frequently accessed | When frequency matters more than recency |
| **FIFO** (First In, First Out) | Evict oldest inserted entry | Simple; streaming data |
| **TTL** (Time To Live) | Evict after expiry time | All caches — use alongside above |
| **Random** | Evict random entry | Lightweight; surprisingly effective |

**LRU Implementation:**
```java
// Java's built-in LRU cache using LinkedHashMap
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity; // auto-evict when over capacity
    }
}

// Usage
LRUCache<Integer, String> cache = new LRUCache<>(100);
cache.put(1, "value1");
cache.get(1); // marks as recently used
```

---

## Cache Invalidation

**"There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton**

### Strategy 1: TTL-based (Time To Live)

```
cache.set("user:123", userData, 300); // expires in 5 minutes
```

**Simple** but may serve stale data up to TTL seconds.

### Strategy 2: Event-driven Invalidation

```
On user update:
  1. Write to DB
  2. Publish event: "user:123:updated"
  3. Cache listener: delete "user:123" from cache

Pattern: DB trigger → Kafka → Cache invalidation service
```

### Strategy 3: Write-Through

Always write to cache AND DB → cache is always up-to-date.

### Strategy 4: Cache Stamping

Tag cache entries with a version. On data change, increment version → old entries become invalid even without deleting them.

```
cache.set("user:123:v5", userData) // versioned key
tag = db.getVersion("user:123")  // = 5
cache.get("user:123:v" + tag)    // reads versioned key
```

---

## Cache Problems and Solutions

### Problem 1: Cache Stampede (Thundering Herd)

**Situation:** Popular cache entry expires. Simultaneously, 10,000 requests miss the cache and all hit the DB.

**Solutions:**
- **Mutex/Lock:** First thread locks, fetches from DB, populates cache. Others wait for the lock.
```java
String data = cache.get(key);
if (data == null) {
    if (tryLock(key)) {
        try {
            data = db.fetch(key); // only one thread fetches
            cache.set(key, data, TTL);
        } finally { unlock(key); }
    } else {
        Thread.sleep(50); // wait and retry
        data = cache.get(key); // now populated
    }
}
```
- **Probabilistic early expiration:** Before TTL expires, some requests proactively refresh
- **Jitter on TTL:** `TTL = baseTime + random(0, 60)` — spread out expirations

### Problem 2: Cache Penetration

**Situation:** Requests for data that doesn't exist (e.g., user ID -1). DB is queried every time — cache never helps.

**Solutions:**
- **Cache null values:** `cache.set("user:-1", null, 60)` — short TTL for non-existent entries
- **Bloom filter:** Before DB query, check Bloom filter. If definitely not in DB, return null.

```
Bloom filter: probabilistic data structure
  "Is this key definitely NOT in the database?" → yes/no
  False positive possible (says "might be in DB" when it's not)
  No false negatives (if says "not in DB", it's definitely not there)
  Space-efficient: 10 billion items in ~1.2 GB
```

### Problem 3: Cache Avalanche

**Situation:** Many cache entries expire at the same time. DB gets flooded.

**Solutions:**
- **Stagger TTLs:** `TTL = base + random(0, 300)` — randomize expiry times
- **Circuit breaker:** If DB is overloaded, return stale data rather than failing
- **Hot key pre-warming:** Before deployment, pre-load critical data into cache

---

## Redis Deep Dive

### Redis Architecture

```
Single-threaded event loop for command processing
  → no locking needed within Redis
  → commands are atomic by default

Persistence:
  RDB (snapshot): periodic point-in-time snapshot to disk
    → fast restart, some data loss possible (since last snapshot)
  AOF (Append Only File): log every write command
    → slower restart, near-zero data loss
  Both: use RDB for restart speed + AOF for durability
```

### Redis Cluster

```
16384 hash slots distributed across nodes
  hash_slot = CRC16(key) % 16384

3 masters + 3 replicas (minimum):
  Master 1: slots 0-5460
  Master 2: slots 5461-10922
  Master 3: slots 10923-16383

Client → MOVED redirect → correct master
Failover: replica promotes when master fails (seconds)
```

### Redis Sorted Set for Leaderboard

```
ZADD leaderboard 1500 "alice"    // add score
ZADD leaderboard 2000 "bob"
ZADD leaderboard 1800 "charlie"

ZRANGE leaderboard 0 9 REV WITHSCORES  // top 10 (highest first)
ZRANK leaderboard "alice" // rank of alice (0-indexed)
ZINCRBY leaderboard 100 "alice"  // add 100 points
ZRANGEBYSCORE leaderboard 1000 2000 WITHSCORES  // score range
```

### Redis Distributed Lock

```
Acquire lock:
  SET lock:resource "owner-id" NX PX 10000
  // NX = set only if key doesn't exist
  // PX 10000 = expire in 10 seconds

Release lock:
  // Check owner first (Lua script for atomicity)
  EVAL "
    if redis.call('get', KEYS[1]) == ARGV[1] then
      return redis.call('del', KEYS[1])
    else
      return 0
    end
  " 1 lock:resource owner-id
```

---

## CDN Caching

**CDN (Content Delivery Network):** Globally distributed servers that cache content closer to users.

```
Without CDN:
  User in Mumbai → Origin server in US → 150ms latency

With CDN:
  User in Mumbai → CDN edge in Mumbai → 5ms latency

How CDN caches:
  1. User requests file
  2. CDN edge checks local cache
  3. On miss: CDN fetches from origin, caches at edge
  4. Next users in same region: served from edge (cache hit)

Cache headers:
  Cache-Control: max-age=86400        // cache for 1 day
  Cache-Control: no-store             // don't cache (sensitive data)
  Cache-Control: stale-while-revalidate=60  // serve stale, revalidate in background
  ETag: "abc123"                      // fingerprint; 304 Not Modified if unchanged
  Vary: Accept-Encoding               // different cached version per encoding
```

**What to put on CDN:**
- Static assets: JS, CSS, images, fonts → long TTL (1 year with versioned filenames)
- API responses (public, non-personalized) → short TTL (60 seconds)
- Do NOT CDN: authenticated content, personalized responses

---

## Interview Questions

### Q: How would you cache user sessions?

```
Redis with TTL:
  key: "session:{session_id}"
  value: JSON (userId, roles, preferences, lastActivity)
  TTL: 30 minutes (rolling — refresh on each request)

On login:
  session_id = UUID.randomUUID()
  redis.setex("session:" + session_id, 1800, userJson)
  Set-Cookie: session={session_id}; HttpOnly; Secure

On request:
  session_id from cookie → redis.get("session:" + session_id)
  if null → redirect to login
  else    → extend TTL, proceed with request

On logout:
  redis.del("session:" + session_id)
```

### Q: Design a rate limiter using Redis

```
Token Bucket (Redis):
  key: "ratelimit:{userId}"
  Script (Lua for atomicity):
    count = INCR key
    if count == 1 then EXPIRE key window_size
    if count > max_requests return REJECT
    return ALLOW

Sliding Window Counter:
  ZADD ratelimit:{userId} {timestamp} {request_id}
  ZREMRANGEBYSCORE ratelimit:{userId} -inf {timestamp - window}
  count = ZCARD ratelimit:{userId}
  if count > limit return REJECT
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Cache-aside | Check cache → miss → DB → populate cache |
| 2 | Write-through | Write to cache + DB; always consistent, slower write |
| 3 | Write-back | Write to cache; async DB flush; fast but data loss risk |
| 4 | LRU eviction | Evict least recently accessed; most common policy |
| 5 | Cache stampede | Many requests hit DB on expiry; fix: mutex + jitter |
| 6 | Cache penetration | Null data causes DB hit; fix: cache null + Bloom filter |
| 7 | Cache avalanche | Many keys expire together; fix: stagger TTLs |
| 8 | Redis persistence | RDB (snapshot) + AOF (append-only); use both |
| 9 | Redis sorted set | Leaderboard; ZADD, ZRANK, ZRANGE all O(log n) |
| 10 | Redis distributed lock | SET NX PX; release with Lua script for atomicity |
| 11 | CDN edge | Serve static/public content from closest edge node |
| 12 | Cache-Control header | max-age=86400 for static; no-store for sensitive |
| 13 | Bloom filter | Space-efficient; no false negatives; prevents cache penetration |
| 14 | Redis cluster | 16384 hash slots; CRC16(key) % 16384 |
| 15 | TTL jitter | base + random(0, n) to spread cache expiry |

---

*Next: [Load Balancing and Service Mesh](04-load-balancing.md)*
