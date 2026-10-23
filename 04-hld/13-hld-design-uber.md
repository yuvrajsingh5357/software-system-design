# HLD Case Study: Design Uber (Ride-Sharing Service)

Design a real-time, geo-spatial matching and dispatch system like Uber or Lyft — tests high-velocity location tracking, spatial indexing, and concurrency.

## Table of Contents
- [Requirements and Clarifications](#requirements-and-clarifications)
- [Capacity Estimation](#capacity-estimation)
- [High-Level Architecture](#high-level-architecture)
- [Deep Dive: Geospatial Indexing (H3, S2, Quadtrees)](#deep-dive-geospatial-indexing-h3-s2-quadtrees)
- [Deep Dive: Location Ingestion & Tracking](#deep-dive-location-ingestion-tracking)
- [Deep Dive: Driver Dispatch and Concurrency](#deep-dive-driver-dispatch-and-concurrency)
- [Interview Tips](#interview-tips)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Requirements and Clarifications

**Functional Requirements:**
1. Riders can see available drivers nearby (within a 5km radius).
2. Riders can request a ride (start and end location).
3. Drivers can accept/reject ride requests.
4. Real-time tracking of driver location during a trip.

**Non-Functional Requirements:**
1. **Low Latency matching:** Driver matching must happen in <10 seconds.
2. **High Consistency:** A driver cannot be matched to two riders at the same time.
3. **High Scale:** 100M active riders, 5M active drivers globally.
4. **Availability:** System must survive partition failures (partitioning drivers by city).

---

## Capacity Estimation

```
Scale:
  Active Drivers = 1M (simultaneously online)
  Active Riders = 10M (simultaneously online)
  
Driver Location Updates:
  Drivers send location coordinates (Latitude, Longitude) every 4 seconds.
  Write QPS = 1,000,000 / 4 = 250,000 writes/sec.
  
Storage for Raw Location History (Audit):
  Record: driverId (8 bytes), timestamp (8 bytes), lat (8 bytes), lon (8 bytes) = 32 bytes
  Daily data: 250,000 writes/sec × 86,400 sec × 32 bytes = ~691 GB/day.
  (Stored in write-heavy Time Series DB like Cassandra or DynamoDB).

Rider Searches (Nearby Drivers):
  Assume 100,000 riders search/refresh every 10 seconds = 10,000 reads/sec.
```

---

## High-Level Architecture

```
                                  ┌──────────────────────────┐
                                  │      DNS/Load Balancer   │
                                  └──────────┬────────┬──────┘
                                             │        │
                     (WebSocket / TCP)       │        │ (HTTPS)
            ┌────────────────────────────────▼──┐  ┌──▼─────────────────┐
            │       Websocket Gateway           │  │    Rider Service   │
            │       (Driver Connection)         │  │ (Search / Request) │
            └────────────────┬──────────────────┘  └──────────┬─────────┘
                             │                                │
            ┌────────────────▼──────────────────┐             │
            │      Location Ingestion Service   │             │
            └────────────────┬──────────────────┘             │
                             │                                │
            ┌────────────────▼──────────────────┐  ┌──────────▼─────────┐
            │           Redis GEO Cache         │◄─┤   Matcher Engine   │
            │        (Driver Lat/Lon, H3)       │  │ (Dispatches Trips) │
            └────────────────┬──────────────────┘  └──────────┬─────────┘
                             │                                │
            ┌────────────────▼──────────────────┐  ┌──────────▼─────────┐
            │       Cassandra (Location Hist)   │  │   Trip Metadata DB │
            └───────────────────────────────────┘  └────────────────────┘
```

**Workflow for Location Updates (Drivers):**
1. Drivers maintain persistent WebSocket connections to the `Websocket Gateway`.
2. Every 4 seconds, driver sends `{driverId, lat, lon}`.
3. `Location Ingestion Service` writes to raw `Cassandra` (for history) and updates `Redis GEO` cache (current state).

**Workflow for Booking (Riders):**
1. Rider requests ride via `Rider Service`.
2. `Matcher Engine` queries `Redis GEO` to find drivers within the closest geospatial cells (e.g. H3 indexes).
3. Matcher sends request to the top driver via WebSockets. If rejected, it tries the next nearest driver.

---

## Deep Dive: Geospatial Indexing (H3, S2, Quadtrees)

Querying databases with `SELECT * FROM drivers WHERE lat BETWEEN x AND y` doesn't scale to 250K QPS. We must index 2D space.

```
       Quadtree (Grid)                  Hexagonal Grid (H3)
┌───────────┬───────────┐                   /\  /\
│           │   Spot    │                  /  \/  \
│  Cell 1   │  (Split)  │                 |    |   |
├───────────┼─────┬─────┤                  \  /\  /
│           │     │     │                   \/  \/
│  Cell 3   │     │     │                  Hexagons have constant 
└───────────┴─────┴─────┘                  neighbor distances!
```

### 1. Quadtrees
A tree where each node has exactly four children. If a cell contains too many drivers, it splits recursively into 4 sub-cells.
- **Cons:** Hard to update dynamically in memory when 1M drivers move constantly.

### 2. Google S2 (Hierarchical cell model)
Projects the Earth onto a cube, dividing it into cells. Each cell has a 64-bit integer ID.
- **Pros:** Fast indexing, great for range queries.

### 3. Uber H3 (Hexagonal Indexing) ← Recommended
Divides the world into nested hexagons.
- **Why Hexagons?** The distance between a hexagon's center and all its 6 neighbors is constant. In squares/rectangles, diagonal neighbors are further than horizontal/vertical neighbors. This makes ride path calculations mathematically simpler.

---

## Deep Dive: Location Ingestion & Tracking

To scale writes:
1. **Redis GEO:** We store driver locations in Redis using Geospatial indexes (`GEOADD`, `GEORADIUS`). Redis is fast enough to handle 250K write QPS.
2. **Decoupled historical storage:** We write raw location points to a Kafka topic. A consumer group flushes batches of coordinates into Cassandra. The API gateway never queries Cassandra for real-time driver tracking; it reads from Redis.

---

## Deep Dive: Driver Dispatch and Concurrency

**The Double Booking Problem:** Two riders request a ride simultaneously. The match engine queries Redis, finds driver D, and offers the ride to D for both riders.

```
Rider A ──► [ Matcher Engine ] ──(Locks Driver D in Redis) ──► Offer sent to Driver
                                       ▲ (Blocked)
Rider B ──► [ Matcher Engine ] ────────┘
```

**Solution:**
1. **Distributed Locks (Redis SETNX):** Before sending a request to driver D, the Matcher acquires a lock: `SETNX lock:driver:D rider_id EX 15` (15-second expiry).
2. If Rider B's matching thread tries to reserve driver D, the lock attempt fails. The Matcher instantly moves to the next closest driver.

---

## Interview Tips

1. **Focus on the writes first:** Acknowledge that the driver location write QPS is the main performance bottleneck.
2. **Explain Hexagonal Indexing (H3):** Explain *why* Uber chose hexagons (constant neighbor distance).
3. **Detail WebSockets:** WebSockets are necessary because HTTP pooling destroys battery life on mobile apps. Explain how WebSocket gateways scale behind a TCP load balancer.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Write QPS driver locations | ~250K writes/sec (1M drivers / 4s interval) |
| 2 | Geospatial querying issue | Relational B-Trees fail on 2D coordinate range queries |
| 3 | Quadtree concept | Hierarchical 2D grid partitioning; splits when cell exceeds limit |
| 4 | Uber H3 benefits | Hexagonal cells have constant distances to all 6 neighbors |
| 5 | Real-time location store | In-memory Redis using Geospatial indexes (`GEOADD`) |
| 6 | Location history storage | Kafka ingestion buffering → Cassandra (write-optimized) |
| 7 | Rider connection type | HTTPS REST (stateless requests/polling trip status) |
| 8 | Driver connection type | WebSockets (persistent duplex TCP socket for constant uploads) |
| 9 | Double booking prevention | Distributed lock on Driver ID (Redis SETNX) with TTL |
| 10 | Cell partitioning | Grouping drivers by H3 cell ID allows lightning-fast range lookups |

---

*Back to [Master Navigation](../README.md)*
