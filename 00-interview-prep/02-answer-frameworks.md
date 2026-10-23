# Answer Frameworks for System Design Interviews

This chapter provides structured frameworks for answering different types of interview questions. Having a framework prevents panic and ensures you cover what interviewers evaluate.

## Table of Contents
- [The RADIO Framework (HLD)](#the-radio-framework-hld)
- [The RCPD Framework (LLD)](#the-rcpd-framework-lld)
- [DSA Problem Framework](#dsa-problem-framework)
- [Behavioral + Technical Combos](#behavioral--technical-combos)
- [Communication Templates](#communication-templates)

---

## The RADIO Framework (HLD)

Use this for any High-Level Design question.

```
R — Requirements (functional + non-functional)
A — API Design (key endpoints)
D — Data Model (schema + DB choice)
I — Infrastructure (components + interactions)
O — Optimization (scale + reliability)
```

### Time Allocation (45-minute HLD interview)

| Phase | Time | What to Cover |
|-------|------|---------------|
| R — Requirements | 5–7 min | Functional, non-functional, scale estimates |
| A — API Design | 3–5 min | Core endpoints, request/response |
| D — Data Model | 5–7 min | Schema, DB type, indexing |
| I — Infrastructure | 12–15 min | Draw architecture, explain components |
| O — Optimization | 5–8 min | Caching, sharding, failure modes |

### Requirements Phase Template

**Ask these questions every time:**

```
"Before I start designing, let me clarify a few things:

1. Scale: How many users? Daily active users? Writes vs reads per second?
2. Latency: Any SLA requirements? 99th percentile?
3. Consistency: Can users see slightly stale data, or must it be real-time?
4. Availability: What's the acceptable downtime? 99.9%? 99.99%?
5. Geography: Single region or global?
6. Out of scope: [State what you're not designing to show judgment]"
```

**Non-functional requirements cheat sheet:**

| Requirement | What to ask |
|-------------|-------------|
| Scale | DAU, QPS reads/writes, data size |
| Latency | P99 acceptable latency, real-time or batch |
| Consistency | Strong vs eventual consistency |
| Availability | Nines of availability, acceptable downtime |
| Durability | Data loss tolerance (RPO) |

### Capacity Estimation Template

```
Daily Active Users (DAU): X million
Writes per user per day: Y
Total writes/day: X * Y million
Writes/sec: Total / 86,400

Reads: typically 10x–100x writes
Storage per record: Z bytes
Total storage over 5 years: Writes/day * 365 * 5 * Z bytes
```

---

## The RCPD Framework (LLD)

Use this for any Low-Level Design / Machine Coding question.

```
R — Requirements: Clarify and list all use cases
C — Core Entities: Identify classes (nouns → classes, verbs → methods)
P — Patterns: Which design patterns fit?
D — Design + Code: Class diagram first, then implement
```

### Requirements Phase (LLD)

**Questions to ask:**
```
"Let me clarify the scope:
- What types of [vehicles/books/users] do we need to support?
- What operations must the system support?
- Any concurrency requirements?
- Do we need persistence or in-memory is fine?
- Any payment/billing functionality?"
```

### Entity Identification

**From requirements, highlight:**
- **Nouns** → potential classes/interfaces
- **Verbs** → potential methods
- **Adjectives** → potential attributes or enums

**Example: Parking Lot**
- Nouns: ParkingLot, Level, Spot, Vehicle, Car, Ticket, Payment
- Verbs: park, unpark, calculateFee, processPayment
- Adjectives: available, occupied, small, large

### Pattern Selection Checklist

| Scenario | Pattern to use |
|----------|---------------|
| Object varies by type/subtype | Factory Method |
| Building complex object | Builder |
| Single instance needed | Singleton |
| Adding behavior to object | Decorator |
| Different algorithms for same operation | Strategy |
| State-dependent behavior | State |
| Notify multiple objects on change | Observer |
| Undo/redo needed | Command |
| Access control / lazy loading | Proxy |

### Code Order

1. Enums and constants
2. Interfaces (if any)
3. Base/abstract classes
4. Concrete implementations
5. Aggregating/controller class
6. Main/demo

---

## DSA Problem Framework

Use this for any coding challenge.

```
U — Understand: Restate the problem, ask edge cases
P — Plan: Brute force → optimize → choose approach
C — Code: Write clean, commented code
T — Test: Trace through examples, edge cases
```

### Step 1: Understand

**Always ask:**
- What are the constraints? (n size, value range)
- Can there be duplicates?
- Is the array sorted?
- What to return if no solution? (-1? empty? exception?)
- Integer overflow possible?

### Step 2: Plan

**Brute force first, then optimize:**

```
"The brute force solution would be O(n²) by checking all pairs.
Can we do better? If I sort the array, I can use two pointers for O(n log n).
Or with a hash map, I can reduce it to O(n) with O(n) space."
```

**Common optimization patterns:**

| Problem Type | Technique | Complexity improvement |
|-------------|-----------|----------------------|
| Find pair with target sum | Two pointers / HashMap | O(n²) → O(n) |
| Sliding window max/min | Deque | O(n²) → O(n) |
| Top-K elements | Heap | O(n log n) → O(n log k) |
| Substring problems | Sliding window | O(n²) → O(n) |
| Tree path problems | DFS recursion | O(n) |
| Shortest path | BFS/Dijkstra | varies |

### Step 3: Code

**Template for clean code:**
```java
public ReturnType solveProblem(InputType input) {
    // 1. Edge cases
    if (input == null || input.length == 0) return defaultValue;
    
    // 2. Initialize variables with meaningful names
    int left = 0, right = input.length - 1;
    
    // 3. Main logic with clear comments
    while (left < right) {
        // explain what you're doing
    }
    
    // 4. Return result
    return result;
}
```

### Step 4: Test

**Always test:**
1. The provided example
2. Empty input
3. Single element
4. All same elements
5. Maximum constraint (for overflow)
6. Negative numbers (if applicable)

---

## Behavioral + Technical Combos

HLD interviews often blend technical depth with "how would you make this decision at your company."

### STAR Format for Technical Decisions

```
S — Situation: "We had a service handling 10K req/sec..."
T — Task: "We needed to add rate limiting without downtime..."
A — Action: "I chose token bucket over leaky bucket because... 
             I implemented it using Redis Lua scripts for atomicity..."
R — Result: "Reduced abuse traffic by 60%, p99 latency down by 40ms"
```

### Common Combo Questions

| Question | Technical angle | Behavioral angle |
|----------|----------------|-----------------|
| "How do you choose SQL vs NoSQL?" | CAP, query patterns, consistency needs | "Tell me about a time you made this decision" |
| "How do you handle a production outage?" | Incident response, runbooks | Leadership, communication |
| "How do you ensure code quality?" | Testing strategies, code review | Team culture |

---

## Communication Templates

### Starting your answer

> "Before I dive in, let me make sure I understand the requirements correctly. [Ask 3-4 questions]. Based on that, my approach will be..."

### Stating tradeoffs

> "I'm choosing X over Y here because [reason]. The tradeoff is [downside of X], which is acceptable because [justification]."

### Handling "I don't know"

> "I'm not 100% certain of the exact implementation, but my intuition would be [approach] because [reasoning]. I'd verify by [checking docs / benchmarking / etc.]."

### Wrapping up

> "So to summarize: [restate key components]. The main tradeoffs I made were [1, 2, 3]. If I had more time, I'd also address [scaling concern / edge case]."

---

*Next: [Common Pitfalls](03-common-pitfalls.md)*
