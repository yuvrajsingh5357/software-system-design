# Arrays and Strings

This chapter covers arrays and strings — the foundation of almost every DSA interview problem. Mastery here means you can recognize patterns in seconds and implement cleanly.

## Table of Contents
- [Arrays in Java](#arrays-in-java)
- [Core Techniques](#core-techniques)
  - [Two Pointers](#two-pointers)
  - [Sliding Window](#sliding-window)
  - [Prefix Sum](#prefix-sum)
  - [Kadane's Algorithm](#kadanes-algorithm)
- [Strings in Java](#strings-in-java)
- [String Techniques](#string-techniques)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Arrays in Java

```java
// Declaration and initialization
int[] arr = new int[10];         // All zeros
int[] arr = {1, 2, 3, 4, 5};    // Inline

// Key operations
arr.length                        // O(1)
Arrays.sort(arr)                  // O(n log n) — in-place
Arrays.copyOf(arr, arr.length)    // O(n) — new array
Arrays.fill(arr, 0)               // O(n) — fill with value

// 2D arrays
int[][] matrix = new int[rows][cols];
int[][] matrix = {{1,2},{3,4},{5,6}};

// ArrayList vs array
List<Integer> list = new ArrayList<>();
list.add(5);           // O(1) amortized
list.get(2);           // O(1)
list.remove(2);        // O(n) — shifts elements
list.size();           // O(1)
Collections.sort(list);// O(n log n)
```

---

## Core Techniques

### Two Pointers

**When to use:** Sorted arrays, finding pairs/triplets, palindrome checks, in-place modifications.

**Pattern:**
```
Start with left = 0, right = n-1
Move pointers based on condition
Stop when they meet or cross
```

#### Two Sum (sorted array)

```java
public int[] twoSum(int[] numbers, int target) {
    int left = 0, right = numbers.length - 1;
    while (left < right) {
        int sum = numbers[left] + numbers[right];
        if (sum == target) return new int[]{left + 1, right + 1};
        else if (sum < target) left++;
        else right--;
    }
    return new int[]{-1, -1};
}
// O(n) time, O(1) space
```

#### Container With Most Water

```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxWater = 0;
    
    while (left < right) {
        int water = Math.min(height[left], height[right]) * (right - left);
        maxWater = Math.max(maxWater, water);
        // Move the shorter wall — only way to potentially increase water
        if (height[left] < height[right]) left++;
        else right--;
    }
    return maxWater;
}
// O(n) time, O(1) space
```

#### 3Sum (sorted, two-pointer extension)

```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);
    
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue; // skip duplicates
        
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left+1]) left++;
                while (left < right && nums[right] == nums[right-1]) right--;
                left++; right--;
            } else if (sum < 0) left++;
            else right--;
        }
    }
    return result;
}
// O(n²) time, O(1) extra space (excluding output)
```

---

### Sliding Window

**When to use:** Contiguous subarray/substring problems with a constraint.

**Two variants:**
1. **Fixed window size:** Both pointers move at same rate
2. **Variable window size:** Right pointer expands, left contracts when constraint violated

#### Fixed Window: Maximum Sum of K Elements

```java
public int maxSumSubarray(int[] nums, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];
    
    int maxSum = windowSum;
    for (int i = k; i < nums.length; i++) {
        windowSum += nums[i] - nums[i - k]; // slide window
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
// O(n) time, O(1) space
```

#### Variable Window: Longest Substring Without Repeating Characters

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> charIndex = new HashMap<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        // If char seen and within current window, move left past it
        if (charIndex.containsKey(c) && charIndex.get(c) >= left) {
            left = charIndex.get(c) + 1;
        }
        charIndex.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
// O(n) time, O(min(n, charset)) space
```

#### Variable Window: Minimum Window Substring

```java
public String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
    
    int left = 0, formed = 0, required = need.size();
    int minLen = Integer.MAX_VALUE, minLeft = 0;
    Map<Character, Integer> window = new HashMap<>();
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).equals(need.get(c))) formed++;
        
        while (formed == required) { // window is valid — try to shrink
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            char lc = s.charAt(left++);
            window.merge(lc, -1, Integer::sum);
            if (need.containsKey(lc) && window.get(lc) < need.get(lc)) formed--;
        }
    }
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}
// O(|s| + |t|) time, O(|t|) space
```

---

### Prefix Sum

**When to use:** Range sum queries, subarray sum equals K, 2D range queries.

**Key insight:** `sum(i, j) = prefix[j+1] - prefix[i]`

```java
// Build prefix sum
int[] prefix = new int[nums.length + 1];
for (int i = 0; i < nums.length; i++) {
    prefix[i + 1] = prefix[i] + nums[i];
}

// Query range sum [l, r] in O(1)
int rangeSum = prefix[r + 1] - prefix[l];
```

#### Subarray Sum Equals K

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1); // empty subarray
    int count = 0, sum = 0;
    
    for (int num : nums) {
        sum += num;
        // If (sum - k) exists as a prefix, we found a valid subarray
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
// O(n) time, O(n) space
```

---

### Kadane's Algorithm

**Maximum Subarray Sum** — classic DP technique.

**Insight:** At each position, either extend the previous subarray or start fresh.

```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0];
    int currentSum = nums[0];
    
    for (int i = 1; i < nums.length; i++) {
        // Either extend current or restart from here
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
// O(n) time, O(1) space

// Variant: also return the subarray indices
public int[] maxSubArrayWithIndices(int[] nums) {
    int maxSum = nums[0], currentSum = nums[0];
    int start = 0, end = 0, tempStart = 0;
    
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] > currentSum + nums[i]) {
            currentSum = nums[i];
            tempStart = i;
        } else {
            currentSum += nums[i];
        }
        if (currentSum > maxSum) {
            maxSum = currentSum;
            start = tempStart;
            end = i;
        }
    }
    return new int[]{maxSum, start, end};
}
```

---

## Strings in Java

```java
String s = "hello";

// Key methods
s.length()                      // O(1)
s.charAt(i)                     // O(1)
s.substring(start, end)         // O(n) — creates new String
s.indexOf('e')                  // O(n)
s.contains("ell")               // O(n)
s.startsWith("he")              // O(n)
s.toCharArray()                 // O(n)
s.split(",")                    // O(n)
s.trim()                        // O(n)
s.toLowerCase()                 // O(n)
String.valueOf(42)              // int → String
Integer.parseInt("42")          // String → int

// StringBuilder for mutable strings
StringBuilder sb = new StringBuilder();
sb.append("hello");             // O(1) amortized
sb.insert(2, "XY");             // O(n)
sb.delete(1, 3);                // O(n)
sb.reverse();                   // O(n)
sb.toString();                  // O(n)
```

**Critical interview reminder:**
```java
// String concatenation in loop = O(n²) — NEVER do this
String result = "";
for (int i = 0; i < n; i++) result += i; // O(n²)

// StringBuilder = O(n) — ALWAYS use for building strings
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) sb.append(i); // O(n)
```

---

## String Techniques

### Character Frequency Array

Use a `int[26]` array for lowercase English letters — much faster than HashMap.

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (char c : t.toCharArray()) count[c - 'a']--;
    for (int n : count) if (n != 0) return false;
    return true;
}
// O(n) time, O(1) space (fixed 26-char array)
```

### Check Palindrome

```java
public boolean isPalindrome(String s) {
    // Clean: keep only alphanumeric, lowercase
    s = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++; right--;
    }
    return true;
}
```

### String Hashing (Rabin-Karp)

```java
// Rolling hash for pattern matching — O(n + m) average
public List<Integer> findOccurrences(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    if (m > n) return result;
    
    int base = 31, mod = 1_000_000_007;
    long patternHash = 0, windowHash = 0, power = 1;
    
    for (int i = 0; i < m; i++) {
        patternHash = (patternHash * base + (pattern.charAt(i) - 'a' + 1)) % mod;
        windowHash = (windowHash * base + (text.charAt(i) - 'a' + 1)) % mod;
        if (i > 0) power = power * base % mod;
    }
    
    if (patternHash == windowHash) result.add(0);
    
    for (int i = 1; i <= n - m; i++) {
        windowHash = (windowHash - (text.charAt(i-1) - 'a' + 1) * power % mod + mod) % mod;
        windowHash = (windowHash * base + (text.charAt(i + m - 1) - 'a' + 1)) % mod;
        if (windowHash == patternHash) result.add(i);
    }
    return result;
}
```

---

## Classic Problems

| Problem | Technique | Complexity |
|---------|-----------|-----------|
| Two Sum (sorted) | Two pointers | O(n) |
| 3Sum | Sort + two pointers | O(n²) |
| Longest substring w/o repeat | Sliding window + HashMap | O(n) |
| Minimum window substring | Sliding window | O(n) |
| Maximum subarray | Kadane's | O(n) |
| Subarray sum = K | Prefix sum + HashMap | O(n) |
| Product except self | Prefix + suffix arrays | O(n) |
| Rotate array | Reverse trick | O(n) |
| Valid anagram | Char frequency array | O(n) |
| Group anagrams | Sort key + HashMap | O(n × k log k) |
| Longest palindromic substring | Expand around center | O(n²) |

---

## Interview Questions

### Q: Rotate an array by k positions. Do it in O(1) space.

**Key insight — reverse trick:**
1. Reverse entire array
2. Reverse first k elements
3. Reverse remaining n-k elements

```java
public void rotate(int[] nums, int k) {
    int n = nums.length;
    k %= n; // handle k > n
    reverse(nums, 0, n - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, n - 1);
}

private void reverse(int[] nums, int left, int right) {
    while (left < right) {
        int tmp = nums[left]; nums[left] = nums[right]; nums[right] = tmp;
        left++; right--;
    }
}
```

### Q: Product of array except self. No division allowed.

```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    
    // result[i] = product of all elements to the left
    result[0] = 1;
    for (int i = 1; i < n; i++) result[i] = result[i-1] * nums[i-1];
    
    // Multiply by product of all elements to the right
    int rightProduct = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= rightProduct;
        rightProduct *= nums[i];
    }
    return result;
}
// O(n) time, O(1) extra space (output array doesn't count)
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Two Sum (LeetCode #1)

**Problem:** Given an array of integers and a target, return indices of two numbers that add up to the target. Each input has exactly one solution.

**Brute:** Check all pairs.
```java
public int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++)
        for (int j = i + 1; j < nums.length; j++)
            if (nums[i] + nums[j] == target)
                return new int[]{i, j};
    return new int[]{};
}
// Time: O(n²)  Space: O(1)
```

**Optimized:** HashMap for O(1) complement lookup.
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement))
            return new int[]{seen.get(complement), i};
        seen.put(nums[i], i);
    }
    return new int[]{};
}
// Time: O(n)  Space: O(n)
```

---

#### 2. Valid Anagram (LeetCode #242)

**Problem:** Given two strings s and t, return true if t is an anagram of s.

**Brute:** Sort both and compare.
```java
public boolean isAnagram(String s, String t) {
    char[] a = s.toCharArray(); char[] b = t.toCharArray();
    Arrays.sort(a); Arrays.sort(b);
    return Arrays.equals(a, b);
}
// Time: O(n log n)  Space: O(n)
```

**Optimized:** Character frequency array.
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (char c : t.toCharArray()) count[c - 'a']--;
    for (int n : count) if (n != 0) return false;
    return true;
}
// Time: O(n)  Space: O(1) — fixed 26-element array
```

