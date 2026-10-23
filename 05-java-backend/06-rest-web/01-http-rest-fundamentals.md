# HTTP and REST Fundamentals

## 1. What Is It?
REST (Representational State Transfer) is an architectural style for distributed systems, heavily used for designing web APIs. It operates over HTTP.

## 2. HTTP Methods
- **GET:** Retrieve a resource. (Safe, Idempotent)
- **POST:** Create a new resource. (Not safe, Not idempotent)
- **PUT:** Replace a resource entirely. (Idempotent)
- **PATCH:** Partially update a resource. (Not guaranteed idempotent)
- **DELETE:** Remove a resource. (Idempotent)

## 3. Status Codes
- **2xx (Success):** 200 OK, 201 Created, 204 No Content.
- **3xx (Redirection):** 301 Moved Permanently, 302 Found, 304 Not Modified.
- **4xx (Client Error):** 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict.
- **5xx (Server Error):** 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable.

## 4. REST Principles
1. **Client-Server Architecture:** Decoupled frontend and backend.
2. **Statelessness:** Every request from the client must contain all the information necessary to understand and process the request. The server cannot store session state.
3. **Cacheability:** Responses must define themselves as cacheable or non-cacheable.
4. **Uniform Interface:** Use standard HTTP methods and standard resource-based URIs (e.g., `/users/{id}`).

## 5. API Design Best Practices
- Use nouns, not verbs: `/users`, not `/getUsers`.
- Use plurals: `/users/123`, not `/user/123`.
- Version your API: `/api/v1/users`.
- Use query parameters for filtering and pagination: `/users?role=admin&page=2`.

## 6. Interview Questions
### Q. What is idempotency in REST?
**Short Answer:** Idempotency means that making the same request multiple times produces the same result (the state of the server doesn't change after the first request). `GET`, `PUT`, and `DELETE` are idempotent. `POST` is not.

### Q. What is the difference between 401 Unauthorized and 403 Forbidden?
**Short Answer:** `401 Unauthorized` means the client has not authenticated (who are you?). `403 Forbidden` means the client is authenticated, but does not have the required permissions to access the resource (you are a user, but you need to be an admin).

---

[← Previous: The N+1 Select Problem](../05-jpa-hibernate/03-n-plus-one-problem.md) | [Next: Spring Web Request Lifecycle (DispatcherServlet) →](02-spring-web-lifecycle.md)
