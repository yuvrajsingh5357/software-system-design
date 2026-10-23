# SOLID Principles and OOP Design

This chapter covers the SOLID principles — the foundation of all good object-oriented design. Every LLD interview question is ultimately asking you to apply these principles.

## Table of Contents
- [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
- [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
- [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
- [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
- [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
- [OOP Pillars Refresher](#oop-pillars-refresher)
- [Applying SOLID in LLD Interviews](#applying-solid-in-lld-interviews)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Single Responsibility Principle (SRP)

**A class should have only one reason to change.**

This means a class should only do one thing — it should have one responsibility, one job.

### Violation

```java
// BAD — UserManager does everything
class UserManager {
    private Connection db;
    
    public User findUser(int id) {
        // Database query logic
        return db.query("SELECT * FROM users WHERE id = " + id);
    }
    
    public void saveUser(User user) {
        // Database persistence logic
        db.execute("INSERT INTO users ...");
    }
    
    public String formatUserForEmail(User user) {
        // Formatting logic
        return "Dear " + user.getName() + ",";
    }
    
    public void sendWelcomeEmail(User user) {
        // Email sending logic
        SmtpClient smtp = new SmtpClient();
        smtp.send(user.getEmail(), formatUserForEmail(user));
    }
    
    public boolean validateEmail(String email) {
        // Validation logic
        return email.contains("@");
    }
}
// Reasons to change: DB query changes, DB schema changes, email template changes,
// SMTP config changes, validation rules change → 5 reasons to change ONE class
```

### Correct Implementation

```java
// GOOD — each class has one responsibility
class UserRepository {
    private Connection db;
    public User findById(int id) { /* DB query */ return null; }
    public void save(User user) { /* DB insert */ }
}

class EmailFormatter {
    public String formatWelcome(User user) { return "Dear " + user.getName() + ","; }
}

class EmailService {
    private SmtpClient smtp;
    private EmailFormatter formatter;
    public void sendWelcome(User user) { smtp.send(user.getEmail(), formatter.formatWelcome(user)); }
}

class UserValidator {
    public boolean isValidEmail(String email) { return email.matches("^[A-Za-z0-9+_.-]+@.+$"); }
}

class UserService {
    private UserRepository repo;
    private EmailService emailService;
    private UserValidator validator;
    
    public void registerUser(User user) {
        if (!validator.isValidEmail(user.getEmail())) throw new IllegalArgumentException("Invalid email");
        repo.save(user);
        emailService.sendWelcome(user);
    }
}
// Each class has exactly ONE reason to change
```

---

## Open/Closed Principle (OCP)

**Software entities should be open for extension but closed for modification.**

Add new functionality by adding new code, not by changing existing, tested code.

### Violation

```java
// BAD — adding a new shape requires modifying AreaCalculator
class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return Math.PI * c.radius * c.radius;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.width * r.height;
        }
        // Adding Triangle → MUST modify this class → violates OCP
        throw new IllegalArgumentException("Unknown shape");
    }
}
```

### Correct Implementation

```java
// GOOD — open for extension via interface, closed for modification
interface Shape {
    double area(); // behavior encapsulated in each shape
}

class Circle implements Shape {
    private double radius;
    Circle(double r) { this.radius = r; }
    public double area() { return Math.PI * radius * radius; }
}

class Rectangle implements Shape {
    private double width, height;
    Rectangle(double w, double h) { this.width = w; this.height = h; }
    public double area() { return width * height; }
}

// Adding Triangle: create NEW class, NO modification to existing code
class Triangle implements Shape {
    private double base, height;
    Triangle(double b, double h) { this.base = b; this.height = h; }
    public double area() { return 0.5 * base * height; }
}

class AreaCalculator {
    // Never changes — works with any Shape, present or future
    public double totalArea(List<Shape> shapes) {
        return shapes.stream().mapToDouble(Shape::area).sum();
    }
}
```

---

## Liskov Substitution Principle (LSP)

**Subtypes must be substitutable for their base types without altering program correctness.**

If S is a subtype of T, then objects of type T may be replaced with objects of type S without changing behavior.

### Violation

```java
// Classic violation: Square extends Rectangle
class Rectangle {
    protected int width, height;
    
    public void setWidth(int w) { this.width = w; }
    public void setHeight(int h) { this.height = h; }
    public int area() { return width * height; }
}

class Square extends Rectangle {
    // Square must have equal sides — overrides both setters
    @Override
    public void setWidth(int w) { this.width = this.height = w; } // changes height too!
    @Override
    public void setHeight(int h) { this.width = this.height = h; } // changes width too!
}

// Code that works with Rectangle breaks with Square
void checkArea(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    assert r.area() == 20; // FAILS for Square (area = 25, not 20)
    // Square is NOT substitutable for Rectangle → LSP violation!
}
```

### Correct Implementation

```java
// GOOD — don't force inheritance where substitutability doesn't hold
interface Shape {
    double area();
}

class Rectangle implements Shape {
    private final int width, height;
    Rectangle(int w, int h) { this.width = w; this.height = h; }
    public double area() { return width * height; }
    public int getWidth() { return width; }
    public int getHeight() { return height; }
}

class Square implements Shape {
    private final int side;
    Square(int s) { this.side = s; }
    public double area() { return side * side; }
    public int getSide() { return side; }
}
// Rectangle and Square are siblings, not parent-child
// Both substituable for Shape
```

**LSP rules for method overriding:**
- Preconditions cannot be strengthened in subtype
- Postconditions cannot be weakened in subtype  
- Invariants of supertype must be preserved
- No new checked exceptions in overriding method

---

## Interface Segregation Principle (ISP)

**Clients should not be forced to depend on interfaces they don't use.**

Prefer many specific interfaces over one "fat" interface.

### Violation

```java
// BAD — one fat interface forces all implementors to implement everything
interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeReport();
}

// Robot can work but can't eat, sleep, attend meetings, or write reports
class Robot implements Worker {
    public void work() { /* real implementation */ }
    public void eat() { throw new UnsupportedOperationException(); }  // forced!
    public void sleep() { throw new UnsupportedOperationException(); } // forced!
    public void attendMeeting() { throw new UnsupportedOperationException(); }
    public void writeReport() { throw new UnsupportedOperationException(); }
}
```

### Correct Implementation

```java
// GOOD — segregated interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }
interface MeetingAttendable { void attendMeeting(); }
interface Reportable { void writeReport(); }

// Human implements all relevant interfaces
class HumanWorker implements Workable, Eatable, Sleepable, MeetingAttendable, Reportable {
    public void work() { System.out.println("Working..."); }
    public void eat() { System.out.println("Eating..."); }
    public void sleep() { System.out.println("Sleeping..."); }
    public void attendMeeting() { System.out.println("In meeting..."); }
    public void writeReport() { System.out.println("Writing report..."); }
}

// Robot only implements what it can do
class Robot implements Workable {
    public void work() { System.out.println("Robot working at max efficiency"); }
}

// Client code declares only what it needs
class WorkScheduler {
    private List<Workable> workers; // only depends on Workable
    
    public void assignWork() {
        workers.forEach(Workable::work); // doesn't need to know about eating/sleeping
    }
}
```

---

## Dependency Inversion Principle (DIP)

**High-level modules should not depend on low-level modules. Both should depend on abstractions.**

This is the foundation of dependency injection.

### Violation

```java
// BAD — high-level OrderProcessor depends on low-level MySQLOrderRepository
class MySQLOrderRepository {
    public void save(Order order) { /* MySQL-specific save */ }
    public Order findById(int id) { /* MySQL query */ return null; }
}

class OrderProcessor {
    private MySQLOrderRepository repository = new MySQLOrderRepository(); // direct dependency!
    
    public void processOrder(Order order) {
        // Process...
        repository.save(order); // tightly coupled to MySQL
    }
    // Switching to MongoDB requires changing OrderProcessor!
    // Testing requires a real MySQL database!
}
```

### Correct Implementation

```java
// GOOD — both depend on abstraction (interface)

// Abstraction
interface OrderRepository {
    void save(Order order);
    Order findById(int id);
    List<Order> findByUserId(int userId);
}

// Low-level modules implement the abstraction
class MySQLOrderRepository implements OrderRepository {
    public void save(Order order) { /* MySQL save */ }
    public Order findById(int id) { /* MySQL query */ return null; }
    public List<Order> findByUserId(int userId) { /* MySQL query */ return null; }
}

class MongoOrderRepository implements OrderRepository {
    public void save(Order order) { /* MongoDB save */ }
    public Order findById(int id) { /* MongoDB query */ return null; }
    public List<Order> findByUserId(int userId) { return null; }
}

// In-memory implementation for tests
class InMemoryOrderRepository implements OrderRepository {
    private Map<Integer, Order> store = new HashMap<>();
    public void save(Order order) { store.put(order.getId(), order); }
    public Order findById(int id) { return store.get(id); }
    public List<Order> findByUserId(int userId) {
        return store.values().stream().filter(o -> o.getUserId() == userId).collect(Collectors.toList());
    }
}

// High-level module depends on abstraction — injected from outside
class OrderProcessor {
    private final OrderRepository repository; // depends on interface!
    
    // Dependency Injection — constructor injection (preferred)
    public OrderProcessor(OrderRepository repository) {
        this.repository = repository;
    }
    
    public void processOrder(Order order) {
        // Process...
        repository.save(order); // works with any implementation!
    }
}

// Wiring — done at composition root (DI framework like Spring)
OrderRepository repo = new MySQLOrderRepository();  // or MongoOrderRepository, or InMemory
OrderProcessor processor = new OrderProcessor(repo);

// For tests:
OrderRepository testRepo = new InMemoryOrderRepository();
OrderProcessor testProcessor = new OrderProcessor(testRepo); // no real DB needed!
```

---

## OOP Pillars Refresher

### Encapsulation

```java
// Hide internal state; expose only what's necessary
class BankAccount {
    private double balance; // private — cannot be directly modified
    
    public double getBalance() { return balance; } // controlled read
    
    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Deposit must be positive");
        balance += amount; // controlled write with validation
    }
    
    public void withdraw(double amount) {
        if (amount > balance) throw new InsufficientFundsException();
        balance -= amount;
    }
}
```

### Inheritance vs Composition

```java
// "Favor composition over inheritance" — GoF

// Inheritance — is-a relationship
class Animal { public String speak() { return "..."; } }
class Dog extends Animal { public String speak() { return "Woof"; } } // is-a Animal ✓

// Composition — has-a relationship (prefer when "is-a" doesn't hold strongly)
class Engine { public void start() { } }
class Car {
    private Engine engine; // has-a Engine
    public Car(Engine engine) { this.engine = engine; }
    public void start() { engine.start(); }
}
// Car IS-NOT-A Engine; Car HAS-A Engine → composition is correct
```

---

## Applying SOLID in LLD Interviews

When designing a system in an interview, demonstrate SOLID by:

1. **SRP:** "I'll split this into `OrderService` (business logic), `OrderRepository` (data access), and `NotificationService` (alerting) to keep responsibilities separate."

2. **OCP:** "I'll define a `PaymentStrategy` interface so we can add PayPal, Stripe, or Bitcoin support later without touching the payment service."

3. **LSP:** "My `PremiumUser` class overrides `getDiscount()` but still returns a non-negative value — it doesn't weaken the contract of `User`."

4. **ISP:** "I'm defining separate interfaces: `Readable`, `Writable`, `Lockable` — so file types that only need reading don't have to implement write/lock."

5. **DIP:** "The `ReservationService` depends on `RoomRepository` interface, not a specific DB implementation. I'll inject the dependency."

---

## Interview Questions

### Q: What is the most important SOLID principle?

Depends on context, but **DIP (Dependency Inversion)** has the broadest impact. When you depend on abstractions and inject dependencies, you automatically enable: easier testing (mock injection), swapping implementations (MySQL → Postgres), and decoupling that makes the other principles easier to follow.

### Q: How do you know if you're violating SRP?

Ask: "How many reasons does this class have to change?" If the answer is more than one, it violates SRP. Concretely: if the class has methods for persistence, formatting, validation, AND business logic — it has too many responsibilities.

### Q: LSP in practice — what is the classic violation?

The Square-Rectangle example. Intuitively a square IS-A rectangle, but `Square` violates the `Rectangle` contract because setting width changes height (and vice versa). The behavioral substitutability fails — client code expecting a Rectangle behaves incorrectly with a Square.

---

## ⚡ Flashcard Summary

| # | Principle | One-liner |
|---|-----------|-----------|
| 1 | SRP | One reason to change; one job per class |
| 2 | SRP sign | God class: persistence + validation + formatting + business logic |
| 3 | OCP | Add new behavior by adding new code, not modifying old |
| 4 | OCP mechanism | Interfaces + polymorphism; Strategy, Template patterns |
| 5 | LSP | Subtype must be behaviorally substitutable for its supertype |
| 6 | LSP violation | Square overrides Rectangle setters — breaks contract |
| 7 | LSP rules | Can't strengthen preconditions or weaken postconditions |
| 8 | ISP | Many small interfaces > one fat interface |
| 9 | ISP sign | Classes throwing UnsupportedOperationException for interface methods |
| 10 | DIP | Depend on abstractions; inject implementations from outside |
| 11 | DIP benefit | Testability (mock injection), swappable backends |
| 12 | Constructor injection | Preferred DI method; dependencies declared explicitly |
| 13 | Composition vs inheritance | "Favor composition" — has-a vs is-a |
| 14 | Encapsulation | Private state + controlled public methods |
| 15 | Tell, don't ask | Objects should do things, not expose state for others to decide |

---

*Next: [LLD Case Study — Parking Lot](02-lld-parking-lot.md)*
