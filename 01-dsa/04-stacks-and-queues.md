# Stacks and Queues

This chapter covers stacks and queues — two fundamental linear data structures with broad applications in parsing, scheduling, BFS/DFS, and monotonic problems.

## Table of Contents
- [Stack Fundamentals](#stack-fundamentals)
- [Queue Fundamentals](#queue-fundamentals)
- [Monotonic Stack](#monotonic-stack)
- [Monotonic Queue (Deque)](#monotonic-queue-deque)
- [Stack Applications](#stack-applications)
- [Queue Applications](#queue-applications)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Stack Fundamentals

Stack follows **LIFO** (Last In, First Out).

```java
// Java Stack — use Deque instead (Stack is legacy)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);        // O(1) — push to top
stack.pop();          // O(1) — remove top
stack.peek();         // O(1) — view top, no remove
stack.isEmpty();      // O(1)
stack.size();         // O(1)

// Why ArrayDeque over Stack?
// Stack extends Vector (synchronized, slow). ArrayDeque is faster, not synchronized.
```

**When to think "Stack":**
- Matching/balancing (parentheses, brackets, braces)
- Expression evaluation (postfix, prefix)
- Undo/redo functionality
- DFS iterative implementation
- Function call simulation
- "Nearest greater/smaller element" problems → Monotonic Stack

---

## Queue Fundamentals

Queue follows **FIFO** (First In, First Out).

```java
// Java Queue
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);       // O(1) — add to rear
queue.poll();         // O(1) — remove from front
queue.peek();         // O(1) — view front, no remove
queue.isEmpty();      // O(1)
queue.size();         // O(1)

// Deque (Double-ended queue) — most versatile
Deque<Integer> deque = new ArrayDeque<>();
deque.offerFirst(1);  // O(1) — add to front
deque.offerLast(2);   // O(1) — add to rear
deque.pollFirst();    // O(1) — remove from front
deque.pollLast();     // O(1) — remove from rear
deque.peekFirst();    // O(1)
deque.peekLast();     // O(1)

// PriorityQueue (min-heap by default)
PriorityQueue<Integer> pq = new PriorityQueue<>();
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(Comparator.reverseOrder());
pq.offer(5);          // O(log n)
pq.poll();            // O(log n) — removes minimum
pq.peek();            // O(1)
```

**When to think "Queue":**
- BFS (level-order traversal)
- Task scheduling
- Producer-Consumer problems
- Cache eviction (FIFO cache)
- Sliding window maximum → Monotonic Deque

---

## Monotonic Stack

**The most important stack technique for interviews.**

A monotonic stack maintains elements in **strictly increasing or decreasing order**.

**Key pattern:** For each element, pop elements from the stack that violate the monotonic property. The popped element's "nearest greater/smaller" is the current element.

### Next Greater Element

```java
public int[] nextGreaterElement(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices
    
    for (int i = 0; i < n; i++) {
        // Pop elements that are smaller than current
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            int idx = stack.pop();
            result[idx] = nums[i]; // nums[i] is the next greater for idx
        }
        stack.push(i);
    }
    return result;
    // O(n) time — each element pushed and popped at most once
}
```

### Largest Rectangle in Histogram

```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;
    int n = heights.length;
    
    for (int i = 0; i <= n; i++) {
        int currentHeight = (i == n) ? 0 : heights[i];
        
        while (!stack.isEmpty() && heights[stack.peek()] > currentHeight) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
// O(n) time, O(n) space
```

### Daily Temperatures (Next Warmer Day)

```java
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices
    
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
            int idx = stack.pop();
            result[idx] = i - idx; // days to wait
        }
        stack.push(i);
    }
    return result;
}
```

### Trap Rainwater

```java
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    
    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = Math.max(leftMax, height[left]);
            water += leftMax - height[left];
            left++;
        } else {
            rightMax = Math.max(rightMax, height[right]);
            water += rightMax - height[right];
            right--;
        }
    }
    return water;
}
// O(n) time, O(1) space — two-pointer approach
```

---

## Monotonic Queue (Deque)

Use a deque to find the **maximum or minimum in a sliding window** in O(1) per step.

**Key insight:** Maintain a deque of indices where values are in decreasing order. Front is always the max of current window.

### Sliding Window Maximum

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>(); // stores indices, front is max
    
    for (int i = 0; i < n; i++) {
        // Remove elements outside window
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
            deque.pollFirst();
        }
        // Remove elements smaller than current from back (they're useless)
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
            deque.pollLast();
        }
        deque.offerLast(i);
        // Window is full
        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
// O(n) time — each element enters and leaves deque at most once
```

---

## Stack Applications

### Valid Parentheses

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
}
```

### Evaluate Reverse Polish Notation (Postfix)

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String token : tokens) {
        if ("+-*/".contains(token)) {
            int b = stack.pop(), a = stack.pop();
            switch (token) {
                case "+" -> stack.push(a + b);
                case "-" -> stack.push(a - b);
                case "*" -> stack.push(a * b);
                case "/" -> stack.push(a / b);
            }
        } else {
            stack.push(Integer.parseInt(token));
        }
    }
    return stack.pop();
}
```

### Implement DFS Iteratively Using Stack

```java
public void dfsIterative(TreeNode root) {
    if (root == null) return;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        process(node);
        // Push right first so left is processed first (LIFO)
        if (node.right != null) stack.push(node.right);
        if (node.left != null) stack.push(node.left);
    }
}
```

---

## Queue Applications

### BFS Level-Order Traversal

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size(); // number of nodes at this level
        List<Integer> level = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

---

## Classic Problems

| Problem | Technique | Complexity |
|---------|-----------|-----------|
| Valid parentheses | Stack | O(n) |
| Min stack | Two stacks | O(1) all ops |
| Next greater element | Monotonic stack | O(n) |
| Daily temperatures | Monotonic stack | O(n) |
| Largest rectangle in histogram | Monotonic stack | O(n) |
| Trapping rain water | Two pointers | O(n) |
| Sliding window maximum | Monotonic deque | O(n) |
| Evaluate RPN | Stack | O(n) |
| BFS / Level order | Queue | O(n) |
| Decode string | Stack | O(n) |

---

## Interview Questions

### Q: Implement a queue using two stacks.

```java
class MyQueue {
    private Deque<Integer> inbox = new ArrayDeque<>();   // for push
    private Deque<Integer> outbox = new ArrayDeque<>();  // for pop
    
    public void push(int x) {
        inbox.push(x);
    }
    
    public int pop() {
        transfer();
        return outbox.pop();
    }
    
    public int peek() {
        transfer();
        return outbox.peek();
    }
    
    public boolean empty() {
        return inbox.isEmpty() && outbox.isEmpty();
    }
    
    private void transfer() {
        if (outbox.isEmpty()) {
            while (!inbox.isEmpty()) outbox.push(inbox.pop());
        }
    }
    // Amortized O(1) per operation — each element transferred at most once
}
```

### Q: Design a stack that returns min in O(1).

```java
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();
    
    public void push(int val) {
        stack.push(val);
        int min = minStack.isEmpty() ? val : Math.min(val, minStack.peek());
        minStack.push(min);
    }
    
    public void pop() { stack.pop(); minStack.pop(); }
    public int top() { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Valid Parentheses (LeetCode #20)

**Problem:** Given a string of brackets, determine if it's valid.

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') stack.push(c);
        else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
}
// Time: O(n)  Space: O(n)
```

---

#### 2. Min Stack (LeetCode #155)

**Problem:** Design a stack that supports push, pop, top, and retrieving the minimum element in O(1).

```java
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();
    public void push(int val) {
        stack.push(val);
        minStack.push(minStack.isEmpty() ? val : Math.min(val, minStack.peek()));
    }
    public void pop() { stack.pop(); minStack.pop(); }
    public int top() { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}
// All ops: O(1)  Space: O(n)
```

---

#### 3. Implement Queue Using Stacks (LeetCode #232)

**Problem:** Implement a first-in-first-out (FIFO) queue using only two stacks.

```java
class MyQueue {
    Deque<Integer> inbox = new ArrayDeque<>(), outbox = new ArrayDeque<>();
    public void push(int x) { inbox.push(x); }
    public int pop() { move(); return outbox.pop(); }
    public int peek() { move(); return outbox.peek(); }
    public boolean empty() { return inbox.isEmpty() && outbox.isEmpty(); }
    private void move() {
        if (outbox.isEmpty()) while (!inbox.isEmpty()) outbox.push(inbox.pop());
    }
}
// Amortized O(1) per operation — each element transferred at most once
```

---

### 🟡 EASY

---

#### 4. Daily Temperatures (LeetCode #739)

**Problem:** For each day, find how many days until a warmer temperature. Return -1 if none.

**Brute:** Nested loop → O(n²).

**Optimized:** Monotonic decreasing stack.
```java
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();  // indices
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
            int idx = stack.pop();
            result[idx] = i - idx;
        }
        stack.push(i);
    }
    return result;
}
// Time: O(n)  Space: O(n)
// Key insight: stack holds indices whose "next warmer day" hasn't been found yet
```

---

#### 5. Next Greater Element I (LeetCode #496)

**Problem:** For each element in nums1, find its next greater element in nums2.

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();
    for (int n : nums2) {
        while (!stack.isEmpty() && stack.peek() < n)
            nextGreater.put(stack.pop(), n);
        stack.push(n);
    }
    int[] result = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++)
        result[i] = nextGreater.getOrDefault(nums1[i], -1);
    return result;
}
// Time: O(m + n)  Space: O(n)
```

