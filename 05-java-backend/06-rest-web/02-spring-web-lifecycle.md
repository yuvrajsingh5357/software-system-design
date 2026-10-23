# Spring Web: Request Lifecycle and Components

## 1. The DispatcherServlet
In Spring MVC, the `DispatcherServlet` is the front controller. It intercepts all incoming HTTP requests.
1. Client sends an HTTP request.
2. The Servlet Container (Tomcat) hands it to the `DispatcherServlet`.
3. `DispatcherServlet` asks the `HandlerMapping` which Controller method should handle this URL.
4. The request is passed to the `@RestController`.
5. The Controller processes it and returns an object.
6. The `HttpMessageConverter` (like Jackson) converts the object to JSON.
7. The JSON is written to the HTTP Response.

## 2. Filters vs Interceptors
Both are used to intercept requests, but they operate at different levels.

- **Filters (Servlet Level):**
  - Part of the Java Servlet API, not Spring.
  - Executes *before* the request reaches the `DispatcherServlet`.
  - Use case: Logging, CORS, Authentication (Spring Security operates via a filter chain).
- **Interceptors (Spring Level):**
  - Part of the Spring Framework.
  - Executes *after* the `DispatcherServlet` but *before* the Controller method.
  - Has access to the actual Controller method and Spring context.
  - Use case: Role-based authorization, adding metadata to models.

## 3. Data Transfer Objects (DTOs) and Validation
Never expose JPA `@Entity` classes directly via your REST API.
- **Why?** It couples your database schema to your API contract. Changing a column name breaks the API. It can also cause infinite recursion with Jackson if bidirectional relationships exist.
- **Solution:** Create simple POJOs (DTOs) and map data between Entities and DTOs.
- **Validation:** Use `@Valid` and JSR-380 annotations (`@NotBlank`, `@Email`, `@Min`) on the DTO. Spring will automatically return a `400 Bad Request` if validation fails.

## 4. Exception Handling (@ControllerAdvice)
Instead of putting `try-catch` blocks in every controller, Spring uses Global Exception Handling.
- `@ControllerAdvice` / `@RestControllerAdvice` defines a global class to intercept exceptions thrown by any controller.
- `@ExceptionHandler(UserNotFoundException.class)` tells Spring what JSON response to return when this specific exception is thrown.
- This centralizes all error mapping (e.g., mapping `IllegalArgumentException` to `400`, `EntityNotFoundException` to `404`).

## 5. Interview Questions
### Q. What is the complete request lifecycle in a Spring Boot web application?
**Short Answer:** Client -> Tomcat -> Filter Chain (Security) -> DispatcherServlet -> HandlerMapping -> Interceptor `preHandle()` -> Controller -> Service -> Repository -> Database -> Service -> Controller -> HttpMessageConverter (Jackson JSON) -> Interceptor `postHandle()` -> DispatcherServlet -> Client.

---

[← Previous: HTTP & REST Fundamentals](01-http-rest-fundamentals.md) | [Next: Spring Security & Stateless JWTs →](../07-spring-security/01-security-and-jwt.md)
