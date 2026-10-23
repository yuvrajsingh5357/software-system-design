# Patterns in Practice

Understanding design patterns is one thing; recognizing them in production frameworks and applying them to solve real-world problems is another. This chapter bridges theory and practice by showcasing how industry-standard frameworks (Spring, React, JDK) utilize GoF and modern patterns, and how to apply them.

## Table of Contents
- [Design Patterns in Spring Framework](#design-patterns-in-spring-framework)
- [Design Patterns in the Java Development Kit (JDK)](#design-patterns-in-the-java-development-kit-jdk)
- [Design Patterns in Modern Frontend (React & Web)](#design-patterns-in-modern-frontend-react--web)
- [Practical Guide: Applying Patterns in Legacy Refactoring](#practical-guide-applying-patterns-in-legacy-refactoring)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Design Patterns in Spring Framework

Spring is a masterclass in software design patterns. It relies heavily on patterns to achieve loose coupling, extensibility, and cross-cutting concerns management (AOP).

### 1. Dependency Injection (DI) as the Core Pattern
While not a GoF pattern, Dependency Injection (and the broader Dependency Inversion Principle) is the foundation of Spring. Instead of objects fetching their dependencies, Spring's IoC container acts as an **Assembler** or **Service Locator** to inject them.

```java
// Spring manages the lifecycle and wiring of beans
@Service
public class OrderService {
    private final PaymentGateway paymentGateway; // Injected dependency

    @Autowired // Constructor Injection is preferred for immutability and testability
    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

### 2. Singleton & Prototype Bean Scopes
In Spring, beans are by default **Singleton-scoped**, meaning the container creates exactly one instance per Spring context. Note that this is different from the GoF Singleton: Spring's singleton is a singleton *per container* (scoped by bean definition), not *per ClassLoader* (hardcoded private constructor).

```java
// Default scope is Singleton
@Component
public class DatabaseConfig { }

// Prototype scope creates a new instance on every request/injection
@Component
@Scope("prototype")
public class UserSessionToken { }
```

### 3. Template Method Pattern (JdbcTemplate, RestTemplate)
Spring uses the Template Method pattern to eliminate boilerplate code. The framework implements the skeleton algorithm (resource acquisition, transaction management, exception translation, resource cleanup) and lets developers provide the custom SQL/mapping logic.

```java
// JDBC implementation without template has ~30 lines of try-catch-finally boilerplate
public class EmployeeDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public Employee getEmployeeById(int id) {
        String query = "SELECT name, role FROM employees WHERE id = ?";
        
        // Custom row mapping callback (the "Hook" in the template method)
        return jdbcTemplate.queryForObject(query, (rs, rowNum) -> 
            new Employee(rs.getString("name"), rs.getString("role")), id);
    }
}
```

### 4. Proxy Pattern (AOP, @Transactional, @Cacheable)
Spring heavily uses Dynamic Proxies (either JDK Dynamic Proxies for interfaces or CGLIB for classes) to implement Aspect-Oriented Programming (AOP). Annotations like `@Transactional` or `@Cacheable` work by wrapping your class in a proxy that intercepts execution to handle transactions or caching.

```
Caller ──> [ Spring Transaction Proxy ] ──> [ Your Service Bean ]
                  │ (Begins TX)                    │
                  ├────────────────────────────────┤ (Executes query)
                  │ (Commits/Rollbacks TX)
```

```java
@Service
public class UserService {
    @Transactional // A Proxy intercepts this call, starts a transaction, and commits it on success
    public void registerUser(User user) {
        userRepository.save(user);
        auditRepository.log("Registered: " + user.getId());
    }
}
```

---

## Design Patterns in the Java Development Kit (JDK)

The Java standard library uses GoF design patterns in almost every package.

### 1. Decorator Pattern: `java.io`
The `java.io` package is the classic example of the Decorator pattern. It allows composing dynamic behavior (buffering, parsing, zip compression) onto input/output streams at runtime.

```java
// base stream (reads raw bytes)
FileInputStream fis = new FileInputStream("data.txt");

// Buffered decorator (adds in-memory buffering)
BufferedInputStream bis = new BufferedInputStream(fis);

// DataInputStream decorator (adds reading primitive Java data types)
DataInputStream dis = new DataInputStream(bis);

// Usage
double price = dis.readDouble(); // Flows through dis -> bis -> fis
```

### 2. Adapter Pattern: Arrays and Collections
`Arrays.asList()` acts as an Adapter between the raw array interface and the standard collections framework.

```java
String[] array = {"A", "B", "C"};
// Adapts the array to the List interface
List<String> list = Arrays.asList(array);

// Modification to list writes back to underlying array
list.set(0, "Z");
System.out.println(array[0]); // Outputs "Z"
```

### 3. Flyweight Pattern: Integer Caching
To save memory, Java caches small `Integer` objects (from `-128` to `127`). When using `Integer.valueOf()`, the JDK returns a cached reference instead of instantiating a new object.

```java
Integer x = Integer.valueOf(100);
Integer y = Integer.valueOf(100);
System.out.println(x == y); // true (same cached instance)

Integer a = Integer.valueOf(300);
Integer b = Integer.valueOf(300);
System.out.println(a == b); // false (different objects, outside cache range)
```

---

## Design Patterns in Modern Frontend (React & Web)

Modern frontend frameworks use design patterns adapted for UI state and component composition.

### 1. Provider Pattern (React Context)
Used to share global data (themes, auth state) with a tree of nested components without manually passing props down through every level ("prop drilling"). This is a variation of the **Service Locator** and **Observer** patterns.

```jsx
// Context definition
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <MainLayout />
    </ThemeContext.Provider>
  );
}

// Deeply nested component
function Button() {
  const theme = useContext(ThemeContext); // Resolves directly from Provider
  return <button className={theme}>Click Me</button>;
}
```

### 2. Observer Pattern: Redux & Event Bus
Redux uses the Observer pattern to notify UI components of global state changes. UI components "subscribe" to the Redux store, and when an action triggers a reducer update, all subscribed components re-render with the new data.

```
[ User Interaction ] ──> Dispatch(Action) ──> [ Redux Store / Reducer ]
                                                     │
                                                     ▼ (State Changes)
[ Re-Render UI ] <──────────────────────────── Notify Subscribed Components
```

---

## Practical Guide: Applying Patterns in Legacy Refactoring

Refactoring legacy code is one of the best times to introduce design patterns.

### Refactoring "Nested If-Else" to Strategy + Factory
**Legacy code:**
```java
public class ShippingCalculator {
    public double calculate(String type, double weight) {
        if (type.equalsIgnoreCase("standard")) {
            return weight * 1.5;
        } else if (type.equalsIgnoreCase("express")) {
            return weight * 3.0 + 5.0;
        } else if (type.equalsIgnoreCase("international")) {
            return weight * 10.0 + 25.0;
        }
        throw new IllegalArgumentException("Unknown shipping method");
    }
}
```

**Refactored with Strategy + Factory:**
```java
// 1. Define Strategy
public interface ShippingStrategy {
    double calculate(double weight);
}

// 2. Concrete Strategies
public class StandardShipping implements ShippingStrategy {
    public double calculate(double weight) { return weight * 1.5; }
}
public class ExpressShipping implements ShippingStrategy {
    public double calculate(double weight) { return weight * 3.0 + 5.0; }
}

// 3. Simple Factory with Registry
public class ShippingFactory {
    private static final Map<String, ShippingStrategy> strategies = new HashMap<>();
    
    static {
        strategies.put("standard", new StandardShipping());
        strategies.put("express", new ExpressShipping());
    }
    
    public static ShippingStrategy getStrategy(String type) {
        ShippingStrategy strategy = strategies.get(type.toLowerCase());
        if (strategy == null) throw new IllegalArgumentException("Unknown type");
        return strategy;
    }
}

// 4. Clean usage — Open-Closed Principle satisfied
public class ShippingCalculator {
    public double calculate(String type, double weight) {
        return ShippingFactory.getStrategy(type).calculate(weight);
    }
}
```

---

## Interview Questions

### Q: How does Spring use the Proxy pattern to manage transactions?
When a bean is annotated with `@Transactional`, Spring replaces it in the application context with a dynamically generated proxy class. When a method is called:
1. The proxy intercepts the call.
2. It opens a database connection and begins a transaction.
3. It calls the original method on the target bean.
4. If the method completes successfully, the proxy commits the transaction.
5. If a runtime exception is thrown, the proxy rolls back the transaction.
6. The database connection is closed.

### Q: Why does JDK's `java.io` package require wrapping so many streams?
`java.io` uses the **Decorator Pattern**. By avoiding one huge class with all features (buffering, file reading, parsing), it uses smaller classes that wrap each other. This adheres to the **Single Responsibility Principle** and allows you to dynamically combine behavior at runtime (e.g., buffering only when needed, parsing integers from files or network streams).

---

## ⚡ Flashcard Summary

| # | Topic | Pattern & Framework/Tool |
|---|-------|--------------------------|
| 1 | Spring Beans | Singleton scope (per context) / Prototype scope (new instance) |
| 2 | Spring database access | Template Method pattern (`JdbcTemplate` handles boilerplate) |
| 3 | Spring `@Transactional` | Proxy pattern (JDK Dynamic Proxy or CGLIB wrappers) |
| 4 | Java `BufferedInputStream` | Decorator pattern wrapping a raw `FileInputStream` |
| 5 | Java `Arrays.asList(array)` | Adapter pattern converting array to `List` interface |
| 6 | Java `Integer.valueOf(100)` | Flyweight pattern caching integers `-128` to `127` |
| 7 | React Context API | Provider pattern to avoid prop-drilling |
| 8 | Redux state updates | Observer pattern (components subscribe to store changes) |
| 9 | Avoid nested if-else | Strategy pattern for behavior, Factory registry for lookup |
| 10 | Spring IoC Container | Service Locator / Factory pattern managing bean creation |

---

*Next: [SOLID and OOP Design](../03-lld/01-solid-and-oop-design.md)*
