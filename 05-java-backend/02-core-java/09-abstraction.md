# Abstraction in Java

## 1. What Is It?
Abstraction is the concept of hiding the complex internal implementation details and showing only the essential features to the user. You define *what* an object does, not *how* it does it.

## 2. Why Do We Need It?
Abstraction reduces system complexity and increases reusability. It allows developers to interact with complex systems using a simple interface without needing to understand the internal wiring.

## 3. Mental Model
**The Car Dashboard:** When you drive a car, you use the steering wheel, accelerator, and brakes. This is the interface. You do not need to know how the fuel injector works, how the crankshaft rotates, or how the transmission shifts gears. The complex logic is hidden behind a simple abstraction.

## 4. How Is It Achieved in Java?
Abstraction is implemented using:
1. **Abstract Classes:** Can have both abstract methods (no body) and concrete methods. Cannot be instantiated. (Partial abstraction).
2. **Interfaces:** Define a strict contract of methods. (Full abstraction, though Java 8 introduced `default` methods).

## 5. Basic Example
```java
// Abstract class
abstract class Payment {
    // Abstract method: only what to do, not how
    public abstract void makePayment();
}

// Concrete class
class CreditCardPayment extends Payment {
    @Override
    public void makePayment() {
        System.out.println("Processing payment via Credit Card Gateway...");
        // Complex HTTP logic hidden here
    }
}

class UPIPayment extends Payment {
    @Override
    public void makePayment() {
        System.out.println("Processing payment via UPI Network...");
    }
}
```

## 6. Backend Context (The Core of Architecture)
In Java backend engineering, Abstraction is arguably the most important OOP principle.
It decouples the code. You write a `UserRepository` interface. Your `UserService` depends ONLY on this interface. At runtime, Spring injects a concrete `JpaUserRepositoryImpl`. 
If you decide to migrate from PostgreSQL to MongoDB, you write a new `MongoUserRepositoryImpl`. Your `UserService` never changes because it relies on the abstraction (the interface), not the specific database implementation.
*(See [SOLID Principles: Dependency Inversion Principle](../../03-lld/01-solid-and-oop-design.md))*

## 7. Interview Questions

### Q. What is the difference between an Abstract Class and an Interface?
#### Short Interview Answer
- **State:** Abstract classes can have instance variables and constructors. Interfaces cannot have instance variables (only `public static final` constants) and no constructors.
- **Inheritance:** A class can implement multiple interfaces, but can only extend one abstract class.
- **Usage:** Use an interface to define a contract across unrelated classes. Use an abstract class to share core implementation code and state among closely related classes.

---

[← Previous: Inheritance](08-inheritance.md) | [Next: Polymorphism (Overloading & Overriding) →](10-polymorphism.md)