---

#### 3. Palindrome Check (LeetCode #125)

**Problem:** Given a string s, return true if it is a palindrome considering only alphanumeric characters and ignoring cases.

```java
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right)))
            return false;
        left++; right--;
    }
    return true;
}
// Time: O(n)  Space: O(1)
```

---

### 🟡 EASY

---

#### 4. Best Time to Buy and Sell Stock (LeetCode #121)

**Problem:** Find the maximum profit from buying and selling a stock once. You must buy before you sell.

**Brute:** Try all pairs.
```java
// Time: O(n²)  Space: O(1)
int maxProfit = 0;
for (int i = 0; i < prices.length; i++)
    for (int j = i + 1; j < prices.length; j++)
        maxProfit = Math.max(maxProfit, prices[j] - prices[i]);
```

**Optimized:** Track minimum price seen so far.
```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
// Time: O(n)  Space: O(1)
// Key insight: profit at index i = prices[i] - min(prices[0..i])
```

---

#### 5. Contains Duplicate (LeetCode #217)

**Problem:** Return true if any value appears at least twice in the array.

**Brute:** Compare all pairs → O(n²).

**Better:** Sort and check adjacent elements.
```java
// Time: O(n log n)  Space: O(1)
Arrays.sort(nums);
for (int i = 1; i < nums.length; i++)
    if (nums[i] == nums[i-1]) return true;
return false;
```

