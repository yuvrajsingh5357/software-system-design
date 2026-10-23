# JPA and Hibernate Fundamentals

## 1. What Is It?
- **JDBC (Java Database Connectivity):** The low-level Java API for connecting to databases and executing raw SQL.
- **JPA (Jakarta Persistence API):** A specification/standard for Object-Relational Mapping (ORM) in Java. It defines the interfaces (like `EntityManager`).
- **Hibernate:** The most popular concrete implementation of the JPA specification.

## 2. Why Do We Need ORM?
*Problem:* Object-Oriented code uses Objects, Classes, and References. Relational Databases use Tables, Rows, and Foreign Keys. This mismatch requires developers to write tedious, repetitive boilerplate code to map SQL result sets into Java Objects.
*Solution:* ORM tools map database tables to Java classes (`@Entity`). You interact with the database in terms of objects, and the ORM generates the underlying SQL automatically.

## 3. The Persistence Context (First-Level Cache)
The Persistence Context is the core concept of JPA. It is an in-memory cache managed by the `EntityManager`.
- When you load an entity (`findById`), Hibernate stores it in the Persistence Context.
- If you request the same entity again within the same transaction, Hibernate returns the cached object without hitting the database (First-Level Cache).
- **Dirty Checking:** Hibernate tracks changes to entities inside the Persistence Context. If you modify a managed entity, Hibernate automatically generates an `UPDATE` statement when the transaction commits. You do not need to call `.save()`.

## 4. Entity States (Lifecycle)
1. **Transient:** An object created with `new`. It has no ID and is not associated with the Persistence Context.
2. **Managed (Persistent):** An object currently tracked by the Persistence Context. Any changes will be synchronized with the DB.
3. **Detached:** The object has an ID, but the Persistence Context is closed (the transaction ended). Changes to this object are ignored until it is re-attached (`merge()`).
4. **Removed:** Scheduled for deletion.

## 5. Interview Questions
### Q. What is the difference between JPA and Hibernate?
**Short Answer:** JPA is just a specification consisting of interfaces and annotations (the "rules"). It doesn't write anything to a database. Hibernate is a concrete implementation of that specification (the "engine") that does the actual work of generating SQL and managing database state.

### Q. What is Dirty Checking?
**Short Answer:** Dirty checking is a feature of Hibernate where it monitors all entities in the Managed state within the current Persistence Context. When the transaction commits (or flushes), Hibernate compares the current state of the entities against their original state. If any fields have changed, it automatically generates and executes the necessary `UPDATE` SQL statements.

---

[← Previous: DevTools & Actuator](../04-spring-boot/04-devtools-and-actuator.md) | [Next: Relationships & Fetching Strategies →](02-relationships-fetching.md)
