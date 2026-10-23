# JVM and Memory Management

## 1. What is it?
The JVM is an abstract computing machine that enables a computer to run a Java program. It manages memory automatically via Garbage Collection.

## 2. Memory Architecture (Internals)
- **Heap:** Shared among all threads. Stores objects (`new Keyword`). This is where Garbage Collection happens.
- **Stack:** One per thread. Stores local variables (primitives) and references to objects in the heap. Method calls push frames here.
- **Metaspace (Java 8+):** Replaced PermGen. Stores class metadata, static variables, and constants. Grows dynamically into native memory.

## 3. Garbage Collection (GC Roots)
An object is eligible for GC if it is not reachable from a **GC Root**.
**What is a GC Root?**
- Local variables in the thread stack.
- Active Java threads.
- Static variables.
- JNI references.

## 4. Production Debugging
**Symptom:** `java.lang.OutOfMemoryError: Java heap space`
**Root Cause:** A memory leak (holding references to objects longer than needed) or genuinely processing too much data at once.
**Backend Debugging:**
1. Configure JVM to generate a dump on crash: `-XX:+HeapDumpOnOutOfMemoryError`.
2. Analyze the `.hprof` file using Eclipse MAT or VisualVM.
3. Look for the "Dominator Tree" to find the collection holding all the memory (often an unbounded `List` or `Map` acting as a local cache).

## 5. Interview Questions
**Q: Does Java have memory leaks?**
**Answer:** Yes. While Java handles reclaiming *unreachable* objects, a logical leak occurs when a developer unintentionally maintains a reference to an object. Example: adding objects to a static `HashMap` cache and never removing them.

**Q: Pass-by-value or Pass-by-reference?**
**Answer:** Java is strictly **pass-by-value**. However, when passing an object, the *value* passed is the *reference* (pointer) to the object. You can mutate the object's state, but you cannot reassign the original caller's reference to a new object.

---

[← Previous: HashMap Internals](16-hashmap-internals.md) | [Next: Design Patterns in Java →](18-design-patterns-backend.md)
