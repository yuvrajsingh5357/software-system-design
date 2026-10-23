# Common Pitfalls & Anti-Patterns

This chapter covers the most common mistakes candidates make in software system design interviews — and how to avoid them. Learn from others' failures.

## Table of Contents
- [DSA Pitfalls](#dsa-pitfalls)
- [Design Pattern Pitfalls](#design-pattern-pitfalls)
- [LLD Pitfalls](#lld-pitfalls)
- [HLD Pitfalls](#hld-pitfalls)

---

## DSA Pitfalls

### ❌ Pitfall 1: Jumping to code without a plan

**What it looks like:**
Interviewer finishes the problem statement. Candidate immediately starts typing.

**Why it fails:**
- Often solves the wrong problem
- Misses edge cases
- No opportunity to show problem-solving thought process

**Fix:** Spend 2–3 minutes understanding the problem. Ask edge case questions. State your approach before coding.

---

### ❌ Pitfall 2: Integer overflow in binary search

```java
// WRONG — can overflow when left + right > Integer.MAX_VALUE
int mid = (left + right) / 2;

// CORRECT
int mid = left + (right - left) / 2;
```

---

### ❌ Pitfall 3: Modifying a list while iterating

```java
// WRONG — ConcurrentModificationException
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4));
for (Integer item : list) {
    if (item % 2 == 0) list.remove(item); // throws exception
}

// CORRECT — Use Iterator
Iterator<Integer> it = list.iterator();
while (it.hasNext()) {
    if (it.next() % 2 == 0) it.remove();
}
```

---

### ❌ Pitfall 4: Ignoring space complexity

A candidate who optimizes time to O(n log n) while using O(n²) space has not fully solved the problem. Always state both complexities.

**Common hidden space costs:**
- Recursion stack depth (can be O(n) for DFS)
- HashMap of lists → O(n) total but reads poorly
- StringBuilder in loops (if appended in quadratic patterns)

---

### ❌ Pitfall 5: Not handling null/empty inputs

```java
// Every function should start with:
if (nums == null || nums.length == 0) return -1;
if (s == null || s.isEmpty()) return "";
if (root == null) return 0;
```

Interviewers notice when you skip this. It shows production mindset.

---

### ❌ Pitfall 6: Using the wrong data structure

| Scenario | Wrong choice | Right choice |
|----------|-------------|--------------|
| Frequent insertions in the middle | ArrayList | LinkedList |
| Ordered unique elements | HashSet | TreeSet |
| Frequent min/max queries | sorted array | Heap |
| Count frequency of elements | sorting | HashMap |
| LRU cache | just HashMap | LinkedHashMap or HashMap + DoublyLinkedList |

---

## Design Pattern Pitfalls

### ❌ Pitfall 7: Singleton overuse

Singletons create hidden global state and make testing very hard.

```java
// BAD — tight coupling, untestable
class UserService {
    private static UserService instance = new UserService();
    private DatabaseConnection db = DatabaseConnection.getInstance(); // another singleton
    // now you can't mock the DB in tests
}

// BETTER — dependency injection
class UserService {
    private DatabaseConnection db;
    public UserService(DatabaseConnection db) { this.db = db; }
    // Now you can inject a mock DB in tests
}
```

**Rule of thumb:** Prefer dependency injection over Singletons. Use Singleton only for truly shared resources like Logger, Configuration.

---

### ❌ Pitfall 8: Pattern for pattern's sake

Not every problem needs a design pattern. Applying patterns where they add complexity without value is an anti-pattern itself.

```java
// OVER-ENGINEERED for a simple greeting
interface GreetingStrategy { String greet(String name); }
class HelloGreeting implements GreetingStrategy {
    public String greet(String name) { return "Hello, " + name; }
}
class Greeter {
    private GreetingStrategy strategy;
    public Greeter(GreetingStrategy s) { this.strategy = s; }
    public String greet(String name) { return strategy.greet(name); }
}

// SIMPLE AND CORRECT for a fixed greeting
class Greeter {
    public String greet(String name) { return "Hello, " + name; }
}
```

**Rule:** Patterns solve recurring design problems. Don't add them if the problem doesn't exist yet.

---

### ❌ Pitfall 9: Breaking the Open/Closed Principle in Strategy

```java
// BAD — adding a new discount type requires modifying this class (violates OCP)
class DiscountCalculator {
    public double calculate(String type, double price) {
        if (type.equals("SEASONAL")) return price * 0.9;
        else if (type.equals("LOYALTY")) return price * 0.85;
        else if (type.equals("FLASH")) return price * 0.7; // must keep editing here
        return price;
    }
}

// GOOD — Strategy pattern — new discount = new class, no modification
interface DiscountStrategy {
    double apply(double price);
}
class SeasonalDiscount implements DiscountStrategy {
    public double apply(double price) { return price * 0.9; }
}
```

---

## LLD Pitfalls

### ❌ Pitfall 10: Modeling behavior as data instead of polymorphism

```java
// BAD — type string driving logic
class Animal {
    private String type;
    public String speak() {
        if (type.equals("dog")) return "Woof";
        else if (type.equals("cat")) return "Meow";
        // violates OCP, every new animal = edit this method
    }
}

// GOOD — polymorphism
abstract class Animal {
    public abstract String speak();
}
class Dog extends Animal { public String speak() { return "Woof"; } }
class Cat extends Animal { public String speak() { return "Meow"; } }
```

---

### ❌ Pitfall 11: God class / violation of SRP

```java
// BAD — ParkingLot does everything
class ParkingLot {
    public void parkVehicle() { }
    public void calculateFee() { }
    public void processPayment() { }
    public void sendSMSReceipt() { }
    public void updateDatabase() { }
    // grows to 1000 lines, impossible to maintain
}

// GOOD — responsibilities separated
class ParkingLot { /* manages spots */ }
class FeeCalculator { /* calculates fees */ }
class PaymentProcessor { /* handles payment */ }
class NotificationService { /* sends receipts */ }
```

---

### ❌ Pitfall 12: Not using interfaces / programming to implementation

```java
// BAD
ArrayList<String> names = new ArrayList<>(); // tied to ArrayList

// GOOD — code to interface
List<String> names = new ArrayList<>(); // can swap to LinkedList, CopyOnWrite, etc.
```

This matters in LLD interviews — it shows you understand polymorphism and extensibility.

---

### ❌ Pitfall 13: Forgetting thread safety in concurrent LLD

Many LLD problems (ATM, parking lot, booking system) have concurrent users.

```java
// BAD — race condition: two threads may both see spot as available
public boolean park(Vehicle v) {
    if (spot.isAvailable()) {  // Thread A reads: true
        // Thread B reads: true (before A writes)
        spot.park(v);           // Both park → double booking!
        return true;
    }
    return false;
}

// GOOD — synchronized
public synchronized boolean park(Vehicle v) {
    if (spot.isAvailable()) {
        spot.park(v);
        return true;
    }
    return false;
}
```

---

## HLD Pitfalls

### ❌ Pitfall 14: Skipping capacity estimation

Candidates who jump to architecture without numbers miss the point. Scale drives architecture.

**Example:** "We need a database" is weak.
**Better:** "With 100K writes/sec and 500-byte records, we need ~50MB/sec write throughput. A single MySQL instance tops out at ~5K writes/sec, so we need sharding from day one."

---

### ❌ Pitfall 15: Single point of failure

Every system component should be evaluated: "What happens when this fails?"

```
❌ Single load balancer → if it fails, everything fails
✅ Active-passive LB pair (HAProxy) or cloud LB (AWS ALB with multi-AZ)

❌ Single DB instance
✅ Primary + replicas + automated failover

❌ Single cache server
✅ Redis Cluster or Redis Sentinel
```

---

### ❌ Pitfall 16: Ignoring the CAP theorem when choosing a database

```
"We'll use MongoDB for everything" — no justification

Better: "This service needs strong consistency for financial data (inventory),
so we choose PostgreSQL (CP). The user activity feed tolerates eventual
consistency, so we use Cassandra (AP) for its write scalability."
```

---

### ❌ Pitfall 17: Not considering idempotency for APIs

Any write operation that can be retried (payments, order creation) must be idempotent.

```java
// BAD — retry creates duplicate order
POST /orders
{userId: 123, item: "Book"}

// GOOD — idempotency key prevents duplicate
POST /orders
Headers: Idempotency-Key: uuid-abc-123
{userId: 123, item: "Book"}
// Server stores result of first call, returns same result on retry
```

---

### ❌ Pitfall 18: Designing for today's scale, not tomorrow's

**Wrong:** "We have 1000 users, so SQLite is fine."

Even in an interview context, you should note: "For MVP, one Postgres instance is enough. When we hit 10K writes/sec, we'll add read replicas. At 100K, we shard by user_id."

Showing awareness of growth paths is a staff-level differentiator.

---

### ❌ Pitfall 19: Forgetting about monitoring/observability

A system without observability is unrunnable in production. Always mention:
- Metrics (request rate, error rate, latency — RED method)
- Distributed tracing (for microservices — Jaeger, Zipkin)
- Alerting (PagerDuty integration)
- Logging (structured logs, centralized in ELK/CloudWatch)

---

*Next: [Whiteboard Exercises](04-whiteboard-exercises.md)*
