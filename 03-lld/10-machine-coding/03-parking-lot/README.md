# Parking Lot — Machine Coding

## Time Limit
90 minutes

## Requirements
Design and implement a Parking Lot.
- Support multiple operations concurrently.
- Maintain a clean domain model.
- (Assume in-memory storage unless otherwise specified).

## Supported Operations
- `init(...)`
- `execute(...)`
- `status(...)`

## Constraints
- Memory is limited to standard heap sizes.
- Sub-millisecond latency expected for core operations.

## Expected Behavior
The system should gracefully handle edge cases and invalid inputs without crashing.

## Extension Requirements
- What if the system needs to be distributed?
- What if a new requirement is added tomorrow? (Design for Open/Closed Principle).

## What the interviewer is evaluating
- Object-Oriented Design (Entities, Encapsulation).
- Clean Code & SOLID principles.
- Working, bug-free implementation.
- Handling of Concurrency/Race conditions.
- Testability.

DO NOT READ THE REFERENCE IMPLEMENTATION BEFORE ATTEMPTING THE PROBLEM.
