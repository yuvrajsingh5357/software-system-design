# Redis Architecture and Use Cases

## 1. What Is It?
Redis (Remote Dictionary Server) is an open-source, in-memory, key-value data store. Because it stores data in RAM, it delivers sub-millisecond response times, making it ideal for caching, session management, real-time analytics, and rate limiting.

## 2. Data Types
Unlike Memcached, which only supports simple strings, Redis supports complex data structures:
- **Strings:** Simple text or binary data (max 512MB). Used for JSON caching or simple counters.
- **Lists:** Linked lists. Used for queues or timeline feeds.
- **Sets:** Unordered collections of unique strings. Used for tracking unique IP addresses.
- **Sorted Sets:** Sets ordered by a "score". Perfect for leaderboards or rate limiting by timestamp.
- **Hashes:** Maps between string fields and string values. Perfect for storing objects (like User Sessions).

## 3. Caching Patterns
- **Cache-Aside (Lazy Loading):** The application first asks the cache. If a cache miss occurs, the application queries the DB, saves the result to the cache, and returns it. (Most common in Spring via `@Cacheable`).
- **Write-Through:** The application writes data to the cache AND the database simultaneously. Ensures the cache is always fresh, but adds latency to writes.

## 4. Cache Invalidation and Eviction
- **TTL (Time-To-Live):** Data automatically expires after a set time.
- **Eviction Policies:** When RAM is full, Redis must delete old data. The most common policy is **LRU (Least Recently Used)**, which drops the keys that haven't been accessed in the longest time.

## 5. Common Cache Failures
- **Cache Stampede:** A highly requested key expires, and thousands of concurrent requests instantly hit the database simultaneously, potentially crashing it. *Solution:* Use distributed locks (Redis Redlock) so only one thread rebuilds the cache while others wait.
- **Cache Penetration:** Attackers request keys that don't exist in the DB. The cache misses, the DB is queried, returns nothing, and nothing is cached. The DB gets overwhelmed. *Solution:* Cache null/empty responses with a short TTL, or use Bloom Filters.
- **Cache Avalanche:** A large portion of cached keys expire at the exact same millisecond. *Solution:* Add random jitter to TTLs (e.g., `expire = 60 mins + random(0-5) mins`).

## 6. Persistence (RDB vs AOF)
By default, RAM is wiped on restart. Redis offers persistence:
- **RDB (Redis Database):** Takes point-in-time snapshots to disk every X minutes. Very fast, but you lose data written since the last snapshot if a crash occurs.
- **AOF (Append Only File):** Logs every write operation to disk. Slower, but ensures minimal to zero data loss.

## 7. Interview Questions
### Q. How do you implement a distributed lock in a Spring Boot application using Redis?
**Short Answer:** A distributed lock prevents multiple microservice instances from executing the same critical code simultaneously. In Redis, you use the `SET key value NX PX 10000` command (`NX` means Set if Not eXists, `PX` sets a TTL). If the command succeeds, the lock is acquired. The TTL is critical so the lock is released even if the microservice crashes. Spring integrates this easily via the `Redisson` library.

---

[← Previous: RabbitMQ Fundamentals](../10-rabbitmq/01-rabbitmq-fundamentals.md) | [Next: Docker Fundamentals for Java Backends →](../12-docker/01-docker-fundamentals.md)
