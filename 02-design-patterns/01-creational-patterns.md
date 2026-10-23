# Creational Patterns

Creational patterns deal with **object creation mechanisms**. They abstract the instantiation process, making systems independent of how objects are created, composed, and represented.

## Table of Contents
- [Singleton](#singleton)
- [Factory Method](#factory-method)
- [Abstract Factory](#abstract-factory)
- [Builder](#builder)
- [Prototype](#prototype)
- [Pattern Comparison](#pattern-comparison)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Singleton

**Intent:** Ensure a class has only one instance and provide a global access point.

**Use cases:** Logger, Configuration manager, Connection pool, Thread pool.

### Thread-Safe Singleton Implementations

```java
// Option 1: Eager initialization — simplest, always thread-safe
public class Logger {
    private static final Logger INSTANCE = new Logger(); // created at class load
    private Logger() {} // private constructor prevents external instantiation
    public static Logger getInstance() { return INSTANCE; }
    public void log(String msg) { System.out.println(msg); }
}

// Option 2: Double-checked locking (lazy + thread-safe)
public class DatabasePool {
    private static volatile DatabasePool instance; // volatile prevents reordering
    private DatabasePool() {}
    
    public static DatabasePool getInstance() {
        if (instance == null) {                  // first check (no lock)
            synchronized (DatabasePool.class) {
                if (instance == null) {           // second check (with lock)
                    instance = new DatabasePool();
                }
            }
        }
        return instance;
    }
}
// volatile is REQUIRED — without it, partial initialization can be visible across threads

// Option 3: Bill Pugh Singleton (best — lazy, thread-safe, no synchronization overhead)
public class ConfigManager {
    private ConfigManager() {}
    
    private static class Holder {
        static final ConfigManager INSTANCE = new ConfigManager();
        // Inner class not loaded until first call to getInstance()
    }
    
    public static ConfigManager getInstance() { return Holder.INSTANCE; }
}
// JVM guarantees class loading is thread-safe → INSTANCE created safely on first call

// Option 4: Enum Singleton (Josh Bloch's recommendation — handles serialization too)
public enum AppConfig {
    INSTANCE;
    private String dbUrl = "jdbc:mysql://localhost/mydb";
    public String getDbUrl() { return dbUrl; }
}
// Usage: AppConfig.INSTANCE.getDbUrl()
// Handles serialization (enum values are singletons by JVM spec), prevents reflection attacks
```

**Why `volatile` matters:**

Without volatile, the CPU or compiler may reorder the instructions for `instance = new DatabasePool()`:
1. Allocate memory
2. ~~Initialize fields~~ ← might be reordered to step 3
3. Assign reference to `instance`

Another thread could see a non-null but uninitialized instance. `volatile` prevents this reordering.

**Singleton problems:**
- Hard to unit test (can't easily substitute)
- Global state (hidden dependencies)
- Violates Single Responsibility
- **Prefer Dependency Injection** over Singleton where possible

---

## Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

**Use cases:** Framework hooks, plugin systems, when the exact type to create isn't known ahead of time.

```java
// Product interface
interface Notification {
    void send(String message);
}

// Concrete products
class EmailNotification implements Notification {
    private String email;
    EmailNotification(String email) { this.email = email; }
    public void send(String message) {
        System.out.println("Email to " + email + ": " + message);
    }
}

class SMSNotification implements Notification {
    private String phone;
    SMSNotification(String phone) { this.phone = phone; }
    public void send(String message) {
        System.out.println("SMS to " + phone + ": " + message);
    }
}

class PushNotification implements Notification {
    private String deviceToken;
    PushNotification(String token) { this.deviceToken = token; }
    public void send(String message) {
        System.out.println("Push to " + deviceToken + ": " + message);
    }
}

// Creator with factory method
abstract class NotificationService {
    // Factory method — subclasses override
    protected abstract Notification createNotification(String recipient);
    
    // Template method using factory method
    public void notifyUser(String recipient, String message) {
        Notification n = createNotification(recipient);
        n.send(message);
        logNotification(recipient, message);
    }
    
    private void logNotification(String recipient, String message) {
        System.out.println("Logged: " + recipient + " -> " + message);
    }
}

// Concrete creators
class EmailNotificationService extends NotificationService {
    protected Notification createNotification(String email) {
        return new EmailNotification(email);
    }
}

class SMSNotificationService extends NotificationService {
    protected Notification createNotification(String phone) {
        return new SMSNotification(phone);
    }
}

// Simple static factory (not GoF Factory Method, but very common)
class NotificationFactory {
    public static Notification create(String type, String recipient) {
        return switch (type.toUpperCase()) {
            case "EMAIL" -> new EmailNotification(recipient);
            case "SMS"   -> new SMSNotification(recipient);
            case "PUSH"  -> new PushNotification(recipient);
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}

// Usage
NotificationService service = new EmailNotificationService();
service.notifyUser("user@example.com", "Your order shipped!");

Notification n = NotificationFactory.create("SMS", "+1234567890");
n.send("Your OTP is 1234");
```

**Factory Method vs Simple Factory:**
- Simple Factory: Static method — not a GoF pattern but very common
- Factory Method: Abstract method in a class — subclass decides the concrete type
- Abstract Factory: Creates families of related objects

---

## Abstract Factory

**Intent:** Provide an interface for creating **families of related objects** without specifying concrete classes.

**Use cases:** Cross-platform UI toolkits, different themes, testing mock factories.

```java
// Abstract product interfaces
interface Button { void render(); void onClick(); }
interface TextField { void render(); void onType(String text); }

// Concrete products — Windows family
class WindowsButton implements Button {
    public void render() { System.out.println("Rendering Windows button"); }
    public void onClick() { System.out.println("Windows button click"); }
}
class WindowsTextField implements TextField {
    public void render() { System.out.println("Rendering Windows text field"); }
    public void onType(String text) { System.out.println("Windows typing: " + text); }
}

// Concrete products — Mac family
class MacButton implements Button {
    public void render() { System.out.println("Rendering Mac button"); }
    public void onClick() { System.out.println("Mac button click"); }
}
class MacTextField implements TextField {
    public void render() { System.out.println("Rendering Mac text field"); }
    public void onType(String text) { System.out.println("Mac typing: " + text); }
}

// Abstract factory
interface UIFactory {
    Button createButton();
    TextField createTextField();
}

// Concrete factories
class WindowsUIFactory implements UIFactory {
    public Button createButton() { return new WindowsButton(); }
    public TextField createTextField() { return new WindowsTextField(); }
}
class MacUIFactory implements UIFactory {
    public Button createButton() { return new MacButton(); }
    public TextField createTextField() { return new MacTextField(); }
}

// Client — works with abstract interfaces, unaware of concrete types
class Application {
    private Button button;
    private TextField textField;
    
    public Application(UIFactory factory) {
        this.button = factory.createButton();
        this.textField = factory.createTextField();
    }
    
    public void render() {
        button.render();
        textField.render();
    }
}

// Usage — swap entire family by changing factory
UIFactory factory = System.getProperty("os.name").contains("Windows")
    ? new WindowsUIFactory()
    : new MacUIFactory();
Application app = new Application(factory);
app.render();
```

---

## Builder

**Intent:** Construct complex objects step by step. Builder allows producing different types and representations of an object using the same construction process.

**Use cases:** HTTP request builders, SQL query builders, complex object constructors with many optional parameters.

```java
// Without Builder: telescoping constructor problem
public User(String name, String email, String phone, String address, int age) { }
// Which is which? User("Alice", null, null, null, 25) — confusing!

// With Builder — fluent API
public class User {
    // Required fields
    private final String name;
    private final String email;
    // Optional fields
    private final String phone;
    private final String address;
    private final int age;
    
    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.phone = builder.phone;
        this.address = builder.address;
        this.age = builder.age;
    }
    
    public static class Builder {
        // Required
        private final String name;
        private final String email;
        // Optional with defaults
        private String phone = "";
        private String address = "";
        private int age = 0;
        
        public Builder(String name, String email) { // required params in constructor
            this.name = name;
            this.email = email;
        }
        
        public Builder phone(String phone) { this.phone = phone; return this; }
        public Builder address(String address) { this.address = address; return this; }
        public Builder age(int age) { this.age = age; return this; }
        
        public User build() {
            // Validate here
            if (name == null || name.isEmpty()) throw new IllegalStateException("Name required");
            return new User(this);
        }
    }
    
    // Getters
    public String getName() { return name; }
    public String getEmail() { return email; }
    // ...
}

// Usage — readable, order-independent, self-documenting
User user = new User.Builder("Alice", "alice@example.com")
    .phone("+1234567890")
    .age(30)
    .build();

// Real-world: StringBuilder, HttpRequest.Builder (Java 11), Lombok @Builder annotation
```

**Lombok @Builder (production shortcut):**
```java
@lombok.Builder
public class User {
    private final String name;
    private final String email;
    @lombok.Builder.Default private String phone = "";
    @lombok.Builder.Default private int age = 0;
}
// Auto-generates Builder class — no boilerplate
User user = User.builder().name("Alice").email("a@b.com").build();
```

---

## Prototype

**Intent:** Create new objects by **copying an existing object** (the prototype).

**Use cases:** When object creation is expensive, when objects need to be duplicates of existing state.

```java
// Java's built-in Cloneable + clone()
class GameCharacter implements Cloneable {
    private String name;
    private int health;
    private int level;
    private List<String> skills; // mutable — needs deep copy!
    
    public GameCharacter(String name, int health, int level) {
        this.name = name;
        this.health = health;
        this.level = level;
        this.skills = new ArrayList<>();
    }
    
    // Deep clone — important! Shallow clone would share skills list
    @Override
    public GameCharacter clone() {
        try {
            GameCharacter copy = (GameCharacter) super.clone(); // shallow copy fields
            copy.skills = new ArrayList<>(this.skills);          // deep copy mutable list
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
    
    public void addSkill(String skill) { skills.add(skill); }
}

// Usage — clone archetype characters
GameCharacter wizard = new GameCharacter("Wizard", 100, 5);
wizard.addSkill("Fireball");
wizard.addSkill("Teleport");

GameCharacter player1Wizard = wizard.clone();
player1Wizard.setName("Player1_Wizard");
// player1Wizard has same skills, level, health — but is a separate object

// Better approach: Copy constructor (preferred over Cloneable in Java)
class GameCharacter {
    // Copy constructor
    public GameCharacter(GameCharacter other) {
        this.name = other.name;
        this.health = other.health;
        this.level = other.level;
        this.skills = new ArrayList<>(other.skills); // deep copy
    }
}
GameCharacter copy = new GameCharacter(wizard); // clearer than clone()
```

**Shallow vs Deep copy:**
```
Shallow: copies references → original and copy SHARE the same object fields
Deep: copies values recursively → fully independent copy
```

---

## Pattern Comparison

| Pattern | Problem Solved | Mechanism | When to Choose |
|---------|---------------|-----------|----------------|
| Singleton | One instance everywhere | Private constructor + static field | Logger, config, connection pool |
| Factory Method | Subclass decides type | Abstract creator method | Framework with pluggable types |
| Abstract Factory | Family of related objects | Factory of factories | UI toolkits, cross-platform |
| Builder | Complex construction | Fluent step-by-step API | Many optional parameters |
| Prototype | Expensive object creation | clone() or copy constructor | Copy existing state |

---

## Interview Questions

### Q: Why is Singleton considered an anti-pattern?

Singleton creates **hidden global state**:
1. Testing is hard — can't inject a mock singleton
2. Classes that use it have hidden dependencies (not declared in constructor)
3. Violates Single Responsibility (manages its own creation AND behavior)
4. Violates Dependency Inversion (high-level depends on low-level singleton directly)

**Better alternative:** Dependency injection. Declare the dependency in the constructor; let the DI framework (Spring, Guice) manage the lifecycle (singleton scope if needed).

### Q: Double-checked locking without volatile — why is it broken?

Without `volatile`, the JIT compiler or CPU can reorder instructions:
1. Allocate memory for instance → write reference to `instance` (steps 2 and 3 swapped!)
2. Assign reference
3. Initialize fields

Thread B may see `instance != null` but get an **uninitialized object**. `volatile` prevents this reordering by establishing a happens-before relationship.

### Q: What is the difference between Factory Method and Abstract Factory?

- **Factory Method:** ONE factory method for ONE product type. Subclasses override to change which concrete type is created. Example: `createButton()` in a per-platform class.
- **Abstract Factory:** MULTIPLE factory methods for a FAMILY of related products. One factory creates buttons AND text fields AND checkboxes — all consistent. Example: `WindowsUIFactory` creates all Windows widgets.

---

## ⚡ Flashcard Summary

| # | Pattern | One-liner |
|---|---------|-----------|
| 1 | Singleton | Private constructor + static INSTANCE |
| 2 | Singleton thread-safety | volatile + double-checked OR Bill Pugh Holder |
| 3 | Best Singleton | Enum singleton (handles serialization, reflection) |
| 4 | Factory Method | Subclass overrides `createProduct()` |
| 5 | Static Factory | Static `create()` method — not a GoF pattern |
| 6 | Abstract Factory | Creates family of products; swap factory = swap family |
| 7 | Builder | Fluent API; `new Builder(required).optional(x).build()` |
| 8 | Builder use case | >3 optional params in constructor |
| 9 | Prototype | copy constructor OR `clone()` with deep copy |
| 10 | Shallow vs Deep | Shallow: shares references; Deep: independent copy |
| 11 | Singleton problems | Hidden state, untestable, hidden deps |
| 12 | DI > Singleton | Inject via constructor instead of global access |
| 13 | volatile without DCL | Risk: uninitialized object visible across threads |
| 14 | Lombok @Builder | Generates Builder automatically — production shortcut |
| 15 | Factory Method vs Abstract | FM: one product; AF: family of products |

---

*Next: [Structural Patterns](02-structural-patterns.md)*
