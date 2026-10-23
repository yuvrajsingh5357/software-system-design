# Concurrency in Java: Executors and Callable

## 1. What Is It?
While raw multithreading (`Thread` and `Runnable`) provides concurrency, managing dozens or hundreds of threads manually is difficult and inefficient. The `java.util.concurrent` package provides high-level APIs like `ExecutorService` and `Callable` to manage concurrent tasks efficiently.

## 2. What Is ExecutorService?
`ExecutorService` is a framework that simplifies thread management.
- Instead of creating threads manually (`new Thread(task).start()`), you submit tasks to the `ExecutorService`.
- It handles thread creation, scheduling, execution, and reuse automatically.

### Why use a Thread Pool?
`ExecutorService` internally uses a **Thread Pool** (a set of worker threads created in advance). Creating and destroying native OS threads is computationally expensive. A thread pool reuses existing threads for multiple tasks, significantly improving performance and preventing the application from crashing due to thread exhaustion.

## 3. Callable vs Runnable
Both are functional interfaces representing tasks to be executed by threads, but they have critical differences:

| Feature | `Runnable` | `Callable<V>` |
|---------|------------|---------------|
| **Return Value** | `void` (Returns nothing) | Returns a value of generic type `V`. |
| **Method** | `run()` | `call()` |
| **Exceptions** | Cannot throw checked exceptions. | Can throw checked exceptions. |

## 4. The Future Object
When you submit a `Callable` task to an `ExecutorService`, it immediately returns a `Future` object.
- The `Future` acts as a placeholder for the result that the `Callable` will produce in the future.
- You can use the `Future` to check if the task is done (`isDone()`), or wait for it to finish and retrieve the result using the `get()` method.
- **Note:** Calling `future.get()` is a *blocking* operation. The calling thread will pause until the `Callable` finishes and returns the value.

## 5. Production Relevance
- **Safe Shutdown:** Always shut down your `ExecutorService` when it's no longer needed (e.g., during application shutdown). If you don't, the worker threads will remain alive, preventing the JVM from exiting cleanly. Use `executor.shutdown()` or `executor.shutdownNow()`.
- **Spring Context:** In Spring Boot, you rarely instantiate an `ExecutorService` manually. You use the `@Async` annotation, which uses a Spring-managed `ThreadPoolTaskExecutor` under the hood.

## 6. Interview Questions

### Q. What is the difference between Runnable and Callable?
#### Short Interview Answer
`Runnable`'s `run()` method returns void and cannot throw checked exceptions. `Callable`'s `call()` method returns a generic value and can throw checked exceptions. When submitted to an `ExecutorService`, a `Callable` returns a `Future` object which allows you to retrieve the result once the background task finishes.

---

[← Previous: Multithreading & Synchronization](22-multithreading.md) | [Next: Spring Core Fundamentals (IoC) →](../03-spring-core/01-spring-core-fundamentals.md)
