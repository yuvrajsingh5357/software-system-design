# Design Patterns and SOLID in the Java Backend

## 1. The Rule of Reuse
Before studying how patterns are applied in Spring Boot, ensure you have mastered the theoretical foundations in the canonical repository sections:
- **SOLID Principles:** See `03-lld/01-solid-and-oop-design.md`
- **Design Patterns:** See `02-design-patterns/`

This document focuses *strictly* on how these concepts are weaponized in enterprise Java and Spring Boot.

## 2. SOLID in Spring Boot

### Single Responsibility Principle (SRP)
- **Concept:** A class should have only one reason to change.
- **Backend Application:** The classic Controller-Service-Repository (Three-Tier) architecture.
  - *Controller:* Only handles HTTP request/response parsing and status codes.
  - *Service:* Only handles business logic.
  - *Repository:* Only handles database interactions.
  If your controller is doing database lookups directly, you are violating SRP.

### Open-Closed Principle (OCP)
- **Concept:** Open for extension, closed for modification.
- **Backend Application:** Spring's `@Configuration` classes. If you need to add a new security filter, you don't modify the core `SecurityFilterChain` class of the Spring framework. You create your own `@Configuration` class and register a new `@Bean` filter.

### Liskov Substitution Principle (LSP)
- **Concept:** Subclasses should be replaceable for their base classes without breaking the system.
- **Backend Application:** Using the `List` interface. 
  `List<User> users = new ArrayList<>();`
  Later, if performance dictates, you can change it to `new LinkedList<>()`. The rest of your service logic doesn't break because both abide by the `List` contract.

### Interface Segregation Principle (ISP)
- **Concept:** Don't force clients to implement methods they don't use.
- **Backend Application:** Spring Data JPA. You extend `CrudRepository` for basic operations, but if you need pagination, you extend `PagingAndSortingRepository`. You aren't forced into a massive God-interface if you only need `save()`.

### Dependency Inversion Principle (DIP)
- **Concept:** High-level modules should not depend on low-level modules. Both should depend on abstractions.
- **Backend Application:** The absolute core of Spring (Inversion of Control).
  ```java
  // Bad (Tightly Coupled)
  private MySQLUserRepository repo = new MySQLUserRepository();

  // Good (DIP via Constructor Injection)
  private final UserRepository repo; 
  public UserService(UserRepository repo) { this.repo = repo; }
  ```

## 3. Design Patterns in Spring

### Factory Pattern
- **Spring Usage:** The `BeanFactory` and `ApplicationContext` are literal implementations of the Factory pattern. They centralize object creation so your business code never calls `new`.

### Builder Pattern
- **Spring Usage:** Widely used when building complex objects, especially with Lombok (`@Builder`). Also used in `MockMvcBuilders` for unit testing and `UriComponentsBuilder` for safe URL construction.

### Proxy Pattern
- **Spring Usage:** `@Transactional`, `@Cacheable`, and Spring AOP. When you inject a service annotated with `@Transactional`, Spring doesn't give you the actual object. It gives you a dynamically generated **Proxy**. 
  - The proxy starts the DB transaction.
  - The proxy calls your actual method.
  - The proxy commits the transaction (or rolls back on exception).

### Strategy Pattern
- **Spring Usage:** Spring Security Authentication. You can configure multiple `AuthenticationProvider` strategies (e.g., LDAP, Database, OAuth2). The `AuthenticationManager` loops through them and delegates to the appropriate strategy at runtime.

### Observer Pattern
- **Spring Usage:** Spring Application Events (`ApplicationEventPublisher` and `@EventListener`). Used heavily in microservices to decouple domains (e.g., `OrderService` publishes `OrderCreatedEvent`, and `NotificationService` listens to it without the two classes ever importing each other).

---

[← Previous: JVM Memory & GC](17-jvm-memory-and-gc.md) | [Next: Multithreading & Synchronization →](22-multithreading.md)
