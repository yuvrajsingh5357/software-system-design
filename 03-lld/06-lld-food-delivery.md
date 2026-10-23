# LLD Case Study: Food Delivery System

Design a Swiggy or Zomato-style food delivery system — tests matching algorithms, real-time tracking, state management, and strategy patterns.

## Table of Contents
- [Requirements Analysis](#requirements-analysis)
- [Class Design](#class-design)
- [Full Java Implementation](#full-java-implementation)
- [Design Patterns Used](#design-patterns-used)
- [Concurrency & Thread Safety](#concurrency--thread-safety)
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
1. **Users:** Register, search restaurants, view menu, place orders, make payment, track orders.
2. **Restaurants:** Register, add/update menu items, accept/prepare orders.
3. **Delivery Partners:** Register, update location, receive delivery requests, accept/reject delivery, deliver order.
4. **Order Management:** Place order, calculate prices, process payment, transition order state (Placed → Preparing → Out for Delivery → Delivered).
5. **Driver Matching:** Dynamically match delivery partners to orders based on geographical location.

**Non-Functional Requirements:**
1. **High Concurrency:** Handle thousands of concurrent orders, location updates, and driver assignments.
2. **Consistency:** Avoid double assignment of a delivery partner to multiple active deliveries.
3. **Real-time Updates:** Push order status changes to users and delivery partners immediately.

---

## Class Design

```
User
├── id, name, location

Restaurant
├── id, name, menu, location
└── addMenuItem(), removeMenuItem()

MenuItem
├── id, name, price, isAvailable

DeliveryPartner
├── id, name, isAvailable, currentLocation
└── updateLocation()

Order
├── id, user, restaurant, List<OrderItem>, status, deliveryPartner
└── calculateTotal(), nextState()

OrderItem
├── menuItem, quantity

DeliveryMatchingStrategy (Interface)
└── DeliveryPartner match(Order order, List<DeliveryPartner> availablePartners)

NearestDriverStrategy (Implements DeliveryMatchingStrategy)
└── calculates Euclidean or Haversine distance

OrderService
└── placeOrder(), assignDriver(), updateStatus()
```

---

## Full Java Implementation

### 1. Enums and Domain Classes
```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;

public enum OrderStatus {
    PLACED, PREPARING, READY_FOR_PICKUP, OUT_FOR_DELIVERY, DELIVERED, CANCELLED
}

public class Location {
    private final double latitude;
    private final double longitude;

    public Location(double lat, double lon) {
        this.latitude = lat;
        this.longitude = lon;
    }

    public double getLatitude() { return latitude; }
    public double getLongitude() { return longitude; }

    public double distanceTo(Location other) {
        // Simple Euclidean distance for illustration (use Haversine in production)
        double latDiff = this.latitude - other.latitude;
        double lonDiff = this.longitude - other.longitude;
        return Math.sqrt(latDiff * latDiff + lonDiff * lonDiff);
    }
}

public class MenuItem {
    private final String itemId;
    private final String name;
    private final double price;

    public MenuItem(String id, String name, double price) {
        this.itemId = id; this.name = name; this.price = price;
    }
    public String getName() { return name; }
    public double getPrice() { return price; }
}
```

### 2. User, Restaurant, and DeliveryPartner
```java
public class User {
    private final String id;
    private final String name;
    private Location location;

    public User(String id, String name, Location location) {
        this.id = id; this.name = name; this.location = location;
    }
    public String getId() { return id; }
    public Location getLocation() { return location; }
    public void setLocation(Location loc) { this.location = loc; }
}

public class Restaurant {
    private final String id;
    private final String name;
    private final List<MenuItem> menu = new CopyOnWriteArrayList<>();
    private final Location location;

    public Restaurant(String id, String name, Location location) {
        this.id = id; this.name = name; this.location = location;
    }
    public String getId() { return id; }
    public Location getLocation() { return location; }
    public List<MenuItem> getMenu() { return menu; }
    public void addMenuItem(MenuItem item) { menu.add(item); }
}

public class DeliveryPartner {
    private final String id;
    private final String name;
    private volatile boolean isAvailable = true;
    private volatile Location currentLocation;

    public DeliveryPartner(String id, String name, Location location) {
        this.id = id; this.name = name; this.currentLocation = location;
    }
    public String getId() { return id; }
    public boolean isAvailable() { return isAvailable; }
    public void setAvailable(boolean avail) { this.isAvailable = avail; }
    public Location getCurrentLocation() { return currentLocation; }
    public void updateLocation(Location loc) { this.currentLocation = loc; }
}
```

### 3. Order and OrderItem
```java
public class OrderItem {
    private final MenuItem menuItem;
    private final int quantity;

    public OrderItem(MenuItem menuItem, int quantity) {
        this.menuItem = menuItem; this.quantity = quantity;
    }
    public MenuItem getMenuItem() { return menuItem; }
    public int getQuantity() { return quantity; }
}

public class Order {
    private final String orderId;
    private final User user;
    private final Restaurant restaurant;
    private final List<OrderItem> items;
    private OrderStatus status;
    private DeliveryPartner deliveryPartner;

    public Order(User user, Restaurant restaurant, List<OrderItem> items) {
        this.orderId = "ORD-" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
        this.user = user;
        this.restaurant = restaurant;
        this.items = items;
        this.status = OrderStatus.PLACED;
    }

    public String getOrderId() { return orderId; }
    public User getUser() { return user; }
    public Restaurant getRestaurant() { return restaurant; }
    public List<OrderItem> getItems() { return items; }
    
    public synchronized OrderStatus getStatus() { return status; }
    public synchronized void setStatus(OrderStatus status) { this.status = status; }

    public synchronized DeliveryPartner getDeliveryPartner() { return deliveryPartner; }
    public synchronized void setDeliveryPartner(DeliveryPartner partner) { this.deliveryPartner = partner; }

    public double calculateTotal() {
        return items.stream().mapToDouble(i -> i.getMenuItem().getPrice() * i.getQuantity()).sum();
    }
}
```

### 4. Matching Strategy (Strategy Pattern)
```java
public interface DeliveryMatchingStrategy {
    Optional<DeliveryPartner> findPartner(Order order, List<DeliveryPartner> partners);
}

public class NearestDriverStrategy implements DeliveryMatchingStrategy {
    @Override
    public Optional<DeliveryPartner> findPartner(Order order, List<DeliveryPartner> partners) {
        Location restaurantLoc = order.getRestaurant().getLocation();
        return partners.stream()
            .filter(DeliveryPartner::isAvailable)
            .min(Comparator.comparingDouble(p -> p.getCurrentLocation().distanceTo(restaurantLoc)));
    }
}
```

### 5. OrderService (Coordinator)
```java
public class OrderService {
    private final Map<String, Order> orders = new ConcurrentHashMap<>();
    private final List<DeliveryPartner> partners = new CopyOnWriteArrayList<>();
    private final DeliveryMatchingStrategy matchingStrategy;

    public OrderService(DeliveryMatchingStrategy strategy) {
        this.matchingStrategy = strategy;
    }

    public void registerPartner(DeliveryPartner partner) { partners.add(partner); }

    public Order placeOrder(User user, Restaurant restaurant, List<OrderItem> items) {
        Order order = new Order(user, restaurant, items);
        orders.put(order.getOrderId(), order);
        System.out.printf("Placed order %s for user %s. Total: $%.2f%n", 
            order.getOrderId(), user.getId(), order.calculateTotal());
        return order;
    }

    public boolean assignDeliveryPartner(String orderId) {
        Order order = orders.get(orderId);
        if (order == null) return false;

        synchronized (order) {
            if (order.getDeliveryPartner() != null) return true; // Already assigned

            Optional<DeliveryPartner> matchedPartner = matchingStrategy.findPartner(order, partners);
            if (matchedPartner.isPresent()) {
                DeliveryPartner dp = matchedPartner.get();
                // Double check / lock assignment
                synchronized (dp) {
                    if (dp.isAvailable()) {
                        dp.setAvailable(false);
                        order.setDeliveryPartner(dp);
                        order.setStatus(OrderStatus.PREPARING);
                        System.out.printf("Assigned driver %s to order %s%n", dp.getId(), order.getOrderId());
                        return true;
                    }
                }
            }
        }
        System.out.printf("No delivery partners available for order %s%n", orderId);
        return false;
    }

    public void updateOrderStatus(String orderId, OrderStatus status) {
        Order order = orders.get(orderId);
        if (order != null) {
            order.setStatus(status);
            System.out.printf("Order %s updated to status: %s%n", orderId, status);
            if (status == OrderStatus.DELIVERED && order.getDeliveryPartner() != null) {
                order.getDeliveryPartner().setAvailable(true);
            }
        }
    }
}
```

---

## Design Patterns Used

1. **Strategy Pattern:** Used for `DeliveryMatchingStrategy` allowing the system to dynamically switch algorithms (e.g., matching by distance, driver ratings, historical speed, or batch-delivery strategies).
2. **State Pattern (Conceptual):** Order lifecycle transitions (Placed → Preparing → Out for Delivery → Delivered). In complex implementations, this can be modeled using explicit state classes (`PlacedState`, `PreparingState`) to encapsulate state-specific validations.
3. **Observer Pattern:** Can be used to notify the User, Restaurant, and Delivery Partner systems whenever order status updates occur.

---

## Concurrency & Thread Safety

1. **State Lock Scope:** Modifying `status` and `deliveryPartner` in the `Order` class is synchronized on the individual `Order` instance to prevent race conditions.
2. **Driver Assignment Guard:** When assigning a driver, we synchronize on the selected `DeliveryPartner` object and double-check `dp.isAvailable()` to guarantee a driver is never double-allocated.
3. **CopyOnWriteArrayList:** Used for thread-safe lists (`partners`, `menu`) that are read frequently but updated rarely.
4. **ConcurrentHashMap:** Used for active orders, avoiding bottlenecks of global locks.

---

## Design Decisions

| Decision | Why |
|----------|-----|
| Strategy Pattern for Driver Matching | Allows the matching rules (e.g., batching, surge pricing adjustments) to vary independently of the core booking. |
| Double locking on Driver Assignment | Guarantees that a driver isn't assigned to multiple concurrent orders if two matcher threads run simultaneously. |
| `CopyOnWriteArrayList` for Menu/Drivers | Menus and drivers change less frequently than read queries (users searching menus). |
| Decoupled Order State transitions | Helps cleanly track order lifecycle and release drivers back to pool upon `DELIVERED`. |

---

## Interview Follow-ups

1. What if traffic increases 100x during a major sports event?
2. What if multiple delivery agents try to accept the same order?
3. What if a restaurant runs out of an item after an order is placed?
4. What if the payment gateway fails after order confirmation?
5. How would you dynamically calculate delivery fees based on weather or demand (surge pricing)?
6. What if a delivery agent goes offline mid-delivery?
7. How do you handle real-time location tracking efficiently?
8. What if the database becomes unavailable?
9. How would you handle batching multiple orders for the same delivery agent?
10. What if an order needs to be cancelled by the user?

---

## Interview Tips

1. **Start with domain entities:** Identify core users, restaurants, menu items, order details, and drivers.
2. **Elaborate on the driver allocation:** Emphasize how matching works, how driver state is updated, and how to prevent double assignment using synchronization.
3. **Mention location storage:** "In a real distributed system, we would use a geospatial database like Redis GEO or PostGIS to store and query driver locations rapidly."
4. **Address push notifications:** "We use WebSockets or server-sent events (SSE) for pushing real-time driver movement and status changes to the client app."

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Key Entities | User, Restaurant, MenuItem, Order, OrderItem, DeliveryPartner, Location |
| 2 | Driver matching pattern | Strategy Pattern (`DeliveryMatchingStrategy`) |
| 3 | Order states | PLACED, PREPARING, READY_FOR_PICKUP, OUT_FOR_DELIVERY, DELIVERED, CANCELLED |
| 4 | Double driver booking | Synchronize on `DeliveryPartner` and check availability |
| 5 | Thread safety collections | `ConcurrentHashMap` for orders, `CopyOnWriteArrayList` for partners |
| 6 | Driver location update | `volatile Location currentLocation` + thread-safe update |
| 7 | Geospatial scaling | In production, use Redis GEO queries (`GEORADIUS`) |
| 8 | Push notification delivery | WebSocket connection, Firebase Cloud Messaging (FCM) |
| 9 | Releasing driver | Mark driver available again when order status becomes `DELIVERED` |
| 10 | SOLID compliance | `OrderService` interacts with `DeliveryMatchingStrategy` abstraction |

---

*Next: [LLD Case Study — ATM Machine](07-lld-atm-machine.md)*
