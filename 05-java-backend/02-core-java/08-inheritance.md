# Inheritance in Java

## 1. What Is It?
Inheritance is an OOP mechanism where one class (called the subclass or child) acquires the properties (fields) and behaviors (methods) of another class (called the superclass or parent).

## 2. Why Do We Need It?
Inheritance promotes **code reusability**. If multiple classes share the same logic, you can extract that common logic into a parent class. It also establishes a natural hierarchical "IS-A" relationship between classes.

## 3. Mental Model
Think of biology. A `Dog` inherits traits from an `Animal`.
- The `Animal` class has properties like `height`, `eyeColor`, and methods like `eat()`.
- The `Dog` class inherits all of those, but also adds its own unique method `bark()`.
- The `Dog` "IS-A" `Animal`.

## 4. Types of Inheritance in Java
1. **Single Inheritance:** Class B extends Class A.
2. **Multilevel Inheritance:** Class C extends Class B, which extends Class A. (It forms a chain).
3. **Hierarchical Inheritance:** Class B extends Class A, AND Class C extends Class A. (Two or more children inherit from the same parent, but define their own unique behavior).
4. **Multiple Inheritance (Via Interfaces ONLY!):** A class inheriting from multiple independent parent classes. **Java does NOT support this for classes** to avoid the "Diamond Problem" (ambiguity if both parents have a method with the same signature). However, Java *does* support multiple inheritance through Interfaces.

## 5. Basic Example
```java
// Parent class
class Vehicle {
    int speed = 60;
    void startEngine() {
        System.out.println("Engine started");
    }
}

// Child class
class Car extends Vehicle {
    void playMusic() {
        System.out.println("Playing music in the car");
    }
}

public class Main {
    public static void main(String[] args) {
        Car myCar = new Car();
        myCar.startEngine(); // Inherited method
        myCar.playMusic();   // Specific to Car
    }
}
```

## 6. Backend Context (Trade-offs)
*Rule of thumb for modern backend design:* **Favor Composition over Inheritance.**
While inheritance is a core OOP concept, deep inheritance hierarchies (e.g., `AbstractBaseController` -> `BaseUserController` -> `AdminController`) become rigid, fragile, and hard to test. 
Spring Boot heavily favors composition—building complex behavior by injecting separate, single-responsibility services rather than extending base classes.

## 7. Interview Questions

### Q. Why doesn't Java support multiple inheritance for classes?
#### Short Interview Answer
To prevent the "Diamond Problem." If Class C inherits from both Class A and Class B, and both A and B have a method named `start()`, the compiler wouldn't know which `start()` method C should inherit. To avoid this ambiguity and keep the language simple, Java restricts classes to single inheritance.
#### Follow-Up
How can you achieve the effect of multiple inheritance in Java?
#### Follow-Up Answer
By using Interfaces. A class can implement multiple interfaces because interfaces (prior to Java 8) only provided method signatures without implementation, eliminating the ambiguity. Even with Java 8 default methods, if a conflict occurs, the compiler forces the implementing class to explicitly override the conflicting method and resolve the ambiguity.

---

[← Previous: Encapsulation](07-encapsulation.md) | [Next: Abstraction →](09-abstraction.md)
