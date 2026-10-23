# Exceptions in Java

## 1. What Is It?
An exception is an event that occurs during the execution of a program that disrupts the normal flow of instructions (e.g., dividing by zero, accessing a missing file, or a network timeout). 

## 2. Why Do We Need Exception Handling?
Without exception handling, when an error occurs, the JVM prints a stack trace and the entire program crashes. Exception handling provides a structured mechanism to catch these errors, execute fallback or cleanup logic, and allow the program to continue running or shut down gracefully.

## 3. How Does It Work? (The try-catch-finally block)
- **`try` block:** Contains the risky code that might throw an exception.
- **`catch` block:** Contains the logic to handle the exception if it is thrown.
- **`finally` block:** Contains code that will *always* execute, regardless of whether an exception was thrown or caught. Typically used to release resources (like closing database connections or file streams).

## 4. Checked vs Unchecked Exceptions
The `Throwable` class is the root of the hierarchy, splitting into `Error` (severe system problems like `OutOfMemoryError`) and `Exception`.
- **Checked Exceptions:** Exceptions that the compiler *forces* you to handle (via `try-catch` or `throws`). They extend `Exception` (but not `RuntimeException`). Examples: `IOException`, `SQLException`. They represent conditions outside the immediate control of the program (like a network failure).
- **Unchecked Exceptions:** Exceptions that the compiler does *not* force you to handle. They extend `RuntimeException`. Examples: `NullPointerException`, `IndexOutOfBoundsException`. They usually represent programming bugs.

## 5. The throw vs throws Keywords
- **`throw`:** Used to manually trigger an exception from within a method body. (`throw new IllegalArgumentException("Invalid ID");`)
- **`throws`:** Used in a method signature to declare that this method *might* throw a checked exception, pushing the responsibility of handling it to the caller.

## 6. Exception Chaining
Exception chaining allows you to catch a low-level technical exception (e.g., `SQLException`) and throw a new, higher-level business exception (e.g., `UserCreationException`), while attaching the original exception as the "cause".
```java
try {
    database.save(user);
} catch (SQLException e) {
    // Wrap and rethrow
    throw new UserCreationException("Failed to create user", e);
}
```
**Why is this useful?** It provides a clean, readable message to the user/API while preserving the original technical stack trace so developers can debug the root cause.

## 7. Custom Exceptions
When built-in exceptions don't clearly describe your business logic failure, you create a custom exception by extending `Exception` (for checked) or `RuntimeException` (for unchecked).
```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

## 8. Interview Questions

### Q. How can you catch multiple exceptions in Java?
#### Short Interview Answer
You can use multiple sequential `catch` blocks, ordering them from the most specific exception subclass to the most general superclass. Alternatively, since Java 7, you can use a multi-catch block catching several exceptions in a single block using the pipe `|` symbol (e.g., `catch (IOException | SQLException e)`).

### Q. Can you throw a checked exception inside a Lambda Expression?
#### Short Interview Answer
Yes, but only if the functional interface that the lambda is implementing declares that checked exception in its `throws` clause. If it doesn't (like `Runnable` or `Consumer`), you cannot throw a checked exception directly; you must wrap it in a `try-catch` block inside the lambda and either handle it or rethrow it as an unchecked `RuntimeException`.

---

[← Previous: Java Strings & Immutability](12-strings.md) | [Next: Collections Framework →](14-collections.md)
