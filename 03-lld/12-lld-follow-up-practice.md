# LLD Follow-up Practice (Extensibility Tests)

## 1. Parking Lot: Add EV Charging
**Requirement Change:** Some spots now have EV chargers. Vehicles plug in, and get billed per kWh + parking time.
**What breaks?** The `Ticket` calculation logic. The `ParkingSpot` type enum.
**What pattern would help?** Decorator Pattern for `ParkingSpot` (to add charging capabilities dynamically) or Strategy Pattern for `BillingCalculation`.

## 2. Movie Booking: Seat Locking Expiration
**Requirement Change:** Seats lock for 10 minutes. If not paid, they unlock.
**What breaks?** State transitions. We need a timeout mechanism.
**Implementation:** Use a `DelayQueue` or a Redis TTL lock. When the timeout event fires, check if state is still `LOCKED`. If so, revert to `AVAILABLE`.

## 3. Rate Limiter: Add New Algorithm
**Requirement Change:** Transition from Token Bucket to Sliding Window.
**What breaks?** Tightly coupled rate-limiting logic inside the Gateway.
**Implementation:** Strategy Pattern. `interface RateLimitStrategy { boolean allow(); }`. Inject this into the Gateway.

## 4. Chess: Add a new fairy piece (e.g. Amazon)
**Requirement Change:** Add a piece that moves like a Queen and a Knight.
**What breaks?** If `isValidMove()` has massive if/else blocks, it breaks.
**Implementation:** The Piece interface forces `isValidMove()`. The Amazon class simply aggregates the logic from `Knight.isValidMove()` and `Queen.isValidMove()`.