---

#### 6. Evaluate Reverse Polish Notation (LeetCode #150)

**Problem:** Evaluate the value of an arithmetic expression in Reverse Polish Notation (postfix).

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String token : tokens) {
        if ("+-*/".contains(token)) {
            int b = stack.pop(), a = stack.pop();
            switch (token) {
                case "+" -> stack.push(a + b);
                case "-" -> stack.push(a - b);
                case "*" -> stack.push(a * b);
                case "/" -> stack.push(a / b);
            }
        } else stack.push(Integer.parseInt(token));
    }
    return stack.pop();
}
// Time: O(n)  Space: O(n)
```

---

### 🟠 MEDIUM

---

#### 7. Largest Rectangle in Histogram (LeetCode #84)

**Problem:** Find the largest rectangle in a histogram.

**Brute:** For each pair (i, j), find the min height → O(n²) or O(n³).

**Optimized:** Monotonic increasing stack.
```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0, n = heights.length;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];
        while (!stack.isEmpty() && heights[stack.peek()] > h) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
// Time: O(n)  Space: O(n)
// Key: when a shorter bar is found, pop taller bars and calculate their max width
```

---

#### 8. Sliding Window Maximum (LeetCode #239)

**Problem:** Return the maximum value in each sliding window of size k.

**Brute:** For each window, scan for max → O(n×k).

**Optimized:** Monotonic deque.
```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>();  // stores indices, front = max
    for (int i = 0; i < n; i++) {
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1) deque.pollFirst();
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast();
        deque.offerLast(i);
        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
