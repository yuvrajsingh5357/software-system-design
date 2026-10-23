# Heaps and Priority Queues

This chapter covers heaps — essential for problems involving "top K", median finding, task scheduling, and greedy algorithms. Java's PriorityQueue is the standard implementation.

## Table of Contents
- [Heap Fundamentals](#heap-fundamentals)
- [Heap Operations in Java](#heap-operations-in-java)
- [Common Heap Patterns](#common-heap-patterns)
  - [Top K Elements](#top-k-elements)
  - [K Closest Points](#k-closest-points)
  - [Kth Largest Element](#kth-largest-element)
  - [Median of Data Stream](#median-of-data-stream)
  - [Merge K Sorted Lists/Arrays](#merge-k-sorted-listsarrays)
- [Custom Comparators](#custom-comparators)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Heap Fundamentals

A **binary heap** is a complete binary tree satisfying the heap property:
- **Min-heap:** Every node ≤ its children → root is minimum
- **Max-heap:** Every node ≥ its children → root is maximum

```
Min-Heap example:
         1
       /   \
      3     2
     / \   / \
    7   5 4   6

Array representation: [1, 3, 2, 7, 5, 4, 6]
Index formulas:
  parent(i) = (i - 1) / 2
  leftChild(i) = 2 * i + 1
  rightChild(i) = 2 * i + 2
```

**Why complete binary tree?**
- Allows compact array storage (no null children mid-array)
- Height is always O(log n) for n elements → guarantees O(log n) operations

**Build heap from array:** O(n) — better than O(n log n) for inserting n elements.

---

## Heap Operations in Java

```java
// Min-heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
// OR equivalently:
PriorityQueue<Integer> maxHeap2 = new PriorityQueue<>((a, b) -> b - a);

// Operations
minHeap.offer(5);        // O(log n) — insert
minHeap.peek();          // O(1) — view min, no removal
minHeap.poll();          // O(log n) — remove and return min
minHeap.size();          // O(1)
minHeap.contains(5);     // O(n) — linear scan! Avoid for large heaps
minHeap.remove(5);       // O(n) — arbitrary removal

// Build heap from collection — O(n)
PriorityQueue<Integer> pq = new PriorityQueue<>(Arrays.asList(3, 1, 4, 1, 5));
```

**Critical to remember:**
- `peek()` → O(1)
- `offer()` / `poll()` → O(log n)
- `contains()` / `remove(Object)` → O(n) — Java's PriorityQueue doesn't maintain a position index
- Build from collection → O(n)

---

## Common Heap Patterns

### Top K Elements

**Pattern:** Use a min-heap of size K for "top K largest" — counterintuitively, a min-heap lets you efficiently compare and evict the smallest of the top K.

```java
// Top K frequent elements
public int[] topKFrequent(int[] nums, int k) {
    // Count frequencies
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    
    // Min-heap of size K ordered by frequency
    PriorityQueue<Map.Entry<Integer, Integer>> pq = 
        new PriorityQueue<>((a, b) -> a.getValue() - b.getValue());
    
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        pq.offer(entry);
        if (pq.size() > k) pq.poll(); // remove least frequent
    }
    
    int[] result = new int[k];
    for (int i = 0; i < k; i++) result[i] = pq.poll().getKey();
    return result;
}
// O(n log k) time, O(n) space
// Better than sorting: O(n log n)
```

**When to use min-heap for top K:**
- `top K largest` → min-heap size K (evict smallest that sneaks in)
- `top K smallest` → max-heap size K (evict largest that sneaks in)

---

### K Closest Points

```java
public int[][] kClosest(int[][] points, int k) {
    // Max-heap by distance — maintain K closest (farthest gets evicted)
    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> dist(b) - dist(a) // max-heap
    );
    
    for (int[] point : points) {
        pq.offer(point);
        if (pq.size() > k) pq.poll(); // remove farthest
    }
    
    int[][] result = new int[k][];
    for (int i = 0; i < k; i++) result[i] = pq.poll();
    return result;
}

private int dist(int[] point) {
    return point[0] * point[0] + point[1] * point[1]; // squared distance
}
// O(n log k) time — no need to sqrt for comparison purposes
```

---

### Kth Largest Element

```java
// Option 1: Min-heap of size K — O(n log k)
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> pq = new PriorityQueue<>(); // min-heap
    for (int n : nums) {
        pq.offer(n);
        if (pq.size() > k) pq.poll();
    }
    return pq.peek(); // top of min-heap = kth largest
}

// Option 2: Quickselect — O(n) average, O(n²) worst
public int findKthLargestQuickselect(int[] nums, int k) {
    return quickselect(nums, 0, nums.length - 1, nums.length - k);
}

private int quickselect(int[] nums, int left, int right, int k) {
    int pivot = partition(nums, left, right);
    if (pivot == k) return nums[pivot];
    else if (pivot < k) return quickselect(nums, pivot + 1, right, k);
    else return quickselect(nums, left, pivot - 1, k);
}

private int partition(int[] nums, int left, int right) {
    int pivot = nums[right];
    int i = left;
    for (int j = left; j < right; j++) {
        if (nums[j] <= pivot) { swap(nums, i, j); i++; }
    }
    swap(nums, i, right);
    return i;
}
private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

---

### Median of Data Stream

**Use two heaps:** max-heap for lower half, min-heap for upper half.

Invariant:
- `maxHeap.size() == minHeap.size()` OR `maxHeap.size() == minHeap.size() + 1`
- `maxHeap.peek() <= minHeap.peek()`

```java
class MedianFinder {
    private PriorityQueue<Integer> lower = new PriorityQueue<>(Comparator.reverseOrder()); // max-heap
    private PriorityQueue<Integer> upper = new PriorityQueue<>(); // min-heap
    
    public void addNum(int num) {
        lower.offer(num);
        upper.offer(lower.poll()); // balance: push smallest of lower to upper
        
        // Maintain size invariant: lower can have 1 more element
        if (lower.size() < upper.size()) {
            lower.offer(upper.poll());
        }
    }
    
    public double findMedian() {
        if (lower.size() == upper.size()) {
            return (lower.peek() + upper.peek()) / 2.0;
        }
        return lower.peek(); // odd total — lower has extra
    }
}
// addNum: O(log n), findMedian: O(1)
```

---

### Merge K Sorted Lists/Arrays

```java
// Merge K sorted lists using min-heap
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    
    // Initialize heap with head of each list
    for (ListNode node : lists) {
        if (node != null) pq.offer(node);
    }
    
    ListNode dummy = new ListNode(0), curr = dummy;
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        curr.next = min;
        curr = curr.next;
        if (min.next != null) pq.offer(min.next);
    }
    return dummy.next;
}
// O(N log K) where N = total nodes, K = number of lists
```

---

## Custom Comparators

```java
// Sort by multiple fields
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> {
    if (a[0] != b[0]) return a[0] - b[0]; // primary: first element asc
    return b[1] - a[1];                    // secondary: second element desc
});

// Custom object
class Task {
    String name; int priority; int deadline;
}
PriorityQueue<Task> taskPQ = new PriorityQueue<>((a, b) -> {
    if (a.priority != b.priority) return b.priority - a.priority; // high priority first
    return a.deadline - b.deadline; // then earlier deadline
});

// Using Comparator.comparing (cleaner for complex objects)
PriorityQueue<Task> pq2 = new PriorityQueue<>(
    Comparator.comparingInt((Task t) -> -t.priority)
              .thenComparingInt(t -> t.deadline)
);
```

**Warning on comparator overflow:**
```java
// WRONG — can overflow for large negatives and positives
(a, b) -> a - b

// CORRECT
(a, b) -> Integer.compare(a, b)
// OR
Comparator.comparingInt(x -> x)
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Kth Largest Element in an Array (LeetCode #215)

**Problem:** Find the kth largest element in an unsorted array.

**Brute:** Sort → return nums[n-k] → O(n log n).

**Better:** Min-heap of size k — O(n log k).
```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int n : nums) {
        minHeap.offer(n);
        if (minHeap.size() > k) minHeap.poll();
    }
    return minHeap.peek();
}
// Time: O(n log k)  Space: O(k)
```

**Optimized:** Quickselect — O(n) average.
```java
public int findKthLargest(int[] nums, int k) {
    return quickselect(nums, 0, nums.length - 1, nums.length - k);
}
private int quickselect(int[] nums, int l, int r, int k) {
    int pivot = partition(nums, l, r);
    if (pivot == k) return nums[pivot];
    return pivot < k ? quickselect(nums, pivot + 1, r, k) : quickselect(nums, l, pivot - 1, k);
}
// Time: O(n) avg, O(n²) worst  Space: O(1)
```

---

#### 2. Last Stone Weight (LeetCode #1046)

**Problem:** Repeatedly smash the two heaviest stones. Return the remaining stone weight or 0.

```java
public int lastStoneWeight(int[] stones) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    for (int s : stones) maxHeap.offer(s);
    while (maxHeap.size() > 1) {
        int a = maxHeap.poll(), b = maxHeap.poll();
        if (a != b) maxHeap.offer(a - b);
    }
    return maxHeap.isEmpty() ? 0 : maxHeap.poll();
}
// Time: O(n log n)  Space: O(n)
```

---

### 🟡 EASY

---

#### 3. Top K Frequent Elements (LeetCode #347)

**Problem:** Return the k most frequent elements.

**Brute:** Sort by frequency → O(n log n).

**Better:** Min-heap of size k → O(n log k).
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    PriorityQueue<Map.Entry<Integer, Integer>> pq =
        new PriorityQueue<>((a, b) -> a.getValue() - b.getValue());
    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        pq.offer(e);
        if (pq.size() > k) pq.poll();
    }
    return pq.stream().mapToInt(e -> e.getKey()).toArray();
}
// Time: O(n log k)  Space: O(n)
```

**Optimized:** Bucket sort → O(n).
```java
// Create frequency array, bucket by frequency, scan from high to low
// Time: O(n)  Space: O(n)
```

---

#### 4. K Closest Points to Origin (LeetCode #973)

**Problem:** Return the k closest points to the origin (0, 0).

```java
public int[][] kClosest(int[][] points, int k) {
    // Max-heap by distance — keep k closest
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
        (a, b) -> (b[0]*b[0] + b[1]*b[1]) - (a[0]*a[0] + a[1]*a[1])
    );
    for (int[] p : points) {
        maxHeap.offer(p);
        if (maxHeap.size() > k) maxHeap.poll();  // evict farthest
    }
    return maxHeap.toArray(new int[0][]);
}
// Time: O(n log k)  Space: O(k)
// Key: use squared distance to avoid sqrt (same ordering)
```

---

### 🟠 MEDIUM

---

#### 5. Find Median from Data Stream (LeetCode #295)

**Problem:** Design a data structure that supports adding integers and finding the median in O(1).

```java
class MedianFinder {
    PriorityQueue<Integer> lower = new PriorityQueue<>(Comparator.reverseOrder()); // max-heap (lower half)
    PriorityQueue<Integer> upper = new PriorityQueue<>();                          // min-heap (upper half)

    public void addNum(int num) {
        lower.offer(num);
        upper.offer(lower.poll());              // balance: ensure upper has valid element
        if (lower.size() < upper.size()) lower.offer(upper.poll());  // size invariant
    }

    public double findMedian() {
        if (lower.size() == upper.size()) return (lower.peek() + upper.peek()) / 2.0;
        return lower.peek();
    }
}
// addNum: O(log n)  findMedian: O(1)
```

---

#### 6. Merge K Sorted Lists (LeetCode #23)

**Problem:** Merge k sorted linked lists into one sorted list.

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) if (node != null) pq.offer(node);
    ListNode dummy = new ListNode(0), curr = dummy;
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        curr.next = min; curr = curr.next;
        if (min.next != null) pq.offer(min.next);
    }
    return dummy.next;
}
// Time: O(N log k) where N = total nodes, k = number of lists
// Space: O(k)
```

---

#### 7. Task Scheduler (LeetCode #621)

**Problem:** Given tasks with a cooldown period n, return minimum intervals to execute all tasks.

```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char c : tasks) freq[c - 'A']++;
    int maxFreq = 0;
    for (int f : freq) maxFreq = Math.max(maxFreq, f);
    long maxCount = 0;
    for (int f : freq) if (f == maxFreq) maxCount++;
    // Formula: idle slots = (maxFreq-1)*(n+1) + maxCount, but at least tasks.length
    return (int) Math.max(tasks.length, (maxFreq - 1L) * (n + 1) + maxCount);
}
// Time: O(n)  Space: O(1)
// Key insight: most frequent task dominates — it creates the bottleneck "frames"
```

---

### 🔴 HARD

---

#### 8. Sliding Window Median (LeetCode #480)

**Problem:** Find the median of each window of size k as it slides.

**Key idea:** Two-heap approach with lazy deletion.
```java
// Use two heaps (maxHeap lower, minHeap upper) + a "to-remove" map for lazy deletion
// When sliding window: add new element, mark old element for removal
// Maintain balance invariant before each median query
// Time: O(n log k)  Space: O(k)
// (Full implementation requires 50+ lines — core idea: same as MedianFinder but with lazy deletion)
```

---

#### 9. IPO (LeetCode #502) — Maximize Capital

**Problem:** Given w projects with capitals and profits, maximize final capital by completing at most k projects.

**Greedy with Heap:** Always pick the most profitable project you can afford.
```java
public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
    int n = profits.length;
    int[][] projects = new int[n][2];
    for (int i = 0; i < n; i++) projects[i] = new int[]{capital[i], profits[i]};
    Arrays.sort(projects, (a, b) -> a[0] - b[0]);  // sort by capital needed
    PriorityQueue<Integer> maxProfit = new PriorityQueue<>(Comparator.reverseOrder());
    int idx = 0;
    for (int i = 0; i < k; i++) {
        while (idx < n && projects[idx][0] <= w) maxProfit.offer(projects[idx++][1]);
        if (maxProfit.isEmpty()) break;
        w += maxProfit.poll();
    }
    return w;
}
// Time: O(n log n + k log n)  Space: O(n)
```

---

#### 10. Find K Pairs with Smallest Sums (LeetCode #373)

**Problem:** Given two sorted arrays, find k pairs (u, v) with the smallest sums.

```java
public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
    List<List<Integer>> result = new ArrayList<>();
    if (nums1.length == 0 || nums2.length == 0) return result;
    // Min-heap: {sum, i, j} — start with all pairs (nums1[i], nums2[0])
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    for (int i = 0; i < Math.min(nums1.length, k); i++)
        pq.offer(new int[]{nums1[i] + nums2[0], i, 0});
    while (k-- > 0 && !pq.isEmpty()) {
        int[] curr = pq.poll();
        int i = curr[1], j = curr[2];
        result.add(Arrays.asList(nums1[i], nums2[j]));
        if (j + 1 < nums2.length)
            pq.offer(new int[]{nums1[i] + nums2[j + 1], i, j + 1});
    }
    return result;
}
// Time: O(k log k)  Space: O(k)
```

---

## ⚡ Flashcard Summary

| Problem | Heap type | Size | Time |
|---------|-----------|------|------|
| Kth largest element | Min-heap | K | O(n log k) |
| K closest points | Max-heap | K | O(n log k) |
| Top K frequent | Min-heap | K | O(n log k) |
| Median finder | Two heaps | n/2 each | add O(log n), find O(1) |
| Merge K lists | Min-heap | K | O(N log K) |
| Task scheduler | Max-heap | alphabet | O(n log n) |
| Reorganize string | Max-heap | alphabet | O(n log n) |
| Find K pairs with smallest sum | Min-heap | K | O(K log K) |

---

## Interview Questions

### Q: Why use min-heap for top K largest?

When we want K largest elements, the candidate to evict from our "top K" set is always the smallest. A min-heap gives O(1) access to the smallest element, making eviction O(log K). Using a max-heap, we'd have to pop the max and can't easily find the minimum.

### Q: What is the time complexity of heapSort?

```
Build heap: O(n) — heapify from n/2 down to 0
Extract min n times: O(n log n)
Total: O(n log n) — but NOT stable and has poor cache performance
```

**Why not O(n²)?** Because heapify after extraction is O(log n), not O(n).

### Q: Implement a custom task scheduler using a priority queue.

```java
// CPU Scheduling: given tasks with priorities, process highest priority first
public void scheduleTasks(List<Task> tasks) {
    PriorityQueue<Task> pq = new PriorityQueue<>(
        (a, b) -> b.priority - a.priority // max priority first
    );
    pq.addAll(tasks);
    
    while (!pq.isEmpty()) {
        Task task = pq.poll();
        execute(task);
    }
}
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Min-heap default | PriorityQueue — root = minimum |
| 2 | Max-heap in Java | `new PriorityQueue<>(Comparator.reverseOrder())` |
| 3 | peek() time | O(1) |
| 4 | offer/poll time | O(log n) |
| 5 | contains() time | O(n) — no position index |
| 6 | Build from collection | O(n) — not O(n log n) |
| 7 | Top K largest trick | Min-heap of size K — evict smallest |
| 8 | Top K smallest trick | Max-heap of size K — evict largest |
| 9 | Median finder | max-heap (lower) + min-heap (upper) |
| 10 | Merge K lists | Min-heap of size K; always add next from extracted list |
| 11 | Kth largest heap vs quickselect | Heap: O(n log k); Quickselect: O(n) avg |
| 12 | Comparator overflow risk | Use Integer.compare(a,b) not a-b |
| 13 | Heap sort time | O(n log n) build + extract |
| 14 | Heap sort space | O(1) — in-place |
| 15 | Distance comparison | Use squared distance — avoids sqrt, same ordering |

---

*Next: [Graphs](07-graphs.md)*
