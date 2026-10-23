# Spring Security and Authentication

## 1. What Is It?
Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.

## 2. Authentication vs Authorization
- **Authentication:** Verifying *who* the user is (e.g., verifying username and password).
- **Authorization:** Verifying *what* the authenticated user is allowed to do (e.g., checking if the user has the `ROLE_ADMIN`).

## 3. How Spring Security Works (The Filter Chain)
Spring Security operates via a chain of Servlet Filters (the `SecurityFilterChain`).
1. A request comes in.
2. It hits the `UsernamePasswordAuthenticationFilter` (or a custom `JwtAuthenticationFilter`).
3. The filter extracts credentials and creates an `Authentication` object.
4. The `AuthenticationManager` verifies the credentials (usually delegating to a `UserDetailsService` to load user data from the DB).
5. If successful, the fully populated `Authentication` object (containing roles/authorities) is stored in the `SecurityContextHolder`.
6. The request moves down the chain. If an endpoint requires authorization, the `FilterSecurityInterceptor` checks the `SecurityContextHolder` to see if the user has the required roles.

## 4. JWT (JSON Web Tokens) and Stateless Authentication
In modern REST APIs, servers should be stateless (no server-side sessions).
- **Login:** User sends credentials. Server validates, generates a JWT, signs it with a secret key, and returns it to the client.
- **Subsequent Requests:** Client sends the JWT in the `Authorization: Bearer <token>` header.
- **Validation:** A custom `JwtFilter` parses the token, verifies the cryptographic signature (proving the server issued it and it wasn't tampered with), extracts the username and roles from the payload, and populates the `SecurityContext`. No database lookup is strictly required!

## 5. Access Tokens vs Refresh Tokens
- **Access Token:** Short-lived (e.g., 15 minutes). Sent with every request.
- **Refresh Token:** Long-lived (e.g., 7 days). Stored securely (e.g., HttpOnly cookie). Used only to get a new Access Token when the old one expires.
- **Why?** Because stateless JWTs cannot be forcefully revoked before they expire. Keeping the access token lifespan short minimizes the window of vulnerability if it is stolen.

## 6. Interview Questions
### Q. How do you securely store passwords?
**Short Answer:** Passwords should never be stored in plain text. They must be hashed using a strong, slow hashing algorithm like `BCrypt`, `Argon2`, or `PBKDF2`, which incorporate cryptographic salt to defend against rainbow table attacks. Spring Security provides `BCryptPasswordEncoder` for this.

### Q. How do you implement method-level security in Spring?
**Short Answer:** By enabling `@EnableMethodSecurity`, you can use annotations like `@PreAuthorize("hasRole('ADMIN')")` directly on service methods. Spring uses AOP to intercept the method call, check the `SecurityContext`, and throw an `AccessDeniedException` if the user lacks the role.

---

[← Previous: Spring Web Request Lifecycle (DispatcherServlet)](../06-rest-web/02-spring-web-lifecycle.md) | [Next: Microservices Architecture & Resilience Patterns →](../08-microservices/01-microservices-patterns.md)