**Optimized:** HashSet.
```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int n : nums)
        if (!seen.add(n)) return true;
    return false;
}
// Time: O(n)  Space: O(n)
```

---

#### 6. Maximum Subarray (LeetCode #53) — Kadane's Algorithm

**Problem:** Find the contiguous subarray with the largest sum.

**Brute:** Try all subarrays.
```java
// Time: O(n²)  Space: O(1)
int max = Integer.MIN_VALUE;
for (int i = 0; i < nums.length; i++) {
    int sum = 0;
    for (int j = i; j < nums.length; j++) {
        sum += nums[j];
        max = Math.max(max, sum);
    }
}
```

**Optimized:** Kadane's Algorithm.
```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0], currentSum = nums[0];
    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
// Time: O(n)  Space: O(1)
// Key insight: if currentSum goes negative, restart from current element
```

---

#### 7. Move Zeroes (LeetCode #283)

**Problem:** Move all zeroes to the end of the array while maintaining the relative order of non-zero elements. Do it in-place.

```java
public void moveZeroes(int[] nums) {
    int insertPos = 0;  // pointer for next non-zero position
    for (int num : nums)
        if (num != 0) nums[insertPos++] = num;
    while (insertPos < nums.length)
        nums[insertPos++] = 0;
}
// Time: O(n)  Space: O(1)
```

