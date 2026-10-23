# Dynamic Programming

Dynamic Programming (DP) is the most feared and most rewarding topic in DSA interviews. This chapter teaches you to recognize, formulate, and implement DP solutions — with Java code for every major pattern.

## Table of Contents
- [What is DP?](#what-is-dp)
- [DP Formulation Process](#dp-formulation-process)
- [Core DP Patterns](#core-dp-patterns)
  - [1D DP (Linear)](#1d-dp-linear)
  - [Knapsack Patterns](#knapsack-patterns)
  - [String DP](#string-dp)
  - [Grid DP](#grid-dp)
  - [Interval DP](#interval-dp)
  - [Tree DP](#tree-dp)
  - [State Machine DP](#state-machine-dp)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## What is DP?

**DP is applicable when a problem has:**
1. **Optimal substructure** — optimal solution is built from optimal solutions to subproblems
2. **Overlapping subproblems** — same subproblems are solved multiple times

**DP vs Greedy:**
- Greedy: always choose locally best option, no backtracking
- DP: try all possibilities, store results to avoid recomputation

**Recognize DP problems:**
- "Minimum/maximum..."
- "Count number of ways..."
- "Is it possible to..."
- Involves sequences, subsets, strings, or grids
- Contains words like "each step", "pick", "include/exclude"

---

## DP Formulation Process

```
Step 1: Define the subproblem
        dp[i] = answer to the problem considering elements up to index i

Step 2: Write the recurrence (transition)
        dp[i] = f(dp[i-1], dp[i-2], ...)

Step 3: Identify base cases
        dp[0] = ?, dp[1] = ?

Step 4: Determine order of computation
        Usually left-to-right for 1D, top-left to bottom-right for 2D

Step 5: Extract answer
        Often dp[n] or max(dp[i]) over all i
```

---

## Core DP Patterns

### 1D DP (Linear)

#### Climbing Stairs (Fibonacci variant)

```java
// dp[i] = number of ways to reach step i
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// O(n) time, O(1) space (optimized from O(n) space)
```

#### House Robber

```java
// dp[i] = max money robbing houses 0..i
// Either rob house i (dp[i-2] + nums[i]) or skip it (dp[i-1])
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    int prev2 = nums[0], prev1 = Math.max(nums[0], nums[1]);
    for (int i = 2; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}

// House Robber II — circular (can't rob both first and last)
public int robII(int[] nums) {
    if (nums.length == 1) return nums[0];
    return Math.max(
        rob(nums, 0, nums.length - 2),  // rob range [0, n-2]
        rob(nums, 1, nums.length - 1)   // rob range [1, n-1]
    );
}
private int rob(int[] nums, int start, int end) {
    int prev2 = 0, prev1 = 0;
    for (int i = start; i <= end; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

#### Longest Increasing Subsequence (LIS)

```java
// O(n²) DP
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1); // each element is a subsequence of length 1
    int maxLen = 1;
    
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }
    return maxLen;
}

// O(n log n) using patience sorting (binary search)
public int lengthOfLISOptimal(int[] nums) {
    List<Integer> tails = new ArrayList<>(); // smallest tail for each length
    for (int n : nums) {
        int pos = Collections.binarySearch(tails, n);
        if (pos < 0) pos = -(pos + 1); // insertion point
        if (pos == tails.size()) tails.add(n);
        else tails.set(pos, n);
    }
    return tails.size();
}
```

---

### Knapsack Patterns

#### 0/1 Knapsack — each item used at most once

```java
// Can we fill a bag of capacity W using items with given weights and values?
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    // dp[i][w] = max value using items 0..i-1 with capacity w
    int[][] dp = new int[n + 1][W + 1];
    
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w]; // don't take item i
            if (weights[i-1] <= w) {
                dp[i][w] = Math.max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1]);
            }
        }
    }
    return dp[n][W];
}

// Space-optimized: O(W) — note: iterate w from right to left!
public int knapsack1D(int[] weights, int[] values, int W) {
    int[] dp = new int[W + 1];
    for (int i = 0; i < weights.length; i++) {
        for (int w = W; w >= weights[i]; w--) { // RIGHT TO LEFT prevents reuse
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }
    return dp[W];
}
```

#### Partition Equal Subset Sum (0/1 Knapsack variant)

```java
public boolean canPartition(int[] nums) {
    int total = Arrays.stream(nums).sum();
    if (total % 2 != 0) return false;
    int target = total / 2;
    
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    
    for (int num : nums) {
        for (int j = target; j >= num; j--) { // right to left — 0/1 knapsack
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
```

#### Coin Change — Unbounded Knapsack (each coin reused)

```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1); // "infinity"
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
// Note: inner loop iterates left to right (unbounded = items reusable)
// Contrast with 0/1 knapsack: inner loop right to left

// Count ways — Coin Change 2
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1; // one way to make 0: use no coins
    
    for (int coin : coins) {             // outer: coins
        for (int i = coin; i <= amount; i++) { // inner: amounts
            dp[i] += dp[i - coin];
        }
    }
    return dp[amount];
    // Outer = coins, inner = amounts: counts combinations (not permutations)
    // Swap loops → counts permutations
}
```

---

### String DP

#### Longest Common Subsequence (LCS)

```java
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i-1) == text2.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1] + 1; // chars match
            } else {
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]); // skip one
            }
        }
    }
    return dp[m][n];
}
// O(mn) time, O(mn) space; can optimize to O(n) space
```

#### Edit Distance (Levenshtein)

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    // Base cases: converting empty string
    for (int i = 0; i <= m; i++) dp[i][0] = i; // delete i chars
    for (int j = 0; j <= n; j++) dp[0][j] = j; // insert j chars
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i-1) == word2.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1]; // no operation needed
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i-1][j-1], // replace
                    Math.min(dp[i-1][j], dp[i][j-1]) // delete, insert
                );
            }
        }
    }
    return dp[m][n];
}
```

