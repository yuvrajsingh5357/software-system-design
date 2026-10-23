# Design Pattern Selection Guide

Design patterns are often overused. Knowing *when* and *why* to use a pattern (or choose between two similar ones) is what separates an SDE-2 from an SDE-1.

---

## 1. Strategy vs. State

**The Confusion:** Both involve an object delegating behavior to a contained sub-object (an interface implementation).
*   **Strategy:** Solves "How do I perform this specific algorithm/task?" (e.g., Sorting algorithm, Payment method). The client usually chooses the strategy and sets it once.
*   **State:** Solves "How does this object's behavior change based on its internal state?" (e.g., Vending Machine, Media Player). The state changes dynamically over time, often transitioning itself to the next state.

**Interview Clue:** 
*   "We have 5 different ways to calculate tax..." $\rightarrow$ Strategy.
*   "The order goes from Created to Paid to Shipped..." $\rightarrow$ State.

---

## 2. Factory Method vs. Abstract Factory

**The Confusion:** Both create objects.
*   **Factory Method:** Creates one specific type of object (an interface). Defers instantiation to subclasses. (e.g., `createDocument()`).
*   **Abstract Factory:** Creates a *family* of related objects. (e.g., `createMacButton()`, `createMacCheckbox()` vs `createWinButton()`, `createWinCheckbox()`).

**Interview Clue:**
*   "We need to instantiate different types of Notification (SMS, Email)..." $\rightarrow$ Factory Method.
*   "We are building a UI framework that must support dark mode and light mode across all components..." $\rightarrow$ Abstract Factory.

---

## 3. Factory vs. Builder

**The Confusion:** Both deal with complex object creation.
*   **Factory:** Best for creating distinct polymorphic types (subclasses) where creation is a single step.
*   **Builder:** Best for creating a *single* complex type that requires multiple steps, optional parameters, or a specific initialization order.

**Interview Clue:**
*   "Create either a Car, Bike, or Truck..." $\rightarrow$ Factory.
*   "Create a Pizza with 10 optional toppings..." $\rightarrow$ Builder.

---

## 4. Adapter vs. Facade

**The Confusion:** Both act as wrappers around existing classes.
*   **Adapter:** Changes the *interface* of a class to match what the client expects. It makes incompatible interfaces work together. (Solves a compatibility problem).
*   **Facade:** Simplifies a *complex subsystem* by providing a unified, higher-level interface. (Solves a complexity problem).

**Interview Clue:**
*   "We bought a 3rd party billing library but it doesn't match our `IBilling` interface..." $\rightarrow$ Adapter.
*   "To process an order, you have to call Inventory, Billing, Shipping, and Email services in a specific order..." $\rightarrow$ Facade.

---

## 5. Decorator vs. Proxy

**The Confusion:** Both wrap an object to change its behavior.
*   **Decorator:** *Adds* responsibilities or features to an object dynamically. You can stack them. (e.g., `new Milk(new Sugar(new Coffee()))`).
*   **Proxy:** *Controls access* to an object. It represents the object. (e.g., Lazy loading, Access control/Security, Caching).

**Interview Clue:**
*   "We want to add logging and timing to this existing class without modifying it..." $\rightarrow$ Proxy (or Decorator, but Proxy is often used for aspect-oriented programming).
*   "Users can customize their weapon with scopes, silencers, and extended mags..." $\rightarrow$ Decorator.

---

## 6. Observer vs. Pub/Sub

**The Confusion:** Both involve notifying interested parties of events.
*   **Observer:** Subject knows its observers directly (tightly coupled). Synchronous execution usually.
*   **Pub/Sub:** Uses a message broker/event bus in the middle. Publishers and Subscribers don't know each other (decoupled). Asynchronous execution usually.

**Interview Clue:**
*   "Update the UI when the model changes..." $\rightarrow$ Observer.
*   "When an order is placed, 5 microservices need to react..." $\rightarrow$ Pub/Sub (Kafka, RabbitMQ).
