# LLD Case Study: Parking Lot System

One of the most common LLD interview questions. This chapter walks through the complete design process — from requirements to full Java implementation.

## Table of Contents
- [Requirements Analysis](#requirements-analysis)
- [Identifying Entities](#identifying-entities)
- [Class Design](#class-design)
- [Full Java Implementation](#full-java-implementation)
- [Design Decisions and Trade-offs](#design-decisions-and-trade-offs)
- [Extension Points](#extension-points)
- [Interview Tips](#interview-tips)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Interview Flow

1. Clarify requirements
2. Identify actors
3. Identify core entities
4. Identify relationships
5. Identify responsibilities
6. Define interfaces
7. Choose abstractions
8. Select patterns only where justified
9. Handle concurrency
10. Discuss extensibility
11. Write code
12. Test
13. Discuss trade-offs

---

## Requirements Analysis

**Functional Requirements:**
1. Support multiple levels/floors in the parking lot
2. Support different vehicle types: motorcycle, car, bus
3. Different spot sizes: small (motorcycle), medium (car), large (bus)
4. Park a vehicle — assign the nearest available spot
5. Unpark a vehicle — free the spot
6. Calculate parking fee (hourly rate by vehicle type)
7. Generate a ticket on parking
8. Display availability counts per spot type

**Non-Functional Requirements:**
1. Thread-safe — multiple entry gates may park/unpark simultaneously
2. Extensible — easy to add new vehicle types, pricing models
3. Testable — parking logic decoupled from persistence

**Edge Cases:**
- Lot is full (no available spots)
- Vehicle type doesn't match spot size
- Parking for the same vehicle already exists (duplicate entry)
- Ticket not found on exit

---

## Identifying Entities

**Noun extraction from requirements:**

```
ParkingLot → manages everything
Floor / Level → has many spots
ParkingSpot → has size, availability
Vehicle → has type (Motorcycle, Car, Bus)
Ticket → records entry time, spot, vehicle
ParkingFeeCalculator → computes fee based on duration and type
Gate (entry/exit) → interacts with system
```

---

## Class Design

```
ParkingLot
├── id: String
├── levels: List<Level>
└── methods: park(vehicle), unpark(ticket), getAvailableCount(SpotType)

Level
├── levelNumber: int
├── spots: List<ParkingSpot>
└── methods: getAvailableSpot(SpotType), parkVehicle(vehicle), unparkVehicle(spotId)

ParkingSpot
├── id: String
├── type: SpotType (SMALL, MEDIUM, LARGE)
├── isOccupied: boolean
└── methods: park(vehicle), unpark()

Vehicle (abstract)
├── licensePlate: String
├── vehicleType: VehicleType
└── getRequiredSpotType(): SpotType  ← determines which spot it needs

Ticket
├── ticketId: String
├── vehicle: Vehicle
├── spot: ParkingSpot
├── entryTime: LocalDateTime
└── exitTime: LocalDateTime

FeeCalculator (interface)
└── calculateFee(ticket): double

HourlyFeeCalculator implements FeeCalculator
VehicleTypeFeeCalculator implements FeeCalculator
```

---

## Full Java Implementation

```java
// ============================================================
// Enums
// ============================================================

public enum VehicleType { MOTORCYCLE, CAR, BUS }
public enum SpotType { SMALL, MEDIUM, LARGE }

// ============================================================
// Vehicles
// ============================================================

public abstract class Vehicle {
    private final String licensePlate;
    private final VehicleType type;
    
    protected Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public String getLicensePlate() { return licensePlate; }
    public VehicleType getType() { return type; }
    public abstract SpotType getRequiredSpotType();
    
    @Override
    public String toString() { return type + "[" + licensePlate + "]"; }
}

public class Motorcycle extends Vehicle {
    public Motorcycle(String plate) { super(plate, VehicleType.MOTORCYCLE); }
    public SpotType getRequiredSpotType() { return SpotType.SMALL; }
}

public class Car extends Vehicle {
    public Car(String plate) { super(plate, VehicleType.CAR); }
    public SpotType getRequiredSpotType() { return SpotType.MEDIUM; }
}

public class Bus extends Vehicle {
    public Bus(String plate) { super(plate, VehicleType.BUS); }
    public SpotType getRequiredSpotType() { return SpotType.LARGE; }
}

// ============================================================
// ParkingSpot
// ============================================================

public class ParkingSpot {
    private final String spotId;
    private final SpotType type;
    private Vehicle parkedVehicle;
    
    public ParkingSpot(String spotId, SpotType type) {
        this.spotId = spotId;
        this.type = type;
    }
    
    public String getSpotId() { return spotId; }
    public SpotType getType() { return type; }
    public boolean isAvailable() { return parkedVehicle == null; }
    
    // Synchronized to prevent race condition at spot level
    public synchronized boolean park(Vehicle vehicle) {
        if (!isAvailable()) return false;
        if (vehicle.getRequiredSpotType() != this.type) return false;
        this.parkedVehicle = vehicle;
        return true;
    }
    
    public synchronized Vehicle unpark() {
        Vehicle v = parkedVehicle;
        parkedVehicle = null;
        return v;
    }
    
    @Override
    public String toString() { return "Spot[" + spotId + "," + type + "," + (isAvailable() ? "FREE" : "OCCUPIED") + "]"; }
}

// ============================================================
// Ticket
// ============================================================

import java.time.LocalDateTime;
import java.util.UUID;

public class Ticket {
    private final String ticketId;
    private final Vehicle vehicle;
    private final ParkingSpot spot;
    private final LocalDateTime entryTime;
    private LocalDateTime exitTime;
    
    public Ticket(Vehicle vehicle, ParkingSpot spot) {
        this.ticketId = UUID.randomUUID().toString();
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = LocalDateTime.now();
    }
    
    public String getTicketId() { return ticketId; }
    public Vehicle getVehicle() { return vehicle; }
    public ParkingSpot getSpot() { return spot; }
    public LocalDateTime getEntryTime() { return entryTime; }
    public LocalDateTime getExitTime() { return exitTime; }
    
    public void setExitTime(LocalDateTime exitTime) { this.exitTime = exitTime; }
    
    @Override
    public String toString() {
        return String.format("Ticket[%s | %s | Spot: %s | In: %s]",
            ticketId.substring(0, 8), vehicle, spot.getSpotId(), entryTime);
    }
}

// ============================================================
// Fee Calculator (Strategy Pattern for extensibility)
// ============================================================

public interface FeeCalculator {
    double calculateFee(Ticket ticket);
}

public class HourlyFeeCalculator implements FeeCalculator {
    private final Map<VehicleType, Double> ratePerHour;
    
    public HourlyFeeCalculator() {
        ratePerHour = new EnumMap<>(VehicleType.class);
        ratePerHour.put(VehicleType.MOTORCYCLE, 10.0); // ₹10/hour
        ratePerHour.put(VehicleType.CAR, 20.0);        // ₹20/hour
        ratePerHour.put(VehicleType.BUS, 50.0);        // ₹50/hour
    }
    
    @Override
    public double calculateFee(Ticket ticket) {
        LocalDateTime exit = ticket.getExitTime() != null ? ticket.getExitTime() : LocalDateTime.now();
        long minutes = java.time.Duration.between(ticket.getEntryTime(), exit).toMinutes();
        double hours = Math.ceil(minutes / 60.0); // round up to nearest hour
        double rate = ratePerHour.getOrDefault(ticket.getVehicle().getType(), 20.0);
        return hours * rate;
    }
}

// ============================================================
// Level / Floor
// ============================================================

public class Level {
    private final int levelNumber;
    private final List<ParkingSpot> spots;
    
    public Level(int levelNumber, int smallSpots, int mediumSpots, int largeSpots) {
        this.levelNumber = levelNumber;
        this.spots = new ArrayList<>();
        
        for (int i = 0; i < smallSpots; i++)
            spots.add(new ParkingSpot("L" + levelNumber + "-S" + i, SpotType.SMALL));
        for (int i = 0; i < mediumSpots; i++)
            spots.add(new ParkingSpot("L" + levelNumber + "-M" + i, SpotType.MEDIUM));
        for (int i = 0; i < largeSpots; i++)
            spots.add(new ParkingSpot("L" + levelNumber + "-L" + i, SpotType.LARGE));
    }
    
    // Returns the first available spot for this vehicle type
    public Optional<ParkingSpot> getAvailableSpot(SpotType type) {
        return spots.stream()
            .filter(s -> s.getType() == type && s.isAvailable())
            .findFirst();
    }
    
    public long getAvailableCount(SpotType type) {
        return spots.stream().filter(s -> s.getType() == type && s.isAvailable()).count();
    }
    
    public int getLevelNumber() { return levelNumber; }
}

// ============================================================
// ParkingLot — the main facade
// ============================================================

public class ParkingLot {
    private static volatile ParkingLot instance; // Singleton (thread-safe)
    
    private final String id;
    private final List<Level> levels;
    private final Map<String, Ticket> activeTickets; // ticketId → Ticket
    private final Map<String, Ticket> vehicleToTicket; // licensePlate → Ticket
    private final FeeCalculator feeCalculator;
    
    private ParkingLot(String id, List<Level> levels, FeeCalculator feeCalculator) {
        this.id = id;
        this.levels = new ArrayList<>(levels);
        this.activeTickets = new ConcurrentHashMap<>();
        this.vehicleToTicket = new ConcurrentHashMap<>();
        this.feeCalculator = feeCalculator;
    }
    
    // Bill Pugh Singleton (thread-safe, lazy)
    public static ParkingLot getInstance(String id, List<Level> levels, FeeCalculator calc) {
        if (instance == null) {
            synchronized (ParkingLot.class) {
                if (instance == null) {
                    instance = new ParkingLot(id, levels, calc);
                }
            }
        }
        return instance;
    }
    
    /**
     * Park a vehicle — returns a ticket or throws exception if full/duplicate
     */
    public Ticket park(Vehicle vehicle) {
        // Check for duplicate entry
        if (vehicleToTicket.containsKey(vehicle.getLicensePlate())) {
            throw new IllegalStateException("Vehicle already parked: " + vehicle.getLicensePlate());
        }
        
        SpotType required = vehicle.getRequiredSpotType();
        
        // Find first available spot across all levels (nearest = lowest level)
        for (Level level : levels) {
            Optional<ParkingSpot> spotOpt = level.getAvailableSpot(required);
            if (spotOpt.isPresent()) {
                ParkingSpot spot = spotOpt.get();
                boolean parked = spot.park(vehicle); // thread-safe at spot level
                if (parked) {
                    Ticket ticket = new Ticket(vehicle, spot);
                    activeTickets.put(ticket.getTicketId(), ticket);
                    vehicleToTicket.put(vehicle.getLicensePlate(), ticket);
                    System.out.printf("✅ Parked %s at %s | Ticket: %s%n",
                        vehicle, spot, ticket.getTicketId().substring(0, 8));
                    return ticket;
                }
            }
        }
        throw new RuntimeException("Parking lot is full for " + required + " spots");
    }
    
    /**
     * Unpark a vehicle — returns the fee to pay
     */
    public double unpark(String ticketId) {
        Ticket ticket = activeTickets.get(ticketId);
        if (ticket == null) throw new IllegalArgumentException("Invalid ticket: " + ticketId);
        
        ticket.setExitTime(LocalDateTime.now());
        double fee = feeCalculator.calculateFee(ticket);
        
        // Free the spot
        ticket.getSpot().unpark();
        
        // Clean up tracking maps
        activeTickets.remove(ticketId);
        vehicleToTicket.remove(ticket.getVehicle().getLicensePlate());
        
        System.out.printf("🚗 Unparked %s | Duration: %s | Fee: ₹%.2f%n",
            ticket.getVehicle(),
            java.time.Duration.between(ticket.getEntryTime(), ticket.getExitTime()),
            fee);
        return fee;
    }
    
    /**
     * Display availability status
     */
    public void displayAvailability() {
        System.out.println("\n=== Parking Lot Availability ===");
        for (Level level : levels) {
            System.out.printf("Level %d: Small=%d | Medium=%d | Large=%d%n",
                level.getLevelNumber(),
                level.getAvailableCount(SpotType.SMALL),
                level.getAvailableCount(SpotType.MEDIUM),
                level.getAvailableCount(SpotType.LARGE));
        }
        System.out.println("================================\n");
    }
}

// ============================================================
// Main / Demo
// ============================================================

public class ParkingLotDemo {
    public static void main(String[] args) throws InterruptedException {
        // Build parking lot: 2 levels, each with 5 small, 10 medium, 3 large spots
        List<Level> levels = List.of(
            new Level(1, 5, 10, 3),
            new Level(2, 5, 10, 3)
        );
        
        ParkingLot lot = ParkingLot.getInstance("LOT-001", levels, new HourlyFeeCalculator());
        lot.displayAvailability();
        
        // Park vehicles
        Vehicle car1 = new Car("MH-01-AB-1234");
        Vehicle moto1 = new Motorcycle("MH-02-CD-5678");
        Vehicle bus1 = new Bus("MH-03-EF-9012");
        
        Ticket carTicket = lot.park(car1);
        Ticket motoTicket = lot.park(moto1);
        Ticket busTicket = lot.park(bus1);
        
        lot.displayAvailability();
        
        // Simulate time passing
        Thread.sleep(2000);
        
        // Unpark
        lot.unpark(carTicket.getTicketId());
        lot.displayAvailability();
    }
}
```

---

## Design Decisions and Trade-offs

### Decision 1: Singleton for ParkingLot

**Why:** Only one parking lot entity exists in the system. Singleton ensures central state management.

**Alternative:** Pass `ParkingLot` via constructor (DI) — better for testing. In a real microservice, the singleton would be the Spring-managed bean.

### Decision 2: Strategy Pattern for FeeCalculator

**Why:** Fee rules change frequently (hourly, flat rate, weekend pricing, member discounts). Strategy pattern means adding new pricing = new class, no modification to ParkingLot.

### Decision 3: Thread safety at ParkingSpot level

**Why:** Two entry gates may try to park in the same spot simultaneously. `synchronized` on `park()` prevents double-booking.

**Alternative:** `AtomicBoolean` for spot availability — faster (no lock contention):
```java
private final AtomicBoolean isOccupied = new AtomicBoolean(false);
public boolean park(Vehicle v) {
    if (isOccupied.compareAndSet(false, true)) {
        this.vehicle = v; // race condition risk! needs volatile or lock
        return true;
    }
    return false;
}
```
Full lock-free safety requires more careful design — `synchronized` is simpler and correct.

### Decision 4: ConcurrentHashMap for ticket tracking

**Why:** Multiple threads reading/writing ticket maps. ConcurrentHashMap provides thread-safe operations without global lock.

---

## Extension Points

```
1. Add VehicleType.ELECTRIC → new ElectricCar extends Vehicle
   → new ChargingSpot with charging capability
   → No changes to ParkingLot

2. Add weekend pricing → new WeekendFeeCalculator implements FeeCalculator
   → Inject based on day of week

3. Add reservation system → ReservationService works with ParkingLot API
   → Spots can be pre-reserved; mark as "RESERVED" state

4. Add mobile app API → ParkingLotController exposes REST endpoints
   → ParkingLot is the service layer; no changes needed

5. Add persistence → JpaTicketRepository implements TicketRepository
   → Inject instead of in-memory map
```

---

## Interview Follow-ups

1. What if traffic increases 100x?
2. What if two threads book the same parking spot?
3. What if payment succeeds but ticket generation fails?
4. What if a new payment provider is added?
5. What if the fee calculation rules change dynamically based on demand?
6. What if a new vehicle type is introduced?
7. What if the database becomes unavailable?
8. What if operations must become asynchronous?
9. How would you handle VIP parking reservations?
10. What if the parking lot spans multiple buildings?

---

## Interview Tips

1. **Always start with requirements** — ask: "Do we support multiple entry gates? Multiple vehicle types? Multiple floors?"

2. **Draw the class diagram first** — identify entities, their attributes, and relationships before writing code.

3. **Mention thread safety early** — "Since multiple gates operate simultaneously, I'll use synchronized or ConcurrentHashMap."

4. **Apply SRP** — separate Vehicle, ParkingSpot, Ticket, FeeCalculator as distinct responsibilities.

5. **Use Strategy for fee calculation** — shows OCP awareness.

6. **Handle edge cases explicitly** — full lot, duplicate entry, invalid ticket.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Main entities | ParkingLot, Level, ParkingSpot, Vehicle, Ticket, FeeCalculator |
| 2 | Vehicle → SpotType mapping | Motorcycle → SMALL, Car → MEDIUM, Bus → LARGE |
| 3 | Thread safety at spot level | synchronized on park()/unpark() |
| 4 | Thread safety for maps | ConcurrentHashMap for activeTickets, vehicleToTicket |
| 5 | Fee extensibility | Strategy pattern — FeeCalculator interface |
| 6 | Singleton usage | ParkingLot is singleton (double-checked locking) |
| 7 | Nearest spot strategy | First available in lowest level |
| 8 | Duplicate entry handling | vehicleToTicket map check before parking |
| 9 | Ticket ID | UUID — globally unique |
| 10 | Full lot response | throw RuntimeException with clear message |
| 11 | Add new vehicle type | Extend Vehicle class; override getRequiredSpotType() |
| 12 | Add new pricing | Implement FeeCalculator interface |
| 13 | Design patterns used | Singleton (ParkingLot), Strategy (FeeCalculator), Facade (ParkingLot hides Level/Spot) |
| 14 | SOLID applied | SRP: each class one job; OCP: new vehicle = new class; DIP: FeeCalculator injected |
| 15 | Extension: reservations | Add RESERVED state to SpotStatus enum; ReservationService uses ParkingLot API |

---

*Next: [LLD Case Study — Elevator System](03-lld-elevator-system.md)*