#### Longest Palindromic Subsequence

```java
public int longestPalindromeSubseq(String s) {
    int n = s.length();
    // LPS(s) = LCS(s, reverse(s))
    return longestCommonSubsequence(s, new StringBuilder(s).reverse().toString());
}
```

---

### Grid DP

#### Unique Paths

```java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    // First row and column: only one way to reach
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;
    
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i-1][j] + dp[i][j-1]; // from above or from left
        }
    }
    return dp[m-1][n-1];
}
```

#### Minimum Path Sum

```java
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];
    
    for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0]; // top column
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j]; // top row
    
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = grid[i][j] + Math.min(dp[i-1][j], dp[i][j-1]);
        }
    }
    return dp[m-1][n-1];
}
```

---

### Interval DP

**Pattern:** `dp[i][j]` = answer for subarray/substring `[i, j]`. Fill diagonally (shorter lengths first).

#### Burst Balloons

```java
public int maxCoins(int[] nums) {
    // Pad with 1s on both ends
    int n = nums.length;
    int[] balloons = new int[n + 2];
    balloons[0] = balloons[n + 1] = 1;
    for (int i = 1; i <= n; i++) balloons[i] = nums[i - 1];
    
    // dp[i][j] = max coins from bursting all balloons in (i, j) exclusive
    int[][] dp = new int[n + 2][n + 2];
    
    for (int len = 1; len <= n; len++) { // length of interval
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            for (int k = i; k <= j; k++) { // k is the LAST balloon to burst in [i,j]
                dp[i][j] = Math.max(dp[i][j],
                    dp[i][k-1] + balloons[i-1] * balloons[k] * balloons[j+1] + dp[k+1][j]);
            }
        }
    }
    return dp[1][n];
}
```

---

### State Machine DP

**Pattern for problems with distinct states that transition.**

#### Best Time to Buy and Sell Stock with Cooldown

