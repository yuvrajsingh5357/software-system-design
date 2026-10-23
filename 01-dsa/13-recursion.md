# Recursion: The Foundation of Trees, Graphs, and DP

Recursion is the process where a function calls itself to solve a smaller instance of the same problem. Mastering recursion is non-negotiable for SDE-1 and SDE-2 interviews because it forms the basis of tree traversals, DFS, backtracking, and dynamic programming.

## 1. Core Concepts

Every recursive function requires two essential parts:
1.  **Base Case:** The condition where the function stops calling itself and returns a result. Without it, you get a `StackOverflowError`.
2.  **Recursive Case:** The part where the function calls itself with a smaller or simpler input, progressing toward the base case.

### The Call Stack
When a function calls itself, the current state (variables, execution context) is pushed onto the **call stack**. Once a base case is reached, the stack begins to "unwind", passing results back up to previous calls. 
*   **Time Complexity:** Generally proportional to the number of nodes in the **recursion tree**.
*   **Space Complexity:** Proportional to the maximum depth of the call stack (which is $O(N)$ for a linear sequence of calls, or $O(\log N)$ for balanced divide-and-conquer).

## 2. Types of Recursion

### A. Non-Tail Recursion
The recursive call is *not* the last operation. The function must wait for the recursive call to return before it can finish computing its own result.
```java
// SDE-1 Must Know
public int factorial(int n) {
    if (n == 0) return 1; // Base case
    return n * factorial(n - 1); // Computation after recursion
}
```

### B. Tail Recursion
The recursive call is the absolute last operation. There is no computation after the call returns.
```java
public int factorialTailRec(int n, int accumulator) {
    if (n == 0) return accumulator;
    return factorialTailRec(n - 1, n * accumulator);
}
```
*Note for Java (17+):* Unlike C++ or Scala, Java does **not** natively optimize tail recursion. A `StackOverflowError` can still occur. However, it's conceptually important for converting to iterative solutions.

## 3. Converting Recursion to Iteration
Any recursive algorithm can be written iteratively using an explicit `Stack` (or `Queue` for BFS style).
*   **Why do it?** To avoid `StackOverflowError` in production code.
*   **Interview Tip:** Start with recursion if it's natural (e.g., Tree DFS). Only convert to iteration if the interviewer explicitly asks to optimize space or avoid stack overflow.

## 4. The "Recursion Tree" and Recurrence Relations
To analyze time complexity, draw the recursion tree.
Example: `fibonacci(n) = fibonacci(n-1) + fibonacci(n-2)`
*   Each call spawns 2 more calls.
*   The tree height is $N$.
*   Total nodes: $2^0 + 2^1 + 2^2 + \dots + 2^N \approx O(2^N)$.
*   This exponential time complexity is a massive red flag. (See Dynamic Programming to optimize this).

## 5. Common Mistakes in Interviews
1.  **Missing Base Case:** Causes `StackOverflowError`.
2.  **Improper State Passing:** Using global variables when you should pass state via method arguments. 
3.  **Redundant Calculations:** Not memoizing overlapping subproblems.

## 6. SDE-1 Practice Progression

### Level 1: Basic Recursion
*   Reverse a string recursively.
*   Calculate power ($x^n$). Try to do it in $O(\log N)$ using divide and conquer.

### Level 2: Tree Recursion
*   Inorder, Preorder, Postorder traversal.
*   Maximum depth of a binary tree.
*   Lowest Common Ancestor (LCA) in a Binary Tree.

### Level 3: Returning Values up the Chain
*   Diameter of a Binary Tree (requires returning height, but updating max diameter globally or via a wrapper object).

> **Interview Pro Tip:** When tracing recursion in an interview, do not trace the entire tree. Trace the base cases and *one* complete branch, then explain how the inductive step guarantees the rest works.
