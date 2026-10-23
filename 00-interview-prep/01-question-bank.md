# Software System Design Interview Question Bank

This chapter provides a comprehensive collection of interview questions for DSA, Design Patterns, LLD, and HLD — organized by topic. Each question includes expected depth, strong answer structure, and common follow-ups.

## Table of Contents

- [DSA Questions](#dsa-questions)
- [Design Pattern Questions](#design-pattern-questions)
- [LLD Questions](#lld-questions)
- [HLD Questions](#hld-questions)
- [Advanced HLD Scenarios](#advanced-hld-scenarios)

---

## DSA Questions

### Q1: What is the time complexity of HashMap operations? When can it degrade?

**What interviewers look for:**
- Understanding of hashing mechanics
- Awareness of worst-case scenarios
- Knowledge of how Java's HashMap handles collisions

**Strong answer:**

HashMap provides **O(1) average** for get, put, and remove. But this relies on a good hash function distributing keys evenly.

**Degradation to O(n):** If all keys hash to the same bucket, you get a linked list of length n. Every lookup traverses the entire list.

**Java 8 improvement:** When a bucket's linked list exceeds 8 elements (and the table has ≥ 64 buckets), Java converts it to a Red-Black Tree → O(log n) worst case instead of O(n).

```java
// Worst-case scenario (all same hashCode)
Map<Integer, String> map = new HashMap<>();
// If all keys return hashCode() = 42, they pile into one bucket
// Java 8 treeifies after 8 collisions → O(log n) per op
```

**Key numbers to know:**
- Default initial capacity: 16
- Load factor: 0.75 (resize when 75% full)
- Resize: doubles capacity, rehashes all entries — O(n) amortized

---

### Q2: Explain how you would find the shortest path in a graph. Which algorithm and when?

**What interviewers look for:**
- Knowledge of BFS, Dijkstra, Bellman-Ford, A*
- Clear decision framework based on graph properties

**Strong answer:**

| Condition | Algorithm | Complexity |
|-----------|-----------|------------|
| Unweighted graph | BFS | O(V + E) |
| Non-negative weights | Dijkstra | O((V+E) log V) with min-heap |
| Negative weights (no neg cycles) | Bellman-Ford | O(VE) |
| Detect negative cycles | Bellman-Ford | O(VE) |
| All-pairs shortest path | Floyd-Warshall | O(V³) |
| Heuristic (grid pathfinding) | A* | O(E log V) typical |

```java
// BFS for unweighted shortest path
public int bfsShortestPath(int[][] graph, int src, int dest) {
    Queue<Integer> queue = new LinkedList<>();
    boolean[] visited = new boolean[graph.length];
    int[] distance = new int[graph.length];
    
    queue.offer(src);
    visited[src] = true;
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        if (node == dest) return distance[node];
        
        for (int neighbor : graph[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                distance[neighbor] = distance[node] + 1;
                queue.offer(neighbor);
            }
        }
    }
    return -1; // unreachable
}
```

**Follow-up:** "How would you handle negative weight edges with Dijkstra?" → You can't. Dijkstra assumes relaxation is permanent once a node is settled; negative edges can invalidate that. Use Bellman-Ford instead.

---

### Q3: What is dynamic programming? How do you recognize a DP problem?

**What interviewers look for:**
- Clear definition of overlapping subproblems + optimal substructure
- Pattern recognition ability
- Top-down vs bottom-up understanding

**Strong answer:**

**DP is applicable when:**
1. **Optimal substructure**: Optimal solution to problem contains optimal solutions to subproblems
2. **Overlapping subproblems**: Same subproblems solved repeatedly

**Recognition signals:**
- "Find minimum/maximum..."
- "Count the number of ways..."
- "Is it possible to..."
- Involves sequences, grids, or subsets

**Top-down (memoization) vs Bottom-up (tabulation):**

```java
// Coin Change — Top-Down (Memoization)
Map<Integer, Integer> memo = new HashMap<>();

public int coinChangeTopDown(int[] coins, int amount) {
    if (amount == 0) return 0;
    if (amount < 0) return -1;
    if (memo.containsKey(amount)) return memo.get(amount);
    
    int min = Integer.MAX_VALUE;
    for (int coin : coins) {
        int result = coinChangeTopDown(coins, amount - coin);
        if (result >= 0) min = Math.min(min, result + 1);
    }
    
    memo.put(amount, min == Integer.MAX_VALUE ? -1 : min);
    return memo.get(amount);
}

// Coin Change — Bottom-Up (Tabulation)
public int coinChangeBottomUp(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1); // Initialize to "impossible"
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

**When to prefer each:**
- **Top-down**: Natural recursion, only some subproblems needed
- **Bottom-up**: All subproblems needed, better constant factor (no stack frames)

---

### Q4: Explain binary search and its variants.

**What interviewers look for:**
- Clean implementation without off-by-one errors
- Knowledge of variants (leftmost, rightmost, on answer)
- Application to non-obvious problems

**Strong answer:**

```java
// Standard binary search
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2; // Avoid integer overflow
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// Find leftmost insertion position
public int lowerBound(int[] nums, int target) {
    int left = 0, right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) left = mid + 1;
        else right = mid;
    }
    return left; // First position where nums[pos] >= target
}

// Binary search on answer: Minimum capacity to ship packages in D days
public int shipWithinDays(int[] weights, int days) {
    int left = Arrays.stream(weights).max().getAsInt(); // min possible capacity
    int right = Arrays.stream(weights).sum();           // max possible capacity
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canShip(weights, days, mid)) right = mid;
        else left = mid + 1;
    }
    return left;
}

private boolean canShip(int[] weights, int days, int capacity) {
    int daysNeeded = 1, currentLoad = 0;
    for (int w : weights) {
        if (currentLoad + w > capacity) {
            daysNeeded++;
            currentLoad = 0;
        }
        currentLoad += w;
    }
    return daysNeeded <= days;
}
```

**Key insight:** `mid = left + (right - left) / 2` prevents overflow when left + right exceeds Integer.MAX_VALUE.

---

### Q5: How does a heap work? Implement a priority queue from scratch.

**What interviewers look for:**
- Understanding of heap property and complete binary tree
- Sift-up and sift-down operations
- Relationship between index and parent/children

**Strong answer:**

```java
public class MinHeap {
    private int[] heap;
    private int size;
    
    public MinHeap(int capacity) {
        heap = new int[capacity];
        size = 0;
    }
    
    // Parent and child index formulas
    private int parent(int i) { return (i - 1) / 2; }
    private int leftChild(int i) { return 2 * i + 1; }
    private int rightChild(int i) { return 2 * i + 2; }
    
    public void insert(int val) {
        heap[size] = val;
        siftUp(size);
        size++;
    }
    
    public int extractMin() {
        int min = heap[0];
        heap[0] = heap[--size];
        siftDown(0);
        return min;
    }
    
    private void siftUp(int i) {
        while (i > 0 && heap[parent(i)] > heap[i]) {
            swap(i, parent(i));
            i = parent(i);
        }
    }
    
    private void siftDown(int i) {
        int smallest = i;
        int left = leftChild(i), right = rightChild(i);
        
        if (left < size && heap[left] < heap[smallest]) smallest = left;
        if (right < size && heap[right] < heap[smallest]) smallest = right;
        
        if (smallest != i) {
            swap(i, smallest);
            siftDown(smallest);
        }
    }
    
    private void swap(int i, int j) {
        int tmp = heap[i]; heap[i] = heap[j]; heap[j] = tmp;
    }
}
```

**Complexity:** Insert O(log n), Extract O(log n), Peek O(1), Build heap O(n) (not O(n log n) — important interview fact!).

---

## Design Pattern Questions

### Q6: When would you use Strategy vs Template Method?

**What interviewers look for:**
- Understanding the difference in composition vs inheritance
- Real-world use cases for each

**Strong answer:**

Both handle algorithm variation, but they differ in mechanism:

| | Strategy | Template Method |
|-|----------|-----------------|
| Mechanism | Composition (inject algorithm) | Inheritance (override steps) |
| Change at | Runtime | Compile time |
| Relationship | HAS-A (holds a strategy) | IS-A (extends base class) |
| Flexibility | Higher — swap strategies | Lower — fixed skeleton |

```java
// Strategy: Sort algorithm swappable at runtime
interface SortStrategy {
    void sort(int[] data);
}

class QuickSort implements SortStrategy {
    public void sort(int[] data) { /* quicksort impl */ }
}

class Sorter {
    private SortStrategy strategy;
    public Sorter(SortStrategy strategy) { this.strategy = strategy; }
    public void sort(int[] data) { strategy.sort(data); }
}

// Template Method: Fixed report generation skeleton
abstract class ReportGenerator {
    // Template method — defines the skeleton
    public final void generateReport() {
        fetchData();
        processData();
        formatOutput(); // hook — subclasses override this
    }
    protected abstract void fetchData();
    protected abstract void processData();
    protected void formatOutput() { /* default PDF output */ }
}
```

**Use Strategy when:** You need to swap algorithms dynamically, or composition is preferred over inheritance.
**Use Template Method when:** The overall algorithm skeleton is fixed, only specific steps vary.

---

### Q7: Explain the difference between Proxy, Decorator, and Adapter.

**What interviewers look for:**
- Ability to distinguish structurally similar patterns
- Intent-based differentiation

**Strong answer:**

All three wrap an object, but for different purposes:

| Pattern | Intent | Adds behavior? | Changes interface? |
|---------|--------|---------------|--------------------|
| **Proxy** | Control access | No (same interface) | No |
| **Decorator** | Add features | Yes | No |
| **Adapter** | Translate interface | No | Yes |

```java
// Proxy — Virtual proxy for lazy loading
interface Image { void display(); }

class RealImage implements Image {
    public RealImage(String filename) { loadFromDisk(filename); } // expensive
    public void display() { System.out.println("Displaying"); }
}

class ProxyImage implements Image {
    private RealImage real;
    private String filename;
    public ProxyImage(String filename) { this.filename = filename; }
    public void display() {
        if (real == null) real = new RealImage(filename); // lazy load
        real.display();
    }
}

// Decorator — Add logging around service
class LoggingService implements Service {
    private Service delegate;
    public LoggingService(Service delegate) { this.delegate = delegate; }
    public void execute() {
        log("Before");
        delegate.execute();
        log("After");
    }
}

// Adapter — Adapt old interface to new expected interface
class OldPrinter { public void printText(String text) { } }
interface NewPrinter { void print(String text); }

class PrinterAdapter implements NewPrinter {
    private OldPrinter old;
    public PrinterAdapter(OldPrinter old) { this.old = old; }
    public void print(String text) { old.printText(text); } // translate
}
```

---

## LLD Questions

### Q8: Design a Parking Lot. Walk me through your classes.

**What interviewers look for:**
- Object modeling from requirements
- Use of design patterns naturally
- Extensibility and SOLID compliance

**Strong answer structure:**

1. **Clarify requirements**: Multi-level? Vehicle types? Payment? Entry/exit?
2. **Identify entities**: ParkingLot, Level, ParkingSpot, Vehicle, Ticket
3. **Identify patterns**: Factory (vehicles), Strategy (pricing), Observer (spot availability), State (spot state)

```java
// Key classes
enum VehicleType { MOTORCYCLE, CAR, TRUCK }
enum SpotStatus { AVAILABLE, OCCUPIED }

abstract class Vehicle {
    protected String licensePlate;
    protected VehicleType type;
    // getters
}

class Car extends Vehicle {
    public Car(String plate) {
        this.licensePlate = plate;
        this.type = VehicleType.CAR;
    }
}

class ParkingSpot {
    private int spotId;
    private VehicleType allowedType;
    private SpotStatus status = SpotStatus.AVAILABLE;
    private Vehicle currentVehicle;
    
    public boolean park(Vehicle vehicle) {
        if (status == SpotStatus.OCCUPIED) return false;
        if (!allowedType.equals(vehicle.getType())) return false;
        currentVehicle = vehicle;
        status = SpotStatus.OCCUPIED;
        return true;
    }
    
    public void free() {
        currentVehicle = null;
        status = SpotStatus.AVAILABLE;
    }
}
```

*Full implementation in [03-lld/04-case-parking-lot.md](../03-lld/04-case-parking-lot.md)*

---

### Q9: How do you approach any LLD problem in an interview?

**What interviewers look for:**
- Structured, calm approach under pressure
- Requirements first, code second
- Pattern awareness

**Strong answer — The RCPD Framework:**

```
R — Requirements: Ask clarifying questions first
C — Core Entities: Identify nouns → classes
P — Patterns: Which GoF patterns apply?
D — Design: Draw class diagram, then code
```

**Time allocation (45-min interview):**
- 5 min: Requirements
- 5 min: Entity identification + relationships
- 5 min: Pattern selection
- 25 min: Core class implementations
- 5 min: Edge cases + enhancements

---

## HLD Questions

### Q10: How does a URL shortener work at scale?

**What interviewers look for:**
- Base62 encoding understanding
- Database choice justification
- Scalability and caching

**Strong answer:**

```
User → API Gateway → App Servers → DB (KV store)
                                 ↘ Cache (Redis)
User redirect ← 301/302 ←────────┘
```

**Key decisions:**

1. **ID generation**: Use a counter (single point of failure) or distributed ID (Snowflake)
2. **Encoding**: `base62(id)` → 7 chars supports 62^7 = 3.5 trillion URLs
3. **DB**: Key-value store (DynamoDB, Cassandra) for short→long mapping
4. **Cache**: Redis for hot URLs (top 20% URLs = 80% traffic)
5. **Redirect type**: 301 (permanent, browser caches) vs 302 (temporary, you track clicks)

**Capacity estimate (1B URLs, 100M redirects/day):**
- Write: 100M/86400 ≈ 1,160 writes/sec
- Read: ~12,000 reads/sec
- Storage: 1B × 500 bytes = 500 GB

*Full case study in [04-hld/11-case-studies/01-design-url-shortener.md](../04-hld/11-case-studies/01-design-url-shortener.md)*

---

### Q11: Explain CAP theorem and how you use it in practice.

**What interviewers look for:**
- Clear explanation without confusion
- Practical application to technology choices
- Understanding of "partition tolerance is mandatory"

**Strong answer:**

CAP: In a distributed system, **you cannot simultaneously guarantee all three of**:
- **C**onsistency — Every read sees the most recent write
- **A**vailability — Every request gets a response (no error)
- **P**artition Tolerance — System works despite network failures

**The practical reality:** Network partitions happen. P is non-negotiable for distributed systems. So the real choice is **CP vs AP**.

| Choose CP | Choose AP |
|-----------|-----------|
| Banking, financial | Social media, shopping carts |
| Inventory (avoid overselling) | DNS, search results |
| Medical records | Product catalog |

Examples:
- **CP**: HBase, ZooKeeper, etcd
- **AP**: Cassandra, CouchDB, DynamoDB (default)
- **CA**: Only possible without partitions (single machine)

---

### Q12: How do you design a news feed (like Twitter/Facebook)?

**What interviewers look for:**
- Fan-out on write vs fan-out on read tradeoffs
- Handling celebrity problem
- Cache strategy

**Strong answer:**

**Two approaches:**

**Fan-out on write (push model):**
- When user posts, push to all followers' feed tables
- ✅ Fast read (pre-computed feed)
- ❌ Slow write for celebrities (10M followers = 10M writes)

**Fan-out on read (pull model):**
- When user opens feed, fetch from all followees
- ✅ Fast write
- ❌ Slow read for users following many people

**Hybrid (Twitter's approach):**
- Regular users: fan-out on write → Redis sorted set
- Celebrities (>1M followers): fan-out on read at query time
- Merge celebrity posts with pre-computed feed at read time

*Full case study in [04-hld/11-case-studies/10-design-news-feed.md](../04-hld/11-case-studies/10-design-news-feed.md)*

---

## Advanced HLD Scenarios

### Q13: Your service handles 10K req/sec normally. Traffic spikes to 100K. What breaks and how do you fix it?

**Strong answer — cascading failure analysis:**

```
Traffic spike hits
    ↓
App servers CPU pegged at 100%
    ↓
Response times increase → timeouts
    ↓
Clients retry → more traffic (retry storm)
    ↓
Database connection pool exhausted
    ↓
DB queries pile up → DB goes down
    ↓
Everything fails
```

**Mitigations in order:**
1. **Rate limiting** at API gateway (token bucket) — shed excess load
2. **Auto-scaling** — spin up app servers (30-60 sec delay)
3. **Circuit breaker** — fail fast instead of queueing
4. **Queue-based load leveling** — put writes in a queue, process at fixed rate
5. **Cache** — serve reads from Redis, reduce DB load
6. **Read replicas** — distribute read traffic

---

### Q14: How would you design a distributed rate limiter?

**What interviewers look for:**
- Algorithm choice (token bucket vs leaky bucket vs sliding window)
- Distributed coordination (Redis)
- Edge cases: multi-datacenter, clock drift

**Strong answer:**

**Token Bucket Algorithm** (most common):
```
Each user has a bucket with max K tokens
Tokens refill at rate R per second
Each request consumes 1 token
If no tokens → reject (429 Too Many Requests)
```

**Redis implementation:**
```java
// Using Redis Lua script for atomicity
String luaScript = """
    local tokens = redis.call('get', KEYS[1])
    if tokens == false then
        tokens = tonumber(ARGV[1])
    else
        tokens = tonumber(tokens)
    end
    if tokens < 1 then
        return 0 -- reject
    end
    redis.call('set', KEYS[1], tokens - 1)
    redis.call('expire', KEYS[1], tonumber(ARGV[2]))
    return 1 -- allow
    """;
```

**Multi-datacenter issue:** Synchronizing across DCs adds latency. Solution: Allow local rate limits per DC, accept slightly higher burst globally.

*Full case study in [04-hld/11-case-studies/08-design-rate-limiter.md](../04-hld/11-case-studies/08-design-rate-limiter.md)*