```java
public int maxProfit(int[] prices) {
    // States: HELD (holding stock), SOLD (just sold, in cooldown), REST
    int held = Integer.MIN_VALUE, sold = 0, rest = 0;
    
    for (int price : prices) {
        int prevHeld = held, prevSold = sold, prevRest = rest;
        held = Math.max(prevHeld, prevRest - price); // hold from prev or buy from rest
        sold = prevHeld + price;                      // sell from held
        rest = Math.max(prevRest, prevSold);          // rest or come out of cooldown
    }
    return Math.max(sold, rest);
}
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Climbing Stairs (LeetCode #70)

**Problem:** You can climb 1 or 2 steps at a time. How many distinct ways to reach step n?

**Brute (Recursion):** O(2^n).

**Better (Memoization):** O(n) time, O(n) space.

**Optimized (Fibonacci pattern):**
```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2; prev2 = prev1; prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
// dp[i] = dp[i-1] + dp[i-2] — exactly Fibonacci
```

---

#### 2. Fibonacci Number (LeetCode #509)

Same as climbing stairs — dp[n] = dp[n-1] + dp[n-2]. O(n) time, O(1) space.

---

### 🟡 EASY

---

#### 3. House Robber (LeetCode #198)

**Problem:** Rob non-adjacent houses to maximize total money.

```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    int prev2 = nums[0], prev1 = Math.max(nums[0], nums[1]);
    for (int i = 2; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1; prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
// dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

---

#### 4. Min Cost Climbing Stairs (LeetCode #746)

**Problem:** Each step has a cost. You can start from step 0 or 1. Find minimum cost to reach the top.

```java
public int minCostClimbingStairs(int[] cost) {
    int n = cost.length;
    int prev2 = cost[0], prev1 = cost[1];
    for (int i = 2; i <= n; i++) {
        int curr = (i == n ? 0 : cost[i]) + Math.min(prev1, prev2);
        prev2 = prev1; prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
```

---

#### 5. Best Time to Buy and Sell Stock (LeetCode #121)

**Problem:** Maximum profit from single buy and sell.

```java
// dp: minPrice = min so far; profit = max(price - minPrice)
// Time: O(n)  Space: O(1)
```

---

### 🟠 MEDIUM

---

#### 6. Coin Change (LeetCode #322) — Unbounded Knapsack

**Problem:** Find minimum number of coins to make amount.

**Brute:** Recursion → exponential.

**Optimized:** DP bottom-up.
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);  // "infinity"
    dp[0] = 0;
    for (int i = 1; i <= amount; i++)
        for (int coin : coins)
            if (coin <= i) dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
// Time: O(amount × coins)  Space: O(amount)
// Left-to-right inner loop = coins are reusable (unbounded knapsack)
```

---

#### 7. Longest Increasing Subsequence (LeetCode #300)

**Problem:** Find the length of the longest strictly increasing subsequence.

**Brute:** Recursion → O(2^n).

**Better:** O(n²) DP.
```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    int max = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++)
            if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
        max = Math.max(max, dp[i]);
    }
    return max;
}
```

**Optimized:** Patience sorting — O(n log n).
```java
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    for (int n : nums) {
        int pos = Collections.binarySearch(tails, n);
        if (pos < 0) pos = -(pos + 1);
        if (pos == tails.size()) tails.add(n);
        else tails.set(pos, n);
    }
    return tails.size();
}
// Time: O(n log n)  Space: O(n)
```

---

#### 8. Unique Paths (LeetCode #62)

**Problem:** Count paths in an m×n grid from top-left to bottom-right moving only right/down.

```java
public int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[j] += dp[j - 1];
    return dp[n - 1];
}
// Time: O(m×n)  Space: O(n) — optimized from O(m×n)
// Also solvable as combinatorics: C(m+n-2, m-1)
```

---

#### 9. Partition Equal Subset Sum (LeetCode #416) — 0/1 Knapsack

**Problem:** Determine if array can be partitioned into two subsets with equal sum.

```java
public boolean canPartition(int[] nums) {
    int total = Arrays.stream(nums).sum();
    if (total % 2 != 0) return false;
    int target = total / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int num : nums)
        for (int j = target; j >= num; j--)  // RIGHT-TO-LEFT = 0/1 knapsack
            dp[j] = dp[j] || dp[j - num];
    return dp[target];
}
// Time: O(n × target)  Space: O(target)
```

---

#### 10. Longest Common Subsequence (LeetCode #1143)

**Problem:** Find length of longest common subsequence between two strings.

```java
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i-1) == text2.charAt(j-1)) dp[i][j] = dp[i-1][j-1] + 1;
            else dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        }
    return dp[m][n];
}
// Time: O(m×n)  Space: O(m×n)
```

---

### 🔴 HARD

---

#### 11. Edit Distance (LeetCode #72) — Levenshtein Distance

**Problem:** Minimum operations (insert, delete, replace) to convert word1 to word2.

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i-1) == word2.charAt(j-1)) dp[i][j] = dp[i-1][j-1];
            else dp[i][j] = 1 + Math.min(dp[i-1][j-1], Math.min(dp[i-1][j], dp[i][j-1]));
        }
    return dp[m][n];
}
// Time: O(m×n)  Space: O(m×n)
// Transitions: match=free, replace/delete/insert=cost 1
```

---

#### 12. Maximum Product Subarray (LeetCode #152)

**Problem:** Find the maximum product of a contiguous subarray.

**Key challenge:** Negative × negative = positive — must track both max and min.
```java
public int maxProduct(int[] nums) {
    int maxProd = nums[0], minProd = nums[0], result = nums[0];
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] < 0) { int tmp = maxProd; maxProd = minProd; minProd = tmp; }
        maxProd = Math.max(nums[i], maxProd * nums[i]);
        minProd = Math.min(nums[i], minProd * nums[i]);
        result = Math.max(result, maxProd);
    }
    return result;
}
// Time: O(n)  Space: O(1)
```

