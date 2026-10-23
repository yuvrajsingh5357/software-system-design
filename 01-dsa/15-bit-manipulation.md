# Bit Manipulation: The Secret Weapon

Bit manipulation is less common than arrays or hash maps, but when it appears in an SDE-1/SDE-2 interview, knowing the tricks makes a hard problem trivial.

## 1. Fundamentals

*   `&` (AND): 1 if both are 1.
*   `|` (OR): 1 if either is 1.
*   `^` (XOR): 1 if they are different.
*   `~` (NOT): Inverts all bits.
*   `<<` (Left Shift): Shifts bits left, filling with 0s. Equivalent to multiplying by $2^k$.
*   `>>` (Arithmetic Right Shift): Shifts bits right, preserving the sign bit (important for negative numbers in Java). Equivalent to dividing by $2^k$.
*   `>>>` (Logical Right Shift): Shifts right, filling with 0s regardless of sign.

## 2. Essential Operations

Memorize these exact lines of Java code:

*   **Check the $i$-th bit:** `(num & (1 << i)) != 0`
*   **Set the $i$-th bit:** `num |= (1 << i)`
*   **Clear the $i$-th bit:** `num &= ~(1 << i)`
*   **Toggle the $i$-th bit:** `num ^= (1 << i)`
*   **Isolate the rightmost 1-bit:** `num & (-num)`
*   **Clear the rightmost 1-bit:** `num & (num - 1)`

## 3. Must-Know Algorithms

### A. Brian Kernighan's Algorithm (Count Set Bits)
Counting how many 1s are in the binary representation of a number.
```java
int count = 0;
while (n != 0) {
    n = n & (n - 1); // Clears the lowest set bit
    count++;
}
```
*Time Complexity:* $O(K)$, where $K$ is the number of set bits (much faster than $O(32)$).

### B. Checking Power of Two
A power of two has exactly one bit set.
```java
public boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

### C. XOR Cancellation (The "Single Number" Problem)
XOR has magical properties:
*   $x \oplus 0 = x$
*   $x \oplus x = 0$
*   $a \oplus b \oplus a = (a \oplus a) \oplus b = 0 \oplus b = b$

**Problem:** Given an array where every element appears twice except for one. Find the single element.
```java
int res = 0;
for (int num : nums) res ^= num;
return res;
```
*Extensions (SDE-2):* 
*   Two unique numbers: XOR all, find the first set bit in the result, split the array into two groups based on that bit, XOR each group.
*   Every element appears 3 times except one: Requires bit counting modulo 3.

### D. Missing Number
Given an array containing $n$ distinct numbers in the range $[0, n]$, find the missing one.
*   Method 1: Sum formula $n(n+1)/2$ minus array sum. (Risk of overflow).
*   Method 2: XOR all array indices and all array values. The result is the missing number.

## 4. Bitmasks (SDE-2 / Advanced DP)
Bitmasks use an integer to represent a subset of items. If the $i$-th bit is 1, the $i$-th item is in the subset.
*   Since integers in Java are 32 bits, bitmasks are perfect for tracking state when $N \le 30$.
*   **Common Use Case:** Traveling Salesperson Problem (TSP) DP, or generating all subsets without recursion.

```java
// Generate all subsets of an array of size N using bitmasking
int totalSubsets = 1 << N; // 2^N
for (int mask = 0; mask < totalSubsets; mask++) {
    List<Integer> subset = new ArrayList<>();
    for (int i = 0; i < N; i++) {
        if ((mask & (1 << i)) != 0) { // If the i-th bit is set
            subset.add(nums[i]);
        }
    }
    result.add(subset);
}
```
