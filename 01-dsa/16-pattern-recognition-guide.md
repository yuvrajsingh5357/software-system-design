# DSA Pattern Recognition Guide

Memorizing problems is useless. Recognizing patterns is everything. When you hear an interview problem, your brain should immediately scan the clues and map them to a known algorithmic pattern.

Use this guide to translate problem descriptions into potential solutions.

---

## 1. Array & String Patterns

### Sliding Window
**Problem clues:**
* "Find the longest/shortest substring/subarray..."
* "Contiguous sequence..."
* "Target sum in an array of positive integers..."
**Likely pattern:** Sliding Window. Use two pointers (`left` and `right`). Expand `right` to meet a condition, shrink `left` to maintain it.

### Two Pointers
**Problem clues:**
* "Sorted array..."
* "Find a pair that sums to X..."
* "Reverse a sequence..."
* "Compare two strings..."
**Likely pattern:** Two Pointers. Either start at ends and move inward, or move them at different speeds (Fast & Slow pointers).

### Fast & Slow Pointers (Tortoise & Hare)
**Problem clues:**
* "Cycle detection..."
* "Find the middle of a linked list..."
* "Linked list intersection..."
**Likely pattern:** Move `fast` by 2 steps, `slow` by 1 step.

### Intervals
**Problem clues:**
* "Overlapping intervals..."
* "Meeting rooms..."
* "Merge ranges..."
**Likely pattern:** Sort by start time. Then iterate and compare `current.start` with `previous.end`.

---

## 2. Search & Sort Patterns

### Binary Search (Standard)
**Problem clues:**
* "Sorted array..."
* "Find an element in $O(\log N)$ time..."
**Likely pattern:** Binary Search. Define `low` and `high`.

### Binary Search on Answer
**Problem clues:**
* "Find the minimum feasible answer..."
* "Find the maximum possible..."
* "Minimize the maximum..."
**Likely pattern:** Binary Search on Answer. Guess an answer $M$. Write a `canAchieve(M)` function. If true, move `high = mid - 1` (or `low = mid + 1`).

### Top K Elements
**Problem clues:**
* "Find the top K largest/smallest..."
* "Kth most frequent..."
**Likely pattern:** Heap (PriorityQueue). Use a Min-Heap of size K for "Top K largest". Alternatively, Quickselect for $O(N)$ average time.

---

## 3. Graph & Tree Patterns

### Tree BFS
**Problem clues:**
* "Level order traversal..."
* "Shortest path in an unweighted tree/graph..."
**Likely pattern:** Queue. Push root, iterate while queue is not empty. Process level by level using a `size` variable.

### Tree DFS
**Problem clues:**
* "Find all paths..."
* "Maximum depth..."
* "Subtree properties..."
**Likely pattern:** Recursion. Pass state down the call stack, return values up.

### Topological Sort
**Problem clues:**
* "Course prerequisites..."
* "Dependency resolution..."
* "Build order..."
**Likely pattern:** Directed Acyclic Graph (DAG) + Kahn's Algorithm (In-degree array + Queue) or DFS.

### Union Find (Disjoint Set)
**Problem clues:**
* "Dynamic connectivity..."
* "Are these nodes in the same network?"
* "Number of connected components (when edges are added dynamically)..."
* "Kruskal's Minimum Spanning Tree..."
**Likely pattern:** Union Find with Path Compression and Union by Rank.

### Shortest Path
**Problem clues:**
* "Shortest path with non-negative weights..." $\rightarrow$ Dijkstra
* "Shortest path in unweighted graph..." $\rightarrow$ BFS
* "Shortest path in DAG..." $\rightarrow$ Topological Sort + Relaxation
* "Shortest path with negative weights..." $\rightarrow$ Bellman-Ford

---

## 4. Combinatorics & Optimization

### Backtracking
**Problem clues:**
* "Find all possible combinations/permutations..."
* "Generate all valid parentheses..."
* "Solve Sudoku/N-Queens..."
**Likely pattern:** Backtracking. (Choose $\rightarrow$ Explore $\rightarrow$ Unchoose).

### Dynamic Programming
**Problem clues:**
* "Find the maximum/minimum..."
* "Number of ways to..."
* "Is it possible to..."
* **AND** the problem involves overlapping subproblems / choices at each step.
**Likely pattern:** 
* 1D DP (fibonacci style)
* 2D DP (grid or strings)
* 0/1 Knapsack (pick or skip with capacity limit)

### Greedy
**Problem clues:**
* "Find the minimum/maximum..."
* Local optimum leads to global optimum (can be hard to prove).
* Often involves sorting first.
**Likely pattern:** Greedy. Be careful: if greedy fails a counter-example, it's probably DP.

---

## 5. System State Clues

* **LIFO behavior required?** $\rightarrow$ Stack (e.g., parsing strings, balancing brackets, next greater element).
* **FIFO behavior required?** $\rightarrow$ Queue.
* **Fast lookups/frequency counting?** $\rightarrow$ HashMap / HashSet.
* **Prefix matching in strings?** $\rightarrow$ Trie.
* **Monotonic properties?** $\rightarrow$ Monotonic Stack (e.g., finding the nearest smaller element).