---

### 🟠 MEDIUM

---

#### 8. Group Anagrams (LeetCode #49)

**Problem:** Group an array of strings such that anagrams are together.

**Approach:** Use sorted string as HashMap key.
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
// Time: O(n × k log k) where k = max string length
// Space: O(n × k)
```

**Optimized (no sort):** Use character count as key.
```java
// Key = "a2b1c0..." — 26-char frequency string — avoids O(k log k) sort
// Time: O(n × k)  — same asymptotically for small k
```

---

#### 9. Product of Array Except Self (LeetCode #238)

**Problem:** Return array where output[i] = product of all elements except nums[i]. No division allowed. O(n) time, O(1) extra space.

**Brute:** For each i, compute product of all others → O(n²).

**Optimized:** Left and right prefix products.
```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    // Left pass: result[i] = product of nums[0..i-1]
    result[0] = 1;
    for (int i = 1; i < n; i++) result[i] = result[i-1] * nums[i-1];
    // Right pass: multiply by product of nums[i+1..n-1]
    int rightProduct = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= rightProduct;
        rightProduct *= nums[i];
    }
    return result;
}
// Time: O(n)  Space: O(1) extra (output doesn't count)
```

---

#### 10. Longest Substring Without Repeating Characters (LeetCode #3)

**Problem:** Find the length of the longest substring without repeating characters.

**Brute:** Check all substrings → O(n³) or O(n²).

**Optimized:** Sliding window with last-seen index map.
```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, maxLen = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left)
            left = lastSeen.get(c) + 1;  // shrink window past duplicate
        lastSeen.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
// Time: O(n)  Space: O(min(n, charset))
```

---

#### 11. 3Sum (LeetCode #15)

**Problem:** Find all unique triplets in the array that sum to zero.

**Brute:** Three nested loops → O(n³).

**Better:** Two loops + HashSet → O(n²) time but O(n) space.

**Optimized:** Sort + two pointers.
```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;  // skip duplicate
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left+1]) left++;
                while (left < right && nums[right] == nums[right-1]) right--;
                left++; right--;
            } else if (sum < 0) left++;
            else right--;
        }
    }
    return result;
}
// Time: O(n²)  Space: O(1) extra
```

---

#### 12. Container With Most Water (LeetCode #11)

**Problem:** Find two vertical lines that form a container with the most water.

**Brute:** Try all pairs → O(n²).

**Optimized:** Two pointers — always move the shorter wall inward.
```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1, maxWater = 0;
    while (left < right) {
        maxWater = Math.max(maxWater, Math.min(height[left], height[right]) * (right - left));
        if (height[left] < height[right]) left++;
        else right--;
    }
    return maxWater;
}
// Time: O(n)  Space: O(1)
// Proof: moving the taller wall can only decrease width without increasing height → not beneficial
```

---

#### 13. Subarray Sum Equals K (LeetCode #560)

**Problem:** Count the number of subarrays that sum to k.

**Brute:** Try all subarrays → O(n²).

**Optimized:** Prefix sum + HashMap.
```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1);
    int count = 0, sum = 0;
    for (int num : nums) {
        sum += num;
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
// Time: O(n)  Space: O(n)
// Key insight: subarray [i+1..j] sums to k iff prefix[j] - prefix[i] = k
```

---

#### 14. Minimum Window Substring (LeetCode #76)

**Problem:** Find the minimum window in string s which contains all characters of string t.

```java
public String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
    int left = 0, formed = 0, required = need.size();
    int minLen = Integer.MAX_VALUE, minLeft = 0;
    Map<Character, Integer> window = new HashMap<>();
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).equals(need.get(c))) formed++;
        while (formed == required) {
            if (right - left + 1 < minLen) { minLen = right - left + 1; minLeft = left; }
            char lc = s.charAt(left++);
            window.merge(lc, -1, Integer::sum);
            if (need.containsKey(lc) && window.get(lc) < need.get(lc)) formed--;
        }
    }
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}
// Time: O(|s| + |t|)  Space: O(|t|)
```

---

### 🔴 HARD

---

#### 15. Trapping Rain Water (LeetCode #42)

**Problem:** Compute how much water is trapped after raining between elevation bars.

**Brute:** For each bar, find max left and max right → O(n²).

**Better:** Precompute leftMax[] and rightMax[] arrays → O(n) time, O(n) space.
```java
// water[i] = min(leftMax[i], rightMax[i]) - height[i]
```

**Optimized:** Two pointers — O(n) time, O(1) space.
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
// Time: O(n)  Space: O(1)
// Key insight: water at position depends on min(leftMax, rightMax). The pointer on the shorter side's max is already known.
```

