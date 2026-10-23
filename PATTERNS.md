# Pattern Catalog — Quick Reference

A one-page cheat sheet of all design patterns covered in this guide.

---

## Creational Patterns

| Pattern | Intent | When To Use |
|---------|--------|-------------|
| **Singleton** | One instance, global access | Config, Logger, Connection Pool |
| **Factory Method** | Subclass decides what to create | Framework hooks, plugin systems |
| **Abstract Factory** | Family of related objects | UI toolkits, cross-platform code |
| **Builder** | Step-by-step complex object construction | HTTP requests, query builders |
| **Prototype** | Clone an existing object | Object pools, copying complex state |

---

## Structural Patterns

| Pattern | Intent | When To Use |
|---------|--------|-------------|
| **Adapter** | Make incompatible interfaces work together | Third-party API integration |
| **Bridge** | Decouple abstraction from implementation | Cross-platform code, drivers |
| **Composite** | Tree structure for part-whole hierarchies | File systems, UI components |
| **Decorator** | Wrap to add behavior dynamically | I/O streams, middleware |
| **Facade** | Simplified interface to complex subsystem | Library wrappers, API gateways |
| **Flyweight** | Share common state across many objects | Text rendering, game entities |
| **Proxy** | Placeholder controlling access | Caching, lazy init, security |

---

## Behavioral Patterns

| Pattern | Intent | When To Use |
|---------|--------|-------------|
| **Chain of Responsibility** | Pass request along handler chain | Middleware, event processing |
| **Command** | Encapsulate request as object | Undo/redo, queuing, transactions |
| **Iterator** | Sequential access to collection | Custom data structures |
| **Mediator** | Centralize object communication | Chat room, event bus |
| **Memento** | Save and restore object state | Undo history, snapshots |
| **Observer** | Notify dependents of state change | Event systems, MVC, pub/sub |
| **State** | Object changes behavior when state changes | Order status, ATM, elevator |
| **Strategy** | Swap algorithms at runtime | Sorting, pricing, routing |
| **Template Method** | Define skeleton, defer steps to subclasses | Frameworks, game loops |
| **Visitor** | Add operations without modifying classes | Compilers, AST traversal |

---

## Concurrency Patterns

| Pattern | Intent | When To Use |
|---------|--------|-------------|
| **Thread Pool** | Reuse worker threads | HTTP servers, task executors |
| **Producer-Consumer** | Decouple production from consumption | Queues, pipelines |
| **Read-Write Lock** | Allow concurrent reads, exclusive writes | Caches, shared state |
| **Scheduler** | Control execution order and timing | Task queues, rate limiters |

---

## Architecture Patterns (HLD)

| Pattern | Intent | When To Use |
|---------|--------|-------------|
| **CQRS** | Separate read and write models | High read:write ratio |
| **Event Sourcing** | Store events, not current state | Audit logs, temporal queries |
| **Saga** | Distributed transaction management | Microservices workflows |
| **Strangler Fig** | Migrate legacy systems incrementally | Monolith → microservices |
| **Circuit Breaker** | Stop calling failing services | Microservice resilience |
| **API Gateway** | Single entry point for clients | Microservices, mobile apps |
| **Sidecar** | Deploy helper services alongside main service | Service mesh, logging agents |

---

## Pattern Selection Guide

```
Need to control object creation?          → Creational
Need to compose objects/classes?          → Structural
Need to handle communication/behavior?    → Behavioral
Need thread-safe shared state?            → Concurrency
Need to scale to millions of users?       → Architecture (HLD)
```

---

## Pattern Relationships

```
Builder ──builds──► complex objects that
                    Composite trees contain
                    
Observer ──notifies──► Mediator that
                       coordinates via Command objects
                       
Strategy ──is swapped via──► State machine
                              (State changes the Strategy)
```

---

*Full implementations with Java code in [02-design-patterns/](02-design-patterns/)*
