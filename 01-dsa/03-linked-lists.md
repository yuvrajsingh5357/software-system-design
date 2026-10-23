# Linked Lists

This chapter covers linked lists — a fundamental data structure and one of the most common interview topics. Mastery requires understanding pointer manipulation, cycle detection, and reversal patterns.

## Table of Contents
- [Linked List Basics](#linked-list-basics)
- [Types of Linked Lists](#types-of-linked-lists)
- [Core Operations](#core-operations)
- [Essential Techniques](#essential-techniques)
  - [Fast and Slow Pointers (Floyd's Cycle Detection)](#fast-and-slow-pointers)
  - [Reversal](#reversal)
  - [Merge](#merge)
  - [Dummy Head Node](#dummy-head-node)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Linked List Basics

```java
// Node definition
class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { 
        this.val = val; 
        this.next = next; 
    }
}

// Doubly linked list node
class DoublyListNode {
    int val;
    DoublyListNode prev, next;
    DoublyListNode(int val) { this.val = val; }
}
```

**Key properties:**
- Dynamic size (unlike arrays, no pre-allocation needed)
- O(1) insert/delete at head — but O(n) to find a node
- No random access — O(n) to reach index i
- No wasted space (no capacity like ArrayList)
- Extra memory per node (for pointers)

---

## Types of Linked Lists

```
Singly Linked:    1 → 2 → 3 → 4 → null

Doubly Linked:    null ← 1 ⇄ 2 ⇄ 3 ⇄ 4 → null

Circular:         1 → 2 → 3 → 4 ↩ (tail points to head)
```

---

## Core Operations

```java
// Insert at head — O(1)
public ListNode insertAtHead(ListNode head, int val) {
    ListNode newNode = new ListNode(val);
    newNode.next = head;
    return newNode; // new head
}

// Insert at tail — O(n)
public ListNode insertAtTail(ListNode head, int val) {
    ListNode newNode = new ListNode(val);
    if (head == null) return newNode;
    ListNode curr = head;
    while (curr.next != null) curr = curr.next;
    curr.next = newNode;
    return head;
}

// Delete by value — O(n)
public ListNode deleteNode(ListNode head, int val) {
    if (head == null) return null;
    if (head.val == val) return head.next; // delete head
    
    ListNode curr = head;
    while (curr.next != null) {
        if (curr.next.val == val) {
            curr.next = curr.next.next; // bypass node
            return head;
        }
        curr = curr.next;
    }
    return head;
}

// Length — O(n)
public int length(ListNode head) {
    int count = 0;
    while (head != null) { count++; head = head.next; }
    return count;
}
```

---

## Essential Techniques

### Fast and Slow Pointers

**Floyd's Cycle Detection:** Use two pointers — slow moves 1 step, fast moves 2 steps.

**Applications:**
- Detect cycle
- Find cycle start
- Find middle of list
- Find kth from end

#### Detect Cycle

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true; // they met → cycle
    }
    return false;
}
// O(n) time, O(1) space
```

#### Find Cycle Start

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    
    // Phase 1: Detect if cycle exists
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }
    
    if (fast == null || fast.next == null) return null; // no cycle
    
    // Phase 2: Find cycle start
    // Mathematical proof: distance from head to start = distance from meeting point to start
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    return slow; // cycle start
}
```

**Why does phase 2 work?**
Let: F = distance from head to cycle start, C = cycle length, h = distance from cycle start to meeting point.
When they meet: 2(F + h) = F + h + C → F = C - h.
Moving slow from head and fast from meeting point at same speed, they meet at cycle start.

#### Find Middle

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // slow is at middle
    // For even-length list: returns the SECOND middle node
}
```

#### Find Kth From End

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy, slow = dummy;
    
    // Move fast n+1 steps ahead
    for (int i = 0; i <= n; i++) fast = fast.next;
    
    // Move both until fast reaches end
    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    
    // slow.next is the node to remove
    slow.next = slow.next.next;
    return dummy.next;
}
```

---

### Reversal

**Iterative reversal — the standard approach:**

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next; // save next
        curr.next = prev;          // reverse pointer
        prev = curr;               // advance prev
        curr = next;               // advance curr
    }
    return prev; // new head
}
// O(n) time, O(1) space
```

**Recursive reversal:**
```java
public ListNode reverseListRecursive(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseListRecursive(head.next);
    head.next.next = head; // reverse the connection
    head.next = null;       // avoid cycle
    return newHead;
}
// O(n) time, O(n) space (call stack)
```

**Reverse a sublist (between positions left and right):**
```java
public ListNode reverseBetween(ListNode head, int left, int right) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;
    
    // Move prev to node just before position 'left'
    for (int i = 1; i < left; i++) prev = prev.next;
    
    ListNode curr = prev.next;
    for (int i = 0; i < right - left; i++) {
        ListNode next = curr.next;
        curr.next = next.next;
        next.next = prev.next;
        prev.next = next;
    }
    return dummy.next;
}
```

---

### Merge

**Merge two sorted linked lists:**

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2; // attach remaining
    return dummy.next;
}
// O(n + m) time, O(1) space
```

---

### Dummy Head Node

**The dummy head pattern eliminates special-casing for the head node.**

Without dummy: `if (head is the node to modify) handle specially`
With dummy: `dummy.next = head; always use prev.next`

```java
// Pattern template
ListNode dummy = new ListNode(0);
dummy.next = head;
ListNode curr = dummy;

// ... manipulate list ...

return dummy.next; // actual head (may have changed)
```

---

## Classic Problems

### Palindrome Linked List

```java
public boolean isPalindrome(ListNode head) {
    // 1. Find middle
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
    }
    
    // 2. Reverse second half
    ListNode secondHalf = reverseList(slow);
    ListNode copy = secondHalf; // save for restoration
    
    // 3. Compare both halves
    ListNode first = head;
    while (secondHalf != null) {
        if (first.val != secondHalf.val) return false;
        first = first.next;
        secondHalf = secondHalf.next;
    }
    return true;
}
```

### Add Two Numbers (Reverse of number stored in list)

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    int carry = 0;
    
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
    }
    return dummy.next;
}
```

### Reorder List (L0→Ln→L1→Ln-1→...)

```java
public void reorderList(ListNode head) {
    if (head == null) return;
    
    // 1. Find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next; fast = fast.next.next;
    }
    
    // 2. Reverse second half
    ListNode secondHalf = reverseList(slow.next);
    slow.next = null; // cut first half
    
    // 3. Interleave
    ListNode first = head, second = secondHalf;
    while (second != null) {
        ListNode tmp1 = first.next, tmp2 = second.next;
        first.next = second;
        second.next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}
```

---

## Interview Questions

### Q: Why use a dummy node?

Without it, you need to special-case operations on the head. With a dummy node before the head, every node (including head) has a predecessor, making pointer manipulation uniform.

### Q: How do you find the intersection of two linked lists?

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA, b = headB;
    // When a reaches end, start from headB. When b reaches end, start from headA.
    // After at most len(A) + len(B) steps, they meet at intersection or both null
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;
}
// O(n + m) time, O(1) space
// Elegant: both pointers travel the same total distance
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Reverse a Linked List (LeetCode #206)

**Problem:** Reverse a singly linked list.

**Brute (extra space):** Collect values into array, rebuild.
```java
// Time: O(n)  Space: O(n)
```

**Optimized (in-place, iterative):**
```java
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
// Time: O(n)  Space: O(1)
```

**Recursive:**
```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
// Time: O(n)  Space: O(n) — call stack
```

---

#### 2. Middle of Linked List (LeetCode #876)

**Problem:** Find the middle node of a linked list. If two middle nodes exist, return the second one.

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
// Time: O(n)  Space: O(1)
```

---

#### 3. Merge Two Sorted Lists (LeetCode #21)

**Problem:** Merge two sorted linked lists and return the sorted result.

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
// Time: O(n + m)  Space: O(1)
```

---

### 🟡 EASY

---

#### 4. Linked List Cycle (LeetCode #141)

**Problem:** Determine if a linked list has a cycle.

**Brute:** Use a HashSet to track visited nodes → O(n) space.

**Optimized:** Floyd's Cycle Detection (fast & slow pointers).
```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
// Time: O(n)  Space: O(1)
```

---

#### 5. Remove Nth Node From End (LeetCode #19)

**Problem:** Remove the nth node from the end of a linked list in one pass.

**Brute:** Two passes — first to find length, second to delete.

**Optimized:** Two-pointer trick with a gap of n.
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy, slow = dummy;
    for (int i = 0; i <= n; i++) fast = fast.next;
    while (fast != null) { slow = slow.next; fast = fast.next; }
    slow.next = slow.next.next;
    return dummy.next;
}
// Time: O(n)  Space: O(1) — single pass!
```

---

#### 6. Palindrome Linked List (LeetCode #234)

**Problem:** Check if a linked list is a palindrome. O(n) time, O(1) space.

**Brute:** Collect all values into a list → compare with reverse → O(n) space.

**Optimized:** Find middle → reverse second half → compare.
```java
public boolean isPalindrome(ListNode head) {
    // 1. Find middle
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
    }
    // 2. Reverse second half
    ListNode prev = null, curr = slow;
    while (curr != null) {
        ListNode next = curr.next; curr.next = prev; prev = curr; curr = next;
    }
    // 3. Compare
    ListNode left = head, right = prev;
    while (right != null) {
        if (left.val != right.val) return false;
        left = left.next; right = right.next;
    }
    return true;
}
// Time: O(n)  Space: O(1)
```

---

### 🟠 MEDIUM

---

#### 7. Linked List Cycle II — Find Cycle Start (LeetCode #142)

**Problem:** Find the node where the cycle begins.

**Brute:** HashSet to track visited nodes.

**Optimized:** Floyd's two-phase algorithm.
```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    // Phase 1: Detect cycle
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
        if (slow == fast) break;
    }
    if (fast == null || fast.next == null) return null;
    // Phase 2: Find entry point
    slow = head;
    while (slow != fast) { slow = slow.next; fast = fast.next; }
    return slow;
}
// Time: O(n)  Space: O(1)
// Math: if F = dist to cycle start, C = cycle length, h = dist to meet point
// then 2(F+h) = F+h+C → F = C-h → reset slow to head, both move 1 step → meet at start
```

---

#### 8. Reorder List (LeetCode #143)

**Problem:** Reorder L0→L1→…→Ln to L0→Ln→L1→Ln-1→…

**Approach:** Find middle + reverse second half + interleave.
```java
public void reorderList(ListNode head) {
    // 1. Find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next; fast = fast.next.next;
    }
    // 2. Reverse second half
    ListNode prev = null, curr = slow.next;
    slow.next = null;
    while (curr != null) {
        ListNode next = curr.next; curr.next = prev; prev = curr; curr = next;
    }
    // 3. Interleave
    ListNode first = head, second = prev;
    while (second != null) {
        ListNode tmp1 = first.next, tmp2 = second.next;
        first.next = second; second.next = tmp1;
        first = tmp1; second = tmp2;
    }
}
// Time: O(n)  Space: O(1)
```

---

#### 9. Add Two Numbers (LeetCode #2)

**Problem:** Two numbers are stored in reversed order in linked lists. Return their sum as a linked list.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    int carry = 0;
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
    }
    return dummy.next;
}
// Time: O(max(m, n))  Space: O(max(m, n))
```

---

#### 10. Intersection of Two Linked Lists (LeetCode #160)

**Problem:** Find the node at which two linked lists intersect. O(n+m) time, O(1) space.

**Brute:** HashSet of all nodes from list A; then scan list B → O(n) space.

**Optimized:** Two-pointer alignment trick.
```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA, b = headB;
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;
}
// Time: O(n + m)  Space: O(1)
// Key insight: both pointers travel lenA + lenB total → aligned at intersection
```

---

### 🔴 HARD

---

#### 11. Merge K Sorted Lists (LeetCode #23)

**Problem:** Merge k sorted linked lists.

**Brute:** Merge lists one by one → O(kn) total.

**Better:** Divide and conquer → O(n log k).

**Optimized:** Min-heap of size k.
```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) if (node != null) pq.offer(node);
    ListNode dummy = new ListNode(0), curr = dummy;
    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        curr.next = min;
        curr = curr.next;
        if (min.next != null) pq.offer(min.next);
    }
    return dummy.next;
}
// Time: O(N log k) where N = total nodes, k = number of lists
// Space: O(k)
```

---

#### 12. Copy List with Random Pointer (LeetCode #138)

**Problem:** Deep copy a linked list where each node has a `next` and a `random` pointer.

**Brute:** Two passes using HashMap<original, copy> → O(n) space.

**Optimized (O(1) space):** Weave copies between original nodes.
```java
public Node copyRandomList(Node head) {
    if (head == null) return null;
    // Step 1: Interleave cloned nodes: A→A'→B→B'→...
    Node curr = head;
    while (curr != null) {
        Node clone = new Node(curr.val);
        clone.next = curr.next;
        curr.next = clone;
        curr = clone.next;
    }
    // Step 2: Set random pointers for clones
    curr = head;
    while (curr != null) {
        if (curr.random != null) curr.next.random = curr.random.next;
        curr = curr.next.next;
    }
    // Step 3: Separate the two lists
    curr = head;
    Node cloneHead = head.next, cloneCurr = cloneHead;
    while (curr != null) {
        curr.next = cloneCurr.next;
        curr = curr.next;
        if (curr != null) { cloneCurr.next = curr.next; cloneCurr = cloneCurr.next; }
    }
    return cloneHead;
}
// Time: O(n)  Space: O(1)
```

---

#### 13. Reverse Nodes in K-Group (LeetCode #25)

**Problem:** Reverse nodes in the list in groups of k. If remaining nodes are fewer than k, leave them as-is.

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode check = head;
    for (int i = 0; i < k; i++) {
        if (check == null) return head;  // fewer than k nodes left
        check = check.next;
    }
    // Reverse k nodes
    ListNode prev = null, curr = head;
    for (int i = 0; i < k; i++) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    // head is now the tail of the reversed group; connect to next group
    head.next = reverseKGroup(curr, k);
    return prev;
}
// Time: O(n)  Space: O(n/k) — recursion stack (O(1) with iterative)
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Insert at head | O(1) — just update new node's next |
| 2 | Insert at tail | O(n) — traverse to end |
| 3 | Fast/slow pointer | Slow moves 1, fast moves 2 |
| 4 | Find middle | Fast/slow — when fast reaches end, slow is at middle |
| 5 | Detect cycle | Fast/slow meet → cycle exists |
| 6 | Find cycle start | Reset slow to head; move both 1-step; meeting = start |
| 7 | kth from end | Move fast k steps ahead, then both together |
| 8 | Iterative reversal | prev=null, curr=head; reverse ptr, advance both |
| 9 | Dummy head pattern | ListNode dummy = new ListNode(0); dummy.next = head |
| 10 | Merge sorted lists | Compare heads; attach smaller; repeat |
| 11 | Palindrome list | Find middle → reverse second half → compare |
| 12 | Intersection | Traverse A then B, B then A — they meet at intersection |
| 13 | Time complexity | Access O(n), insert/delete at known position O(1) |
| 14 | Recursive reversal | O(n) space due to call stack |
| 15 | Reverse sublist | Use "insert at front" technique between prev and rest |

---

*Next: [Stacks and Queues](04-stacks-and-queues.md)*
