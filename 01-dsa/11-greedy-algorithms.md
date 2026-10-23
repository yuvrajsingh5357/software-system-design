# Greedy Algorithms

This chapter covers greedy algorithms — elegant, efficient solutions that make locally optimal choices at each step. Knowing when greedy works (and when it doesn't) is a key differentiator.

## Table of Contents
- [What Makes an Algorithm Greedy?](#what-makes-an-algorithm-greedy)
- [Greedy vs DP](#greedy-vs-dp)
- [Interval Scheduling Greedy Problems](#interval-scheduling-greedy-problems)
- [Task and Resource Scheduling](#task-and-resource-scheduling)
- [Classic Greedy Problems](#classic-greedy-problems)
- [When Greedy Fails](#when-greedy-fails)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## What Makes an Algorithm Greedy?

A greedy algorithm makes the **locally optimal choice at each step** with the hope (and proof) that it leads to the **globally optimal solution**.

**Greedy works when:**
1. **Greedy Choice Property:** A globally optimal solution can be reached by making locally optimal choices
2. **Optimal Substructure:** After making a greedy choice, the remaining subproblem is optimal

**Proof approaches:**
- **Exchange argument:** Show that swapping a greedy choice for any other choice doesn't improve the solution
- **Induction:** Greedy choice is safe at step 1; assume safe up to step k; prove safe at step k+1

---

## Greedy vs DP

| | Greedy | DP |
|-|--------|----|
| Decision | One optimal choice at each step | Considers all choices, remembers best |
| Recalculation | No — move forward only | No — memoize |
| Correctness | Only for certain problem structures | Works whenever optimal substructure + overlapping subproblems |
| Time complexity | Usually O(n log n) or O(n) | Usually O(n²) or worse |
| **Hallmark question** | "Does the locally optimal lead to globally optimal?" | "Do I need to revisit choices?" |

**Coin change example of when greedy fails:**
```
Coins: [1, 3, 4], Amount: 6
Greedy: 4 + 1 + 1 = 3 coins
DP optimal: 3 + 3 = 2 coins  ← greedy gives wrong answer
```
Greedy works for standard coin systems (1, 5, 10, 25) but not arbitrary denominations.

---

## Interval Scheduling Greedy Problems

### Activity Selection — Maximum Non-Overlapping Intervals

**Greedy choice:** Always pick the interval with the **earliest end time**.

**Proof (exchange argument):** If we pick interval A instead of B (earlier-ending), interval A ends no later than B, so everything that follows B also follows A → A is at least as good.

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length == 0) return 0;
    // Sort by end time
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]);
    
    int count = 0; // intervals to remove
    int lastEnd = intervals[0][1];
    
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < lastEnd) {
            // Overlap — remove the current interval (it has later end time due to sort)
            count++;
        } else {
            // No overlap — keep it, update lastEnd
            lastEnd = intervals[i][1];
        }
    }
    return count;
}
// O(n log n) for sort, O(n) for scan
```

### Meeting Rooms — Can One Person Attend All?

```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i-1][1]) return false; // overlap
    }
    return true;
}
```

### Meeting Rooms II — Minimum Conference Rooms

**Greedy:** Use a min-heap tracking end times of ongoing meetings.

```java
public int minMeetingRooms(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
    PriorityQueue<Integer> endTimes = new PriorityQueue<>(); // min-heap of end times
    
    for (int[] interval : intervals) {
        // If a room is free (earliest end ≤ current start), reuse it
        if (!endTimes.isEmpty() && endTimes.peek() <= interval[0]) {
            endTimes.poll();
        }
        endTimes.offer(interval[1]); // add current meeting's end time
    }
    return endTimes.size(); // rooms in use = meetings still ongoing
}
```

### Merge Intervals

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
    List<int[]> merged = new ArrayList<>();
    
    for (int[] interval : intervals) {
        if (!merged.isEmpty() && merged.get(merged.size()-1)[1] >= interval[0]) {
            // Overlapping — extend last interval's end
            merged.get(merged.size()-1)[1] = Math.max(merged.get(merged.size()-1)[1], interval[1]);
        } else {
            merged.add(interval);
        }
    }
    return merged.toArray(new int[0][]);
}
```

---

## Task and Resource Scheduling

### Task Scheduler — CPU Scheduling with Cooldown

**Greedy:** Always execute the most frequent remaining task. If none available, idle.

