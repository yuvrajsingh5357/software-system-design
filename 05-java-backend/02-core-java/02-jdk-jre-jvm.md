# JDK, JRE, and JVM

## 1. What Are They?
The Java ecosystem is built on three core components: the Java Development Kit (JDK), the Java Runtime Environment (JRE), and the Java Virtual Machine (JVM). They form a hierarchy of tools necessary to develop and run Java applications.

## 2. Why Do We Need Them?
Different users have different needs. A developer writing code needs compilers and debugging tools. An end-user just wants to run the application. Separating these tools prevents end-users from having to install unnecessary development bloatware just to run a program.

## 3. Mental Model
- **JVM:** The engine of the car. It actually does the running.
- **JRE:** The engine + the chassis, wheels, and seats. It's the complete car ready to be driven.
- **JDK:** The car + the entire manufacturing factory and mechanic's toolkit. It's everything needed to build, fix, and drive the car.

## 4. Internal Details

### A. Java Virtual Machine (JVM)
- The core engine that actually runs Java bytecode.
- It translates bytecode into native machine code.
- It makes Java platform-independent.
- It hides machine-specific details from the developer.

### B. Java Runtime Environment (JRE)
- It is the package used to *run* Java programs.
- Contains the JVM + core libraries (like `java.lang`, `java.util`) + other supporting files.
- It does **not** include development tools like the compiler.
- If you only want to run a Java application, you only need the JRE.

### C. Java Development Kit (JDK)
- The complete package used to *write, compile, and run* Java programs.
- Contains the JRE + development tools (`javac` compiler, debuggers, Javadoc generators, etc.).
- Essential for Java developers.

## 5. Visual Hierarchy
```
JDK = JRE + Development Tools (javac, debuggers)
JRE = JVM + Core Libraries
JVM = The Execution Engine
```

## 6. Interview Questions

### Q. Can you run a Java program with just the JRE? Can you compile it?
#### Short Interview Answer
You can run a compiled `.class` file with the JRE because it contains the JVM and necessary runtime libraries. However, you cannot compile a `.java` source file with just the JRE; compilation requires the `javac` compiler, which is only found in the JDK.
#### Follow-Up
If you are deploying a Spring Boot application to a Docker container, do you need a JDK or a JRE?
#### Follow-Up Answer
For running the compiled application in production, you only need the JRE. Using a JRE-only base image reduces the container size and minimizes the security attack surface since development tools are omitted. (Note: Since Java 11, the distinction has blurred as Oracle stopped shipping separate JREs, but tools like `jlink` allow you to create custom minimal runtime images).

---

[← Previous: Java Fundamentals & Platform Independence](01-java-fundamentals.md) | [Next: Java Program Execution →](03-java-program-execution.md)
