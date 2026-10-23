# Classes and Objects in Java

## 1. What Are They?
A **Class** is a blueprint or template that defines the state (fields/variables) and behavior (methods) of an object.
An **Object** is a real-world instance of that class residing in memory.

## 2. Why Do We Need Them?
In procedural languages (like C), data and functions are separate. Object-Oriented Programming (OOP) bundles data and the functions that operate on that data into a single, cohesive unit. This models the real world better, improves code maintainability, and allows for concepts like inheritance and polymorphism.

## 3. Mental Model
- **Class (The Blueprint):** Think of an architect's design for a house. The blueprint is not the house itself—it's just the plan. Or a Car Factory's design for a specific model.
- **Object (The Instance):** The actual physical house built from the blueprint. Or the physical Car produced by the factory. You can build many houses (objects) from a single blueprint (class).

## 4. Basic Example
```java
// Define a class
public class Car {
    // Fields (variables)
    String brand;
    String color;
    int speed;

    // Method (behavior)
    void start() {
        System.out.println(brand + " is starting");
    }
}

public class Main {
    public static void main(String[] args) {
        // Create an object
        Car myCar = new Car();
        myCar.brand = "Toyota";
        myCar.color = "Red";
        myCar.speed = 80;

        // Access object members
        myCar.start(); // Output: Toyota is starting
    }
}
```

## 5. Backend Context (Spring Boot)
In Spring Boot backend development, you rarely use the `new` keyword for business logic classes.
- Instead of manually creating `new OrderService()`, you define a class and annotate it with `@Service` or `@Component`.
- Spring creates the object (called a Bean) at startup and stores it in the ApplicationContext (IoC Container).
- However, you still use `new` heavily for Data Transfer Objects (DTOs) and Entities (e.g., `new User()`).

## 6. Interview Questions

### Q. What is the difference between a class and an object?
#### Short Interview Answer
A class is a logical blueprint that defines the properties and behaviors that objects of that type will have. An object is a physical instance of that class allocated in heap memory containing actual data.

---

[← Previous: JVM Internals (ClassLoader, Memory, JIT)](04-jvm-internals.md) | [Next: Constructors →](06-constructors.md)
