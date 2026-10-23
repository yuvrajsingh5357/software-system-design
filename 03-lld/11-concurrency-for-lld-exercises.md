# Concurrency Exercises for LLD

## Exercise 1: Thread-Safe Seat Booking
**Problem:** Multiple users trying to book the same seat.
**Naive implementation:** `if (seat.isAvailable()) { seat.setBooked(true); }`
**Race condition:** Two threads check `isAvailable()` at the exact same time, both get `true`, both set it to `true`.
**Correct implementation (Application layer):** Use `ConcurrentHashMap<String, ReentrantLock>` to lock only the specific seat ID before checking availability.
**Trade-offs:** Granular locking is fast but complex. DB-level pessimistic locking (`SELECT FOR UPDATE`) is safer for single-source-of-truth.

## Exercise 2: Concurrent Rate Limiter (Token Bucket)
**Problem:** Threads consuming tokens.
**Naive implementation:** `synchronized boolean allowRequest()`
**Correct implementation:** `AtomicInteger tokens;` and `compareAndSet`.
**Why it works:** Lock-free, uses hardware CAS for high throughput.

## Exercise 3: Thread-Safe LRU Cache
**Problem:** Read/Write heavily concurrent cache.
**Correct implementation:** `ConcurrentHashMap` combined with `ReadWriteLock` for eviction, OR use a segmented approach like Guava Cache.

## Exercise 4: Task Scheduler
**Problem:** Executing millions of delayed tasks.
**Correct implementation:** `ScheduledThreadPoolExecutor`. 
**Why it works:** Thread pool backed by a `DelayQueue`. Avoids creating 1M threads.

## Exercise 5: Producer-Consumer
**Problem:** Async notification delivery.
**Correct implementation:** `BlockingQueue` (e.g., `ArrayBlockingQueue`). Producers `put()`, consumers `take()`.
