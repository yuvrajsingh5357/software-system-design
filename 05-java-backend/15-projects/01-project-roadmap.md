# Backend Projects Progression

This document outlines the required projects to move from a beginner to a production-ready Backend Engineer.

## Phase 1: Core Fundamentals
**PROJECT 1: Simple HTTP Server (No Frameworks)**
- **Goal:** Understand HTTP under the hood.
- **Tech:** Core Java, `java.net.ServerSocket`, `InputStream`, `OutputStream`.
- **Task:** Listen on port 8080. Parse raw HTTP GET request strings. Return a raw HTTP `200 OK` response with HTML content.

## Phase 2: Standard Backend Dev
**PROJECT 2: RESTful Task API**
- **Goal:** Understand Spring Boot, REST principles, and JSON.
- **Tech:** Spring Boot, Spring Web, Jackson.
- **Task:** Build CRUD endpoints for a Task model. Store data in a thread-safe `ConcurrentHashMap`. Use proper HTTP status codes (201 Created, 404 Not Found).

**PROJECT 3: Production E-Commerce CRUD**
- **Goal:** Relational databases, ORM, and Data Validation.
- **Tech:** Spring Boot, Spring Data JPA, PostgreSQL, Hibernate, Validation.
- **Task:** Build User, Order, and Product entities. Implement OneToMany and ManyToMany relationships. Use Liquibase/Flyway for schema migrations. Fix N+1 query problems using `JOIN FETCH`.

## Phase 3: Security & Performance
**PROJECT 4: Secure Banking API**
- **Goal:** Authentication and Authorization.
- **Tech:** Spring Security, BCrypt, JWT, PostgreSQL.
- **Task:** Implement User Registration (hashing passwords). Implement Login to return a JWT. Secure endpoints using a custom JwtFilter. Add Role-based access control (`@PreAuthorize`).

**PROJECT 5: High-Traffic Leaderboard**
- **Goal:** Caching and in-memory data structures.
- **Tech:** Redis, Spring Data Redis, Jedis/Lettuce.
- **Task:** Implement a gaming leaderboard. Use Redis Sorted Sets to rank millions of users in $O(\log N)$ time. Use Redis Strings to cache heavy database queries with a TTL.

## Phase 4: Distributed Systems
**PROJECT 6: Event-Driven Notification System**
- **Goal:** Asynchronous processing and decoupling.
- **Tech:** Apache Kafka, Spring Kafka.
- **Task:** When a User registers in the User Service, publish a `UserCreatedEvent` to Kafka. Build a separate Notification Service that listens to the topic and "sends an email". Handle consumer crashes and retries.

**PROJECT 7: Complete Microservices Ecosystem**
- **Goal:** Orchestration, API Gateways, and Docker.
- **Tech:** Docker Compose, Spring Cloud Gateway, Eureka (optional), PostgreSQL, Kafka, Redis.
- **Task:** 
  1. API Gateway routing requests.
  2. Order Service (writes to Postgres, publishes to Kafka).
  3. Inventory Service (listens to Kafka, updates stock).
  4. Write a `docker-compose.yml` to spin up all services, databases, and brokers with a single command.
  5. Implement distributed tracing using Micrometer and Zipkin.

---

[← Previous: Backend System Design Integration (Bridging HLD & Code)](../14-backend-system-design-integration/01-implementation-bridge.md)
