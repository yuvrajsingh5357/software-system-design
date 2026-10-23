# Whiteboard Exercises

Practice problems with full walkthroughs. Attempt each problem before reading the solution.

## Table of Contents
- [DSA Exercises](#dsa-exercises)
- [LLD Exercises](#lld-exercises)
- [HLD Exercises](#hld-exercises)

---

## DSA Exercises

### Exercise 1: LRU Cache

**Problem:** Design a data structure for LRU (Least Recently Used) cache with O(1) get and put.

**Attempt it first. Solution below.**

---

**Solution:**

Use a **doubly linked list + HashMap**:
- LinkedList maintains usage order (head = most recent, tail = least recent)
- HashMap stores key → node for O(1) access

```java
public class LRUCache {
    private final int capacity;
    private final Map<Integer, Node> map;
    private final Node head, tail; // dummy sentinels
    
    class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        remove(node);
        insertAtFront(node); // mark as recently used
        return node.val;
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) remove(map.get(key));
        if (map.size() == capacity) {
            Node lru = tail.prev; // least recently used
            remove(lru);
            map.remove(lru.key);
        }
        Node node = new Node(key, value);
        insertAtFront(node);
        map.put(key, node);
    }
    
    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private void insertAtFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

**Complexity:** O(1) for both get and put.

**Alternative:** Java's `LinkedHashMap` with `accessOrder=true` can implement LRU in ~10 lines.

---

### Exercise 2: Merge K Sorted Lists

**Problem:** Merge K sorted linked lists into one sorted list.

**Solution:**

Use a **min-heap** of size K:

```java
public ListNode mergeKLists(ListNode[] lists) {
    // Min-heap ordered by node value
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    
    // Add the head of each list to the heap
    for (ListNode node : lists) {
        if (node != null) pq.offer(node);
    }
    
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        curr.next = min;
        curr = curr.next;
        if (min.next != null) pq.offer(min.next);
    }
    
    return dummy.next;
}
```

**Complexity:** O(N log K) where N = total nodes, K = number of lists.

**Why log K?** Heap has K elements, each push/pop is O(log K).

---

### Exercise 3: Word Ladder (BFS)

**Problem:** Given beginWord, endWord, and a word list, find the shortest transformation sequence length. Each step changes exactly one letter and must be a valid word.

**Solution:**

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(endWord)) return 0;
    
    Queue<String> queue = new LinkedList<>();
    queue.offer(beginWord);
    Set<String> visited = new HashSet<>();
    visited.add(beginWord);
    int steps = 1;
    
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            String word = queue.poll();
            char[] chars = word.toCharArray();
            
            for (int j = 0; j < chars.length; j++) {
                char original = chars[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    if (c == original) continue;
                    chars[j] = c;
                    String newWord = new String(chars);
                    if (newWord.equals(endWord)) return steps + 1;
                    if (wordSet.contains(newWord) && !visited.contains(newWord)) {
                        visited.add(newWord);
                        queue.offer(newWord);
                    }
                    chars[j] = original; // restore
                }
            }
        }
        steps++;
    }
    return 0;
}
```

**Key insight:** BFS guarantees shortest path in unweighted graphs. Each level = one transformation.

---

## LLD Exercises

### Exercise 4: Design a Stack with getMin() in O(1)

**Problem:** Design a stack that supports push, pop, top, and getMin — all in O(1).

**Solution:** Use an auxiliary stack that tracks the minimum.

```java
public class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();
    
    public void push(int val) {
        stack.push(val);
        // Push to minStack only if it's the new minimum
        int currentMin = minStack.isEmpty() ? val : Math.min(val, minStack.peek());
        minStack.push(currentMin);
    }
    
    public void pop() {
        stack.pop();
        minStack.pop();
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return minStack.peek();
    }
}
```

**Why it works:** At every position in the main stack, the minStack holds the minimum of all elements up to and including that position.

---

### Exercise 5: Design a Thread-Safe Bounded Buffer (Producer-Consumer)

**Problem:** Implement a thread-safe queue with a max size. Producers block when full; consumers block when empty.

```java
import java.util.concurrent.locks.*;

public class BoundedBuffer<T> {
    private final Object[] buffer;
    private final int capacity;
    private int head, tail, count;
    
    private final Lock lock = new ReentrantLock();
    private final Condition notFull  = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    
    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
        this.buffer = new Object[capacity];
    }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (count == capacity) notFull.await(); // wait if full
            buffer[tail] = item;
            tail = (tail + 1) % capacity;
            count++;
            notEmpty.signal(); // wake up a consumer
        } finally {
            lock.unlock();
        }
    }
    
    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) notEmpty.await(); // wait if empty
            T item = (T) buffer[head];
            head = (head + 1) % capacity;
            count--;
            notFull.signal(); // wake up a producer
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**Key concepts demonstrated:**
- ReentrantLock for mutual exclusion
- Condition variables for blocking/signaling
- `while` not `if` for spurious wakeup protection

---

## HLD Exercises

### Exercise 6: Design a Distributed Counter (like view counts)

**Problem:** Design a system to track view counts for YouTube videos. Videos can have billions of views. Updates are high-frequency.

**Attempt first. Solution below.**

---

**Solution:**

**Naive approach:** `UPDATE views SET count = count + 1 WHERE video_id = ?` on a single DB → too slow, single point of contention.

**Better approach: Sharded Counters**

```
Client → Load Balancer → Counter Service
                              ↓
              ┌───────────────────────────┐
              │  Redis Shards             │
              │  Shard 1: video_123_s1    │
              │  Shard 2: video_123_s2    │
              │  Shard 3: video_123_s3    │
              └───────────────────────────┘
                              ↓ (periodic aggregation)
              ┌───────────────────────────┐
              │  Analytics DB             │
              │  video_123: total_views   │
              └───────────────────────────┘
```

**Steps:**
1. Increment one of N random shards for the video → `INCR video:123:shard:2`
2. To get total count: sum all N shards
3. Async job aggregates shards into permanent store periodically

**Why this works:**
- N shards = N parallel writes (no contention)
- Reads are eventual (sum all shards)
- Redis INCR is O(1) and atomic

---

### Exercise 7: Draw the architecture for a URL shortener

**Target architecture:**

```
                    ┌─────────────┐
User  ──────────→   │ API Gateway │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
    ┌──────────────────┐    ┌──────────────────────┐
    │  Write Service   │    │   Redirect Service   │
    │  (shorten URL)   │    │   (resolve URL)      │
    └────────┬─────────┘    └──────────┬───────────┘
             │                         │
    ┌────────▼─────────┐    ┌──────────▼───────────┐
    │  ID Generator    │    │   Cache (Redis)      │
    │  (Snowflake)     │    │   shortCode → URL    │
    └────────┬─────────┘    └──────────┬───────────┘
             │                         │ (cache miss)
    ┌────────▼─────────────────────────▼───────────┐
    │             KV Store (DynamoDB/Cassandra)    │
    │             shortCode ↔ longURL              │
    └─────────────────────────────────────────────┘
```

**Walk through each component:**
1. **API Gateway:** Auth, rate limiting, routing
2. **Write Service:** Generates ID, encodes to base62, stores mapping
3. **Redirect Service:** Looks up in Redis → DynamoDB, returns 302
4. **ID Generator:** Snowflake for unique, sortable IDs
5. **Cache:** 80% of traffic hits top 20% URLs — cache hit ratio ~90%

---

*See full case studies in [04-hld/11-case-studies/](../04-hld/11-case-studies/)*
