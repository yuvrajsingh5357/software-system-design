# Concurrency for LLD (Java)

In LLD interviews (especially SDE-2), passing a concurrency follow-up question is the difference between a Hire and a Strong Hire. 

Do not just slap `synchronized` on every method. That causes massive performance bottlenecks. Use the right tool for the job.

## 1. Core Concepts to Know

*   **Race Condition:** Two threads read the same state and update it concurrently, overwriting each other.
*   **Atomicity:** An operation that completes entirely or not at all. (e.g., `count++` is NOT atomic; it's read, increment, write).
*   **Visibility:** Changes made by one thread in CPU cache must be flushed to main memory so other threads can see them.
*   **Lock Contention:** Too many threads waiting for the same lock, bringing the system to a halt.

## 2. Java Concurrency Toolbox

### A. The Basics
*   `volatile`: Ensures **visibility**. Does NOT guarantee atomicity. Good for a boolean flag (e.g., `isShutDown`).
*   `synchronized`: Guarantees **visibility and atomicity**. It locks the entire object (or block) using its intrinsic monitor. *Drawback: Slow, blocking, cannot be interrupted.*

### B. Atomic Classes (`java.util.concurrent.atomic`)
*   `AtomicInteger`, `AtomicLong`, `AtomicReference`.
*   Uses **CAS (Compare-And-Swap)** at the CPU level. Lock-free.
*   *When to use:* Simple counters, IDs, or atomic state transitions.
*   *Example:* Rate Limiter token count.

### C. Advanced Locks (`java.util.concurrent.locks`)
*   `ReentrantLock`: Better than `synchronized`. Provides fairness, try-lock (timeout), and interruptibility.
*   `ReentrantReadWriteLock`: Allows multiple concurrent readers, but only one writer. 
*   *When to use:* When reads vastly outnumber writes. (e.g., A shared configuration cache).

### D. Concurrent Collections
*   `ConcurrentHashMap`: Uses fine-grained locking (bucket-level in Java 8+). High throughput.
*   `CopyOnWriteArrayList`: Thread-safe list. Writes copy the entire array.
*   *When to use:* High reads, rare writes (e.g., Observer Pattern subscriber list).

## 3. Applying Concurrency to LLD Systems

### Scenario 1: Movie Ticket Booking
**Problem:** Two users try to book the exact same seat C4 simultaneously.
**Solution:** 
1. **Pessimistic Locking (DB Level):** `SELECT ... FOR UPDATE`.
2. **Optimistic Locking (DB Level):** Use a `version` column.
3. **Application Level (Distributed):** Redis Distributed Lock on the seat ID.
4. **Application Level (Single Node):** Use a `ConcurrentHashMap<String, ReentrantLock>` to lock specific seats, not the whole theater.

### Scenario 2: Rate Limiter (Token Bucket)
**Problem:** Multiple requests try to consume tokens at the same time.
**Solution:** Use `AtomicInteger` for tokens.
```java
public boolean allowRequest() {
    int current = tokens.get();
    while (current > 0) {
        if (tokens.compareAndSet(current, current - 1)) {
            return true;
        }
        current = tokens.get(); // CAS failed, another thread took a token, try again
    }
    return false;
}
```

### Scenario 3: Task Scheduler / Thread Pool
**Problem:** Executing background jobs efficiently without spawning 10,000 threads.
**Solution:** `ExecutorService` (specifically `ScheduledExecutorService`) backed by a `BlockingQueue`.

### Scenario 4: Cache (In-Memory)
**Problem:** Concurrent reads and writes to a Cache.
**Solution:** `ConcurrentHashMap` combined with `ReentrantReadWriteLock` if custom eviction logic is needed.

## 4. Thread-Safe Singleton
In an interview, if you use a Singleton (like an ID Generator), it MUST be thread-safe.

```java
// SDE-2 Must Know: Double-Checked Locking
public class IDGenerator {
    private static volatile IDGenerator instance;
    
    private IDGenerator() {}
    
    public static IDGenerator getInstance() {
        if (instance == null) {
            synchronized (IDGenerator.class) {
                if (instance == null) {
                    instance = new IDGenerator();
                }
            }
        }
        return instance;
    }
}
```