---

#### 13. Word Break (LeetCode #139)

**Problem:** Determine if string s can be segmented into words from the dictionary.

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;
    for (int i = 1; i <= n; i++)
        for (int j = 0; j < i; j++)
            if (dp[j] && dict.contains(s.substring(j, i))) { dp[i] = true; break; }
    return dp[n];
}
// Time: O(n² × k) where k = avg word length  Space: O(n)
```

---

#### 14. Decode Ways (LeetCode #91)

**Problem:** Count the number of ways to decode a digit string to letters (1='A'..26='Z').

```java
public int numDecodings(String s) {
    int n = s.length();
    int[] dp = new int[n + 1];
    dp[0] = 1;
    dp[1] = s.charAt(0) == '0' ? 0 : 1;
    for (int i = 2; i <= n; i++) {
        int oneDigit = Integer.parseInt(s.substring(i-1, i));
        int twoDigits = Integer.parseInt(s.substring(i-2, i));
        if (oneDigit >= 1) dp[i] += dp[i-1];
        if (twoDigits >= 10 && twoDigits <= 26) dp[i] += dp[i-2];
    }
    return dp[n];
}
// Time: O(n)  Space: O(n) → can optimize to O(1)
```

---

#### 15. Burst Balloons (LeetCode #312) — Interval DP

**Problem:** Burst balloons to maximize coins. Coins = left_neighbor × current × right_neighbor.

**Key insight:** Think backwards — which balloon is burst LAST in interval [i, j]?
```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    int[] balloons = new int[n + 2];
    balloons[0] = balloons[n + 1] = 1;
    for (int i = 1; i <= n; i++) balloons[i] = nums[i - 1];
    int[][] dp = new int[n + 2][n + 2];
    for (int len = 1; len <= n; len++)
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            for (int k = i; k <= j; k++)
                dp[i][j] = Math.max(dp[i][j],
                    dp[i][k-1] + balloons[i-1] * balloons[k] * balloons[j+1] + dp[k+1][j]);
        }
    return dp[1][n];
}
// Time: O(n³)  Space: O(n²)
```

---

## Interview Questions

### Q: What is the difference between top-down and bottom-up DP?

**Top-down (Memoization):**
- Recursive approach: solve(n) → recurse with memo
- Natural to write: follows problem definition
- Lazy: only solves needed subproblems
- Stack overhead for deep recursion

**Bottom-up (Tabulation):**
- Iterative approach: fill table from smallest to largest
- No recursion overhead
- All subproblems solved (even if unused)
- Often faster in practice (better cache behavior)

### Q: Knapsack: why iterate inner loop right-to-left for 0/1 but left-to-right for unbounded?

**0/1 knapsack** (each item once): We must use `dp[w - weight]` from the PREVIOUS row (before item was considered). Iterating right-to-left ensures we only use each item once because we read `dp[w - weight]` before overwriting it.

**Unbounded knapsack** (item reusable): We want to allow the same item multiple times. Left-to-right means `dp[i - coin]` has already been updated with the current coin → current coin can be used again.

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | DP recognition | Min/max, count ways, possible or not |
| 2 | Optimal substructure | Optimal solution uses optimal sub-solutions |
| 3 | Overlapping subproblems | Same subproblems recomputed → memoize |
| 4 | Top-down | Recursion + memoization; solve needed subproblems |
| 5 | Bottom-up | Fill table iteratively; all subproblems |
| 6 | LIS O(n log n) | Patience sorting: maintain tails list + binary search |
| 7 | 0/1 Knapsack 1D | Iterate j from W down to weight[i] |
| 8 | Unbounded Knapsack | Iterate j from weight up to W (reuse allowed) |
| 9 | Coin change = unbounded | Outer: amounts; inner: coins; left-to-right |
| 10 | Coin ways vs Coin count | Count: `dp[i] += dp[i-coin]`; Min: `dp[i] = min(dp[i], dp[i-coin]+1)` |
| 11 | LCS recurrence | If chars match: dp[i][j] = dp[i-1][j-1] + 1 |
| 12 | Edit distance operations | 1 + min(replace, delete, insert) |
| 13 | Grid DP order | Top-left to bottom-right; fill row/col base cases |
| 14 | Interval DP order | By length (short → long); fill diagonally |
| 15 | State machine DP | States = HELD/SOLD/REST; transitions per day |

---

*Next: [Greedy Algorithms](11-greedy-algorithms.md)*
