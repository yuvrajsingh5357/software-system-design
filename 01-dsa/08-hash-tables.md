# Hash Tables

This chapter covers hash tables — O(1) average-case lookups make them the most useful data structure in competitive programming and system design. Deep understanding of internals is frequently tested.

## Table of Contents
- [Hash Table Internals](#hash-table-internals)
- [Java HashMap Deep Dive](#java-hashmap-deep-dive)
- [Common HashMap Patterns](#common-hashmap-patterns)
- [HashSet for Lookups](#hashset-for-lookups)
- [LinkedHashMap for Ordering](#linkedhashmap-for-ordering)
- [Custom hashCode and equals](#custom-hashcode-and-equals)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Hash Table Internals

A hash table maps keys to values using a hash function.

```
Key → hashCode() → compress to bucket index → store in bucket

Example: key = "apple"
hashCode("apple") = 92599395
index = 92599395 % 16 (capacity) = 3
Store in bucket 3
```

**Collision handling (two main approaches):**

| Method | How | Java uses? |
|--------|-----|-----------|
| **Chaining** | Each bucket holds a linked list | Yes (HashMap) |
| **Open Addressing** | Find next empty slot | No (in standard Java) |

**Java HashMap collision resolution:**
1. Bucket = linked list (chaining)
2. When chain length > 8 AND table size ≥ 64: **convert to Red-Black Tree** (Java 8+)
3. When chain length drops below 6: revert to linked list

---

## Java HashMap Deep Dive

```java
// Constructors
Map<String, Integer> map = new HashMap<>();                  // default capacity 16, load factor 0.75
Map<String, Integer> map = new HashMap<>(32);               // initial capacity 32
Map<String, Integer> map = new HashMap<>(32, 0.5f);        // capacity 32, load factor 0.5

// Essential operations
map.put("key", 1);              // O(1) avg — insert or update
map.get("key");                 // O(1) avg — null if not found
map.getOrDefault("key", 0);    // O(1) avg — return default if missing
map.containsKey("key");         // O(1) avg
map.containsValue(1);           // O(n) — scans all values
map.remove("key");              // O(1) avg
map.size();                     // O(1)
map.isEmpty();                  // O(1)

// Iterating
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    int val = entry.getValue();
}
for (String key : map.keySet()) { }
for (int val : map.values()) { }

// Useful Java 8+ operations
map.putIfAbsent("key", 1);                          // only put if key missing
map.merge("key", 1, Integer::sum);                  // merge with BiFunction
map.computeIfAbsent("key", k -> new ArrayList<>()); // create if missing
map.getOrDefault("key", Collections.emptyList());   // safe default

// Frequency counting pattern
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) {
    freq.merge(c, 1, Integer::sum);  // cleaner than put(c, getOrDefault(c,0)+1)
}
```

**Resizing:**
- When `size > capacity × loadFactor`: capacity doubles, all entries rehashed
- O(n) for resize, but amortized O(1) per put
- **Interview tip:** Pass initial capacity to avoid multiple resizes: `new HashMap<>(initialSize * 4 / 3 + 1)`

---

## Common HashMap Patterns

### Pattern 1: Two Sum — Store seen values

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value → index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    return new int[]{};
}
// O(n) time, O(n) space
```

### Pattern 2: Group by key

```java
// Group anagrams
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars); // sorted form = canonical key
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
// O(n × k log k) where k = max string length
```

### Pattern 3: Count then query

```java
// Subarray sum equals K — prefix sum + count
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1); // empty prefix
    int count = 0, prefixSum = 0;
    
    for (int n : nums) {
        prefixSum += n;
        // If prefixSum - k exists as a prefix, those subarrays sum to k
        count += prefixCount.getOrDefault(prefixSum - k, 0);
        prefixCount.merge(prefixSum, 1, Integer::sum);
    }
    return count;
}
```

### Pattern 4: HashMap as adjacency structure

```java
// Clone graph
Map<Node, Node> cloned = new HashMap<>(); // original → clone

public Node cloneGraph(Node node) {
    if (node == null) return null;
    if (cloned.containsKey(node)) return cloned.get(node);
    
    Node clone = new Node(node.val);
    cloned.put(node, clone);
    for (Node neighbor : node.neighbors) {
        clone.neighbors.add(cloneGraph(neighbor));
    }
    return clone;
}
```

### Pattern 5: Sliding window with HashMap

```java
// Longest substring with at most K distinct characters
public int lengthOfLongestSubstringKDistinct(String s, int k) {
    Map<Character, Integer> window = new HashMap<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        window.merge(s.charAt(right), 1, Integer::sum);
        
        while (window.size() > k) { // shrink until valid
            char lc = s.charAt(left++);
            window.merge(lc, -1, Integer::sum);
            if (window.get(lc) == 0) window.remove(lc);
        }
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

## HashSet for Lookups

```java
Set<Integer> set = new HashSet<>();
set.add(1);            // O(1) avg
set.contains(1);       // O(1) avg — this is the main use
set.remove(1);         // O(1) avg
set.size();            // O(1)

// Intersection of two arrays
Set<Integer> setA = new HashSet<>(Arrays.asList(arrayA));
Set<Integer> result = new HashSet<>();
for (int n : arrayB) if (setA.contains(n)) result.add(n);

// Detect cycle using HashSet (seen nodes in path)
Set<Integer> path = new HashSet<>();

// Longest consecutive sequence — O(n) using HashSet
public int longestConsecutive(int[] nums) {
    Set<Integer> numSet = new HashSet<>();
    for (int n : nums) numSet.add(n);
    int longest = 0;
    
    for (int n : numSet) {
        // Only start counting from the beginning of a sequence
        if (!numSet.contains(n - 1)) {
            int length = 1;
            while (numSet.contains(n + length)) length++;
            longest = Math.max(longest, length);
        }
    }
    return longest;
}
```

---

## LinkedHashMap for Ordering

**LinkedHashMap** maintains insertion order OR access order (for LRU).

```java
// Insertion order (default)
Map<String, Integer> linked = new LinkedHashMap<>();
linked.put("c", 3); linked.put("a", 1); linked.put("b", 2);
linked.keySet() // [c, a, b] — insertion order preserved

// Access order (for LRU cache)
Map<Integer, Integer> lru = new LinkedHashMap<>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity; // auto-evict when over capacity
    }
};
// Now get() updates position — LRU eviction is automatic!
```

**LRU Cache with LinkedHashMap:**
```java
class LRUCache {
    private final int capacity;
    private final Map<Integer, Integer> cache;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        // accessOrder=true: get() moves to end → end is most recent
        this.cache = new LinkedHashMap<>(16, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> e) {
                return size() > capacity;
            }
        };
    }
    
    public int get(int key) { return cache.getOrDefault(key, -1); }
    public void put(int key, int value) { cache.put(key, value); }
}
```

---

## Custom hashCode and equals

When using custom objects as HashMap keys, you **must override both hashCode and equals**.

```java
class Point {
    int x, y;
    
    Point(int x, int y) { this.x = x; this.y = y; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(x, y); // uses prime-multiplied combination
        // Equivalent: 31 * x + y
    }
}

// Now works correctly as HashMap key
Map<Point, String> map = new HashMap<>();
map.put(new Point(1, 2), "origin");
map.get(new Point(1, 2)); // returns "origin" — equals + hashCode work correctly
```

**The contract:**
1. If `a.equals(b)` → `a.hashCode() == b.hashCode()` (MUST)
2. If `a.hashCode() == b.hashCode()` → `a.equals(b)` (NOT necessarily — collisions ok)
3. Violation causes HashMap to lose objects entirely!

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Two Sum (LeetCode #1)

**Problem:** Return indices of two numbers that add up to target.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) return new int[]{seen.get(complement), i};
        seen.put(nums[i], i);
    }
    return new int[]{};
}
// Time: O(n)  Space: O(n)
```

---

#### 2. Valid Anagram (LeetCode #242)

**Problem:** Check if two strings are anagrams of each other.

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (char c : t.toCharArray()) if (--count[c - 'a'] < 0) return false;
    return true;
}
// Time: O(n)  Space: O(1)
```

---

#### 3. First Unique Character in String (LeetCode #387)

**Problem:** Find the index of the first non-repeating character in a string.

```java
public int firstUniqChar(String s) {
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (int i = 0; i < s.length(); i++)
        if (count[s.charAt(i) - 'a'] == 1) return i;
    return -1;
}
// Time: O(n)  Space: O(1)
```

---

### 🟡 EASY

---

#### 4. Contains Duplicate II (LeetCode #219)

**Problem:** Check if there are two distinct indices i, j such that nums[i] == nums[j] and |i-j| <= k.

```java
public boolean containsNearbyDuplicate(int[] nums, int k) {
    Map<Integer, Integer> lastSeen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        if (lastSeen.containsKey(nums[i]) && i - lastSeen.get(nums[i]) <= k) return true;
        lastSeen.put(nums[i], i);
    }
    return false;
}
// Time: O(n)  Space: O(min(n, k))
```

---

#### 5. Longest Consecutive Sequence (LeetCode #128)

**Problem:** Find the length of the longest consecutive sequence in an unsorted array. O(n) required.

**Brute:** Sort → scan → O(n log n).

**Optimized:** HashSet — only start counting from sequence starts.
```java
public int longestConsecutive(int[] nums) {
    Set<Integer> numSet = new HashSet<>();
    for (int n : nums) numSet.add(n);
    int longest = 0;
    for (int n : numSet) {
        if (!numSet.contains(n - 1)) {  // start of a sequence
            int length = 1;
            while (numSet.contains(n + length)) length++;
            longest = Math.max(longest, length);
        }
    }
    return longest;
}
// Time: O(n)  Space: O(n)
// Key: only count from sequence start — avoids re-counting
```

---

#### 6. Group Anagrams (LeetCode #49)

**Problem:** Group strings that are anagrams of each other.

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        groups.computeIfAbsent(new String(chars), k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
// Time: O(n × k log k)  Space: O(n × k)
```

---

### 🟠 MEDIUM

---

#### 7. LRU Cache (LeetCode #146)

**Problem:** Design a data structure that implements Least Recently Used eviction. O(1) get and put.

**Using LinkedHashMap (Java shortcut):**
```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int capacity;
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }
    public int get(int key) { return super.getOrDefault(key, -1); }
    public void put(int key, int value) { super.put(key, value); }
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) { return size() > capacity; }
}
// All ops: O(1)  Space: O(capacity)
```

**From scratch (HashMap + Doubly Linked List):**
```java
// Maintain HashMap<key, node> and DLL (head=LRU, tail=MRU)
// get: O(1) — look up node, move to tail
// put: O(1) — if exists: update + move to tail; if new: add to tail; if full: remove head
// Time: O(1) all ops  Space: O(capacity)
```

---

#### 8. Subarray Sum Equals K (LeetCode #560)

**Problem:** Count subarrays that sum to k.

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
```

