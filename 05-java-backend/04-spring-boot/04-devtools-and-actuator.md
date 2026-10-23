# DevTools, Profiles, and Actuator

## 1. Spring Boot DevTools
`spring-boot-devtools` is a development-time tool designed to improve developer productivity. **It is automatically disabled when the application is run in production.**

### Key Features:
- **Automatic Restart:** When you change Java code or resources, DevTools instantly restarts the Spring Boot application. It is extremely fast because it uses two classloaders: one for base libraries (which don't change) and one for your project classes (which are reloaded).
- **Live Reload:** Automatically triggers a browser refresh when HTML, CSS, or static resources change.
- **Cache Disabling:** Automatically disables template caching (like Thymeleaf) during development so UI changes appear instantly.

## 2. Externalized Configuration & Profiles
Spring Boot strongly supports externalizing configuration so the same application code can run in different environments without recompilation.

- Configuration is stored in `application.properties` or `application.yml`.
- **Profiles:** You can define environment-specific settings (e.g., `application-dev.yml`, `application-prod.yml`).
- When starting the app, you activate a profile (`--spring.profiles.active=prod`). Spring Boot will load the specific database credentials, port numbers, and logging levels for that environment.

## 3. Difference Between @Controller and @RestController
When building web applications, you use Controllers.

- **`@Controller`:** Used in traditional Spring MVC applications. The methods return a `String` representing a **view name** (e.g., `"dashboard"`). Spring's ViewResolver finds a template (like `dashboard.html` or `dashboard.jsp`) and renders it as an HTML webpage.
- **`@RestController`:** Used for building REST APIs. It is a convenience annotation that combines `@Controller` and `@ResponseBody`. The methods return raw objects, and Spring automatically serializes those objects into **JSON** or XML using Jackson, sending them directly in the HTTP response body.

## 4. Difference Between @RequestParam and @PathVariable
When capturing data from a URL in a REST API:

- **`@RequestParam`:** Extracts data from the **query string** (after the `?`).
  - Example URL: `/search?keyword=laptop`
  - Use case: Filtering, searching, pagination, or optional parameters.
- **`@PathVariable`:** Extracts data directly from the **URI path**.
  - Example URL: `/users/42`
  - Use case: Identifying a specific, required resource (like an ID).

## 5. Interview Questions

### Q. What is Spring Boot Actuator and why is it important for Microservices?
#### Short Interview Answer
Spring Boot Actuator is a built-in module that provides production-ready management and monitoring endpoints. It exposes critical metrics like CPU usage, memory consumption, active threads, and overall application health. In a microservices architecture, where dozens of services are running across containers, Actuator endpoints are essential for orchestrating tools (like Kubernetes) to perform health checks, and for monitoring tools (like Prometheus) to scrape metrics for observability.

---

[← Previous: Spring Boot Architecture & Auto-Config](01-spring-boot-architecture.md) | [Next: JPA & Hibernate Fundamentals (Persistence Context) →](../05-jpa-hibernate/01-jpa-hibernate-fundamentals.md)
