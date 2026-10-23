# 02 — Design Patterns

Design patterns are proven solutions to recurring software design problems. Mastering them transforms you from someone who writes code to someone who designs systems.

## Reading Order

This section covers the canonical **Gang of Four (GoF)** patterns plus essential concurrency patterns, all implemented in **Java**.

| File | Topic | Patterns Covered |
|------|-------|-----------------|
| [01-creational-patterns.md](01-creational-patterns.md) | Object creation | Singleton, Factory Method, Abstract Factory, Builder, Prototype |
| [02-structural-patterns.md](02-structural-patterns.md) | Object composition | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| [03-behavioral-patterns.md](03-behavioral-patterns.md) | Object interaction | Observer, Strategy, Command, State, Template Method, Chain of Responsibility, Iterator, Mediator, Memento, Visitor |
| [04-concurrency-patterns.md](04-concurrency-patterns.md) | Thread-safe design | Thread Pool, Producer-Consumer, Read-Write Lock, Futures |
| [05-patterns-in-practice.md](05-patterns-in-practice.md) | Real-world usage | Patterns in Spring, JDK, React, and refactoring strategies |

## Quick Reference

### Creational — How objects are created
- **Singleton** — One instance, global access
- **Factory Method** — Subclass decides the type
- **Abstract Factory** — Family of related objects
- **Builder** — Step-by-step complex construction
- **Prototype** — Copy an existing object

### Structural — How objects are composed
- **Adapter** — Translate one interface to another
- **Bridge** — Decouple abstraction from implementation
- **Composite** — Tree structure (leaf = composite)
- **Decorator** — Add behavior by wrapping (Java I/O!)
- **Facade** — Simplify a complex subsystem
- **Flyweight** — Share state for memory efficiency
- **Proxy** — Control access (caching, security, lazy init)

### Behavioral — How objects communicate
- **Observer** — Event/notification system
- **Strategy** — Swappable algorithm
- **Command** — Encapsulate request; enable undo
- **State** — Object behavior changes with state
- **Template Method** — Algorithm skeleton with hooks
- **Chain of Responsibility** — Pass request along handlers
- **Iterator** — Sequential access without exposing structure

## Interview Trigger Words

| If the interviewer says... | Think... |
|--------------------------|---------|
| "Add logging without changing code" | Decorator or AOP Proxy |
| "Support multiple pricing strategies" | Strategy |
| "Undo/redo functionality" | Command + Memento |
| "One instance globally" | Singleton |
| "Notify multiple systems on event" | Observer |
| "Different DB implementations" | Strategy or Abstract Factory |
| "Order status: pending → confirmed → shipped" | State |
| "Simplify complex API" | Facade |
| "Only one reader/writer at a time" | Proxy or Monitor |
| "Build complex object step by step" | Builder |

## Java Real-World Examples

```
Singleton:    Runtime.getRuntime(), Collections.emptyList()
Factory:      Calendar.getInstance(), NumberFormat.getInstance()
Builder:      StringBuilder, HttpRequest.newBuilder()
Decorator:    BufferedInputStream wrapping FileInputStream
Observer:     ActionListener, PropertyChangeListener
Strategy:     Comparator, Executor implementations
Command:      Runnable, Callable
Iterator:     for-each loop (Iterable interface)
Proxy:        Spring AOP (@Transactional, @Cacheable)
Composite:    JComponent hierarchy in Swing
Flyweight:    Integer.valueOf(-128 to 127), String.intern()
Template:     AbstractList, HttpServlet.service()
```
