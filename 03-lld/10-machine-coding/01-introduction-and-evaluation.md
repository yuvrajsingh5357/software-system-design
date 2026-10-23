# Machine Coding: Introduction & Evaluation

Machine coding rounds (common at Flipkart, Swiggy, Uber, etc.) require you to design and implement a fully functional, extensible, and clean LLD system within a strict time limit (usually 90-120 minutes).

These rounds test your transition from whiteboard design to production-ready code.

## 1. The Structure of a Machine Coding Exercise

Each problem in this directory is presented in **Interview Mode**. You are given:
1.  **Problem Statement:** The core business problem.
2.  **Functional Requirements:** What the code *must* do.
3.  **Expected APIs/Commands:** The interface the driver code will call.
4.  **Constraints:** Time limits, memory limits, or design restrictions.

*Do not look at the reference solution until you have written running code.*

## 2. Evaluation Rubric

Rate your implementation after the time expires:

| Area | Needs Improvement | Developing | Interview-Ready | Strong |
| :--- | :--- | :--- | :--- | :--- |
| **Requirements** | Missed core features | Functional but ignored edge cases | Met all requirements | Met all requirements + handled unspoken edge cases |
| **Domain Modeling** | Procedural code in a single class | Basic classes, heavy coupling | Clear entities, good encapsulation | Rich domain model, perfect SRP |
| **Extensibility** | Hardcoded logic, `if-else` chains | Tried to use interfaces, but tightly coupled | Easy to add new features without modifying core | Perfect use of Design Patterns (Strategy, Factory) |
| **Concurrency (SDE-2)** | Ignored thread safety | Added `synchronized` to everything | Used ConcurrentHashMap/Locks where appropriate | Minimal lock contention, atomic operations |
| **Testing** | No tests, didn't run | Manual testing in `main()` | Basic JUnit happy path | Extensive JUnit covering edge/failure cases |

## 3. The 90-Minute Strategy

1.  **Read & Clarify (5 mins):** Note down the entities.
2.  **Model & Interfaces (15 mins):** Write out the POJOs/Entities and the main Service interfaces. Do NOT write business logic yet.
3.  **Core Implementation (40 mins):** Implement the core features. Focus on correctness over elegance first.
4.  **Extensibility Refactoring (15 mins):** If you hardcoded a calculation, extract it to a Strategy. If object creation is messy, add a Factory.
5.  **Driver Code / Tests (15 mins):** Prove that it works. 

## 4. Problem Bank

Proceed to the exercises in this folder (e.g., `LRU Cache`, `Parking Lot`, `Splitwise`).