---

#### 16. Longest Palindromic Substring (LeetCode #5)

**Problem:** Find the longest palindromic substring in s.

**Brute:** Check all substrings → O(n³).

**Better:** Expand-around-center — O(n²).
```java
public String longestPalindrome(String s) {
    int start = 0, maxLen = 1;
    for (int i = 0; i < s.length(); i++) {
        // Odd-length palindromes
        int len1 = expand(s, i, i);
        // Even-length palindromes
        int len2 = expand(s, i, i + 1);
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = i - (len - 1) / 2;
        }
    }
    return s.substring(start, start + maxLen);
}
private int expand(String s, int left, int right) {
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        left--; right++;
    }
    return right - left - 1;
}
// Time: O(n²)  Space: O(1)
```

**Expert (Manacher's Algorithm): O(n)**
- Uses previously computed palindromes to avoid redundant expansions.
- Rarely expected in interviews but impressive to mention.

---

#### 17. Spiral Matrix (LeetCode #54)

**Problem:** Return all elements of a matrix in spiral order.

```java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1;
    int left = 0, right = matrix[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int c = left; c <= right; c++) result.add(matrix[top][c]);
        top++;
        for (int r = top; r <= bottom; r++) result.add(matrix[r][right]);
        right--;
        if (top <= bottom) {
            for (int c = right; c >= left; c--) result.add(matrix[bottom][c]);
            bottom--;
        }
        if (left <= right) {
            for (int r = bottom; r >= top; r--) result.add(matrix[r][left]);
            left++;
        }
    }
    return result;
}
// Time: O(m×n)  Space: O(1) extra
```

---

#### 18. Sliding Window Maximum (LeetCode #239)

**Problem:** Given a window of size k sliding over array, find max in each window.

**Brute:** For each window, find max → O(n×k).

**Optimized:** Monotonic deque — O(n).
```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>();  // stores indices, front = max
    for (int i = 0; i < n; i++) {
        // Remove indices outside window
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1)
            deque.pollFirst();
        // Remove indices with smaller values (they're useless)
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i])
            deque.pollLast();
        deque.offerLast(i);
        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
// Time: O(n)  Space: O(k)
```

---

#### 19. Find All Duplicates in Array (LeetCode #442)

**Problem:** Given array where 1 ≤ a[i] ≤ n, find all elements that appear twice. O(n) time, O(1) space.

**Key insight:** Use sign of nums[abs(nums[i])-1] as a "visited" flag.
```java
public List<Integer> findDuplicates(int[] nums) {
    List<Integer> result = new ArrayList<>();
    for (int num : nums) {
        int idx = Math.abs(num) - 1;
        if (nums[idx] < 0) result.add(Math.abs(num));  // seen before
        else nums[idx] = -nums[idx];  // mark as visited
    }
    return result;
}
// Time: O(n)  Space: O(1) — uses the input array as a hash set via negation trick
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Two pointers | Left from 0, right from n-1; move based on condition |
| 2 | Sliding window | Right expands, left contracts when constraint violated |
| 3 | Prefix sum | sum(l,r) = prefix[r+1] - prefix[l] |
| 4 | Kadane's | currentSum = max(nums[i], currentSum + nums[i]) |
| 5 | String concat in loop | O(n²) — always use StringBuilder |
| 6 | Anagram check | int[26] char frequency array — O(n), O(1) space |
| 7 | Rotate array | Reverse trick: reverse all → reverse k → reverse rest |
| 8 | Product except self | Left prefix pass + right product pass — O(n) O(1) |
| 9 | HashMap for two-sum | complement = target - num; check if in map |
| 10 | Sliding window type | Fixed size: slide together; Variable: left contracts |
| 11 | Subarray sum = K | Prefix sum map; look for (currentSum - k) |
| 12 | `s.substring(i,j)` | O(n) in Java — creates new String |
| 13 | Palindrome check | Two-pointer from both ends; ignore non-alphanumeric |
| 14 | Group anagrams | Sort each word → use as HashMap key |
| 15 | 3Sum complexity | O(n²) — sort + outer loop + inner two-pointer |

---

*Next: [Linked Lists](03-linked-lists.md)*