---

#### 9. Four Sum II (LeetCode #454)

**Problem:** Count tuples (i,j,k,l) such that nums1[i]+nums2[j]+nums3[k]+nums4[l] == 0.

```java
public int fourSumCount(int[] a, int[] b, int[] c, int[] d) {
    Map<Integer, Integer> sumCount = new HashMap<>();
    for (int x : a) for (int y : b) sumCount.merge(x + y, 1, Integer::sum);
    int count = 0;
    for (int x : c) for (int y : d) count += sumCount.getOrDefault(-(x + y), 0);
    return count;
}
// Time: O(n²)  Space: O(n²)
// Key: split into two pairs — hash sums of first two, scan sums of last two
```

---

### 🔴 HARD

---

#### 10. Isomorphic Strings (LeetCode #205)

**Problem:** Determine if two strings s and t are isomorphic (characters can be consistently mapped).

```java
public boolean isIsomorphic(String s, String t) {
    Map<Character, Character> sToT = new HashMap<>(), tToS = new HashMap<>();
    for (int i = 0; i < s.length(); i++) {
        char sc = s.charAt(i), tc = t.charAt(i);
        if (sToT.containsKey(sc) && sToT.get(sc) != tc) return false;
        if (tToS.containsKey(tc) && tToS.get(tc) != sc) return false;
        sToT.put(sc, tc); tToS.put(tc, sc);
    }
    return true;
}
// Time: O(n)  Space: O(1) — at most 256 chars
```