// Time: O(n)  Space: O(k)
```

---

#### 9. Decode String (LeetCode #394)

**Problem:** Decode a string encoded as k[encoded_string]. E.g., "3[a2[c]]" → "accaccacc".

```java
public String decodeString(String s) {
    Deque<Integer> counts = new ArrayDeque<>();
    Deque<StringBuilder> builders = new ArrayDeque<>();
    StringBuilder current = new StringBuilder();
    int k = 0;
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            counts.push(k); k = 0;
            builders.push(current);
            current = new StringBuilder();
        } else if (c == ']') {
            int repeat = counts.pop();
            StringBuilder parent = builders.pop();
            for (int i = 0; i < repeat; i++) parent.append(current);
            current = parent;
        } else {
            current.append(c);
        }
    }
    return current.toString();
}
// Time: O(output length)  Space: O(nesting depth)
```

---

#### 10. Asteroid Collision (LeetCode #735)

**Problem:** An array represents asteroids. Positive = right, negative = left. Find the state after all collisions.

```java
public int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (int a : asteroids) {
        boolean destroyed = false;
        while (!stack.isEmpty() && a < 0 && stack.peek() > 0) {
            if (stack.peek() < -a) { stack.pop(); continue; }  // stack top explodes
            else if (stack.peek() == -a) stack.pop();           // both explode
            destroyed = true; break;
        }
        if (!destroyed) stack.push(a);
    }
    int[] result = new int[stack.size()];
    for (int i = result.length - 1; i >= 0; i--) result[i] = stack.pop();
    return result;
}
// Time: O(n)  Space: O(n)
```

---

### 🔴 HARD

---

#### 11. Trapping Rain Water (LeetCode #42) — Stack Approach

**Problem:** Calculate trapped water. *(Also solvable with two-pointer — O(1) space version shown in Arrays chapter.)*

**Stack approach:**
```java
public int trap(int[] height) {
    Deque<Integer> stack = new ArrayDeque<>();
    int water = 0;
    for (int i = 0; i < height.length; i++) {
        while (!stack.isEmpty() && height[i] > height[stack.peek()]) {
            int bottom = stack.pop();
            if (stack.isEmpty()) break;
            int left = stack.peek();
            int width = i - left - 1;
            int boundedHeight = Math.min(height[left], height[i]) - height[bottom];
            water += width * boundedHeight;
        }
        stack.push(i);
    }
    return water;
}
// Time: O(n)  Space: O(n)
```

---

#### 12. Basic Calculator II (LeetCode #227)

**Problem:** Implement a basic calculator to evaluate a string expression with +, -, *, /.

```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int num = 0;
    char op = '+';
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) num = num * 10 + (c - '0');
        if (!Character.isDigit(c) && c != ' ' || i == s.length() - 1) {
            switch (op) {
                case '+' -> stack.push(num);
                case '-' -> stack.push(-num);
                case '*' -> stack.push(stack.pop() * num);
                case '/' -> stack.push(stack.pop() / num);
            }
            op = c; num = 0;
        }
    }
    return stack.stream().mapToInt(Integer::intValue).sum();
}
// Time: O(n)  Space: O(n)
```

---

#### 13. Remove K Digits (LeetCode #402)

**Problem:** Remove k digits from a number to make the smallest possible number.

**Key insight:** Use monotonic increasing stack — remove larger digits when we have budget.
```java
public String removeKdigits(String num, int k) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : num.toCharArray()) {
        while (k > 0 && !stack.isEmpty() && stack.peek() > c) {
            stack.pop(); k--;
        }
        stack.push(c);
    }
    while (k-- > 0) stack.pop();  // remove from end if still have budget
    StringBuilder sb = new StringBuilder();
    boolean leadingZero = true;
    for (char c : stack) {  // stack iterates front to back
        if (leadingZero && c == '0') continue;
        leadingZero = false;
        sb.append(c);
    }
    return sb.length() == 0 ? "0" : sb.toString();
}
// Time: O(n)  Space: O(n)
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Stack order | LIFO — Last In, First Out |
| 2 | Queue order | FIFO — First In, First Out |
| 3 | Use ArrayDeque not Stack | Stack is legacy (Vector-based, slow) |
| 4 | Monotonic stack | Pop elements violating monotonic property |
| 5 | Next greater element | Pop smaller elements; current = their next greater |
| 6 | Sliding window max | Monotonic deque; front = max of current window |
| 7 | Valid parentheses | Push open; on close, pop and match |
| 8 | Queue with 2 stacks | Inbox + outbox; transfer when outbox empty |
| 9 | Min stack | Parallel minStack tracking minimum at each depth |
| 10 | BFS uses queue | Level-order: offer root, poll level by level |
| 11 | DFS uses stack | Push right then left (LIFO → processes left first) |
| 12 | Monotonic stack time | O(n) — each element pushed/popped at most once |
| 13 | Deque operations | offerFirst/Last, pollFirst/Last, peekFirst/Last — all O(1) |
| 14 | PriorityQueue default | Min-heap; reverseOrder() for max-heap |
| 15 | Trapping rain water | leftMax - height[left] at each step (two-pointer) |

---

*Next: [Trees and Tries](05-trees-and-tries.md)*
