# Elevator System — Reference Solution

## Domain Model & Interfaces
```java
public interface Component {
    void execute();
}
```

## Class Diagram (Mental Model)
- `Manager`: Orchestrates the flow.
- `Entity`: Holds the state.
- `Strategy`: Contains the business rules.

## Java Implementation
```java
// Focus on Separation of Concerns
public class ElevatorSystemManager {
    // Core logic
}
```

## SOLID Decisions
- **SRP**: Separated storage from execution.
- **OCP**: Used strategies for rules.

## Design Patterns Used
- Strategy Pattern
- Factory Pattern

## Concurrency
- Used `ConcurrentHashMap` for state storage.
- Used `ReentrantLock` for fine-grained locking.

## Tests (JUnit 5 Structure)
```java
@Test
void testHappyPath() {
    // setup, execute, assert
}

@Test
void testConcurrency() {
    // setup threads, await, assert state
}
```

## Extension Points & Trade-offs
- Extensibility: Easy to add new rules via Strategy.
- Trade-offs: In-memory state means data loss on crash. Needs a write-ahead log for persistence.