---

#### 11. Randomized Set (LeetCode #380) — Insert Delete GetRandom O(1)

**Problem:** Design a data structure with O(1) average for insert, remove, and getRandom.

```java
class RandomizedSet {
    Map<Integer, Integer> indexMap = new HashMap<>();  // value → index in list
    List<Integer> list = new ArrayList<>();
    Random rand = new Random();

    public boolean insert(int val) {
        if (indexMap.containsKey(val)) return false;
        indexMap.put(val, list.size());
        list.add(val);
        return true;
    }

    public boolean remove(int val) {
        if (!indexMap.containsKey(val)) return false;
        int idx = indexMap.get(val);
        int last = list.get(list.size() - 1);
        list.set(idx, last);
        indexMap.put(last, idx);
        list.remove(list.size() - 1);
        indexMap.remove(val);
        return true;
    }

    public int getRandom() { return list.get(rand.nextInt(list.size())); }
}
// All ops: O(1) average  Space: O(n)
// Key trick: to remove in O(1), swap with last element then remove last
```

---

## Classic Problems

| Problem | Technique | Complexity |
|---------|-----------|-----------|
| Two Sum | HashMap: complement lookup | O(n) |
| Group Anagrams | HashMap: sorted key | O(n × k log k) |
| Top K Frequent | HashMap + Heap | O(n log k) |
| Longest consecutive | HashSet: start of sequences | O(n) |
| Subarray sum = K | HashMap: prefix counts | O(n) |
| Longest substr K distinct | HashMap sliding window | O(n) |
| Clone graph | HashMap: original→clone | O(V + E) |
| First unique char | int[26] char count | O(n) |
| LRU Cache | LinkedHashMap OR HashMap + DLL | O(1) all ops |
| Isomorphic strings | Two HashMaps (both directions) | O(n) |

