# Sorting and Searching

This chapter covers sorting algorithms — their mechanics, complexities, and when to use each — plus advanced searching techniques including binary search variants and external sorting.

## Table of Contents
- [Sorting Algorithms](#sorting-algorithms)
- [Java's Sorting APIs](#javas-sorting-apis)
- [Binary Search Deep Dive](#binary-search-deep-dive)
- [Search in Rotated/Modified Arrays](#search-in-rotatedmodified-arrays)
- [External Sorting](#external-sorting)
- [Custom Sort Problems](#custom-sort-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Sorting Algorithms

### Comparison-Based Sorts

| Algorithm | Best | Average | Worst | Space | Stable? | Use when |
|-----------|------|---------|-------|-------|---------|----------|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes | Never in production |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | No | Learning only |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes | Nearly-sorted arrays, small n |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Linked lists, stable needed |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | No | General purpose (fastest avg) |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No | When O(1) space mandatory |
| **Tim Sort** | O(n) | O(n log n) | O(n log n) | O(n) | Yes | Java Arrays.sort (default) |

### Non-Comparison Sorts (bypass O(n log n) lower bound)

| Algorithm | Complexity | Space | Use when |
|-----------|-----------|-------|----------|
| **Counting Sort** | O(n + k) | O(k) | Small, bounded integer keys |
| **Radix Sort** | O(d × (n + k)) | O(n + k) | Fixed-length integers/strings |
| **Bucket Sort** | O(n + k) avg | O(n) | Uniformly distributed floats |

---

### Merge Sort — Deep Dive

```java
public void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return; // base case: 0 or 1 element
    
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);       // sort left half
    mergeSort(arr, mid + 1, right);  // sort right half
    merge(arr, left, mid, right);    // merge
}

private void merge(int[] arr, int left, int mid, int right) {
    int[] tmp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) tmp[k++] = arr[i++]; // <= for stability
        else                  tmp[k++] = arr[j++];
    }
    while (i <= mid)  tmp[k++] = arr[i++];
    while (j <= right) tmp[k++] = arr[j++];
    
    System.arraycopy(tmp, 0, arr, left, tmp.length);
}
// T(n) = 2T(n/2) + O(n) → O(n log n) by Master Theorem
// Space: O(n) for temp array + O(log n) stack = O(n)
```

**Why merge sort for linked lists?**
- Quick sort needs random access for partitioning (slow on linked lists)
- Merge sort works naturally with sequential access → O(n log n) on linked lists

---

### Quick Sort — Deep Dive

```java
public void quickSort(int[] arr, int left, int right) {
    if (left >= right) return;
    int pivotIdx = partition(arr, left, right);
    quickSort(arr, left, pivotIdx - 1);
    quickSort(arr, pivotIdx + 1, right);
}

// Lomuto partition (simple, commonly taught)
private int partition(int[] arr, int left, int right) {
    int pivot = arr[right]; // choose last element as pivot
    int i = left - 1; // boundary of smaller elements
    
    for (int j = left; j < right; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    swap(arr, i + 1, right); // place pivot in correct position
    return i + 1;
}

// 3-way partition (handles duplicates efficiently — Dutch National Flag)
public void quickSort3Way(int[] arr, int left, int right) {
    if (left >= right) return;
    int lt = left, gt = right, i = left;
    int pivot = arr[left];
    
    while (i <= gt) {
        if      (arr[i] < pivot) swap(arr, lt++, i++);
        else if (arr[i] > pivot) swap(arr, i, gt--);
        else                     i++;
    }
    // arr[left..lt-1] < pivot, arr[lt..gt] == pivot, arr[gt+1..right] > pivot
    quickSort3Way(arr, left, lt - 1);
    quickSort3Way(arr, gt + 1, right);
}
private void swap(int[] a, int i, int j) { int t=a[i]; a[i]=a[j]; a[j]=t; }
```

**Quick sort's O(n²) worst case:** When pivot is always the max/min (already sorted + last-element pivot). Fix: **random pivot** or **median-of-three pivot**.

**Java's Arrays.sort:** Uses Dual-Pivot Quicksort for primitives (O(n log n) avg), TimSort for objects (stable).

---

### Counting Sort

```java
public int[] countingSort(int[] nums, int max) {
    int[] count = new int[max + 1];
    for (int n : nums) count[n]++;
    
    int[] result = new int[nums.length];
    int idx = 0;
    for (int i = 0; i <= max; i++) {
        while (count[i]-- > 0) result[idx++] = i;
    }
    return result;
}
// O(n + k) where k = max value
// Only for non-negative integers with bounded max value
```

---

## Java's Sorting APIs

```java
// Primitive arrays — Dual-Pivot Quicksort O(n log n) — NOT stable
int[] arr = {3, 1, 4, 1, 5};
Arrays.sort(arr);                           // full sort
Arrays.sort(arr, 2, 5);                     // sort range [2, 5)

// Object arrays — TimSort O(n log n) — STABLE
Integer[] arr = {3, 1, 4, 1, 5};
Arrays.sort(arr);                           // natural order
Arrays.sort(arr, Comparator.reverseOrder()); // reverse
Arrays.sort(arr, (a, b) -> a - b);          // custom comparator

// List — TimSort O(n log n) — STABLE
List<Integer> list = Arrays.asList(3, 1, 4);
Collections.sort(list);
list.sort(Comparator.reverseOrder());

// Custom object sort — multiple criteria
List<int[]> intervals = Arrays.asList(new int[][]{{1,3},{2,6},{8,10}});
intervals.sort((a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);

// String sort — multiple common patterns
String[] words = {"banana", "apple", "cherry"};
Arrays.sort(words);                                    // alphabetical
Arrays.sort(words, Comparator.comparingInt(String::length)); // by length
Arrays.sort(words, (a, b) -> b.compareTo(a));          // reverse alpha
```

---

## Binary Search Deep Dive

**The three templates — know all three:**

```java
// Template 1: Standard — finds exact target, -1 if not found
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// Template 2: Lower bound — first position where nums[pos] >= target
// (same as Java's Arrays.binarySearch for insertion point)
public int lowerBound(int[] nums, int target) {
    int left = 0, right = nums.length; // right = nums.length (one past end)
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) left = mid + 1;
        else right = mid;
    }
    return left; // first index where nums[left] >= target
}

// Template 3: Upper bound — first position where nums[pos] > target
public int upperBound(int[] nums, int target) {
    int left = 0, right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] <= target) left = mid + 1;
        else right = mid;
    }
    return left; // first index where nums[left] > target
}

// Count occurrences of target: upperBound(target) - lowerBound(target)
```

### Binary Search on Answer

A powerful pattern: when the answer has a monotonic property (e.g., "minimum K such that condition(K) is true"), binary search on K itself.

```java
// Minimum days to make M bouquets of K consecutive bloomed flowers
public int minDays(int[] bloomDay, int m, int k) {
    if ((long)m * k > bloomDay.length) return -1;
    int left = 1, right = Arrays.stream(bloomDay).max().getAsInt();
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canMake(bloomDay, m, k, mid)) right = mid;
        else left = mid + 1;
    }
    return left;
}

private boolean canMake(int[] bloomDay, int m, int k, int day) {
    int bouquets = 0, consecutive = 0;
    for (int d : bloomDay) {
        if (d <= day) { if (++consecutive == k) { bouquets++; consecutive = 0; } }
        else consecutive = 0;
    }
    return bouquets >= m;
}
```

---

## Search in Rotated/Modified Arrays

### Search in Rotated Sorted Array

```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        
        // Determine which half is sorted
        if (nums[left] <= nums[mid]) { // left half is sorted
            if (target >= nums[left] && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else { // right half is sorted
            if (target > nums[mid] && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
```

### Find Minimum in Rotated Sorted Array

```java
public int findMin(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) left = mid + 1; // min is in right half
        else right = mid;                              // min is in left half (including mid)
    }
    return nums[left];
}
```

### Search in 2D Matrix

```java
// Matrix is sorted row by row and first element of each row > last of previous
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int left = 0, right = m * n - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int val = matrix[mid / n][mid % n]; // convert 1D index to 2D
        if (val == target) return true;
        else if (val < target) left = mid + 1;
        else right = mid - 1;
    }
    return false;
}
```

---

## External Sorting

**Used when data is too large to fit in memory.**

**Merge Sort approach:**
1. **Divide:** Read chunks that fit in RAM → sort each chunk → write sorted chunks to disk (called "runs")
2. **Merge:** K-way merge all sorted runs using a min-heap of size K

```
Disk: [1GB unsorted]
RAM: 100MB

Step 1: Read 100MB → sort in RAM → write sorted run to disk × 10 runs
Step 2: Open all 10 runs → K-way merge with 10-element heap → write sorted output
```

**This is how databases sort during ORDER BY when result doesn't fit in sort_buffer.**

---

## Custom Sort Problems

### Sort Intervals by Start Time

```java
int[][] intervals = {{1,3},{2,6},{8,10},{15,18}};
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
// Merge overlapping:
List<int[]> merged = new ArrayList<>();
for (int[] interval : intervals) {
    if (!merged.isEmpty() && merged.get(merged.size()-1)[1] >= interval[0]) {
        merged.get(merged.size()-1)[1] = Math.max(merged.get(merged.size()-1)[1], interval[1]);
    } else {
        merged.add(interval);
    }
}
```

### Custom Sort String (sort by frequency)

```java
public String frequencySort(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
    
    return freq.entrySet().stream()
        .sorted((a, b) -> b.getValue() - a.getValue()) // descending frequency
        .map(e -> String.valueOf(e.getKey()).repeat(e.getValue()))
        .collect(Collectors.joining());
}
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Merge Sorted Array (LeetCode #88)

**Problem:** Merge nums2 into nums1, both sorted. nums1 has extra space at end.

**Key trick:** Fill from the end to avoid overwriting.
```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    int i = m - 1, j = n - 1, k = m + n - 1;
    while (i >= 0 && j >= 0)
        nums1[k--] = (nums1[i] >= nums2[j]) ? nums1[i--] : nums2[j--];
    while (j >= 0) nums1[k--] = nums2[j--];
}
// Time: O(m + n)  Space: O(1)
```

---

#### 2. Sort Colors (LeetCode #75) — Dutch National Flag

**Problem:** Sort array containing only 0s, 1s, and 2s in-place in one pass.

```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        if (nums[mid] == 0) swap(nums, low++, mid++);
        else if (nums[mid] == 1) mid++;
        else swap(nums, mid, high--);
    }
}
// Time: O(n)  Space: O(1)
// Three-way partition: low..mid-1 = 0s, mid..high = 1s, high+1..n = 2s
```

---

### 🟡 EASY

---

#### 3. Search a 2D Matrix (LeetCode #74)

**Problem:** Search for a target in an m×n matrix where each row is sorted and first element of each row is greater than last of previous.

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int left = 0, right = m * n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int val = matrix[mid / n][mid % n];
        if (val == target) return true;
        else if (val < target) left = mid + 1;
        else right = mid - 1;
    }
    return false;
}
// Time: O(log(m×n))  Space: O(1)
```

---

#### 4. Find Minimum in Rotated Sorted Array (LeetCode #153)

**Problem:** Find the minimum element in a rotated sorted array.

```java
public int findMin(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) left = mid + 1;  // min is in right half
        else right = mid;                               // min is in left half (including mid)
    }
    return nums[left];
}
// Time: O(log n)  Space: O(1)
```

---

#### 5. Search in Rotated Sorted Array (LeetCode #33)

**Problem:** Search for target in a rotated sorted array.

```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        if (nums[left] <= nums[mid]) {  // left half is sorted
            if (target >= nums[left] && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else {                         // right half is sorted
            if (target > nums[mid] && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
// Time: O(log n)  Space: O(1)
```

---

### 🟠 MEDIUM

---

#### 6. Koko Eating Bananas (LeetCode #875) — Binary Search on Answer

**Problem:** Koko can eat k bananas per hour. Find minimum k such that she can eat all piles within h hours.

```java
public int minEatingSpeed(int[] piles, int h) {
    int left = 1, right = Arrays.stream(piles).max().getAsInt();
    while (left < right) {
        int mid = left + (right - left) / 2;
        int hours = 0;
        for (int p : piles) hours += (p + mid - 1) / mid;  // ceil division
        if (hours <= h) right = mid;  // can finish — try less
        else left = mid + 1;
    }
    return left;
}
// Time: O(n log(max_pile))  Space: O(1)
// Classic binary search on answer — condition is monotonic
```

---

#### 7. Merge Intervals (LeetCode #56)

**Problem:** Merge all overlapping intervals.

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> merged = new ArrayList<>();
    for (int[] interval : intervals) {
        if (!merged.isEmpty() && merged.get(merged.size()-1)[1] >= interval[0])
            merged.get(merged.size()-1)[1] = Math.max(merged.get(merged.size()-1)[1], interval[1]);
        else merged.add(interval);
    }
    return merged.toArray(new int[0][]);
}
// Time: O(n log n)  Space: O(n)
```

---

#### 8. Find First and Last Position (LeetCode #34) — Two Binary Searches

**Problem:** Find the starting and ending position of a given target in a sorted array.

```java
public int[] searchRange(int[] nums, int target) {
    return new int[]{lowerBound(nums, target), upperBound(nums, target)};
}
private int lowerBound(int[] nums, int target) {
    int lo = 0, hi = nums.length;
    while (lo < hi) { int mid = lo + (hi - lo) / 2; if (nums[mid] < target) lo = mid+1; else hi = mid; }
    return (lo < nums.length && nums[lo] == target) ? lo : -1;
}
private int upperBound(int[] nums, int target) {
    int lo = 0, hi = nums.length;
    while (lo < hi) { int mid = lo + (hi - lo) / 2; if (nums[mid] <= target) lo = mid+1; else hi = mid; }
    return (lo > 0 && nums[lo-1] == target) ? lo-1 : -1;
}
// Time: O(log n)  Space: O(1)
```

---

### 🔴 HARD

---

#### 9. Median of Two Sorted Arrays (LeetCode #4)

**Problem:** Find the median of two sorted arrays of total size m+n. O(log(min(m,n))) required.

**Brute:** Merge and find median → O(m+n).

**Optimized:** Binary search on the partition point.
```java
public double findMedianSortedArrays(int[] A, int[] B) {
    if (A.length > B.length) return findMedianSortedArrays(B, A);
    int m = A.length, n = B.length;
    int lo = 0, hi = m;
    while (lo <= hi) {
        int i = (lo + hi) / 2, j = (m + n + 1) / 2 - i;
        int aLeft = (i == 0) ? Integer.MIN_VALUE : A[i-1];
        int aRight = (i == m) ? Integer.MAX_VALUE : A[i];
        int bLeft = (j == 0) ? Integer.MIN_VALUE : B[j-1];
        int bRight = (j == n) ? Integer.MAX_VALUE : B[j];
        if (aLeft <= bRight && bLeft <= aRight) {
            if ((m + n) % 2 == 1) return Math.max(aLeft, bLeft);
            return (Math.max(aLeft, bLeft) + Math.min(aRight, bRight)) / 2.0;
        } else if (aLeft > bRight) hi = i - 1;
        else lo = i + 1;
    }
    return 0;
}
// Time: O(log(min(m, n)))  Space: O(1)
// Key: partition both arrays so left half has (m+n)/2 elements with all lefts ≤ all rights
```

---

#### 10. Count of Smaller Numbers After Self (LeetCode #315)

**Problem:** Return a count array where count[i] = number of elements to the right of nums[i] that are smaller.

**Brute:** O(n²) nested loops.

**Optimized:** Modified merge sort — count inversions during merge.
```java
// During merge sort, when we move an element from right to left position,
// it means all remaining left elements are greater → count those.
// Time: O(n log n)  Space: O(n)
```

---

#### 11. Capacity to Ship Packages Within D Days (LeetCode #1011) — Binary Search on Answer

**Problem:** Find the minimum ship capacity to ship all packages within D days.

```java
public int shipWithinDays(int[] weights, int days) {
    int left = Arrays.stream(weights).max().getAsInt();  // min capacity = heaviest item
    int right = Arrays.stream(weights).sum();             // max capacity = all in one day
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canShip(weights, days, mid)) right = mid;
        else left = mid + 1;
    }
    return left;
}
private boolean canShip(int[] w, int days, int capacity) {
    int d = 1, load = 0;
    for (int weight : w) {
        if (load + weight > capacity) { d++; load = 0; }
        load += weight;
    }
    return d <= days;
}
// Time: O(n log(sum))  Space: O(1)
```

---

## Interview Questions

### Q: Why is merge sort preferred over quicksort for linked lists?

Quicksort requires random access for efficient pivot selection and partitioning. On a linked list, getting the middle element or swapping elements requires O(n) traversal. Merge sort naturally splits from the middle using fast/slow pointers and merges in O(n) — it's well-suited to sequential access.

### Q: What makes TimSort fast in practice?

TimSort:
1. Detects existing "runs" (already sorted subsequences) in the data
2. Merges runs using merge sort
3. Uses insertion sort for small arrays (< 64 elements) — faster for nearly-sorted small data
4. Best case O(n) for already-sorted data

This is why `Arrays.sort` for objects in Java is often faster in practice than the theoretical O(n log n) suggests.

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Merge sort stability | Stable — equal elements preserve order |
| 2 | Quicksort stability | Not stable |
| 3 | Quicksort worst case | O(n²) — already sorted + last pivot |
| 4 | Quicksort fix | Random pivot or median-of-three |
| 5 | Java Arrays.sort (primitives) | Dual-Pivot Quicksort — NOT stable |
| 6 | Java Arrays.sort (objects) | TimSort — STABLE |
| 7 | Counting sort when | Non-negative integers, bounded max (k small) |
| 8 | Binary search overflow | mid = left + (right - left) / 2 |
| 9 | Lower bound | First position ≥ target; use `right = nums.length` |
| 10 | Binary search on answer | When condition is monotonic — binary search on K |
| 11 | Rotated array search | Check which half is sorted; binary search that half |
| 12 | External sort | Divide into runs → sort in RAM → K-way merge |
| 13 | Merge sort linked lists | Use fast/slow pointer for middle; merge is O(n) |
| 14 | Heap sort space | O(1) — sorts in place |
| 15 | 2D matrix binary search | `matrix[mid/n][mid%n]` to treat as 1D |

---

*Next: [Dynamic Programming](10-dynamic-programming.md)*
