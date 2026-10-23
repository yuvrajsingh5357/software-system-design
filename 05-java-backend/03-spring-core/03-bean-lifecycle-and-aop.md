# Spring Bean Lifecycle and AOP

## 1. Bean Lifecycle Internals
Understanding the exact lifecycle of a Spring Bean is critical for debugging complex configuration issues.

When `ApplicationContext` starts, it creates singleton beans in the following order:
1. **Instantiation:** Spring calls the constructor (`new Service()`).
2. **Populate Properties:** Spring injects dependencies (resolving `@Autowired`).
3. **BeanPostProcessor (Before Initialization):** Spring allows custom processors to intercept the bean and modify it.
4. **Initialization:** Spring calls any custom setup methods (e.g., methods annotated with `@PostConstruct` or `InitializingBean.afterPropertiesSet()`).
5. **BeanPostProcessor (After Initialization):** **CRITICAL STEP.** This is where Spring generates AOP Proxies. If your bean has `@Transactional`, Spring replaces the raw object with a CGLIB proxy object.
6. **Ready for Use:** The bean is added to the container's cache.
7. **Destruction:** On app shutdown, `@PreDestroy` methods are called.

## 2. Aspect-Oriented Programming (AOP)
AOP solves the problem of "cross-cutting concerns"—logic that repeats across many classes (logging, security, transaction management).

Instead of writing `em.getTransaction().begin()` in every method, you use `@Transactional`.
- Spring AOP uses **Dynamic Proxies**.
- When you request `UserService`, Spring actually gives you a proxy object that *wraps* `UserService`.
- When you call `userService.save()`, the proxy intercepts the call, starts the transaction, calls the real `userService.save()`, and then commits the transaction.

## 3. The Self-Invocation Problem
Because Spring uses proxies, internal method calls bypass the proxy entirely.
```java
@Service
public class OrderService {
    public void processOrder() {
        // This internal call BYPASSES the proxy!
        saveOrderToDb(); 
    }

    @Transactional
    public void saveOrderToDb() { ... }
}
```
In the above example, calling `processOrder()` will *not* execute `saveOrderToDb()` within a transaction because `this.saveOrderToDb()` is called on the raw object, not the proxy.

## 4. Circular Dependencies
If Service A requires Service B, and Service B requires Service A, you have a circular dependency.
- **Constructor Injection:** Fails immediately at startup with `BeanCurrentlyInCreationException`. (This is good; it forces you to fix your architecture).
- **Field/Setter Injection:** Spring attempts to solve it using a "Three-level cache" (singleton objects, early singleton objects, singleton factories), which injects a partially constructed bean. This is a code smell.

## 5. Interview Questions
### Q. What actually happens when you annotate a method with @Transactional?
**Short Answer:** At startup, during the BeanPostProcessor phase, Spring wraps the bean in a CGLIB or JDK Dynamic Proxy. At runtime, any external call to that method hits the proxy first. The proxy opens a database connection, starts a transaction, delegates to the actual method, and upon successful return, commits the transaction. If a `RuntimeException` is thrown, the proxy catches it and issues a rollback.

---

[← Previous: Dependency Injection Deep Dive](02-dependency-injection.md) | [Next: Spring Boot Architecture & Auto-Config →](../04-spring-boot/01-spring-boot-architecture.md)
