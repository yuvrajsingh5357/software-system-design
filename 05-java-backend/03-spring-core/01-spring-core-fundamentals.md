# Spring Core Fundamentals and Architecture

## 1. What Is It?
The Spring Framework is a comprehensive, open-source Java framework that provides infrastructure support for developing Java applications. The "Spring Core" module is the foundational layer upon which the entire Spring ecosystem (Spring Boot, Spring MVC, Spring Data) is built.

## 2. The Problem Before Spring
Before Spring, enterprise Java applications (J2EE) were notoriously heavy and complicated:
- Every class created its own dependencies (`new Service()`), leading to **tight coupling**. Changing one part required changing many others.
- Testing was extremely difficult because you couldn't easily replace actual objects with mock objects.
- Configuration and object wiring involved massive amounts of repetitive, unorganized code.

## 3. What Problem Does Spring Solve?
Spring reduces complexity by providing a structured system to manage application components. It standardizes:
- How objects are created.
- How objects interact with each other.
- How they are configured.

In simple terms: **Spring handles the complex setup, allowing developers to focus purely on business logic.**

## 4. Key Features of Spring Framework

### A. Inversion of Control (IoC)
- **Normal Java:** Your application code controls the creation of objects (`UserRepository repo = new UserRepositoryImpl();`).
- **Spring (IoC):** The control of object creation is *inverted*. You hand over the responsibility of creating and managing objects to the Spring Framework.
- **Mental Model:** Instead of cooking your own food, you order from a restaurant. You don't control the cooking process; the restaurant (framework) handles it and gives you the final product.

### B. Dependency Injection (DI)
Dependency Injection is the specific design pattern used to implement Inversion of Control. 
- Classes do not create the objects they depend on.
- Spring creates the required objects (dependencies) and injects them into the classes that need them at runtime.
- *(See [Dependency Injection Deep Dive](02-dependency-injection.md) for details)*.

### C. Bean Management
- Objects created and managed by Spring are called **Spring Beans**.
- Beans are stored inside the **Spring IoC Container** (typically the `ApplicationContext`).
- Spring acts as a warehouse that manages the lifecycle (creation, initialization, destruction) and availability of these Beans.

### D. Loose Coupling
By relying on DI and Interfaces, Spring promotes loose coupling.
- Example: A Controller depends on a `NotificationService` interface, not an `EmailNotificationServiceImpl`. If requirements change, you just swap the implementation bean, and the Controller code doesn't change.

### E. Declarative Support (AOP)
Spring Aspect-Oriented Programming (AOP) solves repeating cross-cutting concerns.
- Instead of writing `log.info()`, transaction commit/rollback, or security checks inside every single method, you use annotations (e.g., `@Transactional`).
- Spring automatically wraps your bean in a proxy and handles the repetitive logic.

## 5. Spring IoC Container: BeanFactory vs ApplicationContext

Both are interfaces representing the Spring IoC container, but they differ in capabilities:

| Feature | `BeanFactory` | `ApplicationContext` |
|---------|---------------|----------------------|
| **Loading** | Lazy loading (creates beans only when requested). | Eager loading (creates all singleton beans at startup). |
| **Complexity** | Simple, lightweight, basic container. | Advanced, feature-rich container. |
| **AOP Support** | No default support. | Full AOP integration. |
| **Event Publishing** | No. | Yes. |
| **Use Case** | Rare low-memory environments (IoT). | Almost all modern applications (Spring Boot default). |

## 6. Important Spring Sub-Projects
- **Spring Core:** Foundation (IoC, DI, Context).
- **Spring MVC:** Model-View-Controller architecture for web apps and REST APIs.
- **Spring Boot:** Opinionated layer on top of Spring for rapid, auto-configured development.
- **Spring Data:** Simplifies database access (JPA, Mongo, Redis).
- **Spring Security:** Authentication, authorization, JWT, OAuth2.
- **Spring Cloud:** Microservices tools (gateway, load balancing, service discovery).

## 7. Interview Questions

### Q. What is Inversion of Control (IoC)?
#### Short Interview Answer
Inversion of Control is a design principle where the control of object creation and lifecycle management is transferred from the application code to a framework or container. In Spring, the IoC container reads configuration metadata, instantiates objects (Beans), and wires their dependencies together, allowing developers to write loosely coupled, highly testable code.

### Q. What is the difference between BeanFactory and ApplicationContext?
#### Short Interview Answer
`BeanFactory` is the fundamental, lightweight IoC container that instantiates beans lazily. `ApplicationContext` is a sub-interface of `BeanFactory` that adds enterprise-specific features like eager initialization of singletons, event publishing, AOP integration, and internationalization. In modern Spring applications, `ApplicationContext` is used almost exclusively.

---

[← Previous: Concurrency (Executors & Callable)](../02-core-java/23-concurrency.md) | [Next: Dependency Injection Deep Dive →](02-dependency-injection.md)
