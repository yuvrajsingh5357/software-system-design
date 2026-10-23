# SOLID Refactoring Exercises (Expanded)

*Includes 5 Beginner, 5 Intermediate, 5 Advanced exercises.*

## Beginner
1. God Object `SystemManager` (SRP)
2. Open-Ended Enums for Payment (OCP)
3. Throwing `UnsupportedOperationException` (LSP)
4. Massive `Worker` Interface (ISP)
5. Hardcoded `new MySQLConnection()` (DIP)

## Intermediate
6. Tax Calculator with Switch Statements (OCP)
7. Rectangle vs Square Area (LSP)
8. `IAnimal` with `fly()` applied to `Dog` (ISP)
9. Notification Service coupled to `Twilio` (DIP)
10. `UserAuth` handling both DB validation and JWT signing (SRP)

## Advanced
11. Event bus tied directly to concrete Handlers (DIP)
12. Base `Cache` class where `set()` behaves differently for `LRUCache` vs `RedisCache` (LSP)
13. Overly broad `Controller` handling HTTP parsing, business logic, and error formatting (SRP/OCP)
14. A `PricingEngine` that requires modifying 4 files to add a new discount rule (OCP)
15. Injecting a massive `GodService` into a class that only needs one specific method (ISP)
