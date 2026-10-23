# API Design: REST, GraphQL, gRPC, and WebSockets

APIs are the contracts that tie distributed systems together. Designing robust, scalable, and intuitive APIs is a core skill tested in high-level system design interviews.

## Table of Contents
- [REST API Design](#rest-api-design)
- [GraphQL: Schema and Resolvers](#graphql-schema-and-resolvers)
- [gRPC: High-Performance RPC](#grpc-high-performance-rpc)
- [Real-Time: WebSockets vs Server-Sent Events](#real-time-websockets-vs-server-sent-events)
- [API protocol Comparison](#api-protocol-comparison)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## REST API Design

Representational State Transfer (REST) is the architectural style for designing APIs over HTTP. It is stateless and resource-oriented.

### 1. HTTP Methods and Semantics
Resources should be identified in the URL (nouns, not verbs) and manipulated using standard HTTP verbs:

| Verb | Path | Action | Idempotent | Safe |
|------|------|--------|------------|------|
| **GET** | `/users/{id}` | Retrieve user details | Yes | Yes |
| **POST** | `/users` | Create new user | No | No |
| **PUT** | `/users/{id}` | Replace/update entire user resource | Yes | No |
| **PATCH** | `/users/{id}` | Partially update user details | No | No |
| **DELETE** | `/users/{id}` | Remove user resource | Yes | No |

*Idempotency:* Making multiple identical requests has the same effect as making a single request. E.g., deleting a resource twice has the same side-effects as deleting it once.

### 2. HTTP Status Codes
Always return correct semantic status codes:
- **200 OK**: Request succeeded.
- **201 Created**: Resource successfully created.
- **202 Accepted**: Request accepted for async background processing.
- **400 Bad Request**: Client-side validation failure.
- **401 Unauthorized**: Authentication required/failed.
- **403 Forbidden**: Authenticated, but lacks permissions (authorization).
- **404 Not Found**: Resource doesn't exist.
- **429 Too Many Requests**: Rate limit breached.
- **500 Internal Server Error**: Bug/unexpected error on server.
- **503 Service Unavailable**: Server overloaded or down.

### 3. API Versioning
Change APIs without breaking existing clients.
- **URI Versioning (Recommended)**: `/api/v1/users` (Most explicit, easy to route/cache).
- **Header Versioning**: `Accept: application/vnd.myapi.v1+json`.

---

## GraphQL: Schema and Resolvers

GraphQL is a query language for APIs, designed by Facebook to solve mobile data overhead problems.

### 1. Key Concepts
- **Schema**: Strong-typed contract defining queries, mutations, and types.
- **Declarative Fetching**: Client requests exactly the fields they need, avoiding over-fetching and under-fetching.
- **Single Endpoint**: All operations pass through `POST /graphql`.

```graphql
# Schema Definition
type User {
  id: ID!
  name: String!
  orders: [Order!]!
}

# Client Query
query GetUserProfile {
  user(id: "123") {
    name
    orders {
      totalAmount
    }
  }
}
```

### 2. The N+1 Query Problem in GraphQL
If a user query asks for `orders`, a naive resolver will fetch the user once, then run 1 SQL query for each of the $N$ users to retrieve their orders.
- **Solution:** Use **DataLoader**. It batches and caches requests. Instead of individual queries, it batches IDs into `SELECT * FROM orders WHERE user_id IN (1, 2, ..., N)`.

---

## gRPC: High-Performance RPC

gRPC is a high-performance, open-source RPC framework developed by Google.

### 1. Key Features
- **Protocol Buffers (Protobuf)**: Binary serialization mechanism (much faster and smaller than JSON/XML).
- **HTTP/2 Transport**: Supports multiplexing (multiple streams over one TCP connection), header compression, and bi-directional streaming.
- **Code Generation**: Auto-generates client and server stubs in multiple languages (Java, Go, C++, Python).

```protobuf
// protobuf schema: user.proto
syntax = "proto3";

message UserRequest {
  string user_id = 1;
}

message UserResponse {
  string name = 1;
  string email = 2;
}

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}
```

---

## Real-Time: WebSockets vs Server-Sent Events

When the server needs to push data to clients (e.g., chat apps, stock tickers, ride tracking).

### 1. WebSockets
- **Type**: Bi-directional, full-duplex persistent connection over a single TCP socket.
- **Protocol**: Initiated via HTTP handshake, then upgraded to `ws://` or `wss://`.
- **Use cases**: Chat systems, multiplayer gaming, collaborative editors.

### 2. Server-Sent Events (SSE)
- **Type**: Mono-directional (Server-to-Client) persistent connection over standard HTTP.
- **Protocol**: Uses standard HTTP with header `Content-Type: text/event-stream`.
- **Use cases**: Live dashboard counters, notification feeds, stock tickers.

---

## API Protocol Comparison

| Dimension | REST | GraphQL | gRPC | WebSockets |
|-----------|------|---------|------|------------|
| **Format** | JSON/XML (text) | JSON (text) | Protobuf (binary) | Custom/JSON |
| **Transport** | HTTP/1.1 or HTTP/2 | HTTP/1.1 or HTTP/2 | HTTP/2 (Required) | TCP upgrade |
| **Communication** | Request-Response | Request-Response | Request-Response & Streaming | Bi-directional Duplex |
| **Strict Schema** | Optional (OpenAPI) | Yes (GraphQL Schema) | Yes (Proto Files) | No |
| **Performance** | Medium | Low-Medium (parser cost) | Extremely High | High (low overhead) |

---

## Interview Questions

### Q: When would you choose gRPC over REST?
Choose **gRPC** for **internal microservice-to-microservice communication** where low latency, high throughput, and strict typing are critical. REST is preferred for **public-facing external APIs** because JSON is universally understood, easy to consume via browser, and integrates cleanly with standard CDNs and reverse proxies.

### Q: Explain what happens during a WebSocket handshake.
1. The client sends a standard HTTP GET request with upgrade headers:
   `Upgrade: websocket`, `Connection: Upgrade`.
2. The server receives the request, verifies headers, and returns HTTP code `101 Switching Protocols`.
3. The underlying TCP socket remains open, switching from HTTP protocol to WebSocket framing protocol (`ws://`).

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Idempotent HTTP methods | GET, PUT, DELETE, HEAD |
| 2 | POST vs PUT | POST: non-idempotent (creates); PUT: idempotent (replaces) |
| 3 | HTTP 401 vs 403 | 401: Unauthenticated (who are you?); 403: Unauthorized (no permission) |
| 4 | HTTP 202 Accepted | Request accepted for async background processing |
| 5 | GraphQL N+1 problem | Resolving nested lists triggers $N$ database calls |
| 6 | N+1 Solution | DataLoader (batches and caches keys) |
| 7 | gRPC transport protocol | HTTP/2 (multiplexing, bi-directional streaming) |
| 8 | Protobuf advantage | Compact binary serialization; schema validation built-in |
| 9 | Bi-directional real-time | WebSockets (low latency, full duplex over single TCP) |
| 10 | Unidirectional server-to-client | Server-Sent Events (SSE) over standard HTTP |

---

*Next: [Microservices and Service Mesh](06-microservices-and-service-mesh.md)*
