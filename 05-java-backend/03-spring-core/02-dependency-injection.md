# Dependency Injection in Spring

## 1. What Is It?
Dependency Injection (DI) is a technique where objects do not create their own dependencies. Instead, the Spring Framework creates the dependencies and injects them into the classes that require them. 

## 2. Why Do We Need It?
When a class creates its own dependencies using the `new` keyword, it becomes tightly coupled to that specific implementation. This makes the code rigid, difficult to maintain, and almost impossible to unit test (because you cannot replace the real dependency with a mock object). DI promotes loose coupling and testability.

## 3. Types of Dependency Injection

Spring supports three main types of dependency injection:

### A. Constructor-Based Dependency Injection (Recommended)
Spring injects all required dependencies through the class constructor.
- **Why it is best:** It makes dependencies **mandatory**. The object cannot be instantiated in a half-initialized state. It also supports **immutability** (fields can be marked as `final`), and it is extremely test-friendly because you can manually pass mock objects into the constructor without starting the Spring container.
- **Usage:** In modern Spring Boot, if a class has only one constructor, the `@Autowired` annotation is entirely optional. Spring will automatically inject the dependencies.

```java
@Service
public class OrderService {
    private final PaymentService paymentService; // Can be final!

    // @Autowired is optional here
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### B. Setter-Based Dependency Injection
Spring injects dependencies by calling a setter method annotated with `@Autowired` after the object is constructed.
- **Use Case:** Only recommended when the dependency is genuinely **optional** or needs to be swapped/reconfigured at runtime.
- **Drawback:** The object can exist without the dependency, leading to potential `NullPointerException`s if the setter isn't called.

### C. Field-Based Dependency Injection (Discouraged)
Dependencies are injected directly into the field variables using reflection, without a constructor or setter.
- **Why it is bad:** It hides the class's dependencies, prevents fields from being `final` (breaking immutability), and makes unit testing incredibly difficult because you have to use reflection tools (like Mockito's `@InjectMocks`) to set the private fields.
- **Usage:** Highly discouraged by the Spring team for production code, though still seen in older legacy applications.

## 4. @Component vs @Bean

Both annotations are used to tell Spring to manage an object, but they are used in different ways:

| Aspect | `@Component` | `@Bean` |
|--------|--------------|---------|
| **Definition** | Class-level annotation. | Method-level annotation inside a `@Configuration` class. |
| **How it Works** | Spring automatically scans the classpath, finds the class, and creates an instance. | You manually write the method that instantiates the object, and Spring registers the returned object as a bean. |
| **Use Case** | Use for **your own classes** (e.g., your custom `UserService`). | Use when you need **full control** over creation, or when injecting **third-party classes** (e.g., a library's `RestTemplate` or `ObjectMapper` where you cannot add `@Component` to their source code). |

## 5. Interview Questions

### Q. Which type of Dependency Injection is recommended and why?
#### Short Interview Answer
Constructor injection is the recommended approach. It ensures that required dependencies are mandatory, preventing half-initialized objects. It allows dependencies to be marked as `final`, ensuring immutability. Finally, it makes unit testing much easier because dependencies can be passed directly via the constructor without relying on Spring's context or reflection.

### Q. What is the default Bean Scope in Spring?
#### Short Interview Answer
The default bean scope is **Singleton**. This means Spring creates exactly one instance of the bean for the entire application context. Every time a class requests that bean, Spring returns the same shared instance, which optimizes memory usage and performance. If a new instance is needed every time, the scope must be explicitly changed to `Prototype`.

---

[← Previous: Spring Core Fundamentals (IoC)](01-spring-core-fundamentals.md) | [Next: Bean Lifecycle and AOP →](03-bean-lifecycle-and-aop.md)
