# DSA Patterns for Interviews

This chapter maps interview problems to patterns — the key skill that separates candidates who struggle to "see" the solution from those who recognize the pattern in 30 seconds.

## Table of Contents
- [The 14 Core Patterns](#the-14-core-patterns)
- [Pattern 1: Two Pointers](#pattern-1-two-pointers)
- [Pattern 2: Sliding Window](#pattern-2-sliding-window)
- [Pattern 3: Fast and Slow Pointers](#pattern-3-fast-and-slow-pointers)
- [Pattern 4: Merge Intervals](#pattern-4-merge-intervals)
- [Pattern 5: Cyclic Sort](#pattern-5-cyclic-sort)
- [Pattern 6: In-place Reversal](#pattern-6-in-place-reversal)
- [Pattern 7: BFS](#pattern-7-bfs)
- [Pattern 8: DFS + Backtracking](#pattern-8-dfs--backtracking)
- [Pattern 9: Two Heaps](#pattern-9-two-heaps)
- [Pattern 10: Subsets](#pattern-10-subsets)
- [Pattern 11: Modified Binary Search](#pattern-11-modified-binary-search)
- [Pattern 12: Top K Elements](#pattern-12-top-k-elements)
- [Pattern 13: K-Way Merge](#pattern-13-k-way-merge)
- [Pattern 14: Monotonic Stack](#pattern-14-monotonic-stack)
- [Pattern Recognition Cheat Sheet](#pattern-recognition-cheat-sheet)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## The 14 Core Patterns

The following 14 patterns cover 90%+ of DSA interview questions at top tech companies. For each problem, ask: **"Which pattern does this resemble?"**

---

## Pattern 1: Two Pointers

**Trigger keywords:** Sorted array, pair/triplet with target sum, palindrome, in-place modification.

```java
// Signature: left = 0, right = n-1, move based on condition
// Time: O(n), Space: O(1)

// Remove duplicates in-place
public int removeDuplicates(int[] nums) {
    int slow = 0;
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) nums[++slow] = nums[fast];
    }
    return slow + 1;
}
```

**Problems:** Two Sum (sorted), 3Sum, 4Sum, Container with Most Water, Trapping Rain Water, Remove Duplicates, Palindrome Check.

---

## Pattern 2: Sliding Window

**Trigger keywords:** Subarray/substring of length K, longest/shortest with constraint.

```java
// Signature: expand right, contract left when constraint violated
// Time: O(n), Space: O(1) or O(k)

// Fixed window: max sum of k elements
public int maxSumK(int[] nums, int k) {
    int sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i];
    int max = sum;
    for (int i = k; i < nums.length; i++) {
        sum += nums[i] - nums[i - k];
        max = Math.max(max, sum);
    }
    return max;
}

// Variable window: longest with constraint
// while (constraint violated) contract left
```

**Problems:** Maximum Sum Subarray of K, Longest Substring Without Repeating, Minimum Window Substring, Fruit Into Baskets, Permutation in String.

---

## Pattern 3: Fast and Slow Pointers

**Trigger keywords:** Linked list cycle, middle of linked list, detect pattern in sequence.

```java
// Signature: slow moves 1, fast moves 2 per step
// Meeting point reveals cycle or middle

public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Problems:** Linked List Cycle, Find Cycle Start, Middle of Linked List, Happy Number (cycle detection in math), Palindrome Linked List.

---

## Pattern 4: Merge Intervals

**Trigger keywords:** Overlapping intervals, scheduling, merging ranges.

```java
// Signature: sort by start → iterate, extend or add
// Time: O(n log n)

// Always sort first! Then greedily merge.
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> result = new ArrayList<>();
    for (int[] curr : intervals) {
        if (!result.isEmpty() && result.get(result.size()-1)[1] >= curr[0]) {
            result.get(result.size()-1)[1] = Math.max(result.get(result.size()-1)[1], curr[1]);
        } else result.add(curr);
    }
    return result.toArray(new int[0][]);
}
```

**Problems:** Merge Intervals, Insert Interval, Non-overlapping Intervals, Meeting Rooms, Employee Free Time.

---

## Pattern 5: Cyclic Sort

**Trigger keywords:** Array with numbers in range [1, n], missing number, duplicate.

```java
// Signature: each number should be at index (num - 1), swap until correct
// Time: O(n), Space: O(1)

public int[] cyclicSort(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correct = nums[i] - 1; // where nums[i] should be
        if (nums[i] != nums[correct]) { // swap
            int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
        } else i++;
    }
    return nums;
}

// Find all missing numbers
public List<Integer> findAllMissing(int[] nums) {
    cyclicSort(nums);
    List<Integer> missing = new ArrayList<>();
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] != i + 1) missing.add(i + 1);
    }
    return missing;
}
```

**Problems:** Missing Number, Find All Duplicates, Find All Missing Numbers, First Missing Positive.

---

## Pattern 6: In-place Reversal

**Trigger keywords:** Reverse a linked list, reverse a sublist.

```java
// Signature: prev = null; while loop reversing .next pointers
// Time: O(n), Space: O(1)

public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

**Problems:** Reverse Linked List, Reverse Sublist, Reverse Every K Elements, Rotate a Linked List.

---

## Pattern 7: BFS

**Trigger keywords:** Shortest path, level-by-level, minimum steps.

```java
// Signature: queue + visited set + level counter
// Time: O(V + E), Space: O(V)

public int shortestPath(int[][] graph, int start, int end) {
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    queue.offer(start); visited.add(start);
    int steps = 0;
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int node = queue.poll();
            if (node == end) return steps;
            for (int neighbor : getNeighbors(graph, node)) {
                if (!visited.contains(neighbor)) { visited.add(neighbor); queue.offer(neighbor); }
            }
        }
        steps++;
    }
    return -1;
}
```

**Problems:** Binary Tree Level Order, Zigzag Level Order, Minimum Depth, Connect Level Order Siblings, Word Ladder, 0/1 Matrix, Rotting Oranges.

---

## Pattern 8: DFS + Backtracking

**Trigger keywords:** All permutations, all subsets, generate all combinations, path in grid.

```java
// Signature: dfs(state, choices) → recurse with choice → undo choice (backtrack)
// Time: varies (often exponential), Space: O(depth)

// Subsets
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    dfs(nums, 0, new ArrayList<>(), result);
    return result;
}
private void dfs(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current)); // add current subset
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);       // choose
        dfs(nums, i + 1, current, result); // explore
        current.remove(current.size() - 1); // unchoose (backtrack)
    }
}
```

**Problems:** Subsets, Permutations, Combinations, Word Search, N-Queens, Sudoku Solver, Generate Parentheses, Letter Combinations.

---

## Pattern 9: Two Heaps

**Trigger keywords:** Median, streaming data with both max and min needed.

```java
// Signature: maxHeap (lower half) + minHeap (upper half)
// Size invariant: maxHeap.size() == minHeap.size() or maxHeap.size() == minHeap.size() + 1
// Median: maxHeap.peek() if odd, average of both tops if even

class MedianFinder {
    PriorityQueue<Integer> lower = new PriorityQueue<>(Comparator.reverseOrder()); // max-heap
    PriorityQueue<Integer> upper = new PriorityQueue<>();                          // min-heap
    
    public void addNum(int num) {
        lower.offer(num);
        upper.offer(lower.poll());
        if (lower.size() < upper.size()) lower.offer(upper.poll());
    }
    
    public double findMedian() {
        if (lower.size() == upper.size()) return (lower.peek() + upper.peek()) / 2.0;
        return lower.peek();
    }
}
```

**Problems:** Find Median from Data Stream, Sliding Window Median, IPO (maximize capital).

---

## Pattern 10: Subsets (using BFS or Backtracking)

**Trigger keywords:** All subsets, power set, combinations.

```java
// BFS approach: start with empty set, add each number to all existing subsets
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    result.add(new ArrayList<>());
    
    for (int num : nums) {
        int size = result.size();
        for (int i = 0; i < size; i++) {
            List<Integer> newSubset = new ArrayList<>(result.get(i));
            newSubset.add(num);
            result.add(newSubset);
        }
    }
    return result;
}
// For n elements: 2^n subsets → O(n × 2^n) time
```

**Problems:** Subsets, Subsets with Duplicates, String Permutations with Duplicates.

---

## Pattern 11: Modified Binary Search

**Trigger keywords:** Sorted array with modification (rotated, missing elements, condition).

```java
// Signature: while (left <= right) with careful mid/condition analysis
// Time: O(log n)

// Variations:
// 1. Standard: find exact target
// 2. Find first/last occurrence (lowerBound/upperBound)
// 3. Search in rotated array: determine which half is sorted
// 4. Binary search on answer: search on K, not on array
```

**Problems:** Binary Search, Search in Rotated Array, Find First and Last Position, Peak Index in Mountain, Split Array Largest Sum, Capacity to Ship Packages, Koko Eating Bananas.

---

## Pattern 12: Top K Elements

**Trigger keywords:** K largest/smallest/most frequent.

```java
// Min-heap size K for K largest (evict smallest)
// Max-heap size K for K smallest (evict largest)
// Time: O(n log k) — better than sorting O(n log n) when k << n

public int[] topKLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int n : nums) {
        minHeap.offer(n);
        if (minHeap.size() > k) minHeap.poll(); // evict smallest
    }
    return minHeap.stream().mapToInt(Integer::intValue).toArray();
}
```

**Problems:** Kth Largest Element, Top K Frequent Elements, K Closest Points, Sort Characters by Frequency, Reorganize String.

---

## Pattern 13: K-Way Merge

**Trigger keywords:** Merge K sorted structures, smallest range covering K lists.

```java
// Min-heap with one element from each list, always pop-and-push
// Time: O(N log K) where N = total elements

public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode head : lists) if (head != null) pq.offer(head);
    ListNode dummy = new ListNode(0), curr = dummy;
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        curr.next = min; curr = curr.next;
        if (min.next != null) pq.offer(min.next);
    }
    return dummy.next;
}
```

**Problems:** Merge K Sorted Lists, Kth Smallest in M Sorted Lists, Find K Pairs with Smallest Sums, Smallest Range.

---

## Pattern 14: Monotonic Stack

**Trigger keywords:** "Next greater/smaller", "previous greater/smaller", daily temperatures, histogram.

```java
// Signature: maintain stack in monotonic order; pop when violated
// Popped element's answer is the current element
// Time: O(n) — each element pushed/popped exactly once

public int[] nextGreaterElement(int[] nums) {
    int[] result = new int[nums.length];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // indices
    for (int i = 0; i < nums.length; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```

**Problems:** Daily Temperatures, Next Greater Element, Largest Rectangle in Histogram, Trapping Rain Water, Remove K Digits, 132 Pattern.

---

## Pattern Recognition Cheat Sheet

| You see... | Think... |
|-----------|---------|
| Sorted array + pair with target | Two Pointers |
| Subarray/substring with constraint | Sliding Window |
| Linked list + cycle/middle | Fast & Slow Pointers |
| Overlapping intervals | Merge Intervals (sort by start) |
| Numbers in [1, n], missing/duplicate | Cyclic Sort |
| Reverse linked list | In-place Reversal |
| Shortest path, minimum steps | BFS |
| All permutations/subsets/paths | DFS + Backtracking |
| Median, streaming min+max | Two Heaps |
| K largest/smallest/frequent | Top K (Heap of size K) |
| Merge K sorted things | K-way Merge (Heap) |
| Next greater/smaller element | Monotonic Stack |
| Sorted but modified array | Modified Binary Search |
| Min/max, count ways | DP (check for optimal substructure) |
| Local optimal → global | Greedy (verify greedy choice property) |

---

## ⚡ Flashcard Summary

| # | Pattern | Signature |
|---|---------|-----------|
| 1 | Two Pointers | left=0, right=n-1, move based on sum/condition |
| 2 | Sliding Window | expand right, contract left when violated |
| 3 | Fast & Slow | slow×1, fast×2; meet = cycle; slow@end = middle |
| 4 | Merge Intervals | sort by start; extend or add |
| 5 | Cyclic Sort | nums[i] should be at index nums[i]-1; swap until correct |
| 6 | In-place Reversal | prev=null; reverse .next; advance both |
| 7 | BFS | Queue + visited; level-by-level |
| 8 | DFS Backtracking | choose → recurse → unchoose |
| 9 | Two Heaps | maxHeap(lower) + minHeap(upper); median = top(s) |
| 10 | Subsets | BFS: add each num to all existing subsets |
| 11 | Modified Binary Search | Identify which half is valid; carefully move left/right |
| 12 | Top K | Min-heap size K for K largest; O(n log k) |
| 13 | K-Way Merge | Min-heap with one from each list; O(N log K) |
| 14 | Monotonic Stack | Pop when violated; popped element's answer = current |

---

*End of DSA section. Next: [Design Patterns → Creational Patterns](../02-design-patterns/01-creational-patterns.md)*
