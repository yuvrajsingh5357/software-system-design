# Glossary — Software System Design

Every term used across this guide, defined concisely.

---

## A

**Abstract Class** — A class that cannot be instantiated and may contain abstract methods (no body). Subclasses must implement abstract methods.

**Abstract Factory** — A creational pattern that produces families of related objects without specifying their concrete classes.

**ACID** — Properties of database transactions: Atomicity, Consistency, Isolation, Durability.

**Adapter Pattern** — A structural pattern that allows incompatible interfaces to work together by wrapping one interface in another.

**API Gateway** — A server that acts as the single entry point for client requests, routing them to appropriate microservices.

**Atomicity** — Either all operations in a transaction succeed, or none do.

**Availability** — In CAP theorem: every request receives a (non-error) response, without guarantee that it contains the most recent data.

---

## B

**BASE** — Basically Available, Soft state, Eventually consistent. The NoSQL alternative to ACID.

**Big-O Notation** — Mathematical notation to describe the upper bound of algorithm complexity as input size grows.

**BFS (Breadth-First Search)** — Graph/tree traversal that explores all neighbors at present depth before moving to next level.

**Binary Heap** — A complete binary tree where every parent is ≤ (min-heap) or ≥ (max-heap) its children.

**Binary Search Tree (BST)** — A binary tree where left child < parent < right child.

**Bridge Pattern** — Structural pattern that decouples an abstraction from its implementation so both can vary independently.

**Builder Pattern** — Creational pattern for constructing complex objects step by step.

---

## C

**CAP Theorem** — In a distributed system, you can only guarantee 2 of 3: Consistency, Availability, Partition Tolerance.

**CDN (Content Delivery Network)** — Distributed servers that cache static content close to users geographically.

**Chain of Responsibility** — Behavioral pattern where a request is passed along a chain of handlers, each deciding to handle or forward it.

**Cohesion** — How strongly related the responsibilities within a single class/module are. High cohesion is desirable.

**Command Pattern** — Behavioral pattern that encapsulates a request as an object, allowing undo, redo, and queuing.

**Composite Pattern** — Structural pattern that composes objects into tree structures to represent part-whole hierarchies.

**Concurrency** — Multiple computations happening in overlapping time periods.

**Consistency** — In CAP: every read receives the most recent write or an error.

**Coupling** — The degree of interdependence between modules. Low coupling is desirable.

**CQRS (Command Query Responsibility Segregation)** — Pattern that separates read (query) and write (command) models.

---

## D

**Deadlock** — A situation where two or more threads are waiting for each other to release resources.

**Decorator Pattern** — Structural pattern that attaches additional responsibilities to an object dynamically.

**DFS (Depth-First Search)** — Graph/tree traversal that explores as far as possible along each branch before backtracking.

**Dijkstra's Algorithm** — Shortest path algorithm for graphs with non-negative weights. O((V+E) log V).

**DIP (Dependency Inversion Principle)** — High-level modules should not depend on low-level modules; both should depend on abstractions.

