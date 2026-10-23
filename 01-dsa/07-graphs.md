# Graphs

This chapter covers graphs — the most versatile and complex data structure in DSA. Graph problems appear in pathfinding, connectivity, scheduling, and network design.

## Table of Contents
- [Graph Fundamentals](#graph-fundamentals)
- [Graph Representations](#graph-representations)
- [BFS on Graphs](#bfs-on-graphs)
- [DFS on Graphs](#dfs-on-graphs)
- [Topological Sort](#topological-sort)
- [Union Find (Disjoint Set Union)](#union-find-disjoint-set-union)
- [Shortest Path Algorithms](#shortest-path-algorithms)
- [Minimum Spanning Tree](#minimum-spanning-tree)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Graph Fundamentals

```
Directed vs Undirected:
  Undirected: A -- B (edge goes both ways)
  Directed:   A --> B (edge goes one way, also called "digraph")

Weighted vs Unweighted:
  Unweighted: all edges cost 1
  Weighted:   each edge has a cost/weight

Cyclic vs Acyclic:
  Cyclic: contains a cycle
  Acyclic: no cycles (DAG = Directed Acyclic Graph)

Connected vs Disconnected:
  Connected: path exists between every pair of nodes
  Disconnected: some nodes unreachable from others
```

**Key terms:**
- **V** = number of vertices (nodes)
- **E** = number of edges
- **Degree** = number of edges at a vertex
- **In-degree / Out-degree** for directed graphs

---

## Graph Representations

### Adjacency List (most common for sparse graphs)

```java
// For V vertices, using HashMap for flexibility
Map<Integer, List<Integer>> adj = new HashMap<>();

// Add edge
adj.computeIfAbsent(u, k -> new ArrayList<>()).add(v);
// For undirected: also add adj[v] → u

// Using array of lists (faster for dense, fixed V)
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < V; i++) adj.add(new ArrayList<>());
adj.get(u).add(v);

// Weighted adjacency list
List<List<int[]>> adj = new ArrayList<>(); // int[] = {neighbor, weight}
adj.get(u).add(new int[]{v, weight});
```

| | Adjacency List | Adjacency Matrix |
|-|---------------|-----------------|
| Space | O(V + E) | O(V²) |
| Check edge (u,v) | O(degree(u)) | O(1) |
| Find all neighbors | O(degree(u)) | O(V) |
| Best for | Sparse graphs | Dense graphs |

### Edge List (for MST algorithms)

```java
int[][] edges = {{0, 1, 4}, {0, 2, 1}, {1, 2, 2}}; // {from, to, weight}
```

---

## BFS on Graphs

**BFS key properties:**
- Finds shortest path in unweighted graphs
- Explores layer by layer
- Uses a Queue
- Marks visited nodes to avoid cycles

```java
// BFS template for graph
public void bfs(Map<Integer, List<Integer>> adj, int start) {
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    
    queue.offer(start);
    visited.add(start);
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        process(node);
        
        for (int neighbor : adj.getOrDefault(node, Collections.emptyList())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
}

// BFS shortest path with distance tracking
public int[] bfsDistances(int[][] adj, int src, int V) {
    int[] dist = new int[V];
    Arrays.fill(dist, -1);
    Queue<Integer> queue = new LinkedList<>();
    queue.offer(src);
    dist[src] = 0;
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (int neighbor : adj[node]) {
            if (dist[neighbor] == -1) {
                dist[neighbor] = dist[node] + 1;
                queue.offer(neighbor);
            }
        }
    }
    return dist;
}
```

### Number of Islands (BFS)

```java
public int numIslands(char[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int count = 0;
    
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == '1') {
                count++;
                bfsIsland(grid, r, c);
            }
        }
    }
    return count;
}

private void bfsIsland(char[][] grid, int r, int c) {
    Queue<int[]> queue = new LinkedList<>();
    queue.offer(new int[]{r, c});
    grid[r][c] = '0'; // mark visited
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    
    while (!queue.isEmpty()) {
        int[] cell = queue.poll();
        for (int[] d : dirs) {
            int nr = cell[0] + d[0], nc = cell[1] + d[1];
            if (nr >= 0 && nr < grid.length && nc >= 0 && nc < grid[0].length && grid[nr][nc] == '1') {
                grid[nr][nc] = '0';
                queue.offer(new int[]{nr, nc});
            }
        }
    }
}
```

---

## DFS on Graphs

```java
// DFS template — iterative
public void dfsIterative(Map<Integer, List<Integer>> adj, int start) {
    Deque<Integer> stack = new ArrayDeque<>();
    Set<Integer> visited = new HashSet<>();
    stack.push(start);
    
    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (visited.contains(node)) continue;
        visited.add(node);
        process(node);
        for (int neighbor : adj.getOrDefault(node, Collections.emptyList())) {
            if (!visited.contains(neighbor)) stack.push(neighbor);
        }
    }
}

// DFS template — recursive
public void dfsRecursive(Map<Integer, List<Integer>> adj, int node, Set<Integer> visited) {
    visited.add(node);
    process(node);
    for (int neighbor : adj.getOrDefault(node, Collections.emptyList())) {
        if (!visited.contains(neighbor)) dfsRecursive(adj, neighbor, visited);
    }
}

// Detect cycle in directed graph (using coloring: WHITE/GRAY/BLACK)
static final int WHITE = 0, GRAY = 1, BLACK = 2;
public boolean hasCycle(int V, List<List<Integer>> adj) {
    int[] color = new int[V];
    for (int i = 0; i < V; i++) {
        if (color[i] == WHITE && dfsCycle(adj, i, color)) return true;
    }
    return false;
}
private boolean dfsCycle(List<List<Integer>> adj, int node, int[] color) {
    color[node] = GRAY;
    for (int neighbor : adj.get(node)) {
        if (color[neighbor] == GRAY) return true; // back edge = cycle
        if (color[neighbor] == WHITE && dfsCycle(adj, neighbor, color)) return true;
    }
    color[node] = BLACK;
    return false;
}
```

---

## Topological Sort

**Only valid for DAGs (Directed Acyclic Graphs).** Gives a linear ordering where every directed edge u→v has u before v.

**Use cases:** Build systems, dependency resolution, course scheduling.

### Kahn's Algorithm (BFS-based)

```java
public int[] topologicalSort(int V, List<List<Integer>> adj) {
    int[] indegree = new int[V];
    for (int u = 0; u < V; u++)
        for (int v : adj.get(u)) indegree[v]++;
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < V; i++) if (indegree[i] == 0) queue.offer(i);
    
    int[] order = new int[V];
    int idx = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[idx++] = node;
        for (int neighbor : adj.get(node)) {
            if (--indegree[neighbor] == 0) queue.offer(neighbor);
        }
    }
    return idx == V ? order : new int[0]; // empty if cycle detected
}
```

### Course Schedule (Can you finish all courses?)

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[numCourses];
    
    for (int[] pre : prerequisites) {
        adj.get(pre[1]).add(pre[0]);
        indegree[pre[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (indegree[i] == 0) queue.offer(i);
    
    int count = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        count++;
        for (int next : adj.get(course)) {
            if (--indegree[next] == 0) queue.offer(next);
        }
    }
    return count == numCourses;
}
```

---

## Union Find (Disjoint Set Union)

**DSU** efficiently answers: "Are nodes u and v in the same connected component?"

**Operations:** `find(x)`, `union(x, y)` — both nearly O(1) with optimizations.

```java
class UnionFind {
    private int[] parent, rank;
    private int components;
    
    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        components = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    
    // Find with path compression
    public int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // compress path
        return parent[x];
    }
    
    // Union by rank
    public boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false; // already connected
        if (rank[rx] < rank[ry]) { int tmp = rx; rx = ry; ry = tmp; }
        parent[ry] = rx;
        if (rank[rx] == rank[ry]) rank[rx]++;
        components--;
        return true;
    }
    
    public boolean connected(int x, int y) { return find(x) == find(y); }
    public int getComponents() { return components; }
}
// With path compression + union by rank: nearly O(1) amortized
// Technically O(α(n)) — inverse Ackermann, essentially constant for any practical n
```

**Use cases:**
- Number of connected components
- Detect cycle in undirected graph
- Kruskal's MST
- Dynamic connectivity

---

## Shortest Path Algorithms

### Dijkstra (non-negative weights)

```java
public int[] dijkstra(int V, List<List<int[]>> adj, int src) {
    int[] dist = new int[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    
    // Min-heap: {distance, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, src});
    
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int d = curr[0], u = curr[1];
        
        if (d > dist[u]) continue; // outdated entry
        
        for (int[] edge : adj.get(u)) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.offer(new int[]{dist[v], v});
            }
        }
    }
    return dist;
}
// O((V + E) log V) with min-heap
```

### Bellman-Ford (handles negative weights)

```java
public int[] bellmanFord(int V, int[][] edges, int src) {
    int[] dist = new int[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    
    // Relax all edges V-1 times
    for (int i = 0; i < V - 1; i++) {
        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }
    
    // Check for negative cycles
    for (int[] edge : edges) {
        if (dist[edge[0]] + edge[2] < dist[edge[1]])
            throw new RuntimeException("Negative cycle detected");
    }
    return dist;
}
// O(VE) time
```

---

## Minimum Spanning Tree

### Kruskal's Algorithm (Union-Find based)

```java
public int kruskal(int V, int[][] edges) {
    Arrays.sort(edges, (a, b) -> a[2] - b[2]); // sort by weight
    UnionFind uf = new UnionFind(V);
    int totalWeight = 0, edgesUsed = 0;
    
    for (int[] edge : edges) {
        if (uf.union(edge[0], edge[1])) {
            totalWeight += edge[2];
            edgesUsed++;
            if (edgesUsed == V - 1) break; // MST complete
        }
    }
    return edgesUsed == V - 1 ? totalWeight : -1; // -1 if disconnected
}
// O(E log E) for sorting
```

---

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Number of Islands (LeetCode #200)

**Problem:** Count the number of islands in a 2D grid of '1' (land) and '0' (water).

```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == '1') { count++; dfs(grid, r, c); }
    return count;
}
private void dfs(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != '1') return;
    grid[r][c] = '0';  // mark visited
    dfs(grid, r+1, c); dfs(grid, r-1, c); dfs(grid, r, c+1); dfs(grid, r, c-1);
}
// Time: O(m×n)  Space: O(m×n) — recursion stack
```

---

#### 2. Clone Graph (LeetCode #133)

**Problem:** Return a deep copy of a connected undirected graph.

```java
Map<Node, Node> cloned = new HashMap<>();
public Node cloneGraph(Node node) {
    if (node == null) return null;
    if (cloned.containsKey(node)) return cloned.get(node);
    Node clone = new Node(node.val);
    cloned.put(node, clone);
    for (Node neighbor : node.neighbors) clone.neighbors.add(cloneGraph(neighbor));
    return clone;
}
// Time: O(V + E)  Space: O(V)
```

---

### 🟡 EASY

---

#### 3. Flood Fill (LeetCode #733)

**Problem:** Perform a flood fill starting from (sr, sc) changing all connected cells of the same color.

```java
public int[][] floodFill(int[][] image, int sr, int sc, int color) {
    int oldColor = image[sr][sc];
    if (oldColor != color) fill(image, sr, sc, oldColor, color);
    return image;
}
private void fill(int[][] image, int r, int c, int oldColor, int newColor) {
    if (r < 0 || r >= image.length || c < 0 || c >= image[0].length
        || image[r][c] != oldColor) return;
    image[r][c] = newColor;
    fill(image, r+1, c, oldColor, newColor); fill(image, r-1, c, oldColor, newColor);
    fill(image, r, c+1, oldColor, newColor); fill(image, r, c-1, oldColor, newColor);
}
// Time: O(m×n)  Space: O(m×n)
```

---

#### 4. Course Schedule (LeetCode #207) — Cycle Detection in DAG

**Problem:** Can you finish all courses given prerequisites (detect cycle in directed graph)?

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[numCourses];
    for (int[] p : prerequisites) { adj.get(p[1]).add(p[0]); indegree[p[0]]++; }
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (indegree[i] == 0) queue.offer(i);
    int count = 0;
    while (!queue.isEmpty()) {
        int c = queue.poll(); count++;
        for (int next : adj.get(c)) if (--indegree[next] == 0) queue.offer(next);
    }
    return count == numCourses;
}
// Time: O(V + E)  Space: O(V + E)
// Kahn's algorithm: if all courses processed, no cycle
```

---

### 🟠 MEDIUM

---

#### 5. Rotting Oranges (LeetCode #994)

**Problem:** Find minimum minutes until all fresh oranges rot (BFS multi-source).

**Key insight:** Multi-source BFS from all initially rotten oranges simultaneously.
```java
public int orangesRotting(int[][] grid) {
    Queue<int[]> queue = new LinkedList<>();
    int fresh = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == 2) queue.offer(new int[]{r, c});
            if (grid[r][c] == 1) fresh++;
        }
    int minutes = 0;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!queue.isEmpty() && fresh > 0) {
        minutes++;
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr >= 0 && nr < grid.length && nc >= 0 && nc < grid[0].length
                    && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2; fresh--; queue.offer(new int[]{nr, nc});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
// Time: O(m×n)  Space: O(m×n)
```

---

#### 6. Number of Connected Components in Undirected Graph (LeetCode #323)

**Problem:** Count connected components using Union-Find.

```java
public int countComponents(int n, int[][] edges) {
    int[] parent = new int[n];
    for (int i = 0; i < n; i++) parent[i] = i;
    int components = n;
    for (int[] e : edges) {
        int a = find(parent, e[0]), b = find(parent, e[1]);
        if (a != b) { parent[a] = b; components--; }
    }
    return components;
}
private int find(int[] parent, int x) {
    if (parent[x] != x) parent[x] = find(parent, parent[x]);
    return parent[x];
}
// Time: O(E × α(V))  Space: O(V)
```

---

#### 7. Pacific Atlantic Water Flow (LeetCode #417)

**Problem:** Find cells from which water can flow to both Pacific and Atlantic oceans.

**Key insight:** Reverse the problem — BFS/DFS from the ocean borders inward.
```java
public List<List<Integer>> pacificAtlantic(int[][] heights) {
    int m = heights.length, n = heights[0].length;
    boolean[][] pacific = new boolean[m][n], atlantic = new boolean[m][n];
    // BFS from Pacific (top and left borders)
    Queue<int[]> q = new LinkedList<>();
    for (int i = 0; i < m; i++) { q.offer(new int[]{i, 0}); pacific[i][0] = true; }
    for (int j = 1; j < n; j++) { q.offer(new int[]{0, j}); pacific[0][j] = true; }
    bfs(heights, q, pacific);
    // BFS from Atlantic (bottom and right borders)
    q.clear();
    for (int i = 0; i < m; i++) { q.offer(new int[]{i, n-1}); atlantic[i][n-1] = true; }
    for (int j = 0; j < n-1; j++) { q.offer(new int[]{m-1, j}); atlantic[m-1][j] = true; }
    bfs(heights, q, atlantic);
    // Collect cells reachable from both
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (pacific[i][j] && atlantic[i][j]) result.add(Arrays.asList(i, j));
    return result;
}
private void bfs(int[][] h, Queue<int[]> q, boolean[][] visited) {
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!q.isEmpty()) {
        int[] cell = q.poll();
        for (int[] d : dirs) {
            int nr = cell[0]+d[0], nc = cell[1]+d[1];
            if (nr >= 0 && nr < h.length && nc >= 0 && nc < h[0].length
                && !visited[nr][nc] && h[nr][nc] >= h[cell[0]][cell[1]]) {
                visited[nr][nc] = true; q.offer(new int[]{nr, nc});
            }
        }
    }
}
// Time: O(m×n)  Space: O(m×n)
```

---

### 🔴 HARD

---

#### 8. Word Ladder (LeetCode #127)

**Problem:** Find the shortest transformation sequence from beginWord to endWord, changing one letter at a time, where each intermediate word must be in the wordList.

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(endWord)) return 0;
    Queue<String> queue = new LinkedList<>();
    queue.offer(beginWord); wordSet.remove(beginWord);
    int steps = 1;
    while (!queue.isEmpty()) {
        int size = queue.size(); steps++;
        for (int i = 0; i < size; i++) {
            char[] word = queue.poll().toCharArray();
            for (int j = 0; j < word.length; j++) {
                char orig = word[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    word[j] = c;
                    String next = new String(word);
                    if (next.equals(endWord)) return steps;
                    if (wordSet.contains(next)) { wordSet.remove(next); queue.offer(next); }
                }
                word[j] = orig;
            }
        }
    }
    return 0;
}
// Time: O(M² × N) where M = word length, N = wordList size
// Space: O(M² × N)
```

---

#### 9. Network Delay Time (LeetCode #743) — Dijkstra

**Problem:** Find the minimum time for a signal sent from node k to reach all other nodes.

```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
    for (int[] t : times) adj.get(t[0]).add(new int[]{t[1], t[2]});
    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, k});
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int d = curr[0], u = curr[1];
        if (d > dist[u]) continue;
        for (int[] edge : adj.get(u)) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) { dist[v] = dist[u] + w; pq.offer(new int[]{dist[v], v}); }
        }
    }
    int maxDist = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1;
        maxDist = Math.max(maxDist, dist[i]);
    }
    return maxDist;
}
// Time: O((V + E) log V)  Space: O(V + E)
```

---

#### 10. Alien Dictionary (LeetCode #269) — Topological Sort

**Problem:** Given a sorted list of words in an alien language, determine the order of the alphabet.

```java
public String alienOrder(String[] words) {
    Map<Character, Set<Character>> adj = new HashMap<>();
    Map<Character, Integer> indegree = new HashMap<>();
    for (String w : words) for (char c : w.toCharArray()) { adj.putIfAbsent(c, new HashSet<>()); indegree.putIfAbsent(c, 0); }
    for (int i = 0; i < words.length - 1; i++) {
        String w1 = words[i], w2 = words[i+1];
        if (w1.length() > w2.length() && w1.startsWith(w2)) return "";  // invalid
        for (int j = 0; j < Math.min(w1.length(), w2.length()); j++) {
            if (w1.charAt(j) != w2.charAt(j)) {
                if (!adj.get(w1.charAt(j)).contains(w2.charAt(j))) {
                    adj.get(w1.charAt(j)).add(w2.charAt(j));
                    indegree.merge(w2.charAt(j), 1, Integer::sum);
                }
                break;
            }
        }
    }
    Queue<Character> q = new LinkedList<>();
    for (char c : indegree.keySet()) if (indegree.get(c) == 0) q.offer(c);
    StringBuilder sb = new StringBuilder();
    while (!q.isEmpty()) {
        char c = q.poll(); sb.append(c);
        for (char next : adj.get(c)) if (indegree.merge(next, -1, Integer::sum) == 0) q.offer(next);
    }
    return sb.length() == indegree.size() ? sb.toString() : "";
}
// Time: O(C) where C = total chars in all words  Space: O(1) — max 26 nodes
```

---

## Classic Problems

| Problem | Algorithm | Complexity |
|---------|-----------|-----------|
| Number of islands | DFS/BFS | O(V×E) |
| Shortest path (unweighted) | BFS | O(V + E) |
| Shortest path (weighted) | Dijkstra | O((V+E) log V) |
| Shortest path (negative) | Bellman-Ford | O(VE) |
| All-pairs shortest path | Floyd-Warshall | O(V³) |
| Topological sort | Kahn's BFS or DFS | O(V + E) |
| Cycle detection (directed) | DFS (3-color) | O(V + E) |
| Cycle detection (undirected) | Union-Find | O(E α(V)) |
| Connected components | DFS/BFS/Union-Find | O(V + E) |
| MST | Kruskal / Prim | O(E log E) |
| Bipartite check | BFS (2-coloring) | O(V + E) |
| Course schedule | Topological sort | O(V + E) |

---

## Interview Questions

### Q: How to detect a cycle in an undirected graph?

**DFS approach:** During DFS, if we visit a neighbor that's already visited AND it's not the parent of current node → cycle.

```java
public boolean hasCycleUndirected(int V, List<List<Integer>> adj) {
    boolean[] visited = new boolean[V];
    for (int i = 0; i < V; i++) {
        if (!visited[i] && dfs(adj, i, -1, visited)) return true;
    }
    return false;
}
private boolean dfs(List<List<Integer>> adj, int node, int parent, boolean[] visited) {
    visited[node] = true;
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            if (dfs(adj, neighbor, node, visited)) return true;
        } else if (neighbor != parent) {
            return true; // back edge (not to parent) = cycle
        }
    }
    return false;
}
```

### Q: Check if a graph is bipartite.

**BFS 2-coloring:** Assign colors alternately. If we find same color neighbors → not bipartite.

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n]; // 0=uncolored, 1=red, -1=blue
    
    for (int start = 0; start < n; start++) {
        if (color[start] != 0) continue;
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(start);
        color[start] = 1;
        
        while (!queue.isEmpty()) {
            int node = queue.poll();
            for (int neighbor : graph[node]) {
                if (color[neighbor] == color[node]) return false; // same color = not bipartite
                if (color[neighbor] == 0) {
                    color[neighbor] = -color[node];
                    queue.offer(neighbor);
                }
            }
        }
    }
    return true;
}
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Adjacency list space | O(V + E) |
| 2 | Adjacency matrix space | O(V²) |
| 3 | BFS finds | Shortest path in unweighted graphs |
| 4 | DFS finds | Connectivity, cycles, topological order |
| 5 | BFS uses | Queue |
| 6 | DFS uses | Stack (iterative) or recursion |
| 7 | Topological sort requires | DAG (no cycles) |
| 8 | Kahn's algorithm | BFS-based topo sort using in-degree |
| 9 | Union-Find for | Connectivity, cycle detection (undirected), MST |
| 10 | Dijkstra limitation | Fails with negative weights |
| 11 | Bellman-Ford vs Dijkstra | BF: handles negatives, O(VE); Dijkstra: O((V+E) log V) |
| 12 | Cycle in directed graph | DFS 3-color: WHITE, GRAY (in progress), BLACK (done) |
| 13 | Cycle in undirected graph | DFS: back edge to non-parent = cycle |
| 14 | Bipartite check | BFS 2-coloring — color alternately |
| 15 | MST Kruskal | Sort edges → union-find; skip edges that would form cycle |

---

*Next: [Hash Tables](08-hash-tables.md)*
