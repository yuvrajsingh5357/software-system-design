# Source PDF Review and Integration Status

This document tracks the integration of the canonical source PDFs into the Markdown knowledge base. All PDFs have been fully extrapolated; they were not summarized, but rather used as the educational baseline and expanded with production and interview context.

## 1. Java Fundamentals & OOP
- **Source:** `01-java-fundamentals.pdf`, `02-oop.pdf`
- **Topics:** JDK/JRE/JVM, bytecode, platform independence, Classes, Inheritance, Polymorphism.
- **Destination:** `02-core-java/01-java-fundamentals.md` through `02-core-java/10-polymorphism.md`
- **Integration Status:** Fully Integrated
- **Preserved Content:** Diagram concepts for JVM architecture and textbook definitions of OOP principles.
- **Expanded Content:** Added deep dives on exact bytecode compilation processes and memory layouts (Heap vs Stack).
- **Cross-links:** Links to `17-jvm-memory-and-gc.md`.
- **Verification:** Passed (No missing core concepts).

## 2. Collections Framework & HashMaps
- **Source:** `03-collections.pdf`, `04-hashmap-internals.pdf`
- **Topics:** List, Set, Map, HashMap collisions, treeification, fail-fast vs fail-safe.
- **Destination:** `02-core-java/14-collections.md`, `15-list-set-map.md`, `16-hashmap-internals.md`
- **Integration Status:** Fully Integrated
- **Preserved Content:** The `hashCode()` and `equals()` contract explanation.
- **Expanded Content:** Added exact Java 8+ Treeification thresholds ($O(1)$ to $O(\log N)$ degradation handling) and SDE-2 interview questions.
- **Cross-links:** Links to DSA Hash Tables.
- **Verification:** Passed.

## 3. Concurrency & Multithreading
- **Source:** `05-concurrency.pdf`, `06-multithreading.pdf`
- **Topics:** Threads, Runnable, synchronization, volatile, Executors.
- **Destination:** `02-core-java/22-multithreading.md`, `23-concurrency.md`
- **Integration Status:** Fully Integrated
- **Preserved Content:** Thread lifecycle states.
- **Expanded Content:** Shifted focus heavily to modern `ExecutorService` and `CompletableFuture` rather than raw `Thread` manipulation.
- **Cross-links:** None required outside module.
- **Verification:** Passed.

## 4. Spring Framework & Spring Boot
- **Source:** `07-spring-core.pdf`, `08-spring-boot.pdf`
- **Topics:** IoC, DI, BeanFactory, Auto-configuration, Starters.
- **Destination:** `03-spring-core/`, `04-spring-boot/`
- **Integration Status:** Fully Integrated
- **Preserved Content:** ApplicationContext vs BeanFactory comparisons.
- **Expanded Content:** Added deep dives on the Bean Lifecycle (`BeanPostProcessor`), AOP Proxies, and the "Self-Invocation" `@Transactional` bug.
- **Cross-links:** Links to `05-jpa-hibernate`.
- **Verification:** Passed.

## 5. Microservices & Distributed Systems
- **Source:** `09-microservices.pdf`, `10-kafka.pdf`
- **Topics:** API Gateways, Circuit Breakers, Saga, Kafka Architecture, Consumer Groups.
- **Destination:** `08-microservices/`, `09-apache-kafka/`
- **Integration Status:** Fully Integrated
- **Preserved Content:** Saga Choreography definitions and Kafka partition logic.
- **Expanded Content:** Added the Transactional Outbox pattern, exact Kafka Replica/ISR failure scenarios, and a direct RabbitMQ comparison.
- **Cross-links:** Links to HLD Distributed Systems.
- **Verification:** Passed.
