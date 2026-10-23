# Timed Interview Practice

Practicing without a timer gives a false sense of security. An interview tests your ability to solve problems *under pressure*. 

Use this guide to structure your mock sessions.

## 1. Time Limits by Difficulty

### The 20-Minute Problem (Easy / Warm-up)
*   **0-3 mins:** Read, clarify, find edge cases.
*   **3-8 mins:** Think, explain brute force, identify optimal pattern.
*   **8-15 mins:** Code perfectly.
*   **15-20 mins:** Dry run, complexity analysis, handle follow-ups.

### The 35-Minute Problem (Standard Medium)
*   **0-5 mins:** Read, define constraints, identify hidden requirements.
*   **5-15 mins:** Develop optimal approach. Discuss trade-offs (e.g. "I can do this in $O(N)$ space with a HashMap, or $O(1)$ space if I sort it first, taking $O(N \log N)$ time. Which do you prefer?").
*   **15-28 mins:** Implementation.
*   **28-35 mins:** Testing with an edge case, complexity analysis.

### The 45-Minute Problem (Hard / Complex Medium)
*   **0-10 mins:** Thoroughly understand the problem. Trace a complex example manually.
*   **10-25 mins:** Formulate approach. If it's DP, explicitly define state and transitions on the "whiteboard".
*   **25-40 mins:** Careful implementation. 
*   **40-45 mins:** Dry run and debugging.

## 2. Self-Evaluation Rubric

After every timed practice, rate yourself strictly:

| Criteria | 1 (Needs Work) | 2 (Developing) | 3 (Interview Ready) | 4 (Strong) |
| :--- | :--- | :--- | :--- | :--- |
| **Understanding** | Jumped to code immediately | Needed heavy hints to understand | Understood with 1-2 clarifying questions | Proactively identified missing constraints |
| **Pattern ID** | Blanked out | Chose wrong pattern initially | Found correct pattern after brute force | Instantly recognized pattern and optimal approach |
| **Communication** | Silent while thinking/coding | Mumbled, hard to follow | Explained steps clearly before coding | Treated it like a pair-programming session |
| **Implementation** | Messy, incomplete | Functional but buggy/slow | Clean code, minor syntax errors | Flawless, modular, readable Java code |
| **Testing** | Skipped testing | Tested only happy path | Found bugs during own dry run | Tested edge cases (null, empty, negative) before interviewer pointed them out |

*(Note: Do not use this to calculate a "hire/no-hire" score. Use it to identify your weakest phase and drill it.)*
