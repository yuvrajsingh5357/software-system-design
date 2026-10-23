# Java Strings

## 1. What Is It?
In Java, a `String` is an object that represents a sequence of characters (e.g., `"Hello", "Java"`). It is widely used for text manipulation, input handling, and data transfer.

## 2. Important Rule: Immutability
The most critical feature of the Java `String` class is that it is **immutable** and **final**.
- **Immutable:** Once a String object is created, its state (the characters inside it) cannot be modified. Any operation that appears to modify a String (like concatenation or `.toUpperCase()`) actually creates and returns a brand-new String object in memory, leaving the original unchanged.
- **Final:** The `String` class is declared as `final`, meaning no other class can extend it and override its behavior.

## 3. Why Is String Immutable?
This was a deliberate design decision by the creators of Java for several reasons:
- **String Constant Pool:** Because Strings never change, the JVM can safely cache them. If 1,000 users have the role `"ADMIN"`, the JVM only needs to store one `"ADMIN"` object in memory. All 1,000 references point to the same object.
- **Thread Safety:** Immutable objects are inherently thread-safe. Multiple threads can read the same String object concurrently without any synchronization overhead.
- **Security:** Strings are used everywhere for sensitive data (database URLs, network ports, file paths). If Strings were mutable, a malicious thread could change a file path *after* a security check has passed but *before* the file is opened.
- **Caching HashCodes:** Because the contents never change, a String's hash code never changes. The `String` class calculates its hash code once and caches it, making it extremely fast and the ideal key for a `HashMap`.

## 4. Internal Working: The String Constant Pool
The String Constant Pool is a special area inside the JVM **Heap Memory**.
- **Literal Creation:** `String s1 = "Java";`
  The JVM checks the pool. If `"Java"` exists, `s1` points to it. If not, it creates it in the pool.
- **New Keyword Creation:** `String s2 = new String("Java");`
  This forces the JVM to create a brand-new String object in the main Heap memory, completely bypassing the pool's caching mechanism. (This is generally considered bad practice unless specifically required).

## 5. Comparisons: == vs .equals()
- `==` operator: Checks **Reference Equality**. Do these two variables point to the exact same memory address?
- `.equals()` method: Checks **Value Equality**. Do these two objects contain the exact same characters?

```java
String a = "Hello";
String b = "Hello";
String c = new String("Hello");

System.out.println(a == b);      // true (Both point to the same object in the Pool)
System.out.println(a == c);      // false (c is a new object in the Heap)
System.out.println(a.equals(c)); // true (They contain the same text)
```

## 6. Interview Questions

### Q. What is the difference between String, StringBuilder, and StringBuffer?
#### Short Interview Answer
- `String` is immutable. Every modification creates a new object, which is slow for heavy concatenations.
- `StringBuilder` is mutable. It allows you to append and modify text in place without creating new objects. It is fast but not thread-safe.
- `StringBuffer` is also mutable but its methods are synchronized, making it thread-safe but slower than `StringBuilder`.

### Q. How do you find the length of a string in Java?
#### Short Interview Answer
You use the `length()` method. For example, `"Java".length()` returns `4`. It counts every single character, including letters, numbers, and spaces.

---

[← Previous: Polymorphism (Overloading & Overriding)](10-polymorphism.md) | [Next: Exception Handling →](13-exceptions.md)
