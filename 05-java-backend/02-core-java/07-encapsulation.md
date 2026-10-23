# Encapsulation in Java

## 1. What Is It?
Encapsulation is the mechanism of wrapping the data (variables) and the code acting on the data (methods) together as a single unit, while keeping the internal details hidden from the outside world.

## 2. Why Do We Need It?
Without encapsulation, any external class could directly modify an object's internal state. This could lead to invalid data, broken logic, and tight coupling. Encapsulation protects the integrity of the data.

## 3. Mental Model
**The ATM Example:** When a user interacts with an ATM, they only see the interface (the screen, the buttons, the card reader). They can withdraw money. They do not see or have access to the internal cash dispensing mechanics, the network validation system, or the account database. The ATM hides its complex internal logic and exposes only what is necessary and safe.

**The Coffee Machine Example:** You press "Espresso". The machine hides the internal logic of grinding beans, boiling water, and mixing. It just gives you coffee. 

## 4. How Does It Work?
1. Declare the variables of a class as `private`. This hides the data.
2. Provide `public` setter and getter methods to modify and view the variables values.

## 5. Basic Example
```java
public class Student {
    // Private data - hidden from outside
    private String name;
    private int rollNumber;

    // Getter for name
    public String getName() {
        return name;
    }

    // Setter for name with validation
    public void setName(String name) {
        if (name != null && !name.isEmpty()) {
            this.name = name;
        }
    }
}
```

## 6. Why Is It Designed This Way?
- **Control over data:** You can make a class read-only (only provide getters) or write-only.
- **Data Validation:** Inside a setter method, you can validate the input before updating the field (e.g., preventing a negative age or an empty string).
- **Flexibility / Refactoring:** You can change the internal implementation of a class without breaking the external code that consumes it, because the external code only interacts with the public methods.

## 7. Production Backend Example
In JPA/Hibernate Entities, Encapsulation is critical.
If you have an `Order` entity with a `totalPrice`, you shouldn't have a public `setTotalPrice(BigDecimal price)` method. The total price should only be updated internally when an `addOrderItem()` method is called. Exposing `setTotalPrice()` allows any service to arbitrarily change an order's price, breaking business rules.

## 8. Interview Questions

### Q. How does Encapsulation differ from Abstraction?
#### Short Interview Answer
Encapsulation is about **information hiding and data protection**—bundling data and methods and restricting direct access to the state (using private fields). Abstraction is about **implementation hiding**—exposing only the essential behavior (what it does) while hiding the internal complex logic (how it does it) typically using interfaces or abstract classes. Encapsulation solves implementation-level issues, while Abstraction solves design-level issues.

---

[← Previous: Constructors](06-constructors.md) | [Next: Inheritance →](08-inheritance.md)
