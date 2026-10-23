# Microservices and Service Mesh

Microservices decompose complex monoliths into small, autonomous, and decoupled services. This chapter covers the patterns, communications, gateway routing, and service mesh internals required to run microservices at scale.

## Table of Contents
- [Monolith vs Microservices](#monolith-vs-microservices)
- [API Gateway Pattern](#api-gateway-pattern)
- [Service Discovery and Registry](#service-discovery-and-registry)
- [Service Mesh: Sidecars and Traffic Control](#service-mesh-sidecars-and-traffic-control)
- [Distributed Transactions: Saga Pattern](#distributed-transactions-saga-pattern)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Monolith vs Microservices

Deciding how to partition system components is a primary high-level architecture choice.

```
       Monolith                   Microservices
┌─────────────────────┐      ┌────────┐   ┌────────┐
│  UI / Web Layer     │      │ Order  │   │Payment │
├─────────────────────┤      │ Service│   │ Service│
│  Order Logic        │      └────┬───┘   └────┬───┘
├─────────────────────┤           │            │
│  Payment Logic      │      ┌────▼───┐   ┌────▼───┐
├─────────────────────┤      │ Order  │   │Payment │
│  Shared Database    │      │   DB   │   │   DB   │
└─────────────────────┘      └────────┘   └────────┘
```

### 1. The Monolith
- **Pros:** Single codebase (easy deploy), fast local communication, simple transaction boundaries.
- **Cons:** Single point of failure, scaling bottleneck (must scale entire app), deployment blocking (teams block each other).

### 2. Microservices
- **Pros:** Independent deployments, technology stack flexibility, fault isolation, scale-to-demand per service.
- **Cons:** Network latency, data consistency (distributed transactions), operation complexity (CI/CD, observability).
- **Rule of Thumb:** **Database-per-service** is mandatory. If services share a database, they are still tightly coupled (distributed monolith).

---

## API Gateway Pattern

Instead of clients calling 50 microservices directly, an **API Gateway** acts as the single entry point.

```
Client ──> [ API Gateway ] ──┬──> Order Service
                             ├──> Payment Service
                             └──> Auth Service
```

**Responsibilities:**
1. **Routing:** Reverse proxy queries to respective services.
2. **Authentication / Authorization:** Validate JWTs at the gateway, avoiding duplicate auth code in each microservice.
3. **Rate Limiting:** Protect downstream services from DDoS and spikes.
4. **Resiliency (Circuit Breaking):** Fail fast if a downstream service is down (e.g., return cached fallback data).

---

## Service Discovery and Registry

With auto-scaling, microservice IP addresses change dynamically. We need a way to locate services.

```
┌─────────────────┐             1. Register             ┌───────────────────┐
│  Order Service  │ ──────────────────────────────────> │  Service Registry │
└─────────────────┘                                     │  (e.g., Consul)   │
                                                        └───────────────────┘
                                                                  ▲
┌─────────────────┐             2. Query/Lookup                   │
│   API Gateway   │ ──────────────────────────────────────────────┘
└─────────────────┘
```

- **Service Registry:** A database of active service instances and their IPs (e.g., Consul, Netflix Eureka).
- **Client-Side Discovery (e.g., Ribbon):** Client queries the registry, caches the IPs, and load-balances requests locally.
- **Server-Side Discovery (e.g., Kubernetes DNS):** Client calls a load balancer, which queries registry and routes the traffic.

---

## Service Mesh: Sidecars and Traffic Control

As a microservice network grows, cross-cutting concerns (encryption, retries, metrics) become hard to manage in application code. A **Service Mesh** is a dedicated infrastructure layer to handle service-to-service communication.

```
Microservice A [App] ◄──(localhost)──► Sidecar Proxy (Envoy)
                                             │
                                     (mTLS encrypted TCP)
                                             │
                                             ▼
Microservice B [App] ◄──(localhost)──► Sidecar Proxy (Envoy)
```

### 1. Data Plane vs Control Plane
- **Data Plane (Proxies):** Sidecars (e.g., Envoy) deployed alongside each app. They intercept all network traffic, handling retries, circuit breaking, and load balancing.
- **Control Plane (Manager):** Central controller (e.g., Istio) that configures the proxies, manages security certificates, and gathers telemetry.

### 2. Core Capabilities
- **mTLS (Mutual TLS):** Automatic end-to-end encryption between all services.
- **Traffic Splitting:** Route 95% traffic to `v1` and 5% to `v2` (Canary deployments).

---

## Distributed Transactions: Saga Pattern

Since each microservice has its own DB, we cannot use database-level ACID transactions. Instead, we use the **Saga Pattern**.

A Saga is a sequence of local transactions. Each transaction updates database state within a single service. If a step fails, the Saga runs **compensating transactions** to undo the preceding changes.

```
Step 1: Order Service [Create Order]  ──(Success)──> Step 2: Payment Service [Deduct Money]
                                                                  │ (Failure)
                                                                  ▼
Step 1 Compensate: Order Service [Cancel Order] <─────────────────┘
```

### 1. Implementation Styles
- **Choreography:** Services publish and listen to events (e.g., Kafka) to trigger their steps. Simple, but hard to track dependencies as services grow.
- **Orchestration:** A central controller class orchestrates the workflow. Easier to monitor, but becomes a single point of dependency.

---

## Interview Questions

### Q: Why is a shared database considered a microservices anti-pattern?
A shared database breaks **loose coupling**. If Service A changes a table schema, Service B immediately breaks. It also causes database lock contention, bypasses domain logic validation (Service B bypasses Service A's code to manipulate A's tables), and prevents scaling services independently based on their database demands.

### Q: Explain the difference between Choreography and Orchestration in Sagas.
- **Choreography:** Decoupled. Services listen to Kafka events and take action independently. No central coordinator. High risk of circular dependencies.
- **Orchestration:** Centralized. A dedicated Orchestrator service explicitly invokes Service A, waits for response, then invokes Service B. It has a clear view of the state machine, but introduces a central orchestrator component.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Microservices rule | Database-per-service (never share database schemas) |
| 2 | API Gateway jobs | Routing, Authentication, Rate Limiting, Circuit Breaking |
| 3 | Service Discovery role | Tracks dynamic IP locations of service instances |
| 4 | Client-side discovery | Client queries registry directly and load-balances locally |
| 5 | Server-side discovery | Client hits load balancer, which resolves address internally |
| 6 | Service Mesh role | Dedicated network proxy layer handling service communication |
| 7 | Sidecar proxy | Intercepts app ingress/egress traffic (e.g., Envoy proxy) |
| 8 | Control Plane | Configures proxies, pushes policy and certificates (e.g., Istio) |
| 9 | Distributed transactions | Saga Pattern (sequence of local transactions + compensations) |
| 10 | Compensating transaction | Operation that explicitly undoes the action of a previous step |

---

*Next: [Security Fundamentals](07-security-fundamentals.md)*
