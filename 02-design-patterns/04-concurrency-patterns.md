# Concurrency Patterns

This chapter covers patterns for writing correct, efficient multi-threaded Java programs — essential for LLD interviews and production systems.

## Table of Contents
- [Java Concurrency Fundamentals](#java-concurrency-fundamentals)
- [Thread Pool Pattern](#thread-pool-pattern)
- [Producer-Consumer Pattern](#producer-consumer-pattern)
- [Read-Write Lock Pattern](#read-write-lock-pattern)
- [Future and CompletableFuture](#future-and-completablefuture)
- [Scheduler / Rate Limiter Pattern](#scheduler--rate-limiter-pattern)
- [Common Concurrency Pitfalls](#common-concurrency-pitfalls)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Java Concurrency Fundamentals

### Key Primitives

```java
// synchronized — mutual exclusion on method or block
synchronized void increment() { count++; }
synchronized(this) { count++; }
synchronized(MyClass.class) { /* class-level lock */ }

// volatile — visibility guarantee (no caching in CPU registers)
volatile boolean running = true; // changes visible across threads immediately

// Atomic classes — lock-free thread-safe operations
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();          // atomic increment
count.compareAndSet(expected, new); // CAS — compare-and-swap

// ReentrantLock — more flexible than synchronized
Lock lock = new ReentrantLock();
lock.lock();
try { /* critical section */ }
finally { lock.unlock(); } // ALWAYS unlock in finally!

// tryLock — non-blocking lock attempt
if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
    try { /* critical section */ }
    finally { lock.unlock(); }
} else {
    // could not acquire lock — do something else
}
```

### Memory Model Key Rules

```
happens-before relationships:
1. unlock(m) → subsequent lock(m)
2. volatile write → subsequent volatile read
3. Thread.start() → any action in the started thread
4. Thread.join() → any action after the join call
5. Constructor completion → Object.finalize()
```

---

## Thread Pool Pattern

**Intent:** Maintain a pool of reusable worker threads. Eliminates thread creation overhead and controls resource usage.

```java
// Java's built-in thread pool — Executor framework
ExecutorService pool = Executors.newFixedThreadPool(10); // 10 worker threads

// Submit tasks
Future<Integer> future = pool.submit(() -> {
    return heavyComputation(); // runs in a worker thread
});

// Execute (fire and forget)
pool.execute(() -> System.out.println("Task running in pool"));

// Shutdown gracefully
pool.shutdown(); // stop accepting new tasks
pool.awaitTermination(60, TimeUnit.SECONDS); // wait for completion

// Different pool types
Executors.newCachedThreadPool();         // unlimited threads, recycle idle
Executors.newSingleThreadExecutor();    // 1 thread, preserves task order
Executors.newScheduledThreadPool(5);   // for delayed/periodic tasks
```

### Custom ThreadPoolExecutor

```java
// Most configurable — use when you need custom rejection policy or queue type
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                                   // corePoolSize: always alive
    20,                                  // maximumPoolSize: max in peak
    60L, TimeUnit.SECONDS,              // keepAliveTime: idle thread cleanup
    new ArrayBlockingQueue<>(100),       // work queue (bounded is preferred!)
    new ThreadFactory() {
        private final AtomicInteger count = new AtomicInteger();
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setName("worker-" + count.incrementAndGet());
            t.setDaemon(true); // daemon threads die with JVM
            return t;
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection: caller thread runs it
);
// Rejection policies:
// AbortPolicy (default): throw RejectedExecutionException
// CallerRunsPolicy: caller thread executes — creates backpressure
// DiscardPolicy: silently discard
// DiscardOldestPolicy: discard oldest waiting task, retry submit
```

### Object Pool Pattern

```java
// Pool of expensive objects (connections, parsers)
class ConnectionPool {
    private final BlockingQueue<Connection> pool;
    private final int maxSize;
    
    public ConnectionPool(int maxSize) {
        this.maxSize = maxSize;
        this.pool = new LinkedBlockingQueue<>(maxSize);
        for (int i = 0; i < maxSize; i++) {
            pool.offer(createConnection());
        }
    }
    
    public Connection borrow() throws InterruptedException {
        return pool.poll(5, TimeUnit.SECONDS); // wait up to 5 seconds
    }
    
    public void returnConnection(Connection conn) {
        if (!pool.offer(conn)) {
            conn.close(); // pool full — discard
        }
    }
    
    private Connection createConnection() { return new Connection(); }
}

// Usage pattern — always return in finally
Connection conn = pool.borrow();
try {
    conn.query("SELECT ...");
} finally {
    pool.returnConnection(conn);
}
```

---

## Producer-Consumer Pattern

**Intent:** Decouple producers (generate work) from consumers (process work) via a shared buffer.

```java
// Using BlockingQueue — the idiomatic Java way
class ProducerConsumerDemo {
    private static final BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);
    private static volatile boolean running = true;
    
    // Producer
    static class Producer implements Runnable {
        private final int id;
        Producer(int id) { this.id = id; }
        
        @Override
        public void run() {
            try {
                while (running) {
                    Task task = new Task("Task-" + id + "-" + System.currentTimeMillis());
                    queue.put(task); // BLOCKS if queue is full
                    System.out.println("Produced: " + task.getName());
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    // Consumer
    static class Consumer implements Runnable {
        private final int id;
        Consumer(int id) { this.id = id; }
        
        @Override
        public void run() {
            try {
                while (running || !queue.isEmpty()) {
                    Task task = queue.poll(1, TimeUnit.SECONDS); // BLOCKS up to 1 sec
                    if (task != null) {
                        System.out.println("Consumer-" + id + " processing: " + task.getName());
                        processTask(task);
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        private void processTask(Task task) throws InterruptedException {
            Thread.sleep(200); // simulate work
        }
    }
    
    public static void main(String[] args) throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(5);
        pool.execute(new Producer(1));
        pool.execute(new Producer(2));
        pool.execute(new Consumer(1));
        pool.execute(new Consumer(2));
        pool.execute(new Consumer(3));
        
        Thread.sleep(5000); // run for 5 seconds
        running = false;
        pool.shutdown();
    }
}
```

### Manual Implementation (with Lock + Condition)

```java
class BoundedBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    
    public BoundedBlockingQueue(int capacity) { this.capacity = capacity; }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) notFull.await(); // block if full
            queue.offer(item);
            notEmpty.signal(); // wake up one consumer
        } finally { lock.unlock(); }
    }
    
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) notEmpty.await(); // block if empty
            T item = queue.poll();
            notFull.signal(); // wake up one producer
            return item;
        } finally { lock.unlock(); }
    }
}
```

---

## Read-Write Lock Pattern

**Intent:** Allow multiple readers OR one writer at a time. Improves throughput when reads dominate.

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class ThreadSafeCache<K, V> {
    private final Map<K, V> cache = new HashMap<>();
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    
    // Multiple readers can execute simultaneously
    public V get(K key) {
        readLock.lock();
        try {
            return cache.get(key);
        } finally {
            readLock.unlock();
        }
    }
    
    // Only one writer at a time; blocks all readers while writing
    public void put(K key, V value) {
        writeLock.lock();
        try {
            cache.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
    
    // Upgrade from read to write lock requires releasing read lock first
    public V computeIfAbsent(K key, Function<K, V> compute) {
        readLock.lock();
        try {
            V val = cache.get(key);
            if (val != null) return val; // fast path — read only
        } finally {
            readLock.unlock();
        }
        
        writeLock.lock(); // must re-check after acquiring write lock
        try {
            return cache.computeIfAbsent(key, compute); // double-check in cache
        } finally {
            writeLock.unlock();
        }
    }
}
```

---

## Future and CompletableFuture

```java
// CompletableFuture — non-blocking async composition
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchUserFromDB(userId))    // runs in ForkJoinPool
    .thenApply(user -> enrichWithProfile(user))    // transform result
    .thenCompose(user -> fetchUserOrders(user))    // chain another async op
    .exceptionally(ex -> {                          // error handling
        log.error("Failed", ex);
        return defaultUser();
    });

// Combine multiple futures
CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> fetchUser(id));
CompletableFuture<List<Order>> ordersFuture = CompletableFuture.supplyAsync(() -> fetchOrders(id));

CompletableFuture.allOf(userFuture, ordersFuture) // wait for all
    .thenRun(() -> {
        String user = userFuture.join();
        List<Order> orders = ordersFuture.join();
        display(user, orders);
    });

// Timeout handling
CompletableFuture<String> withTimeout = future
    .completeOnTimeout("default", 3, TimeUnit.SECONDS) // Java 9+
    .orTimeout(5, TimeUnit.SECONDS); // throws TimeoutException if exceeded
```

---

## Scheduler / Rate Limiter Pattern

### Token Bucket (Rate Limiter)

```java
class TokenBucketRateLimiter {
    private final int maxTokens;
    private final double refillRate; // tokens per second
    private double tokens;
    private long lastRefillTime;
    
    public TokenBucketRateLimiter(int maxTokens, double refillRate) {
        this.maxTokens = maxTokens;
        this.refillRate = refillRate;
        this.tokens = maxTokens;
        this.lastRefillTime = System.nanoTime();
    }
    
    public synchronized boolean tryConsume(int tokensNeeded) {
        refill();
        if (tokens >= tokensNeeded) {
            tokens -= tokensNeeded;
            return true; // allow request
        }
        return false; // reject request
    }
    
    private void refill() {
        long now = System.nanoTime();
        double elapsed = (now - lastRefillTime) / 1_000_000_000.0; // seconds
        tokens = Math.min(maxTokens, tokens + elapsed * refillRate);
        lastRefillTime = now;
    }
}

// Usage
TokenBucketRateLimiter limiter = new TokenBucketRateLimiter(100, 10.0); // 100 tokens, refill 10/sec

if (limiter.tryConsume(1)) {
    processRequest(); // allowed
} else {
    throw new RateLimitExceededException("429 Too Many Requests");
}
```

---

## Common Concurrency Pitfalls

### Deadlock

```java
// DEADLOCK — thread A holds lock1, waits for lock2; thread B holds lock2, waits for lock1
Object lock1 = new Object(), lock2 = new Object();

Thread a = new Thread(() -> {
    synchronized(lock1) {
        synchronized(lock2) { /* work */ }
    }
});
Thread b = new Thread(() -> {
    synchronized(lock2) { // swapped order!
        synchronized(lock1) { /* work */ }
    }
});

// Prevention: always acquire locks in the same global order
// Thread B should also acquire lock1 before lock2
```

### Race Condition

```java
// RACE CONDITION — non-atomic check-then-act
class BankAccount {
    private double balance = 1000.0;
    
    // BROKEN — two threads can both see balance > amount and both withdraw
    public void withdraw(double amount) {
        if (balance >= amount) {      // check
            balance -= amount;        // act — not atomic!
        }
    }
    
    // FIXED — synchronized makes check+act atomic
    public synchronized void withdraw(double amount) {
        if (balance >= amount) balance -= amount;
    }
    
    // OR use AtomicReference/compare-and-set for lock-free:
    // Requires careful design
}
```

---

## Interview Questions

### Q: What is the difference between wait() and sleep()?

| | `wait()` | `sleep()` |
|-|----------|-----------|
| Belongs to | Object | Thread |
| Releases lock? | **Yes** | No |
| Must be in synchronized | Yes | No |
| Woken by | `notify()` / `notifyAll()` | Timer expiry |
| Use case | Producer-Consumer waiting | Pause for fixed time |

### Q: Why use `while` not `if` for condition check in wait?

```java
// WRONG — spurious wakeups can happen
synchronized(lock) {
    if (queue.isEmpty()) lock.wait(); // if spurious wakeup → proceeds with empty queue!
    process(queue.poll());
}

// CORRECT — re-check condition after wakeup
synchronized(lock) {
    while (queue.isEmpty()) lock.wait(); // loop: check → wait → check again
    process(queue.poll());
}
```

JVM can wake up threads spuriously (without notify). The `while` loop guards against this.

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | volatile | Visibility guarantee; no caching; NOT atomicity |
| 2 | synchronized | Mutual exclusion + visibility; one thread at a time |
| 3 | AtomicInteger | Lock-free atomic ops via CAS; prefer over synchronized for counters |
| 4 | ReentrantLock | Like synchronized but tryLock, timed lock, fairness |
| 5 | ThreadPool types | Fixed, Cached, Single, Scheduled |
| 6 | ThreadPoolExecutor | core/max pool size, queue, keepAlive, rejection policy |
| 7 | BlockingQueue | put() blocks if full; take() blocks if empty |
| 8 | Producer-Consumer | BlockingQueue decouples producers from consumers |
| 9 | ReadWriteLock | Many readers OR one writer; great for read-heavy caches |
| 10 | wait() | Releases lock; must be in synchronized; use while loop |
| 11 | sleep() | Does NOT release lock; Thread method |
| 12 | CompletableFuture | Async composition: supplyAsync, thenApply, thenCompose, allOf |
| 13 | Deadlock prevention | Always acquire locks in same global order |
| 14 | Token bucket | maxTokens + refillRate; tryConsume returns allow/deny |
| 15 | Spurious wakeup | JVM can wake without notify(); always use while not if |

---

*Next: [Patterns in Practice](05-patterns-in-practice.md)*
