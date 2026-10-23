# Multithreading in Java

## 1. What Is It?
Multithreading is a feature that allows a program to execute multiple tasks simultaneously. A thread is the smallest unit of execution within a program. 

## 2. Why Do We Need It?
By enabling multithreading, an application can perform several operations concurrently—such as serving multiple HTTP requests, reading files, writing logs, or doing background calculations—without blocking the main execution flow. This makes applications faster, more responsive, and capable of handling heavy workloads.

## 3. How Can You Create a Thread?
In Java, there are two main ways to create a thread:
1. **Extending the `Thread` class:**
   You inherit from `Thread` and override its `run()` method.
2. **Implementing the `Runnable` interface:**
   You implement `Runnable` and override its `run()` method, then pass the instance to a `Thread` constructor.

**Which is better?**
Implementing `Runnable` is highly preferred. Java does not support multiple class inheritance. If you extend `Thread`, your class cannot extend any other class. Implementing `Runnable` keeps your class hierarchy flexible and clearly separates the task (the `Runnable`) from the runner (the `Thread`).

## 4. Thread States and Lifecycle
Java threads go through a fixed set of states defined in the `Thread.State` enum:

- **NEW:** The `Thread` object has been created (using `new`), but the `start()` method has not been called yet.
- **RUNNABLE:** The thread enters this state immediately after calling `start()`. It means the thread is eligible to run; it might be actively running on the CPU or waiting for the OS thread scheduler to assign it CPU time.
- **BLOCKED:** The thread tries to enter a `synchronized` block or method that is currently locked by another thread. It is waiting for the monitor lock to become available.
- **WAITING:** The thread is waiting indefinitely for another thread to perform a specific action. Happens when calling `Object.wait()` without a timeout, or `Thread.join()` without a timeout.
- **TIMED_WAITING:** The thread waits for a specific amount of time. Happens when calling `Thread.sleep(milliseconds)` or timed versions of `wait()`.
- **TERMINATED:** The `run()` method has completed normally, or ended because of an unhandled exception. The thread is permanently dead and cannot be restarted.

## 5. Synchronization and Race Conditions
- **Race Condition:** When multiple threads update the same shared variable at the exact same time, the program may produce unpredictable, incorrect results (e.g., two threads incrementing a counter simultaneously may result in the counter increasing by 1 instead of 2).
- **Synchronization:** The `synchronized` keyword ensures that only one thread at a time can execute a critical section of code. When a thread enters a synchronized block, it acquires the object's monitor lock. No other thread can enter any synchronized section using that same lock until the first thread releases it.

## 6. Interview Questions

### Q. What happens if you call `run()` directly instead of `start()`?
#### Short Interview Answer
If you call `run()` directly, the code will execute synchronously on the current thread (the main thread). It will not spawn a new separate path of execution. The `start()` method is required to tell the JVM to allocate a new native thread and then invoke the `run()` method inside that new thread.

---

[← Previous: Design Patterns in Java](18-design-patterns-backend.md) | [Next: Concurrency (Executors & Callable) →](23-concurrency.md)
