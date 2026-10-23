# Java Program Execution Lifecycle

## 1. What Is It?
The Java Program Execution Lifecycle describes the journey of Java code from human-readable source code to machine-executable instructions.

## 2. How Does It Work?
The lifecycle consists of four main steps: Writing, Compiling, Bytecode generation, and Execution.

### Step 1: Writing the Source Code
A developer writes code in a `.java` file (e.g., `MyClass.java`).

### Step 2: Compiling with javac
The Java compiler (`javac`) reads the source code line by line and checks it for syntax errors. If everything is correct, the compiler converts the source code into an intermediate format called bytecode.

### Step 3: Bytecode and Platform Independence
The compiler generates a `.class` file (e.g., `MyClass.class`). This file contains the bytecode. This bytecode is platform-independent—it can run on any operating system as long as that system has a JVM installed.

### Step 4: Execution with JVM
The JVM is the engine that runs the program. It acts as a translator and runtime environment.
1. The JVM loads the `MyClass.class` file.
2. It performs a verification step to ensure the bytecode is safe and valid.
3. Once verified, the JVM executes the bytecode. It translates it into native machine instructions and runs the program.

## 3. Internal Working of JVM Execution
When the JVM executes the bytecode, it can do so in two ways:
- **Interpretation:** Reading and executing the bytecode line-by-line.
- **Just-In-Time (JIT) Compilation:** Converting frequently executed bytecode into native machine code on the fly for faster performance.

## 4. Interview Questions

### Q. What happens during the bytecode verification step?
#### Short Interview Answer
Before executing bytecode, the JVM's Bytecode Verifier checks the `.class` file to ensure it doesn't violate security constraints, perform illegal memory accesses, or contain structural errors. This prevents malicious or corrupted code from crashing the JVM.

---

[← Previous: JDK, JRE, JVM Architecture](02-jdk-jre-jvm.md) | [Next: JVM Internals (ClassLoader, Memory, JIT) →](04-jvm-internals.md)
