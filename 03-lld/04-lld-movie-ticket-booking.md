# LLD Case Study: Movie Ticket Booking System

Design a Bookmyshow-style movie ticket booking system — tests concurrent booking, seat locking, and complex entity relationships.

## Table of Contents
- [Requirements Analysis](#requirements-analysis)
- [Class Design](#class-design)
- [Full Java Implementation](#full-java-implementation)
- [Concurrency: Seat Locking](#concurrency-seat-locking)
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
1. Browse movies currently showing
2. Search shows by movie, city, date
3. View seat layout for a show (AVAILABLE / BOOKED / LOCKED)
4. Select and temporarily lock seats (10-minute hold)
5. Book locked seats with payment
6. Cancel a booking — seats released
7. Generate booking confirmation with ticket details

**Non-Functional Requirements:**
1. High concurrency — many users may try to book the same seat simultaneously
2. Seat locking must prevent double-booking (consistency over availability)
3. Lock timeout — seats released if payment not made in 10 minutes

**Constraints:**
- Seat locking is the hardest problem — must be atomic

---

## Class Design

```
Movie
├── id, title, language, durationMinutes, genre

Theater
├── id, name, city
└── List<Auditorium>

Auditorium
├── id, name, totalSeats
└── List<Row>

Row
├── rowLabel (A, B, C...)
└── List<Seat>

Seat
├── seatId, rowLabel, seatNumber, SeatType (SILVER, GOLD, PLATINUM)
└── status: SeatStatus (AVAILABLE, LOCKED, BOOKED)

Show
├── id, movie, auditorium, startTime, endTime
└── Map<String, ShowSeat>  (seatId → ShowSeat)

ShowSeat
├── seat, status, lockedByUser, lockExpiry
└── methods: lock(userId), unlock(), book()

Booking
├── bookingId, show, user, List<ShowSeat>, totalPrice, BookingStatus

BookingService
└── lockSeats(), book(), cancelBooking()
```

---

## Full Java Implementation

```java
// ============================================================
// Enums
// ============================================================

public enum SeatType { SILVER, GOLD, PLATINUM }

public enum SeatStatus { AVAILABLE, LOCKED, BOOKED }

public enum BookingStatus { PENDING, CONFIRMED, CANCELLED }

// ============================================================
// Core Domain Objects
// ============================================================

public class Movie {
    private final String movieId;
    private final String title;
    private final String language;
    private final int durationMinutes;
    private final String genre;
    
    public Movie(String id, String title, String language, int duration, String genre) {
        this.movieId = id; this.title = title; this.language = language;
        this.durationMinutes = duration; this.genre = genre;
    }
    
    public String getTitle() { return title; }
    public String getMovieId() { return movieId; }
    @Override public String toString() { return title + " [" + language + ", " + durationMinutes + " min]"; }
}

public class Seat {
    private final String seatId;
    private final String rowLabel;
    private final int seatNumber;
    private final SeatType type;
    
    public Seat(String rowLabel, int seatNumber, SeatType type) {
        this.seatId = rowLabel + seatNumber;
        this.rowLabel = rowLabel;
        this.seatNumber = seatNumber;
        this.type = type;
    }
    
    public String getSeatId() { return seatId; }
    public SeatType getType() { return type; }
    @Override public String toString() { return seatId + "[" + type + "]"; }
}

// ============================================================
// ShowSeat — per-show seat state (critical: thread-safe)
// ============================================================

public class ShowSeat {
    private final Seat seat;
    private SeatStatus status;
    private String lockedByUserId;
    private LocalDateTime lockExpiry;
    
    private static final Duration LOCK_DURATION = Duration.ofMinutes(10);
    
    public ShowSeat(Seat seat) {
        this.seat = seat;
        this.status = SeatStatus.AVAILABLE;
    }
    
    public Seat getSeat() { return seat; }
    public SeatStatus getStatus() { return status; }
    
    /**
     * Attempt to lock this seat for a user.
     * Thread-safety: caller must hold lock on this ShowSeat.
     */
    public boolean lock(String userId) {
        // Release expired locks
        if (status == SeatStatus.LOCKED && LocalDateTime.now().isAfter(lockExpiry)) {
            status = SeatStatus.AVAILABLE;
        }
        
        if (status != SeatStatus.AVAILABLE) return false;
        
        status = SeatStatus.LOCKED;
        lockedByUserId = userId;
        lockExpiry = LocalDateTime.now().plus(LOCK_DURATION);
        return true;
    }
    
    /**
     * Confirm booking (LOCKED → BOOKED).
     * Thread-safety: caller must hold lock on this ShowSeat.
     */
    public boolean book(String userId) {
        if (status != SeatStatus.LOCKED || !userId.equals(lockedByUserId)) return false;
        if (LocalDateTime.now().isAfter(lockExpiry)) {
            status = SeatStatus.AVAILABLE;
            return false; // lock expired
        }
        status = SeatStatus.BOOKED;
        return true;
    }
    
    /**
     * Release seat (LOCKED/BOOKED → AVAILABLE).
     */
    public void release() {
        status = SeatStatus.AVAILABLE;
        lockedByUserId = null;
        lockExpiry = null;
    }
    
    @Override
    public String toString() {
        return seat.getSeatId() + "[" + status + "]";
    }
}

// ============================================================
// Show — a movie playing at a specific time in an auditorium
// ============================================================

public class Show {
    private final String showId;
    private final Movie movie;
    private final String auditoriumId;
    private final LocalDateTime startTime;
    private final LocalDateTime endTime;
    
    // seatId → ShowSeat; accessed concurrently
    private final Map<String, ShowSeat> showSeats = new ConcurrentHashMap<>();
    
    // Pricing by seat type
    private final Map<SeatType, Double> pricing;
    
    public Show(String showId, Movie movie, String auditoriumId,
                LocalDateTime startTime, List<Seat> seats, Map<SeatType, Double> pricing) {
        this.showId = showId;
        this.movie = movie;
        this.auditoriumId = auditoriumId;
        this.startTime = startTime;
        this.endTime = startTime.plusMinutes(movie.getDurationMinutes() + 30); // 30 min buffer
        this.pricing = new EnumMap<>(pricing);
        
        for (Seat seat : seats) {
            showSeats.put(seat.getSeatId(), new ShowSeat(seat));
        }
    }
    
    public String getShowId() { return showId; }
    public Movie getMovie() { return movie; }
    
    public double getPriceForSeat(Seat seat) {
        return pricing.getOrDefault(seat.getType(), 150.0);
    }
    
    /**
     * Get available seats for display
     */
    public List<ShowSeat> getAvailableSeats() {
        return showSeats.values().stream()
            .filter(ss -> ss.getStatus() == SeatStatus.AVAILABLE)
            .collect(Collectors.toList());
    }
    
    public ShowSeat getShowSeat(String seatId) {
        return showSeats.get(seatId);
    }
    
    public Map<String, ShowSeat> getAllShowSeats() { return Collections.unmodifiableMap(showSeats); }
}

// ============================================================
// Booking
// ============================================================

public class Booking {
    private final String bookingId;
    private final Show show;
    private final String userId;
    private final List<ShowSeat> seats;
    private final double totalPrice;
    private BookingStatus status;
    private final LocalDateTime createdAt;
    
    public Booking(Show show, String userId, List<ShowSeat> seats) {
        this.bookingId = "BKG-" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
        this.show = show;
        this.userId = userId;
        this.seats = new ArrayList<>(seats);
        this.totalPrice = seats.stream()
            .mapToDouble(ss -> show.getPriceForSeat(ss.getSeat()))
            .sum();
        this.status = BookingStatus.PENDING;
        this.createdAt = LocalDateTime.now();
    }
    
    public String getBookingId() { return bookingId; }
    public BookingStatus getStatus() { return status; }
    public void setStatus(BookingStatus status) { this.status = status; }
    public List<ShowSeat> getSeats() { return Collections.unmodifiableList(seats); }
    public double getTotalPrice() { return totalPrice; }
    
    @Override
    public String toString() {
        return String.format("Booking[%s | %s | Seats: %s | ₹%.2f | %s]",
            bookingId, show.getMovie().getTitle(), seats, totalPrice, status);
    }
}

// ============================================================
// BookingService — core business logic
// ============================================================

public class BookingService {
    private final Map<String, Show> shows = new ConcurrentHashMap<>();
    private final Map<String, Booking> bookings = new ConcurrentHashMap<>();
    private final PaymentService paymentService;
    private final NotificationService notificationService;
    
    public BookingService(PaymentService paymentService, NotificationService notificationService) {
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }
    
    public void addShow(Show show) { shows.put(show.getShowId(), show); }
    
    /**
     * Step 1: Lock seats for a user — returns temporary hold
     * Thread-safe: synchronizes on each ShowSeat independently
     */
    public Booking lockSeats(String showId, String userId, List<String> seatIds) {
        Show show = getShow(showId);
        List<ShowSeat> toBook = new ArrayList<>();
        
        // Lock seats one by one; if any fails, release all already locked
        // Sort by seatId to prevent deadlock when two users lock overlapping sets
        List<String> sortedSeatIds = new ArrayList<>(seatIds);
        Collections.sort(sortedSeatIds);
        
        List<ShowSeat> locked = new ArrayList<>();
        try {
            for (String seatId : sortedSeatIds) {
                ShowSeat showSeat = show.getShowSeat(seatId);
                if (showSeat == null) throw new IllegalArgumentException("Seat not found: " + seatId);
                
                synchronized (showSeat) {
                    if (!showSeat.lock(userId)) {
                        throw new SeatUnavailableException("Seat " + seatId + " is not available");
                    }
                    locked.add(showSeat);
                }
            }
        } catch (Exception e) {
            // Release all seats we already locked
            locked.forEach(ss -> { synchronized(ss) { ss.release(); } });
            throw new RuntimeException("Failed to lock seats: " + e.getMessage(), e);
        }
        
        Booking booking = new Booking(show, userId, locked);
        bookings.put(booking.getBookingId(), booking);
        System.out.printf("🔒 Seats locked: %s for user %s (hold for 10 min)%n", seatIds, userId);
        return booking;
    }
    
    /**
     * Step 2: Confirm booking after payment
     */
    public Booking confirmBooking(String bookingId, PaymentDetails payment) {
        Booking booking = getBooking(bookingId);
        if (booking.getStatus() != BookingStatus.PENDING) {
            throw new IllegalStateException("Booking is not in PENDING state: " + bookingId);
        }
        
        // Process payment
        boolean paid = paymentService.processPayment(payment, booking.getTotalPrice());
        if (!paid) {
            // Release seats
            cancelBooking(bookingId);
            throw new PaymentFailedException("Payment failed for booking: " + bookingId);
        }
        
        // Confirm each seat
        String userId = booking.getSeats().get(0).getSeat().getSeatId(); // simplified
        for (ShowSeat showSeat : booking.getSeats()) {
            synchronized (showSeat) {
                showSeat.book(booking.getBookingId()); // lock expires → BOOKED
            }
        }
        
        booking.setStatus(BookingStatus.CONFIRMED);
        notificationService.sendConfirmation(booking);
        System.out.println("✅ Booking confirmed: " + booking);
        return booking;
    }
    
    /**
     * Cancel booking — release seats
     */
    public void cancelBooking(String bookingId) {
        Booking booking = getBooking(bookingId);
        if (booking.getStatus() == BookingStatus.CANCELLED) return;
        
        for (ShowSeat showSeat : booking.getSeats()) {
            synchronized (showSeat) {
                showSeat.release();
            }
        }
        
        booking.setStatus(BookingStatus.CANCELLED);
        System.out.println("❌ Booking cancelled: " + bookingId);
    }
    
    private Show getShow(String showId) {
        Show show = shows.get(showId);
        if (show == null) throw new IllegalArgumentException("Show not found: " + showId);
        return show;
    }
    
    private Booking getBooking(String bookingId) {
        Booking booking = bookings.get(bookingId);
        if (booking == null) throw new IllegalArgumentException("Booking not found: " + bookingId);
        return booking;
    }
}

// ============================================================
// Support interfaces/classes
// ============================================================

interface PaymentService {
    boolean processPayment(PaymentDetails details, double amount);
}

interface NotificationService {
    void sendConfirmation(Booking booking);
}

class PaymentDetails {
    String cardNumber, cvv;
    // ...
}

class SeatUnavailableException extends RuntimeException {
    public SeatUnavailableException(String msg) { super(msg); }
}

class PaymentFailedException extends RuntimeException {
    public PaymentFailedException(String msg) { super(msg); }
}
```

---

## Concurrency: Seat Locking

The hardest part of this system. The key insight:

**Problem:** Two users (A and B) both see seat X as available and try to book it.

**Solution 1: Synchronized block on ShowSeat (our approach)**
```java
synchronized(showSeat) {
    if (!showSeat.lock(userId)) throw new SeatUnavailableException(...);
}
```
Only one thread enters the synchronized block at a time. First one gets the seat; second gets the exception.

**Solution 2: Optimistic locking (database version)**
```sql
UPDATE show_seats SET status='LOCKED', locked_by=?, lock_expiry=?
WHERE seat_id=? AND show_id=? AND status='AVAILABLE'
-- Returns 0 rows updated → seat was already taken
```

**Solution 3: Database advisory lock or Redis SETNX (distributed)**
```java
// Redis: SETNX (Set if Not Exists)
boolean locked = redis.setnx("lock:show123:seatA1", userId, 600); // 10 min TTL
```

### Deadlock Prevention

When locking multiple seats: **sort by seatId before locking**.

```java
List<String> sortedSeatIds = new ArrayList<>(seatIds);
Collections.sort(sortedSeatIds); // consistent global order
// Thread A locking A1, A2 and Thread B locking A2, A1 can deadlock
// With sorted order: both try A1 first → one waits → no deadlock
```

---

## Design Decisions

| Decision | Why |
|----------|-----|
| ShowSeat as separate from Seat | Seat is static data; ShowSeat is per-show mutable state |
| ConcurrentHashMap for shows | Multiple threads read/write concurrently |
| synchronized on ShowSeat | Fine-grained locking — only blocks contention on the same seat |
| Two-step: lock then confirm | Allows user to review before payment; releases seat if payment fails |
| Sort seatIds before locking | Prevents deadlock when users lock overlapping seat sets |

---

## Interview Follow-ups

1. What if traffic increases 100x (e.g., Avengers Endgame release)?
2. What if two threads book the same seat at the exact same millisecond?
3. What if payment succeeds but booking fails in our DB?
4. What if a new payment provider is added?
5. What if the fee calculation changes based on time of day?
6. What if a user locks a seat but never pays?
7. What if the database becomes unavailable?
8. What if operations must become asynchronous?
9. How would you handle refund processing?
10. What if the cinema layout changes for a special event?

---

## Interview Tips

1. **Identify the concurrency problem** — seat double-booking is the crux. "Two users may attempt to book the same seat."

2. **Explain the lock→confirm flow** — "First we lock seats temporarily (10-minute hold), then confirm after payment. This prevents ghost bookings."

3. **Mention sorted locking** — "To prevent deadlock, I sort seat IDs before locking in order."

4. **Mention lock expiry** — "If the user doesn't pay within 10 minutes, the lock expires and the seat becomes available again."

5. **Extension: waitlist** — "Users can join a waitlist for sold-out shows; when a seat is released, notify the next user."

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Main entities | Movie, Theater, Auditorium, Show, Seat, ShowSeat, Booking |
| 2 | ShowSeat vs Seat | Seat: static data; ShowSeat: per-show mutable status |
| 3 | Seat states | AVAILABLE, LOCKED, BOOKED |
| 4 | Double booking prevention | synchronized on ShowSeat in lock() method |
| 5 | Lock hold duration | 10 minutes; seat released if payment not made |
| 6 | Deadlock prevention | Sort seatIds before locking (consistent global order) |
| 7 | Two-step booking | lockSeats() → user pays → confirmBooking() |
| 8 | ConcurrentHashMap for | shows, bookings — thread-safe without global lock |
| 9 | Cancel → | release() all ShowSeats → set booking CANCELLED |
| 10 | Pricing | Map<SeatType, Double> on Show |
| 11 | Design patterns | Strategy (payment processor), Observer (notification on confirm) |
| 12 | SOLID applied | DIP: BookingService depends on PaymentService/NotificationService interfaces |
| 13 | Database approach | Optimistic locking: UPDATE...WHERE status='AVAILABLE' |
| 14 | Distributed approach | Redis SETNX per seat; TTL for auto-expiry |
| 15 | Extension: waitlist | Queue per show; on cancel → notify head of queue |

---

*Next: [LLD Case Study — Rate Limiter](05-lld-rate-limiter.md)*
