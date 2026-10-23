# Advanced Graphs and DP Theory

## 1. 0-1 BFS
**Problem it solves:** Shortest path in a graph where edge weights are only 0 or 1.
**Recognition clues:** Grid with free/costly moves. "Minimum obstacles removed".
**Core idea:** Use a Deque. If weight is 0, add to `front()`. If weight is 1, add to `back()`.
**Complexity:** $O(V + E)$ time, avoiding the $O((V+E)\log V)$ of Dijkstra.
**Common mistakes:** Using a standard queue instead of a deque.

## 2. Strongly Connected Components (Kosaraju / Tarjan)
**Problem it solves:** Finding groups of nodes in a directed graph where every node can reach every other node.
**Recognition clues:** "Is there a cycle encompassing these nodes?", "Critical dependencies".
**Algorithm (Kosaraju):** DFS to get finish times (stack). Reverse graph. DFS based on stack pop.

## 3. Bridges and Articulation Points
**Problem it solves:** Finding single points of failure in a network.
**Recognition clues:** "Critical connections", "Servers that, if removed, disconnect the network".
**Algorithm (Tarjan):** Maintain `discovery_time` and `lowest_reachable_time`. If a child's lowest reachable time is > your discovery time, the edge to the child is a bridge.

## 4. Interval DP
**Problem it solves:** Problems where you can combine adjacent elements.
**Recognition clues:** "Merge adjacent", "Burst balloons", "Cost to multiply matrices".
**Core idea:** State is `dp[i][j]` representing the optimal cost for the interval `i` to `j`. Iterate length `L` from 2 to `N`, and try all splits `k` between `i` and `j`.

## 5. State Machine DP
**Problem it solves:** Sequence problems where you can be in a discrete number of "states" on any given day.
**Recognition clues:** "Stock trading with cooldown/fee", "Multiple states of action".
**Core idea:** Mutually recursive DP relations. `hold[i] = max(hold[i-1], rest[i-1] - price)`, etc.
