# Trees and Tries

This chapter covers trees and tries — among the most heavily tested topics in interviews. Trees test recursion fluency; tries test string processing and prefix matching.

## Table of Contents
- [Binary Tree Fundamentals](#binary-tree-fundamentals)
- [Tree Traversals](#tree-traversals)
- [Binary Search Tree (BST)](#binary-search-tree-bst)
- [Balanced Trees](#balanced-trees)
- [Common Tree Patterns](#common-tree-patterns)
- [Trie (Prefix Tree)](#trie-prefix-tree)
- [Classic Problems](#classic-problems)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Binary Tree Fundamentals

```java
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}

// Key properties
// Height of tree = longest path from root to leaf
// Depth of node = distance from root
// Full binary tree: every node has 0 or 2 children
// Complete binary tree: all levels full except last, last filled left to right
// Perfect binary tree: all leaves at same level (2^h - 1 nodes for height h)
```

**Complexity reference:**

| Operation | Balanced BST | Unbalanced BST | Full Binary Tree |
|-----------|-------------|----------------|-----------------|
| Search | O(log n) | O(n) | O(n) |
| Insert | O(log n) | O(n) | O(n) |
| Delete | O(log n) | O(n) | O(n) |
| Traversal | O(n) | O(n) | O(n) |

---

## Tree Traversals

### DFS Traversals (Recursive)

```java
// Inorder: Left → Root → Right (gives sorted order for BST)
public void inorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    inorder(root.left, result);
    result.add(root.val);
    inorder(root.right, result);
}

// Preorder: Root → Left → Right (used for serialization)
public void preorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    result.add(root.val);
    preorder(root.left, result);
    preorder(root.right, result);
}

// Postorder: Left → Right → Root (used for deletion, expression trees)
public void postorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    postorder(root.left, result);
    postorder(root.right, result);
    result.add(root.val);
}
```

### DFS Traversals (Iterative)

```java
// Iterative Inorder
public List<Integer> inorderIterative(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) { stack.push(curr); curr = curr.left; }
        curr = stack.pop();
        result.add(curr.val);
        curr = curr.right;
    }
    return result;
}
```

### BFS (Level-Order)

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
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

## Binary Search Tree (BST)

**BST property:** `left.val < root.val < right.val` for every node.

**Key insight:** Inorder traversal of a BST gives a sorted sequence.

```java
// Search in BST
public TreeNode search(TreeNode root, int target) {
    if (root == null || root.val == target) return root;
    if (target < root.val) return search(root.left, target);
    return search(root.right, target);
}

// Insert in BST
public TreeNode insert(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);
    if (val < root.val) root.left = insert(root.left, val);
    else if (val > root.val) root.right = insert(root.right, val);
    return root; // if val == root.val, ignore duplicates
}

// Delete in BST — 3 cases
public TreeNode delete(TreeNode root, int key) {
    if (root == null) return null;
    if (key < root.val) root.left = delete(root.left, key);
    else if (key > root.val) root.right = delete(root.right, key);
    else {
        // Case 1: No child
        if (root.left == null && root.right == null) return null;
        // Case 2: One child
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;
        // Case 3: Two children — replace with inorder successor (min of right subtree)
        TreeNode successor = findMin(root.right);
        root.val = successor.val;
        root.right = delete(root.right, successor.val);
    }
    return root;
}

private TreeNode findMin(TreeNode node) {
    while (node.left != null) node = node.left;
    return node;
}

// Validate BST
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) && validate(node.right, node.val, max);
}
```

---

## Balanced Trees

### AVL Tree
- Height difference between left and right subtree ≤ 1 for every node
- Maintained via rotations after insert/delete
- Guarantees O(log n) operations

### Red-Black Tree (Java TreeMap/TreeSet)
- 5 properties ensure approximate balance
- Used in Java's TreeMap, TreeSet, and HashMap (Java 8+ for treeified buckets)
- O(log n) guaranteed operations

**Interview tip:** You rarely implement these from scratch. Know that Java's TreeMap is a Red-Black Tree giving O(log n) get/put.

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(3, "three");     // O(log n)
map.get(3);              // O(log n)
map.firstKey();          // O(log n) — smallest key
map.lastKey();           // O(log n) — largest key
map.floorKey(4);         // O(log n) — largest key ≤ 4
map.ceilingKey(2);       // O(log n) — smallest key ≥ 2
map.headMap(5);          // keys < 5
map.tailMap(3);          // keys >= 3
```

---

## Common Tree Patterns

### Pattern 1: Return value from recursion to propagate state up

```java
// Diameter of binary tree — at each node, diameter through this node = left height + right height
public int diameterOfBinaryTree(TreeNode root) {
    int[] maxDiameter = {0};
    height(root, maxDiameter);
    return maxDiameter[0];
}

private int height(TreeNode node, int[] maxDiameter) {
    if (node == null) return 0;
    int left = height(node.left, maxDiameter);
    int right = height(node.right, maxDiameter);
    maxDiameter[0] = Math.max(maxDiameter[0], left + right);
    return 1 + Math.max(left, right);
}
```

### Pattern 2: Path problems — carry accumulated state down

```java
// Path sum — does a root-to-leaf path sum to target?
public boolean hasPathSum(TreeNode root, int target) {
    if (root == null) return false;
    if (root.left == null && root.right == null) return target == root.val;
    return hasPathSum(root.left, target - root.val) ||
           hasPathSum(root.right, target - root.val);
}
```

### Pattern 3: Lowest Common Ancestor (LCA)

```java
// LCA in binary tree
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    // If both sides found a target — this node is LCA
    if (left != null && right != null) return root;
    return left != null ? left : right;
}

// LCA in BST — use BST property
public TreeNode lcaBST(TreeNode root, TreeNode p, TreeNode q) {
    if (p.val < root.val && q.val < root.val) return lcaBST(root.left, p, q);
    if (p.val > root.val && q.val > root.val) return lcaBST(root.right, p, q);
    return root; // p and q are on different sides — root is LCA
}
```

### Pattern 4: Serialize / Deserialize

```java
// Preorder + null markers
public String serialize(TreeNode root) {
    if (root == null) return "null,";
    return root.val + "," + serialize(root.left) + serialize(root.right);
}

public TreeNode deserialize(String data) {
    Queue<String> nodes = new LinkedList<>(Arrays.asList(data.split(",")));
    return buildTree(nodes);
}

private TreeNode buildTree(Queue<String> nodes) {
    String val = nodes.poll();
    if (val.equals("null")) return null;
    TreeNode node = new TreeNode(Integer.parseInt(val));
    node.left = buildTree(nodes);
    node.right = buildTree(nodes);
    return node;
}
```

---

## Trie (Prefix Tree)

A Trie is a tree where each node represents a character. It excels at prefix-based operations.

**Use cases:** Autocomplete, spell checking, IP routing, dictionary operations.

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEndOfWord = false;
}

class Trie {
    private TrieNode root = new TrieNode();
    
    // Insert a word — O(m) where m = word length
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }
    
    // Search exact word — O(m)
    public boolean search(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return false;
            node = node.children[idx];
        }
        return node.isEndOfWord;
    }
    
    // Check if any word starts with prefix — O(m)
    public boolean startsWith(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return false;
            node = node.children[idx];
        }
        return true; // prefix exists
    }
    
    // Get all words with given prefix (for autocomplete)
    public List<String> getWordsWithPrefix(String prefix) {
        List<String> result = new ArrayList<>();
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return result;
            node = node.children[idx];
        }
        dfs(node, new StringBuilder(prefix), result);
        return result;
    }
    
    private void dfs(TrieNode node, StringBuilder current, List<String> result) {
        if (node.isEndOfWord) result.add(current.toString());
        for (int i = 0; i < 26; i++) {
            if (node.children[i] != null) {
                current.append((char)('a' + i));
                dfs(node.children[i], current, result);
                current.deleteCharAt(current.length() - 1); // backtrack
            }
        }
    }
}
```

**Trie Complexity:**
- Insert: O(m) time, O(m × alphabet) space
- Search: O(m)
- StartsWith: O(m)
- Total space: O(N × m × alphabet) where N = number of words

---

## Classic Problems

| Problem | Technique | Complexity |
|---------|-----------|-----------|
| Max depth of binary tree | DFS (return height) | O(n) |
| Min depth | BFS (first leaf) | O(n) |
| Diameter of tree | DFS with global max | O(n) |
| Level order | BFS | O(n) |
| Zigzag level order | BFS + reverse alternating levels | O(n) |
| Lowest Common Ancestor | Recursive post-order | O(n) |
| Path Sum II | DFS + backtracking | O(n²) |
| Flatten to linked list | Preorder + rewire | O(n) |
| Construct BST from inorder | Recursion | O(n) |
| Validate BST | DFS with min/max bounds | O(n) |
| Kth smallest in BST | Inorder traversal | O(k) |
| Word Search in grid | DFS + Trie | O(m×n×4^L) |

---

## Interview Questions

### Q: Difference between inorder, preorder, postorder?

- **Inorder (L, Root, R):** Gives sorted order for BST. Used to validate BST.
- **Preorder (Root, L, R):** Root first — used for serialization, copying tree.
- **Postorder (L, R, Root):** Children before parent — used for deletion, expression evaluation.

### Q: How to check if two trees are identical?

```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) return true;
    if (p == null || q == null) return false;
    return p.val == q.val && isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}
```

### Q: Find the Kth smallest in a BST.

```java
public int kthSmallest(TreeNode root, int k) {
    int[] count = {0};
    int[] result = {0};
    inorder(root, count, result, k);
    return result[0];
}
private void inorder(TreeNode node, int[] count, int[] result, int k) {
    if (node == null) return;
    inorder(node.left, count, result, k);
    if (++count[0] == k) { result[0] = node.val; return; }
    inorder(node.right, count, result, k);
}
// O(k) average if tree is balanced
```

---

## 🎯 Famous Interview Problems

> Sorted: Basic → Easy → Medium → Hard → Expert

---

### 🟢 BASIC

---

#### 1. Maximum Depth of Binary Tree (LeetCode #104)

**Problem:** Find the maximum depth (height) of a binary tree.

```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
// Time: O(n)  Space: O(h) — h = height (O(log n) balanced, O(n) skewed)
```

---

#### 2. Same Tree (LeetCode #100)

**Problem:** Check if two binary trees are structurally identical with the same node values.

```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) return true;
    if (p == null || q == null) return false;
    return p.val == q.val && isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}
// Time: O(n)  Space: O(h)
```

---

#### 3. Invert Binary Tree (LeetCode #226)

**Problem:** Invert (mirror) a binary tree.

```java
public TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    TreeNode tmp = root.left;
    root.left = invertTree(root.right);
    root.right = invertTree(tmp);
    return root;
}
// Time: O(n)  Space: O(h)
```

---

### 🟡 EASY

---

#### 4. Level Order Traversal (LeetCode #102)

**Problem:** Return the level-order traversal of a binary tree's node values.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
// Time: O(n)  Space: O(n)
```

---

#### 5. Validate Binary Search Tree (LeetCode #98)

**Problem:** Determine if a binary tree is a valid BST.

**Wrong approach:** Only check each node against its parent — misses cases like left subtree having a value > grandparent.

**Correct approach:** Pass valid range (min, max) down recursion.
```java
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) && validate(node.right, node.val, max);
}
// Time: O(n)  Space: O(h)
```

---

#### 6. Symmetric Tree (LeetCode #101)

**Problem:** Check if a binary tree is a mirror of itself (symmetric around its center).

```java
public boolean isSymmetric(TreeNode root) {
    return isMirror(root.left, root.right);
}
private boolean isMirror(TreeNode l, TreeNode r) {
    if (l == null && r == null) return true;
    if (l == null || r == null) return false;
    return l.val == r.val && isMirror(l.left, r.right) && isMirror(l.right, r.left);
}
// Time: O(n)  Space: O(h)
```

---

### 🟠 MEDIUM

---

#### 7. Lowest Common Ancestor of BST (LeetCode #235)

**Problem:** Find the LCA of two nodes p and q in a BST.

**Optimized (use BST property):**
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while (root != null) {
        if (p.val < root.val && q.val < root.val) root = root.left;
        else if (p.val > root.val && q.val > root.val) root = root.right;
        else return root;  // root is between p and q (or is p or q)
    }
    return null;
}
// Time: O(h)  Space: O(1) — iterative, no recursion stack
```

---

#### 8. Lowest Common Ancestor (General Binary Tree) (LeetCode #236)

**Problem:** Find the LCA of two nodes in a binary tree (not necessarily BST).

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root;  // p and q on different sides
    return left != null ? left : right;
}
// Time: O(n)  Space: O(h)
```

---

#### 9. Binary Tree Right Side View (LeetCode #199)

**Problem:** Return the values of nodes visible from the right side of a binary tree.

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            if (i == size - 1) result.add(node.val);  // rightmost node
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    return result;
}
// Time: O(n)  Space: O(n)
```

---

#### 10. Kth Smallest Element in BST (LeetCode #230)

**Problem:** Find the kth smallest element in a BST.

**Key insight:** Inorder traversal of BST gives sorted order.
```java
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    int count = 0;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) { stack.push(curr); curr = curr.left; }
        curr = stack.pop();
        if (++count == k) return curr.val;
        curr = curr.right;
    }
    return -1;
}
// Time: O(h + k)  Space: O(h)
```

---

#### 11. Serialize and Deserialize Binary Tree (LeetCode #297)

**Problem:** Design an algorithm to serialize and deserialize a binary tree.

```java
// Serialize: preorder with null markers
public String serialize(TreeNode root) {
    if (root == null) return "null,";
    return root.val + "," + serialize(root.left) + serialize(root.right);
}

// Deserialize: reconstruct from preorder
public TreeNode deserialize(String data) {
    Queue<String> q = new LinkedList<>(Arrays.asList(data.split(",")));
    return build(q);
}
private TreeNode build(Queue<String> q) {
    String val = q.poll();
    if (val.equals("null")) return null;
    TreeNode node = new TreeNode(Integer.parseInt(val));
    node.left = build(q);
    node.right = build(q);
    return node;
}
// Time: O(n)  Space: O(n)
```

---

### 🔴 HARD

---

#### 12. Binary Tree Maximum Path Sum (LeetCode #124)

**Problem:** Find the maximum path sum in a binary tree. A path can start and end at any node.

```java
int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    maxGain(root);
    return maxSum;
}

private int maxGain(TreeNode node) {
    if (node == null) return 0;
    int leftGain = Math.max(maxGain(node.left), 0);   // ignore negative paths
    int rightGain = Math.max(maxGain(node.right), 0);
    // Path through this node (can't branch further up)
    maxSum = Math.max(maxSum, node.val + leftGain + rightGain);
    // Return the max gain from this node going up (can only use one branch)
    return node.val + Math.max(leftGain, rightGain);
}
// Time: O(n)  Space: O(h)
```

---

#### 13. Diameter of Binary Tree (LeetCode #543)

**Problem:** Find the length of the longest path between any two nodes in a tree (doesn't need to pass through root).

```java
int maxDiameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return maxDiameter;
}

private int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    int right = height(node.right);
    maxDiameter = Math.max(maxDiameter, left + right);
    return 1 + Math.max(left, right);
}
// Time: O(n)  Space: O(h)
```

---

#### 14. Word Search II (LeetCode #212) — Trie + DFS

**Problem:** Find all words from a dictionary that exist in a 2D board of characters.

**Brute:** Run DFS for each word → O(words × m × n × 4^L).

**Optimized:** Build a Trie from dictionary, then DFS once on board.
```java
public List<String> findWords(char[][] board, String[] words) {
    TrieNode root = buildTrie(words);
    List<String> result = new ArrayList<>();
    for (int r = 0; r < board.length; r++)
        for (int c = 0; c < board[0].length; c++)
            dfs(board, r, c, root, result);
    return result;
}

private void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
    char ch = board[r][c];
    if (ch == '#' || node.children[ch - 'a'] == null) return;
    node = node.children[ch - 'a'];
    if (node.word != null) { result.add(node.word); node.word = null; }  // avoid duplicates
    board[r][c] = '#';  // mark visited
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    for (int[] d : dirs) {
        int nr = r + d[0], nc = c + d[1];
        if (nr >= 0 && nr < board.length && nc >= 0 && nc < board[0].length)
            dfs(board, nr, nc, node, result);
    }
    board[r][c] = ch;  // restore
}
// Time: O(m × n × 4^L) with Trie pruning — much faster in practice
// Space: O(total chars in dictionary)
```

---

#### 15. Construct Binary Tree from Preorder and Inorder Traversal (LeetCode #105)

**Problem:** Build a unique binary tree from its preorder and inorder traversal arrays.

```java
public TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> inorderIdx = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) inorderIdx.put(inorder[i], i);
    return build(preorder, 0, 0, inorder.length - 1, inorderIdx);
}

private int[] pre;
private TreeNode build(int[] preorder, int preStart, int inStart, int inEnd,
                       Map<Integer, Integer> inorderIdx) {
    if (inStart > inEnd) return null;
    int rootVal = preorder[preStart];
    int inRoot = inorderIdx.get(rootVal);
    int leftSize = inRoot - inStart;
    TreeNode root = new TreeNode(rootVal);
    root.left = build(preorder, preStart + 1, inStart, inRoot - 1, inorderIdx);
    root.right = build(preorder, preStart + 1 + leftSize, inRoot + 1, inEnd, inorderIdx);
    return root;
}
// Time: O(n)  Space: O(n)
```

---

## ⚡ Flashcard Summary

| # | Concept | Key Point |
|---|---------|-----------|
| 1 | Inorder BST | Gives sorted sequence |
| 2 | Preorder use case | Serialization, copying |
| 3 | Postorder use case | Deletion, expression trees |
| 4 | BFS tool | Queue — level by level |
| 5 | DFS tool | Stack (iterative) or recursion |
| 6 | BST delete 2 children | Replace with inorder successor (min of right subtree) |
| 7 | Validate BST | Pass min/max bounds down recursion |
| 8 | Diameter pattern | At each node: left_height + right_height; track global max |
| 9 | LCA pattern | If both p and q found below — current node is LCA |
| 10 | Trie insert | O(m) — traverse/create nodes for each char |
| 11 | Trie startsWith | O(m) — traverse; return true if all chars found |
| 12 | Height of tree | O(n) DFS: 1 + max(left, right) |
| 13 | Balanced BST ops | O(log n) all; unbalanced → O(n) |
| 14 | TreeMap in Java | Red-Black Tree; O(log n) get/put; ordered iteration |
| 15 | Trie vs HashMap | Trie: prefix queries, memory efficient for shared prefixes |

---

*Next: [Heaps and Priority Queues](06-heaps-and-priority-queues.md)*