```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char c : tasks) freq[c - 'A']++;
    
    int maxFreq = Arrays.stream(freq).max().getAsInt();
    long maxCount = Arrays.stream(freq).filter(f -> f == maxFreq).count();
    
    // Formula: (maxFreq - 1) * (n + 1) + maxCount tasks with max frequency
    // But must be at least tasks.length if no idle needed
    return (int) Math.max(tasks.length, (maxFreq - 1) * (long)(n + 1) + maxCount);
}
// Key insight: tasks is the lower bound; the formula gives the minimum time with idle slots
```

### Jump Game — Can You Reach the End?

```java
public boolean canJump(int[] nums) {
    int maxReach = 0; // farthest index we can reach
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false; // can't reach index i
        maxReach = Math.max(maxReach, i + nums[i]);
    }
    return true;
}

// Jump Game II — Minimum jumps to reach end
public int jump(int[] nums) {
    int jumps = 0, currentEnd = 0, farthest = 0;
    for (int i = 0; i < nums.length - 1; i++) {
        farthest = Math.max(farthest, i + nums[i]);
        if (i == currentEnd) { // must jump now
            jumps++;
            currentEnd = farthest;
        }
    }
    return jumps;
}
// Greedy: delay jump as long as possible, jump to farthest reachable position
```

---

## Classic Greedy Problems

### Gas Station — Can You Complete the Circuit?

```java
public int canCompleteCircuit(int[] gas, int[] cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < gas.length; i++) {
        int net = gas[i] - cost[i];
        total += net;
        tank += net;
        if (tank < 0) { // can't continue from 'start'
            start = i + 1; // try starting from next
            tank = 0;
        }
    }
    return total >= 0 ? start : -1; // if total >= 0, a solution exists
}
// Key insight: if total gas >= total cost, a solution exists. 
// The greedy finds it by tracking where we go negative and resetting.
```

### Assign Cookies — Children with Greed Factor

```java
public int findContentChildren(int[] g, int[] s) {
    Arrays.sort(g); // greed factors
    Arrays.sort(s); // cookie sizes
    int child = 0, cookie = 0;
    while (child < g.length && cookie < s.length) {
        if (s[cookie] >= g[child]) child++; // satisfy this child
        cookie++; // move to next cookie either way
    }
    return child;
}
// Greedy: assign smallest sufficient cookie to least greedy child
```

### Partition Labels — Split String Into Maximum Labels

```java
public List<Integer> partitionLabels(String s) {
    // Find last occurrence of each character
    int[] last = new int[26];
    for (int i = 0; i < s.length(); i++) last[s.charAt(i) - 'a'] = i;
    
    List<Integer> result = new ArrayList<>();
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, last[s.charAt(i) - 'a']); // extend partition
        if (i == end) { // partition ends here
            result.add(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}
```

### Minimum Number of Arrows to Burst Balloons

```java
public int findMinArrowShots(int[][] points) {
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1])); // sort by end
    int arrows = 1, arrowPos = points[0][1];
    
    for (int i = 1; i < points.length; i++) {
        if (points[i][0] > arrowPos) { // balloon starts after current arrow
            arrows++;
            arrowPos = points[i][1]; // shoot new arrow at this balloon's end
        }
    }
    return arrows;
}
```

### Reconstruct Queue by Height

```java
// People have [height, k] where k = number of taller/equal people in front
public int[][] reconstructQueue(int[][] people) {
    // Sort by height desc; ties broken by k asc
    Arrays.sort(people, (a, b) -> a[0] != b[0] ? b[0] - a[0] : a[1] - b[1]);
    
    List<int[]> queue = new LinkedList<>();
    for (int[] p : people) {
        queue.add(p[1], p); // insert at position k (LinkedList O(n) but acceptable for interviews)
    }
    return queue.toArray(new int[0][]);
}
// Greedy: taller people don't care about shorter → place taller first, then insert shorter by their k
```

---

## When Greedy Fails

```
Problem: Minimum coins to make change
Coins: {1, 3, 4}, Amount: 6

Greedy (pick largest coin first):
  4 → amount = 2
  1 → amount = 1
  1 → amount = 0
  Result: 3 coins ❌

DP optimal:
  3 + 3 = 2 coins ✅

Why greedy fails: Picking 4 "seems" optimal locally, but prevents finding the 
better 3+3 combination. The greedy choice property doesn't hold here.
```

