# LLD Case Study: Elevator System

Design an elevator control system — a classic LLD interview question that tests state machine design, thread safety, and scheduling algorithm thinking.

## Table of Contents
- [Requirements Analysis](#requirements-analysis)
- [Class Design](#class-design)
- [Full Java Implementation](#full-java-implementation)
- [Elevator Scheduling Algorithms](#elevator-scheduling-algorithms)
- [Design Decisions](#design-decisions)
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
1. Building has multiple floors and multiple elevators
2. User can press Up/Down button on any floor (hall call)
3. User inside elevator presses destination floor (cabin call)
4. Elevator moves up/down, stops at requested floors
5. Elevator has states: IDLE, MOVING_UP, MOVING_DOWN
6. Doors open/close at each stop
7. Dispatch algorithm: assign most suitable elevator to a hall call

**Non-Functional:**
1. Thread-safe — elevators run independently in separate threads
2. Extensible — swap scheduling algorithm

**Edge Cases:**
- Elevator at capacity (skip floor)
- Button pressed while elevator already going that direction
- Emergency stop

---

## Class Design

```
ElevatorSystem
├── List<Elevator>
├── dispatch(hallCall)
└── algorithms: NearestElevator, SCAN (look algorithm)

Elevator
├── id: int
├── currentFloor: int
├── state: ElevatorState (IDLE / MOVING_UP / MOVING_DOWN)
├── pendingFloors: TreeSet (sorted floors to stop at)
└── methods: addFloor(), run() (Runnable — runs in its own thread)

HallCall
├── floor: int
├── direction: Direction (UP / DOWN)

ElevatorState (enum): IDLE, MOVING_UP, MOVING_DOWN

Direction (enum): UP, DOWN
```

---

## Full Java Implementation

```java
// ============================================================
// Enums
// ============================================================

public enum Direction { UP, DOWN }

public enum ElevatorState { IDLE, MOVING_UP, MOVING_DOWN }

// ============================================================
// Hall Call — request from a floor
// ============================================================

public class HallCall {
    private final int floor;
    private final Direction direction;
    
    public HallCall(int floor, Direction direction) {
        this.floor = floor;
        this.direction = direction;
    }
    
    public int getFloor() { return floor; }
    public Direction getDirection() { return direction; }
    
    @Override
    public String toString() { return "HallCall[floor=" + floor + ", dir=" + direction + "]"; }
}

// ============================================================
// Elevator — State Machine + Thread
// ============================================================

public class Elevator implements Runnable {
    private final int id;
    private volatile int currentFloor;
    private volatile ElevatorState state;
    
    // TreeSet keeps floors sorted; we always go to next floor in order
    private final TreeSet<Integer> pendingFloors = new TreeSet<>();
    private final Object lock = new Object();
    
    private static final int MOVE_TIME_MS = 1000; // 1 second per floor
    private static final int DOOR_OPEN_TIME_MS = 2000;
    
    public Elevator(int id, int startFloor) {
        this.id = id;
        this.currentFloor = startFloor;
        this.state = ElevatorState.IDLE;
    }
    
    public int getId() { return id; }
    public int getCurrentFloor() { return currentFloor; }
    public ElevatorState getState() { return state; }
    
    /**
     * Add a floor to service (hall call or cabin call)
     */
    public void addFloor(int floor) {
        synchronized (lock) {
            pendingFloors.add(floor);
            lock.notifyAll(); // wake up the elevator thread
        }
    }
    
    /**
     * Elevator cost function — for dispatcher to pick best elevator
     * Lower is better
     */
    public int cost(int requestedFloor, Direction requestedDir) {
        if (state == ElevatorState.IDLE) {
            return Math.abs(currentFloor - requestedFloor); // distance only
        }
        
        if (state == ElevatorState.MOVING_UP && requestedFloor > currentFloor
            && requestedDir == Direction.UP) {
            return requestedFloor - currentFloor; // on the way — very efficient
        }
        
        if (state == ElevatorState.MOVING_DOWN && requestedFloor < currentFloor
            && requestedDir == Direction.DOWN) {
            return currentFloor - requestedFloor; // on the way — very efficient
        }
        
        // Elevator moving in wrong direction — needs full reversal
        return 100 + Math.abs(currentFloor - requestedFloor);
    }
    
    /**
     * Elevator thread — SCAN algorithm (like disk scheduling)
     * Goes in one direction until no more floors in that direction, then reverses
     */
    @Override
    public void run() {
        while (true) {
            Integer nextFloor = null;
            
            synchronized (lock) {
                while (pendingFloors.isEmpty()) {
                    try {
                        state = ElevatorState.IDLE;
                        System.out.printf("[Elevator %d] Idle at floor %d%n", id, currentFloor);
                        lock.wait(); // wait for a request
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
                
                // SCAN: prefer floors in current direction
                if (state == ElevatorState.MOVING_UP || state == ElevatorState.IDLE) {
                    // Try to continue up
                    nextFloor = pendingFloors.ceiling(currentFloor + 1);
                    if (nextFloor == null) {
                        // No floors above — go down
                        nextFloor = pendingFloors.floor(currentFloor - 1);
                    }
                } else { // MOVING_DOWN
                    nextFloor = pendingFloors.floor(currentFloor - 1);
                    if (nextFloor == null) {
                        nextFloor = pendingFloors.ceiling(currentFloor + 1);
                    }
                }
                
                if (nextFloor != null) {
                    pendingFloors.remove(nextFloor);
                }
            }
            
            if (nextFloor != null) {
                moveToFloor(nextFloor);
            }
        }
    }
    
    private void moveToFloor(int targetFloor) {
        if (targetFloor > currentFloor) state = ElevatorState.MOVING_UP;
        else if (targetFloor < currentFloor) state = ElevatorState.MOVING_DOWN;
        
        System.out.printf("[Elevator %d] Moving %s from floor %d to floor %d%n",
            id, state, currentFloor, targetFloor);
        
        // Simulate movement
        int floors = Math.abs(targetFloor - currentFloor);
        try {
            Thread.sleep((long) floors * MOVE_TIME_MS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return;
        }
        
        currentFloor = targetFloor;
        openDoors();
    }
    
    private void openDoors() {
        System.out.printf("[Elevator %d] 🚪 Doors opening at floor %d%n", id, currentFloor);
        try {
            Thread.sleep(DOOR_OPEN_TIME_MS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.printf("[Elevator %d] 🚪 Doors closing at floor %d%n", id, currentFloor);
    }
    
    @Override
    public String toString() {
        return String.format("Elevator[id=%d, floor=%d, state=%s, pending=%s]",
            id, currentFloor, state, pendingFloors);
    }
}

// ============================================================
// Scheduling Strategy (Strategy Pattern for extensibility)
// ============================================================

public interface ElevatorScheduler {
    Elevator dispatch(List<Elevator> elevators, HallCall call);
}

// Nearest elevator that is already serving the call direction
public class OptimalScheduler implements ElevatorScheduler {
    @Override
    public Elevator dispatch(List<Elevator> elevators, HallCall call) {
        return elevators.stream()
            .min(Comparator.comparingInt(e -> e.cost(call.getFloor(), call.getDirection())))
            .orElseThrow(() -> new RuntimeException("No elevators available"));
    }
}

// Simple nearest elevator (ignores direction)
public class NearestElevatorScheduler implements ElevatorScheduler {
    @Override
    public Elevator dispatch(List<Elevator> elevators, HallCall call) {
        return elevators.stream()
            .min(Comparator.comparingInt(e -> Math.abs(e.getCurrentFloor() - call.getFloor())))
            .orElseThrow();
    }
}

// ============================================================
// ElevatorSystem — Facade
// ============================================================

public class ElevatorSystem {
    private final List<Elevator> elevators;
    private final ElevatorScheduler scheduler;
    private final ExecutorService pool;
    
    public ElevatorSystem(int numElevators, int startFloor, ElevatorScheduler scheduler) {
        this.scheduler = scheduler;
        this.elevators = new ArrayList<>();
        this.pool = Executors.newFixedThreadPool(numElevators);
        
        for (int i = 1; i <= numElevators; i++) {
            Elevator elevator = new Elevator(i, startFloor);
            elevators.add(elevator);
            pool.execute(elevator); // start each elevator's thread
        }
    }
    
    /**
     * User pressed Up/Down button on a floor
     */
    public void hallCall(int floor, Direction direction) {
        HallCall call = new HallCall(floor, direction);
        Elevator chosen = scheduler.dispatch(elevators, call);
        System.out.printf("📍 Hall call at floor %d (%s) → dispatching Elevator %d%n",
            floor, direction, chosen.getId());
        chosen.addFloor(floor);
    }
    
    /**
     * User inside elevator pressed a floor button
     */
    public void cabinCall(int elevatorId, int destinationFloor) {
        elevators.stream()
            .filter(e -> e.getId() == elevatorId)
            .findFirst()
            .ifPresent(elevator -> {
                System.out.printf("🎯 Cabin call in Elevator %d → floor %d%n", elevatorId, destinationFloor);
                elevator.addFloor(destinationFloor);
            });
    }
    
    public void displayStatus() {
        System.out.println("\n=== Elevator System Status ===");
        elevators.forEach(System.out::println);
        System.out.println("==============================\n");
    }
    
    public void shutdown() {
        pool.shutdownNow();
    }
}

// ============================================================
// Demo
// ============================================================

public class ElevatorDemo {
    public static void main(String[] args) throws InterruptedException {
        ElevatorSystem system = new ElevatorSystem(3, 1, new OptimalScheduler());
        
        Thread.sleep(500); // let elevators initialize
        
        // Hall calls
        system.hallCall(5, Direction.UP);
        system.hallCall(3, Direction.DOWN);
        system.hallCall(8, Direction.UP);
        
        Thread.sleep(3000); // wait for elevators to arrive
        
        // Cabin calls (user inside elevator 1 wants to go to floor 10)
        system.cabinCall(1, 10);
        system.cabinCall(2, 1);
        
        system.displayStatus();
        
        Thread.sleep(15000); // let everything settle
        system.shutdown();
    }
}
```

---

## Elevator Scheduling Algorithms

| Algorithm | How it works | Pros | Cons |
|-----------|-------------|------|------|
| **FCFS** | First come, first served | Simple | Inefficient; long waits |
| **Nearest Car** | Closest idle elevator | Low average travel | Ignores direction |
| **SCAN (Look)** | Goes one direction until end, reverses | Good throughput | Can starve far floors |
| **Optimal** | Pick elevator with lowest "cost" considering direction | Best real-world | More complex |

**Our implementation uses SCAN** for the elevator movement (TreeSet + ceiling/floor) and **Optimal scheduling** for dispatch (cost function).

---

## Design Decisions

### Why TreeSet for pendingFloors?

TreeSet keeps floors sorted and supports `ceiling(x)` and `floor(x)` operations in O(log n) — perfect for SCAN:
- `ceiling(currentFloor + 1)` → next floor above
- `floor(currentFloor - 1)` → next floor below

### Why volatile for currentFloor and state?

The elevator's thread updates `currentFloor` and `state`, while the dispatcher thread reads them to compute `cost()`. Without `volatile`, the dispatcher might read stale cached values.

### Why separate Runnable for each Elevator?

Each elevator runs independently — blocking one doesn't affect others. `Thread.sleep()` for movement simulation is fine because each elevator is in its own thread.

---

## Interview Follow-ups

1. What if traffic increases 100x?
2. What if two people press the button simultaneously?
3. What if the elevator breaks down mid-transit?
4. What if a new scheduling algorithm is introduced?
5. What if the building adds a VIP or service elevator?
6. What if the building has 100 floors and 20 elevators? How do you optimize?
7. What if the power goes out?
8. How do you handle emergency fire modes?
9. What if the weight limit is exceeded?
10. How do you load balance requests across multiple elevators effectively?

---

## Interview Tips

1. **State machine is key** — draw IDLE → MOVING_UP → MOVING_DOWN → IDLE transitions first.
2. **Mention the scheduling algorithm** — shows you've thought beyond basic CRUD.
3. **Thread safety** — "Each elevator runs in its own thread; I use `synchronized` + `wait/notifyAll` for the pending floors set."
4. **Strategy pattern for scheduler** — shows extensibility thinking.
5. **Ask about scale** — "Is this 10 floors or 100? 2 elevators or 20?"

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Elevator states | IDLE, MOVING_UP, MOVING_DOWN |
| 2 | Pending floors data structure | TreeSet — sorted, supports ceiling/floor ops |
| 3 | SCAN algorithm | Go up until no more floors above → go down |
| 4 | Hall call | User presses button on a floor; dispatcher assigns elevator |
| 5 | Cabin call | User inside elevator presses destination floor |
| 6 | Cost function | IDLE: distance; same direction: distance; wrong dir: 100+distance |
| 7 | Thread per elevator | Each Elevator implements Runnable; runs independently |
| 8 | volatile usage | currentFloor and state — read by dispatcher thread |
| 9 | synchronized usage | pendingFloors modifications + wait/notifyAll |
| 10 | Scheduler as Strategy | OptimalScheduler, NearestScheduler — swappable |
| 11 | wake elevator | lock.notifyAll() when floor added; elevator was lock.wait() |
| 12 | ElevatorSystem role | Facade: hides elevator management, dispatching |
| 13 | Design patterns | State, Strategy (scheduler), Facade (ElevatorSystem) |
| 14 | SOLID in design | OCP: add new scheduler without changing ElevatorSystem |
| 15 | Extension: capacity | Add currentPassengers field; check before addFloor() |

---

*Next: [LLD Case Study — Movie Ticket Booking](04-lld-movie-ticket-booking.md)*