**DRY (Don't Repeat Yourself)** — Every piece of knowledge should have a single, unambiguous representation in a system.

**Dynamic Programming (DP)** — Solving complex problems by breaking them into simpler overlapping subproblems and storing results.

---

## E

**Encapsulation** — Bundling data and methods that operate on that data within a class, hiding internal implementation.

**Eventual Consistency** — A consistency model where, given enough time with no new updates, all replicas will converge to the same value.

**Event Sourcing** — Storing state as a sequence of events rather than the current state.

---

## F

**Facade Pattern** — Structural pattern providing a simplified interface to a complex subsystem.

**Factory Method** — Creational pattern that defines an interface for creating objects, but lets subclasses decide which class to instantiate.

**Flyweight Pattern** — Structural pattern that shares common state between many fine-grained objects to reduce memory usage.

**FIFO** — First In, First Out. Queue behavior.

---

## G

**Gang of Four (GoF)** — The four authors (Gamma, Helm, Johnson, Vlissides) of "Design Patterns: Elements of Reusable Object-Oriented Software" (1994).

**Graph** — A non-linear data structure with nodes (vertices) and edges connecting them.

**gRPC** — Google's high-performance RPC framework using Protocol Buffers.

---

## H

**Hash Map / Hash Table** — Data structure providing O(1) average-case insert, delete, and lookup using a hash function.

**Hash Function** — A function that maps arbitrary data to a fixed-size value (hash code).

**Heap** — See Binary Heap. Also: memory region for dynamic allocation.

**Horizontal Scaling (Scale-Out)** — Adding more machines to distribute load.

**HLD (High-Level Design)** — System design at the architectural level: components, databases, queues, APIs, and how they interact.

---

## I

**Idempotency** — An operation that produces the same result no matter how many times it is applied.

**Inheritance** — An OOP mechanism where a class inherits fields and methods from a parent class.

**Interface Segregation Principle (ISP)** — Clients should not be forced to depend on interfaces they do not use.

**Iterator Pattern** — Behavioral pattern providing a way to sequentially access elements of a collection without exposing its underlying representation.

---

## J

**JWT (JSON Web Token)** — A compact, URL-safe way to represent claims between two parties, typically used for authentication.

---

## K

**Kafka** — A distributed streaming platform for high-throughput event streaming.

**KISS (Keep It Simple, Stupid)** — Design principle favoring simplicity over complexity.

---

## L

**Latency** — Time taken for a single operation to complete (e.g., one database query).

**LIFO** — Last In, First Out. Stack behavior.

**LLD (Low-Level Design)** — System design at the class level: objects, methods, attributes, and their relationships.

**Load Balancer** — A server that distributes incoming network traffic across multiple backend servers.

**LSP (Liskov Substitution Principle)** — Subtypes must be substitutable for their base types without altering program correctness.

---

## M

**Mediator Pattern** — Behavioral pattern that defines an object that encapsulates how a set of objects interact.

**Memento Pattern** — Behavioral pattern for capturing and externalizing an object's internal state for later restoration.

**Message Queue** — A communication method where messages are stored in a queue until processed. Examples: Kafka, RabbitMQ, SQS.

**Microservices** — Architectural style where an application is a collection of small, independently deployable services.

**Monolith** — A single, unified software application where all components are interconnected and interdependent.

---

## N

**NoSQL** — Non-relational databases. Categories: document (MongoDB), key-value (Redis), column (Cassandra), graph (Neo4j).

**NP-Hard** — A class of problems at least as hard as the hardest problems in NP; no known polynomial-time solution.

---

## O

**Observer Pattern** — Behavioral pattern where an object (subject) maintains a list of dependents (observers) and notifies them of state changes.

**OCP (Open/Closed Principle)** — Classes should be open for extension but closed for modification.

**OOP (Object-Oriented Programming)** — Programming paradigm based on objects that contain data and behavior.

---

## P

**Partition Tolerance** — In CAP: the system continues operating despite arbitrary network partitions.

**Polymorphism** — The ability of different classes to be treated as instances of the same class through shared interface or inheritance.

**Prototype Pattern** — Creational pattern that creates objects by copying an existing object (prototype).

**Proxy Pattern** — Structural pattern providing a placeholder or surrogate for another object.

---

## Q

**Queue** — A linear data structure following FIFO order.

---

## R

**Race Condition** — A bug where the system's behavior depends on the relative timing of events.

**Redis** — An in-memory data structure store used as cache, database, and message broker.

**Replication** — Copying data across multiple servers for fault tolerance and read scalability.

**REST (Representational State Transfer)** — An architectural style for distributed hypermedia systems using HTTP.

**RDBMS** — Relational Database Management System. Examples: MySQL, PostgreSQL.

---

## S

**Saga Pattern** — A pattern for managing long-running distributed transactions via a series of local transactions.

**Sharding** — Partitioning a database across multiple machines by splitting data horizontally.

**Single Responsibility Principle (SRP)** — A class should have only one reason to change.

**Singleton Pattern** — Creational pattern ensuring a class has only one instance.

**SOLID** — Five OOP design principles: SRP, OCP, LSP, ISP, DIP.

**SQL** — Structured Query Language. Used with relational databases.

**State Pattern** — Behavioral pattern allowing an object to alter its behavior when its internal state changes.

**Strategy Pattern** — Behavioral pattern defining a family of algorithms, encapsulating each, and making them interchangeable.

---

## T

**Template Method Pattern** — Behavioral pattern defining the skeleton of an algorithm in a base class, deferring steps to subclasses.

**Throughput** — Number of operations per unit time (e.g., requests per second).

**Thread Pool** — A collection of pre-instantiated reusable threads for executing tasks.

**Trie** — A tree-like data structure for storing strings where each node represents a character.

---

## U

**UML (Unified Modeling Language)** — A standardized modeling language for visualizing software design.

---

## V

**Vertical Scaling (Scale-Up)** — Increasing the resources (CPU, RAM) of an existing machine.

**Visitor Pattern** — Behavioral pattern that adds new operations to existing class hierarchies without modifying them.

---

## W

**WebSocket** — A protocol providing full-duplex communication channels over a single TCP connection.

---

## Y

**YAGNI (You Aren't Gonna Need It)** — Only implement things when you actually need them.

---

*See also: [PATTERNS.md](PATTERNS.md) for a quick pattern reference.*
