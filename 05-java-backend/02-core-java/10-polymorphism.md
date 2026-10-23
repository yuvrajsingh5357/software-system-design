# Polymorphism in Java (Overloading & Overriding)

## 1. What Is It?
Polymorphism translates to "many forms". In Java, it is the ability of a single action or method to behave differently depending on the context or the object that invokes it.

## 2. Types of Polymorphism

### A. Compile-Time Polymorphism (Method Overloading)
Method overloading means defining multiple methods with the **same name** in the **same class**, but with a **different parameter list** (different number, type, or order of parameters).
- The Java compiler decides which method to call based on the arguments passed at compile time.

**Mental Model:** A `CoffeeMachine`. You can press `makeCoffee()`, `makeCoffee(String sugar)`, or `makeCoffee(String sugar, String milk)`. Same action name, different inputs, slightly different process.

```java
public class Calculator {
    // Adds two integers
    public int add(int a, int b) {
        return a + b;
    }

    // Adds three integers
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    // Adds two doubles
    public double add(double a, double b) {
        return a + b;
    }
}
```

### B. Run-Time Polymorphism (Method Overriding)
Method overriding happens when a child class provides a specific implementation for a method that is already defined in its parent class (or interface).
- It must have the exact same name, return type, and parameters.
- The JVM decides which method to execute at **runtime** based on the actual object type in memory, not the reference type. (This is called Dynamic Method Dispatch).

```java
class Animal {
    void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Bark");
    }
}

public class Main {
    public static void main(String[] args) {
        // Parent reference pointing to Child object
        Animal myAnimal = new Dog(); 
        
        // At runtime, the JVM sees the object is a Dog, so it calls Dog's method
        myAnimal.makeSound(); // Output: Bark
    }
}
```

## 3. Production Backend Context
Run-time polymorphism is the backbone of the Strategy Design Pattern and Spring's Dependency Injection.
If you have a `NotificationService` interface with a `send()` method, and two implementations: `EmailNotificationService` and `SmsNotificationService`. When the controller calls `notificationService.send()`, the JVM dynamically routes the call to the correct implementation injected by Spring at runtime.

## 4. Interview Questions

### Q. Can you override a static method?
#### Short Interview Answer
No. Static methods belong to the class, not the instance. If a child class defines a static method with the same signature as a static method in the parent class, it *hides* the parent's method (Method Hiding), but it does not override it. There is no dynamic polymorphic dispatch at runtime for static methods.

### Q. Can we overload the main() method?
#### Short Interview Answer
Yes, you can overload the `main()` method by creating other methods named `main` with different parameter types (e.g., `public static void main(int[] args)`). However, when you run the program, the JVM will strictly look for the standard `public static void main(String[] args)` method to start execution.

---

[← Previous: Abstraction](09-abstraction.md) | [Next: Java Strings & Immutability →](12-strings.md)