---

## Interview Questions

### Q: Why must you override hashCode when you override equals?

If two objects are equal (`a.equals(b) == true`), they MUST have the same hashCode. If they don't:
- Both would hash to different buckets
- `map.get(b)` would fail to find the entry put with `map.put(a, value)` because it looks in the wrong bucket
- The HashMap contract is violated, and you'd lose data

### Q: How does HashMap handle a null key?

Java's HashMap allows **one null key**. Null keys always hash to bucket 0. `map.put(null, value)` and `map.get(null)` work correctly.

HashTable and ConcurrentHashMap do NOT allow null keys.

### Q: When is HashMap O(n) instead of O(1)?

1. **Before Java 8:** With heavy collisions (same hashCode), bucket becomes a linked list of length n → O(n) lookup
2. **After Java 8:** Treeification kicks in at 8 collisions → O(log n) worst case
3. **During resize (rehash):** The resize operation itself is O(n), but amortized O(1) per put

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | HashMap get/put average | O(1) |
| 2 | HashMap worst case | O(log n) — Java 8 treeified bucket |
| 3 | Java 8 treeification | After 8 collisions in one bucket → Red-Black Tree |
| 4 | containsValue time | O(n) — must scan all values |
| 5 | Load factor default | 0.75 — resize at 75% full |
| 6 | Null key support | HashMap: yes (1 null key); ConcurrentHashMap: no |
| 7 | merge() usage | `map.merge(key, 1, Integer::sum)` for frequency counts |
| 8 | computeIfAbsent | Creates value if key missing — avoids two-step put |
| 9 | LinkedHashMap | Maintains insertion or access order |
| 10 | LRU in one class | LinkedHashMap with accessOrder=true + removeEldestEntry |
| 11 | Custom key contract | Override BOTH equals() AND hashCode() |
| 12 | hashCode contract | a.equals(b) → must have same hashCode |
| 13 | Group anagram key | Sort string → use as canonical key |
| 14 | Two Sum O(n) | HashMap: store seen values; check complement |
| 15 | Longest consecutive O(n) | HashSet; only count from sequence start |

---

*Next: [Sorting and Searching](09-sorting-and-searching.md)*
