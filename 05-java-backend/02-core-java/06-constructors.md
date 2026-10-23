# Constructors in Java

## 1. What Is It?
A constructor is a special block of code that runs automatically when you create an object in Java using the `new` keyword.

## 2. Why Do We Need It?
The main purpose of a constructor is to set initial values to the object's fields. In simple words, it helps us set up and initialize the object at the exact moment it is created, ensuring the object starts in a valid state.

## 3. Important Rules / Contracts
1. **Name Match:** The name of the constructor must be exactly the same as the class name.
2. **No Return Type:** A constructor does not have a return type—not even `void`. This is how Java distinguishes a constructor from a regular method.
3. **Automatic Invocation:** You do not call constructors explicitly like `myObject.constructor()`. They are called automatically by the JVM during the `new` operation.

## 4. Types of Constructors
- **Default Constructor:** If you don't write any constructor in your class, Java automatically provides an empty, no-argument constructor. It assigns default values (like `null` for objects, `0` for ints) to the fields.
- **Parameterized Constructor:** A constructor that accepts one or more parameters, allowing you to pass specific values during object creation.

## 5. Constructor Overloading
Just like methods, you can have multiple constructors in the same class as long as their parameter lists differ (different number or type of parameters).

## 6. Basic Example
```java
public class Student {
    String name;
    int age;

    // Default Constructor
    public Student() {
        this.name = "Unknown";
        this.age = 0;
    }

    // Parameterized Constructor
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

// Usage
Student s1 = new Student(); // Calls default
Student s2 = new Student("Ravi", 20); // Calls parameterized
```

## 7. Backend Context (Dependency Injection)
Constructor Injection is the gold standard in Spring Boot for injecting dependencies.
```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    // Spring automatically calls this constructor and passes the dependency!
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
Using constructor injection allows the fields to be `final` (immutable) and makes the class easier to unit test without starting the Spring context.

## 8. Interview Questions

### Q. Can a constructor be private?
#### Short Interview Answer
Yes. A private constructor prevents other classes from instantiating the class. This is commonly used in the Singleton design pattern, where the class itself manages a single instance of itself, or in Utility classes that only contain static methods and should never be instantiated.

### Q. What happens if you define a parameterized constructor but don't define a default constructor?
#### Short Interview Answer
If you define *any* constructor, the Java compiler will **not** automatically provide a default no-argument constructor. If another part of your code tries to instantiate the class without passing arguments (e.g., `new MyClass()`), it will result in a compilation error unless you explicitly write a no-argument constructor.

---

[← Previous: Classes and Objects](05-classes-and-objects.md) | [Next: Encapsulation →](07-encapsulation.md)
