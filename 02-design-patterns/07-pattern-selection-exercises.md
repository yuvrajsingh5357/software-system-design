# Pattern Selection Exercises

## Exercise 1
**Scenario:** You have a reporting engine. Reports can be generated in PDF, XML, or JSON.
**Which pattern?** Strategy.
**Why?** The algorithm (formatting) changes, but the core data generation is the same. The client selects the format at runtime.

## Exercise 2
**Scenario:** You need to build a UI framework for both Windows and Mac. Each has unique Buttons, Checkboxes, and Windows.
**Which pattern?** Abstract Factory.
**Why?** You need to create *families* of related objects. A WindowsFactory creates WinButton, WinCheckbox.

## Exercise 3
**Scenario:** You want to add retry logic, logging, and caching to an existing network client without modifying its code.
**Which pattern?** Proxy (or Decorator).
**Why?** Wrap the real client. Intercept the `send()` call, log it, attempt cache hit, and if network fails, retry.

## Exercise 4
**Scenario:** A stock ticker needs to update 5 different display widgets on the dashboard whenever a price changes.
**Which pattern?** Observer.
**Why?** One-to-many dependency. Subject (ticker) notifies Observers (displays).
