# LLD Case Study: Rate Limiter

Design a concurrent, thread-safe rate limiter (like a API Gateway rate limiter) implementing Token Bucket and Sliding Window Log algorithms.

## Table of Contents
- [Requirements Analysis](#requirements-analysis)
- [Class Design](#class-design)
- [Full Java Implementation](#full-java-implementation)
- [Concurrency & Thread Safety](#concurrency--thread-safety)
- [Design Decisions](#design-decisions)
- [Interview Tips](#interview-tips)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Interview Flow

1. Clarify requirements
2. Identify actors
3. Identify core entities
4. Identify relationships
5. Identify responsibilities
6. Define interfaces
7. Choose abstractions
8. Select patterns only where justified
9. Handle concurrency
10. Discuss extensibility
11. Write code
12. Test
13. Discuss trade-offs

---

## Requirements Analysis

**Functional Requirements:**
1. Limit the number of requests a client can make to an API within a given time window.
2. Support rate limiting based on client identifiers (e.g., Client ID, IP Address, API Key).
3. Support multiple rate-limiting algorithms (Token Bucket, Sliding Window Log).
4. Return true if a request is allowed, false if it is throttled.

**Non-Functional Requirements:**
1. **Thread-safety:** The rate limiter must support high concurrency, as multiple threads will invoke it simultaneously for different or same clients.
2. **Low Latency:** Checking if a request is allowed must be extremely fast (<2ms) to avoid adding latency to API requests.
3. **Memory Efficiency:** Since rate limiters track states for millions of users, memory footprint per user must be minimized.

---

## Class Design

```
RateLimiter (Interface)
└── boolean allowRequest(String clientId)

TokenBucketRateLimiter (Implements RateLimiter)
├── long capacity
├── long refillRatePerSecond
├── Map<String, Bucket> buckets
└── class Bucket:
    ├── double tokens
    ├── long lastRefillTime
    └── synchronized boolean allow()

SlidingWindowRateLimiter (Implements RateLimiter)
├── int maxRequests
├── long windowSizeMs
├── Map<String, Queue<Long>> requestTimestamps
└── synchronized boolean allow()

RateLimiterFactory
└── RateLimiter getRateLimiter(String type, int maxRequests, int windowSec)
```

---

## Full Java Implementation

### 1. Rate Limiter Interface
```java
public interface RateLimiter {
    boolean allowRequest(String clientId);
}
```

### 2. Token Bucket Implementation
The token bucket algorithm has a maximum capacity and refills tokens at a constant rate. When a request comes, we try to consume a token. If a token is available, the request is allowed; otherwise, it is throttled.
Instead of having a background thread refilling tokens (which wastes CPU and doesn't scale), we calculate the refilled tokens lazily on each request.

```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class TokenBucketRateLimiter implements RateLimiter {
    private final long capacity;
    private final long refillRatePerSecond;
    private final Map<String, Bucket> clientBuckets = new ConcurrentHashMap<>();

    public TokenBucketRateLimiter(long capacity, long refillRatePerSecond) {
        this.capacity = capacity;
        this.refillRatePerSecond = refillRatePerSecond;
    }

    @Override
    public boolean allowRequest(String clientId) {
        Bucket bucket = clientBuckets.computeIfAbsent(clientId, k -> new Bucket(capacity, refillRatePerSecond));
        return bucket.allow();
    }

    private static class Bucket {
        private final long capacity;
        private final long refillRate;
        private double tokens;
        private long lastRefillTimestamp;

        public Bucket(long capacity, long refillRate) {
            this.capacity = capacity;
            this.refillRate = refillRate;
            this.tokens = capacity;
            this.lastRefillTimestamp = System.currentTimeMillis();
        }

        public synchronized boolean allow() {
            refill();
            if (tokens >= 1.0) {
                tokens -= 1.0;
                return true;
            }
            return false;
        }

        private void refill() {
            long now = System.currentTimeMillis();
            long elapsed = now - lastRefillTimestamp;
            double tokensToAdd = (elapsed / 1000.0) * refillRate;
            
            if (tokensToAdd > 0) {
                tokens = Math.min(capacity, tokens + tokensToAdd);
                lastRefillTimestamp = now;
            }
        }
    }
}
```

### 3. Sliding Window Log Implementation
Sliding window log keeps track of the timestamps of all requests in a queue (or list) for each client. When a request arrives, we prune all timestamps older than the current window, then check if the size of the queue is less than the maximum allowed requests.

```java
import java.util.Queue;
import java.util.Map;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.ConcurrentHashMap;

public class SlidingWindowRateLimiter implements RateLimiter {
    private final int maxRequests;
    private final long windowSizeMs;
    private final Map<String, Queue<Long>> clientTimestamps = new ConcurrentHashMap<>();

    public SlidingWindowRateLimiter(int maxRequests, int windowSeconds) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSeconds * 1000L;
    }

    @Override
    public boolean allowRequest(String clientId) {
        Queue<Long> timestamps = clientTimestamps.computeIfAbsent(clientId, k -> new ConcurrentLinkedQueue<>());
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;

        synchronized (timestamps) {
            // Remove outdated requests outside the window
            while (!timestamps.isEmpty() && timestamps.peek() < windowStart) {
                timestamps.poll();
            }

            if (timestamps.size() < maxRequests) {
                timestamps.offer(now);
                return true;
            }
            return false;
        }
    }
}
```

---

## Concurrency & Thread Safety

Rate limiting involves shared mutable state (token counts, timestamps) accessed concurrently:

1. **Lazy Refill synchronization:** In `TokenBucketRateLimiter.Bucket`, the methods `allow()` and `refill()` are synchronized on the individual bucket object. This ensures that calculations for a single user's token bucket are serialized, preventing race conditions where two threads read the same token count and decrement it simultaneously.
2. **ConcurrentHashMap:** We map `clientId → Bucket` or `clientId → Queue` using a `ConcurrentHashMap` to allow concurrent access for different clients without blocking.
3. **Deadlock prevention:** Synchronizing on individual buckets means no two locks are acquired simultaneously. Hence, there is zero risk of deadlocks.

---

## Design Decisions

| Decision | Why |
|----------|-----|
| Lazy token refilling | Avoids wasting CPU cycles on background threads refilling tokens for millions of inactive users. |
| Nested `Bucket` class | Keeps the rate limiter state cleanly encapsulated for each client. |
| Synchronizing on bucket/timestamps | Fine-grained lock scope. Multiple clients checking their limits do not block each other. |
| Sliding Window Log over Sliding Window Counter | Sliding Window Log is mathematically precise, although it consumes more memory (storing timestamps vs counters). |

---

## Interview Follow-ups

1. What if traffic increases 100x?
2. What if a client spams the rate limiter itself, causing CPU spikes?
3. How do you implement this in a distributed system across 100 servers?
4. What if the Redis cache goes down?
5. How would you handle variable rate limits based on subscription tiers?
6. What if the time windows need to be perfectly sliding without the memory overhead of logs?
7. What if we need to rate limit by IP and User ID simultaneously?
8. How do you return appropriate HTTP status codes (429 Too Many Requests) with retry-after headers?
9. What if the rate limit rules change dynamically?
10. How do you handle clock synchronization in a distributed rate limiter?

---

## Interview Tips

1. **Discuss algorithms tradeoffs:**
   - **Token Bucket:** Allows bursts of traffic (up to bucket capacity), memory-efficient.
   - **Leaky Bucket:** Smoothens out traffic, enforces a steady rate of processing, but adds latency to bursts.
   - **Fixed Window Counter:** Easy to implement, but prone to a double-rate burst at the boundary of windows.
   - **Sliding Window Log:** Most accurate, but high memory footprint (must store all timestamps).
2. **Explain lazy refilling:** Proactively mention that you do not use a background cron thread to refill tokens, as it doesn't scale to millions of users.
3. **Discuss Distributed Rate Limiting:** In distributed environments, explain that you would use Redis (using Lua scripts or Redis sorted sets) instead of local memory.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Token Bucket | Allows bursts of traffic; refilled at a constant rate |
| 2 | Leaky Bucket | Queues requests and processes them at a steady rate |
| 3 | Fixed Window issue | Traffic spike at window boundaries can double the rate limit |
| 4 | Sliding Window Log | Stores all timestamps; mathematically precise; high memory usage |
| 5 | Refill optimization | Lazy refill on-demand instead of background thread refilling |
| 6 | Thread safety mechanism | `ConcurrentHashMap` + synchronization on client-specific state |
| 7 | Sliding Window memory | O(N) where N is number of requests in current window |
| 8 | Token Bucket memory | O(1) space per user (only stores current tokens & timestamp) |
| 9 | Distributed rate limiting | Redis using Lua scripting or Redisson library |
| 10 | Response headers | `X-Ratelimit-Limit`, `X-Ratelimit-Remaining`, `Retry-After` |

---

*Next: [LLD Case Study — Food Delivery](06-lld-food-delivery.md)*
