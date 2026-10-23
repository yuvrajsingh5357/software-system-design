# Production Debugging: CPU and Memory Issues

## 1. OutOfMemoryError (OOM) / Memory Leaks

### Symptoms
- The JVM throws `java.lang.OutOfMemoryError: Java heap space`.
- Application becomes extremely slow before crashing (spending 98% of CPU time doing Garbage Collection trying to free space).

### Possible Causes
- **Memory Leak:** Objects are stored in a Collection (like a static `HashMap`) and never removed. The Garbage Collector cannot delete them because a valid reference still exists.
- **Querying Too Much Data:** A `SELECT *` from a database returning 5 million rows into a `List<User>`, exceeding the JVM Heap size.
- **Unclosed Resources:** Not closing DB connections or input streams.

### Debugging Process
1. **Enable Heap Dumps:** Add `-XX:+HeapDumpOnOutOfMemoryError` to JVM args. When OOM occurs, the JVM will write a `.hprof` file to disk.
2. **Analyze the Dump:** Open the file using tools like **Eclipse MAT (Memory Analyzer Tool)** or VisualVM.
3. **Find the Leak:** MAT will show you the "Dominator Tree" and the "Leak Suspects" report, revealing exactly which class/collection is holding onto 90% of the RAM.

## 2. High CPU Utilization

### Symptoms
- CPU usage is consistently at 99%.
- Application response times spike.

### Possible Causes
- **Infinite Loops:** A `while(true)` condition that never breaks.
- **Inefficient Algorithms:** Doing an $O(N^2)$ operation on a massive dataset instead of $O(N)$.
- **GC Thrashing:** Memory is almost full, so the JVM continuously runs full Garbage Collection cycles, burning CPU cycles without doing actual work.

### Debugging Process
1. **Find the Process:** Use Linux `top` to find the Java Process ID (PID) consuming CPU.
2. **Find the Thread:** Use `top -H -p <PID>` to see which specific thread inside the Java process is burning CPU. Note the Thread ID (TID) and convert it to Hexadecimal.
3. **Take a Thread Dump:** Run `jcmd <PID> Thread.print` or `jstack <PID> > threaddump.txt`.
4. **Locate the Code:** Open the thread dump, search for the Hexadecimal Thread ID, and look at its stack trace. It will point you to the exact Java Class and Line Number executing the infinite loop.

## 3. Blocked Threads / Deadlocks

### Symptoms
- Application CPU is very low, but no requests are being processed.
- API requests hang indefinitely until they time out.

### Possible Causes
- **Deadlock:** Thread A holds Lock 1 and waits for Lock 2. Thread B holds Lock 2 and waits for Lock 1.
- **Connection Pool Exhaustion:** All 10 DB connections are in use, and new requests are waiting forever for a connection.

### Debugging Process
1. Take a Thread Dump (`jstack`).
2. Search the dump for the word `BLOCKED`. 
3. If it's a deadlock, `jstack` automatically detects it and prints a "Found one Java-level deadlock" report at the bottom, showing exactly which threads and locks are involved.

---

[← Previous: Observability (Logs, Metrics, Traces)](../17-observability/01-observability-metrics.md) | [Next: Backend System Design Integration (Bridging HLD & Code) →](../14-backend-system-design-integration/01-implementation-bridge.md)