**Greedy DOES work for:** Standard coin denominations (multiples: 1, 5, 10, 25), fractional knapsack, activity selection, shortest path in positive-weight graphs (Dijkstra).

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Assign Cookies (LeetCode #455)

**Problem:** Assign cookies to children. Each child has a greed factor. Maximize content children.

```java
public int findContentChildren(int[] g, int[] s) {
    Arrays.sort(g); Arrays.sort(s);
    int child = 0, cookie = 0;
    while (child < g.length && cookie < s.length) {
        if (s[cookie] >= g[child]) child++;
        cookie++;
    }
    return child;
}
// Time: O(n log n)  Space: O(1)
```

---

#### 2. Lemonade Change (LeetCode #860)

**Problem:** Customers pay with $5, $10, or $20. Can you always give correct change starting with no money?

**Greedy:** Always prefer to give $10 as change rather than two $5 (save $5 bills).
```java
public boolean lemonadeChange(int[] bills) {
    int fives = 0, tens = 0;
    for (int bill : bills) {
        if (bill == 5) fives++;
        else if (bill == 10) { if (fives == 0) return false; fives--; tens++; }
        else {
            if (tens > 0 && fives > 0) { tens--; fives--; }
            else if (fives >= 3) fives -= 3;
            else return false;
        }
    }
    return true;
}
// Time: O(n)  Space: O(1)
```

---

### 🟡 EASY

---

#### 3. Jump Game (LeetCode #55)

**Problem:** Given array nums, where nums[i] is max jump length at position i, can you reach the last index?

```java
public boolean canJump(int[] nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false;
        maxReach = Math.max(maxReach, i + nums[i]);
    }
    return true;
}
// Time: O(n)  Space: O(1)
```

---

#### 4. Maximum Subarray (LeetCode #53) — Greedy Perspective

**Problem:** Find maximum sum contiguous subarray.

**Greedy interpretation:** If current sum goes negative, restart — there's no benefit in keeping a negative prefix.
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
```

---

### 🟠 MEDIUM

---

#### 5. Jump Game II (LeetCode #45) — Minimum Jumps

**Problem:** Return the minimum number of jumps to reach the last index.

```java
public int jump(int[] nums) {
    int jumps = 0, currentEnd = 0, farthest = 0;
    for (int i = 0; i < nums.length - 1; i++) {
        farthest = Math.max(farthest, i + nums[i]);
        if (i == currentEnd) {  // must jump now
            jumps++;
            currentEnd = farthest;
        }
    }
    return jumps;
}
// Time: O(n)  Space: O(1)
// Greedy: delay jumping as long as possible — always jump to farthest reachable
```

---

#### 6. Gas Station (LeetCode #134)

**Problem:** Find the starting gas station to complete a circular route, or return -1.

```java
public int canCompleteCircuit(int[] gas, int[] cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < gas.length; i++) {
        int net = gas[i] - cost[i];
        total += net; tank += net;
        if (tank < 0) { start = i + 1; tank = 0; }
    }
    return total >= 0 ? start : -1;
}
// Time: O(n)  Space: O(1)
// Proof: if total gas >= total cost, a solution exists. Greedy finds it.
```

---

#### 7. Non-overlapping Intervals (LeetCode #435) — Minimum Removals

**Problem:** Find the minimum number of intervals to remove to make the rest non-overlapping.

**Greedy:** Sort by end time, keep as many intervals as possible.
```java
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]);
    int count = 0, lastEnd = intervals[0][1];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < lastEnd) count++;  // overlap — remove
        else lastEnd = intervals[i][1];           // no overlap — keep
    }
    return count;
}
// Time: O(n log n)  Space: O(1)
```

---

#### 8. Partition Labels (LeetCode #763)

**Problem:** Partition string into as many parts as possible such that each letter appears in at most one part.

```java
public List<Integer> partitionLabels(String s) {
    int[] last = new int[26];
    for (int i = 0; i < s.length(); i++) last[s.charAt(i) - 'a'] = i;
    List<Integer> result = new ArrayList<>();
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, last[s.charAt(i) - 'a']);
        if (i == end) { result.add(end - start + 1); start = i + 1; }
    }
    return result;
}
// Time: O(n)  Space: O(1)
```

---

### 🔴 HARD

---

#### 9. Candy (LeetCode #135)

**Problem:** Give minimum candies to n children where children with higher rating than neighbors get more.

**Two-pass greedy:**
```java
public int candy(int[] ratings) {
    int n = ratings.length;
    int[] candies = new int[n];
    Arrays.fill(candies, 1);
    // Left to right: if rating > left neighbor, give one more
    for (int i = 1; i < n; i++)
        if (ratings[i] > ratings[i-1]) candies[i] = candies[i-1] + 1;
    // Right to left: if rating > right neighbor, take max
    for (int i = n - 2; i >= 0; i--)
        if (ratings[i] > ratings[i+1]) candies[i] = Math.max(candies[i], candies[i+1] + 1);
    int total = 0;
    for (int c : candies) total += c;
    return total;
}
// Time: O(n)  Space: O(n)
```

---

#### 10. Minimum Number of Arrows to Burst Balloons (LeetCode #452)

**Problem:** Find the minimum number of arrows needed to burst all balloons (intervals).

```java
public int findMinArrowShots(int[][] points) {
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));  // sort by end
    int arrows = 1, arrowPos = points[0][1];
    for (int i = 1; i < points.length; i++) {
        if (points[i][0] > arrowPos) {  // balloon starts after current arrow
            arrows++; arrowPos = points[i][1];
        }
    }
    return arrows;
}
// Time: O(n log n)  Space: O(1)
```

---

#### 11. Course Schedule III (LeetCode #630) — Greedy + Heap

**Problem:** Given courses with duration and deadline, find the maximum number of courses you can take.

**Greedy:** Take the shortest deadline course first. If adding a course exceeds deadline, replace the longest course taken so far.
```java
public int scheduleCourse(int[][] courses) {
    Arrays.sort(courses, (a, b) -> a[1] - b[1]);  // sort by deadline
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    int time = 0;
    for (int[] course : courses) {
        time += course[0];
        maxHeap.offer(course[0]);
        if (time > course[1]) {  // exceeded deadline — replace longest
            time -= maxHeap.poll();
        }
    }
    return maxHeap.size();
}
// Time: O(n log n)  Space: O(n)
```

---

#### 12. Reconstruct Queue by Height (LeetCode #406)

**Problem:** Reconstruct a queue from (height, k) pairs where k = number of taller/equal people in front.

**Greedy:** Place tallest people first, then insert shorter ones by their k index.
```java
public int[][] reconstructQueue(int[][] people) {
    Arrays.sort(people, (a, b) -> a[0] != b[0] ? b[0] - a[0] : a[1] - b[1]);
    List<int[]> queue = new LinkedList<>();
    for (int[] p : people) queue.add(p[1], p);
    return queue.toArray(new int[0][]);
}
// Time: O(n²) for insertions  Space: O(n)
```

---

## Interview Questions

### Q: How do you prove a greedy algorithm is correct?

The most common proof technique is the **exchange argument:**
1. Assume there's an optimal solution O that differs from the greedy solution G
2. Find the first point of difference
3. Show that swapping O's choice for G's choice doesn't decrease the solution quality
4. By repeated swaps, O → G without loss → G is also optimal

### Q: Interval scheduling — why sort by end time and not start time?

If we sort by start time: we might pick a long interval that blocks many short ones.
If we sort by end time: the earliest-ending interval leaves the most room for future intervals. This is the classical result: greedy by end time = optimal for maximum non-overlapping intervals.

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Greedy works when | Greedy choice property + optimal substructure |
| 2 | Greedy vs DP | Greedy: one local decision; DP: explores all |
| 3 | Activity selection | Sort by end time; pick earliest-ending non-overlapping |
| 4 | Min meeting rooms | Min-heap of end times; reuse if room free |
| 5 | Merge intervals | Sort by start; extend last interval if overlapping |
| 6 | Jump game greedy | Track maxReach; if i > maxReach → impossible |
| 7 | Jump game II | Delay jumps; jump when forced to; track farthest |
| 8 | Gas station | If total gas ≥ total cost, solution exists; start resets on negative |
| 9 | Task scheduler formula | max(tasks.length, (maxFreq-1)*(n+1) + maxCount) |
| 10 | Coin change greedy fails | Arbitrary denominations → use DP |
| 11 | Partition labels | Last occurrence map; extend end as you scan |
| 12 | Burst balloons with arrows | Sort by end; shoot at end; count new arrows needed |
| 13 | Reconstruct queue | Sort tall-first; insert by k position |
| 14 | Exchange argument | Swap greedy for any other → not better → greedy is optimal |
| 15 | Greedy sort choice | Problem usually dictates sort key — analyze what "locally optimal" means |

---

*Next: [DSA Patterns for Interviews](12-dsa-patterns-for-interviews.md)*
