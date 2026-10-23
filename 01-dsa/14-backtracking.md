# Backtracking: The Systematic Search

Backtracking is a form of recursion used to systematically explore all possible configurations (or "states") in a search space. If a configuration is found to be invalid, the algorithm "backtracks" to the previous state and tries a different path.

## 1. The Universal Backtracking Template

Memorize this template. Almost every backtracking problem in an interview fits this exact structure.

```java
private void backtrack(State state, List<Result> results, /* specific parameters */) {
    // 1. Base case: have we reached a valid solution?
    if (isGoal(state)) {
        results.add(new Result(state)); // NOTE: Must create a DEEP COPY!
        return;
    }
    
    // 2. Iterate through all possible choices at the current state
    for (Choice choice : getChoices(state)) {
        // 3. Check constraints (pruning)
        if (isValid(choice, state)) {
            
            // 4. CHOOSE: apply the choice to the state
            apply(choice, state);
            
            // 5. EXPLORE: recurse to the next state
            backtrack(state, results, ...);
            
            // 6. UNCHOOSE: undo the choice (backtrack)
            undo(choice, state);
        }
    }
}
```

## 2. Critical Analysis for Every Problem

Before coding a backtracking solution, explicitly state:
1.  **State:** What variables define my current position? (e.g., current index, current path list).
2.  **Choices:** What are my options right now?
3.  **Constraints:** How do I know a choice is invalid early on? (Pruning).
4.  **Base case:** When do I stop and add to the result?
5.  **Undo operation:** How do I revert the state?

## 3. Core SDE-1 / SDE-2 Backtracking Problems

### A. Subsets (Power Set)
*   **State:** Current index, current subset (`List<Integer>`).
*   **Choices:** Include the current element OR exclude it.
*   **Base case:** Reached the end of the array.
```java
// Example snippet
public void generateSubsets(int[] nums, int index, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current)); // Deep copy is critical
    
    for (int i = index; i < nums.length; i++) {
        current.add(nums[i]);       // Choose
        generateSubsets(nums, i + 1, current, result); // Explore
        current.remove(current.size() - 1); // Unchoose
    }
}
```
*Complexity:* $O(N \cdot 2^N)$ time, $O(N)$ space.

### B. Permutations
*   **State:** Current permutation.
*   **Choices:** Any unused element.
*   **Constraints:** Can use a `boolean[] used` array or swap elements in-place.
*   **Base case:** Current permutation length == input length.
*Complexity:* $O(N \cdot N!)$ time, $O(N)$ space.

### C. Combinations / Combination Sum
*   **Choices:** Which element to pick next. To avoid duplicates (like `[2,3]` and `[3,2]`), only pick elements at or after the current index.
*   **Constraints:** Target sum exceeded.

### D. Palindrome Partitioning
*   **Choices:** Where to cut the string next.
*   **Constraints:** Is the substring from `start` to `end` a palindrome?

### E. Grid-Based Backtracking (Word Search, N-Queens, Sudoku)
*   **State:** Current cell `(row, col)`.
*   **Choices:** Go Up, Down, Left, Right (Word Search) OR put a queen in column `c` (N-Queens).
*   **Constraints:** Bounds checking, visited cells (use a `boolean[][] visited` or mutate grid temporarily).
*   **Undo:** `visited[r][c] = false;` or reset grid cell.

## 4. Complexity Analysis Rules of Thumb
*   **Permutations:** $O(N!)$
*   **Subsets / Combinations:** $O(2^N)$
*   **N-Queens / Sudoku:** $O(N!)$ or $O(9^{81})$ (upper bounds, highly pruned).

## 5. Interview Pitfalls
*   **Adding References:** `result.add(currentPath)`. When the function finishes, `currentPath` is empty. You MUST do `result.add(new ArrayList<>(currentPath))`.
*   **Forgetting to Unchoose:** If you modify a shared state (like a `StringBuilder` or `List`), you must revert it before the next loop iteration. String concatenation (`path + "A"`) creates a new object and avoids explicit unchoosing, but wastes memory. Using `StringBuilder.deleteCharAt()` is much faster but requires explicit undo.
