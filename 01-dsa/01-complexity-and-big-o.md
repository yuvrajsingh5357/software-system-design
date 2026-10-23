# Complexity and Big-O Notation

This chapter covers algorithm complexity analysis — the foundation of every DSA decision. Without this, you cannot reason about which algorithm to choose or why.

## Table of Contents
- [Why Complexity Analysis Matters](#why-complexity-analysis-matters)
- [Big-O, Big-Ω, Big-Θ](#big-o-big-ω-big-θ)
- [Common Complexity Classes](#common-complexity-classes)
- [Rules for Calculating Big-O](#rules-for-calculating-big-o)
- [Space Complexity](#space-complexity)
- [Amortized Analysis](#amortized-analysis)
- [Complexity of Common Operations](#complexity-of-common-operations)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)
- [References](#references)

---

## Why Complexity Analysis Matters

Without complexity analysis, you have no principled way to compare algorithms. Two algorithms might both "work" but one takes 1 second and the other takes 1 hour on a million-element input.

**Interview reality:** Every algorithm question is a complexity question in disguise. "Can you do better than O(n²)?" is the most common follow-up.

---

## Big-O, Big-Ω, Big-Θ

| Notation | Meaning | Practical use |
|----------|---------|---------------|
| **O(f(n))** | Upper bound (worst case) | Most commonly used |
| **Ω(f(n))** | Lower bound (best case) | Rarely discussed in interviews |
| **Θ(f(n))** | Tight bound (both upper and lower) | When best = worst case |

**In interviews, "what is the time complexity" always means Big-O (worst case).**

---

## Common Complexity Classes

From fastest to slowest:

```
O(1)        Constant       — Array index access, HashMap get (average)
O(log n)    Logarithmic    — Binary search, balanced BST operations
O(n)        Linear         — Single loop, linear scan
O(n log n)  Linearithmic   — Merge sort, heapsort, efficient sorts
O(n²)       Quadratic      — Nested loops (bubble sort, insertion sort)
O(n³)       Cubic          — Triple nested loops (naive matrix multiply)
O(2ⁿ)       Exponential    — Brute force subsets (2^n subsets of n elements)
O(n!)       Factorial      — Brute force permutations (traveling salesman naive)
```

**Visual growth comparison (n = 1,000):**

```
O(1)        =  1 operation
O(log n)    ≈  10 operations
O(n)        =  1,000 operations
O(n log n)  ≈  10,000 operations
O(n²)       =  1,000,000 operations
O(2ⁿ)       =  10^301 operations  ← completely infeasible
```

---

## Rules for Calculating Big-O

### Rule 1: Drop Constants

```java
// O(2n) → O(n)
for (int i = 0; i < n; i++) { doWork(); }
for (int i = 0; i < n; i++) { doWork(); }

// O(500) → O(1)
for (int i = 0; i < 500; i++) { doWork(); }
```

### Rule 2: Drop Non-Dominant Terms

```java
// O(n² + n) → O(n²)   (n² dominates n for large n)
// O(n + log n) → O(n)  (n dominates log n)
// O(2ⁿ + n!) → O(n!)
```

### Rule 3: Loops Multiply

```java
// Single loop: O(n)
for (int i = 0; i < n; i++) { }

// Nested loops: O(n²)
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) { }
}

// But inner loop reduces: still O(n²)
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) { }  // O(n²/2) → O(n²)
}
```

### Rule 4: Sequential Code Adds

```java
// O(n) + O(m) = O(n + m)
for (int i = 0; i < n; i++) { }  // O(n)
for (int j = 0; j < m; j++) { }  // O(m)
// Total: O(n + m), NOT O(n²) — they are independent loops
```

### Rule 5: Recursion — Use Recurrence Relations

```
T(n) = T(n/2) + O(1)    → O(log n)     (binary search)
T(n) = T(n-1) + O(1)    → O(n)         (linear recursion)
T(n) = 2T(n/2) + O(n)   → O(n log n)  (merge sort)
T(n) = 2T(n-1) + O(1)   → O(2ⁿ)       (naive Fibonacci)
```

**Master Theorem** (for T(n) = aT(n/b) + O(n^d)):
- If d > log_b(a): O(n^d)
- If d = log_b(a): O(n^d log n)
- If d < log_b(a): O(n^(log_b a))

---

## Space Complexity

Space complexity measures **extra memory used** (excluding the input itself).

```java
// O(1) space — no extra memory
public int sum(int[] nums) {
    int total = 0;
    for (int n : nums) total += n;
    return total;
}

// O(n) space — new array of size n
public int[] copyArray(int[] nums) {
    int[] copy = new int[nums.length]; // O(n)
    for (int i = 0; i < nums.length; i++) copy[i] = nums[i];
    return copy;
}

// O(n) space — recursion stack
public int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1); // n frames on stack → O(n)
}

// O(log n) space — recursion stack for balanced divide-and-conquer
public int binarySearch(int[] arr, int target, int l, int r) {
    if (l > r) return -1;
    int mid = l + (r - l) / 2;
    if (arr[mid] == target) return mid;
    if (arr[mid] < target) return binarySearch(arr, target, mid + 1, r);
    return binarySearch(arr, target, l, mid - 1);
    // log n recursive calls → O(log n) stack space
}
```

---

## Amortized Analysis

Amortized analysis gives the **average cost per operation over a sequence of operations** — even if individual operations can be expensive.

**Example: Dynamic Array (ArrayList) resizing**

When an ArrayList is full, it doubles in size:
- Most pushes: O(1)
- Occasional resize: O(n) — copies all elements

**Amortized cost:** O(1) per push.

**Why?** If you push n elements, you resize at sizes 1, 2, 4, 8, ..., n/2.
Total copy work = 1 + 2 + 4 + ... + n/2 = n - 1 < n.
So n pushes cost O(n) total → **O(1) amortized each**.

```java
// Java ArrayList internals — similar logic
// capacity doubles: [10 → 15 → 22 → 33 → 49...]
// Actually grows by 1.5x in Java, not 2x — but still amortized O(1)
list.add(element); // usually O(1), occasionally O(n) for resize
```

---

## Complexity of Common Operations

### Array

| Operation | Complexity |
|-----------|-----------|
| Access by index | O(1) |
| Search (unsorted) | O(n) |
| Search (sorted, binary) | O(log n) |
| Insert at end | O(1) amortized |
| Insert at beginning | O(n) |
| Delete at end | O(1) |
| Delete at index | O(n) |

### LinkedList (Java LinkedList / doubly-linked)

| Operation | Complexity |
|-----------|-----------|
| Access by index | O(n) |
| Search | O(n) |
| Insert at head/tail | O(1) |
| Insert in middle (given node) | O(1) |
| Delete at head/tail | O(1) |
| Delete in middle (given node) | O(1) |

### HashMap (Java HashMap)

| Operation | Average | Worst |
|-----------|---------|-------|
| get | O(1) | O(log n) ← Java 8 treeified bucket |
| put | O(1) | O(log n) |
| remove | O(1) | O(log n) |
| containsKey | O(1) | O(log n) |

### TreeMap (Java TreeMap — Red-Black Tree)

| Operation | Complexity |
|-----------|-----------|
| get/put/remove | O(log n) |
| Ordered iteration | O(n) |
| Floor/ceiling key | O(log n) |

### Heap (PriorityQueue in Java)

| Operation | Complexity |
|-----------|-----------|
| peek (min/max) | O(1) |
| offer (insert) | O(log n) |
| poll (extract min/max) | O(log n) |
| remove arbitrary element | O(n) |
| heapify (build heap) | **O(n)** ← not O(n log n)! |

### String

| Operation | Complexity |
|-----------|-----------|
| length() | O(1) |
| charAt() | O(1) |
| substring() | O(n) in Java (allocates new String) |
| String concatenation in loop | O(n²) — use StringBuilder! |
| StringBuilder append | O(1) amortized |

---

## Interview Questions

### Q: What is the time complexity of this code?

```java
for (int i = 1; i < n; i *= 2) {  // i doubles each time → log n iterations
    System.out.println(i);
}
// Answer: O(log n)
```

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < i; j++) { }  // inner loop runs 0,1,2,...,n-1 times
}
// Total = 0 + 1 + 2 + ... + (n-1) = n(n-1)/2 → O(n²)
```

```java
// Fibonacci (naive recursion)
public int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}
// Each call branches into 2 → T(n) = 2T(n-1) → O(2ⁿ)
// With memoization: O(n) time, O(n) space
```

### Q: Why is building a heap O(n) and not O(n log n)?

When building from an array (heapify), we call sift-down on each non-leaf node.
- Half the nodes are leaves: 0 work
- Quarter at height 1: 1 sift-down step each
- ...
The sum = n × (average height) where average height ≈ 1 (most nodes are near leaves).
Precise math: Σ n/2^h × h = O(n).

This is a commonly asked "trick" question to distinguish candidates who know complexity from those who just memorize it.

### Q: String concatenation — why is it O(n²)?

```java
String result = "";
for (int i = 0; i < n; i++) {
    result += "a";  // creates new String of length i+1 each iteration
    // copies: 1 + 2 + 3 + ... + n = O(n²)
}

// Solution: StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append("a");  // O(1) amortized
}
String result = sb.toString(); // O(n) final copy
```

---

## ⚡ Flashcard Summary

| Card | Front | Back |
|------|-------|------|
| 1 | O(1) example | Array index, HashMap get (avg) |
| 2 | O(log n) example | Binary search, balanced BST |
| 3 | O(n) example | Linear scan, one loop |
| 4 | O(n log n) example | Merge sort, heap sort |
| 5 | O(n²) example | Nested loops, bubble sort |
| 6 | O(2ⁿ) example | Recursive Fibonacci, all subsets |
| 7 | Drop constants rule | O(2n) → O(n) |
| 8 | Sequential code | O(n) + O(m) = O(n + m) |
| 9 | Nested loops | O(n) × O(n) = O(n²) |
| 10 | Heap build complexity | O(n) — not O(n log n)! |
| 11 | Amortized ArrayList add | O(1) amortized |
| 12 | String concat in loop | O(n²) — use StringBuilder |
| 13 | Recursion with memoization | Reduces O(2ⁿ) → O(n) |
| 14 | Java 8 HashMap worst case | O(log n) — treeified buckets |
| 15 | Space of DFS recursion | O(h) where h = tree height |

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard

---

### 🟢 BASIC

---

#### 1. Factorial — What is the time complexity?

**Problem:** Find the time and space complexity of computing `n!` recursively.

```java
public long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

**Answer:**
- **Time:** O(n) — n recursive calls
- **Space:** O(n) — n frames on the call stack

---

#### 2. Fibonacci — Brute vs Memoized

**Problem:** Compute the nth Fibonacci number.

**Brute (Exponential):**
```java
public int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
// Time: O(2^n)  Space: O(n) stack
```

**Better (Memoization):**
```java
public int fib(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] != 0) return memo[n];
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
    return memo[n];
}
// Time: O(n)  Space: O(n)
```

**Optimized (Iterative):**
```java
public int fib(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        int c = a + b; a = b; b = c;
    }
    return b;
}
// Time: O(n)  Space: O(1)
```

---

#### 3. Find Maximum in Array

**Problem:** Find the maximum element in an unsorted array.

**Approach (Single Pass):**
```java
public int findMax(int[] nums) {
    int max = nums[0];
    for (int n : nums) max = Math.max(max, n);
    return max;
}
// Time: O(n)  Space: O(1)
// Why not binary search? Array is not sorted — must look at all elements.
```

---

### 🟡 EASY

---

#### 4. Count Operations — Nested Loop Analysis

**Problem:** What is the time complexity of this code?
```java
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        System.out.println(i + " " + j);
    }
}
```

**Answer:**
- Inner loop runs n, n-1, n-2, ..., 1 times.
- Total = n(n+1)/2 ≈ O(n²).
- **Even though inner loop starts at i**, it's still O(n²).

---

#### 5. Log Loop Analysis

**Problem:** What is the time complexity?
```java
for (int i = 1; i < n; i *= 2) {
    doWork(); // O(1)
}
```

**Answer:** O(log n) — `i` doubles each iteration, so the loop runs log₂(n) times.

---

#### 6. Sum of Digits

**Problem:** What is the complexity of summing digits of a number n?

```java
public int digitSum(int n) {
    int sum = 0;
    while (n > 0) {
        sum += n % 10;
        n /= 10;
    }
    return sum;
}
```

**Answer:** O(log₁₀ n) — the number of digits in n is floor(log₁₀(n)) + 1.

---

### 🟠 MEDIUM

---

#### 7. Why is ArrayList.add() Amortized O(1)?

**Problem:** Explain why Java ArrayList's `add()` is amortized O(1) even though it occasionally copies all elements.

**Answer:**
- Most adds: O(1)
- Occasional resize (capacity doubles): O(n) — copies all n elements
- If n elements are added starting from capacity 1:
  - Copies at sizes: 1, 2, 4, 8, … n/2
  - Total copies = 1 + 2 + 4 + … + n/2 = n - 1 < n
- n operations cost O(n) total → O(1) amortized per operation.

---

#### 8. Recursive Complexity with Tree

**Problem:** What is the time complexity of a full DFS on a binary tree?

```java
public void dfs(TreeNode root) {
    if (root == null) return;
    dfs(root.left);
    dfs(root.right);
    process(root);  // O(1)
}
```

**Answer:**
- Each node visited exactly once → **O(n)**
- Space: O(h) where h = height of tree
  - Balanced: O(log n)
  - Skewed (degenerate): O(n)

---

#### 9. Power Function — Naive vs Fast Exponentiation

**Problem:** Compute x^n.

**Brute:**
```java
public double myPow(double x, int n) {
    double result = 1.0;
    for (int i = 0; i < Math.abs(n); i++) result *= x;
    return n >= 0 ? result : 1.0 / result;
}
// Time: O(n)
```

**Optimized (Fast Power / Binary Exponentiation):**
```java
public double myPow(double x, int n) {
    long N = n;  // avoid overflow when n = Integer.MIN_VALUE
    if (N < 0) { x = 1.0 / x; N = -N; }
    double result = 1.0;
    while (N > 0) {
        if ((N & 1) == 1) result *= x;  // if N is odd
        x *= x;   // square the base
        N >>= 1;  // halve the exponent
    }
    return result;
}
// Time: O(log n)  Space: O(1)
// Key insight: x^8 = (x^4)^2 = ((x^2)^2)^2 — halve exponent repeatedly
```

---

#### 10. Is This O(n) or O(n²)?

**Problem:** Analyze this code:
```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < 100; j++) {  // constant inner loop
        doWork();
    }
}
```

**Answer:** O(n) — the inner loop runs 100 times (a constant), so total = 100n = O(n). **Don't mistake constant inner loops for O(n²)!**

---

### 🔴 HARD

---

#### 11. Master Theorem Application

**Problem:** Solve these recurrences:

| Recurrence | a | b | d | Answer |
|------------|---|---|---|--------|
| T(n) = 2T(n/2) + O(n) | 2 | 2 | 1 | O(n log n) — d = log_b(a) |
| T(n) = 4T(n/2) + O(n) | 4 | 2 | 1 | O(n²) — d < log_b(a) |
| T(n) = 2T(n/2) + O(n²) | 2 | 2 | 2 | O(n²) — d > log_b(a) |
| T(n) = 3T(n/3) + O(n) | 3 | 3 | 1 | O(n log n) — d = log_b(a) |

**Master Theorem:** For T(n) = aT(n/b) + O(n^d):
- d > log_b(a): **O(n^d)**
- d = log_b(a): **O(n^d log n)**
- d < log_b(a): **O(n^(log_b a))**

---

#### 12. HashMap Collision Analysis

**Problem:** What is the worst-case time complexity of HashMap.get() in Java before and after Java 8?

**Answer:**
- **Before Java 8:** All keys hash to same bucket → linked list of length n → **O(n)** worst case
- **After Java 8:** When chain length exceeds 8 AND table size ≥ 64, bucket converts to Red-Black Tree → **O(log n)** worst case
- **Why does this matter?** A poorly designed hashCode() can degrade HashMap from O(1) to O(n).

---

#### 13. Space Complexity of N-Queens

**Problem:** What is the space complexity of solving N-Queens with backtracking?

**Answer:**
- The recursion stack goes O(n) deep (one queen per row).
- Board representation: O(n²) or O(n) with bitmask.
- **Total: O(n)** stack frames × O(n) state per frame = **O(n²)** if storing board.
- With bitmask optimization for column/diagonal attacks: O(n) space.

---

#### 14. Amortized Analysis — Stack with Multipop

**Problem:** A stack supports push(), pop(), and multipop(k) which pops up to k elements. Prove that n operations cost O(n) total.

**Answer (Potential Method):**
- Define potential Φ = number of elements on the stack.
- Push: actual cost 1, potential increases 1 → amortized cost = 1 + 1 = 2 = O(1)
- Pop: actual cost 1, potential decreases 1 → amortized cost = 1 - 1 = 0 = O(1)
- Multipop(k): actual cost min(k, stack_size), potential decreases same amount → amortized = 0 = O(1)
- **n operations → total O(n)**

---

#### 15. Tight Bound on Sorting Lower Bound

**Problem:** Prove that any comparison-based sorting algorithm requires Ω(n log n) comparisons.

**Answer (Decision Tree Argument):**
- A sorting algorithm corresponds to a binary decision tree where each internal node is a comparison.
- For n elements, there are n! possible orderings → need ≥ n! leaves.
- A binary tree with n! leaves has height ≥ log₂(n!).
- By Stirling's approximation: log₂(n!) ≈ n log₂(n) - n log₂(e) = **Ω(n log n)**.
- Therefore, no comparison-based sort can do better than **Ω(n log n)**.

---

## References

- Cormen, T.H. et al. "Introduction to Algorithms (CLRS)" — Chapter 3
- Sedgewick, R. "Algorithms in Java"
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- Skiena, S. "The Algorithm Design Manual" — Chapter 2

---

*Next: [Arrays and Strings](02-arrays-and-strings.md)*
