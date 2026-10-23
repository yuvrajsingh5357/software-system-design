# Spring Boot Architecture and Core Concepts

## 1. What Is It?
Spring Boot is an opinionated framework built on top of the Spring Framework. Its primary mission is to radically simplify Java backend development by providing auto-configuration, starter dependencies, and an embedded server.

## 2. The Problem Before Spring Boot
While the original Spring Framework solved the problem of tight coupling (via DI and IoC), setting up a Spring project was still painful. Developers had to:
- Write heavy, repetitive XML configuration files.
- Manually register beans, view resolvers, and component scanners.
- Search for compatible versions of various libraries (Jackson, Hibernate, Tomcat).
- Download Tomcat separately, build a `.war` file, and deploy it to the external server.

## 3. What Does Spring Boot Solve?
Spring Boot removes the boilerplate. 
- It uses an **opinionated approach**: it assumes you want to build a standard production-ready application and configures sensible defaults automatically.
- You don't configure `DispatcherServlet` or Jackson manually; Spring Boot does it behind the scenes.
- It changes the deployment model from deploying `.war` files to external servers to running standalone `.jar` files with embedded servers.

## 4. Key Features of Spring Boot

### A. Auto-Configuration
Auto-configuration is the heart of Spring Boot. When the application starts, Spring Boot scans your classpath, checks your dependencies, and automatically configures the components it thinks you need.
- *Example:* If Spring Boot sees `spring-boot-starter-web` on the classpath, it automatically configures Tomcat and Spring MVC. If it sees `spring-data-jpa` and a PostgreSQL driver, it automatically configures Hibernate, a DataSource, and a TransactionManager.

### B. Starter Dependencies
Instead of hunting for compatible versions of dozens of libraries, Spring Boot provides pre-packaged "Starters".
- `spring-boot-starter-web`: Pulls in Spring MVC, REST support, Jackson, and embedded Tomcat.
- `spring-boot-starter-data-jpa`: Pulls in Hibernate, JPA API, and HikariCP.
This eliminates dependency version conflicts.

### C. Embedded Servers
Spring Boot includes an embedded server (Tomcat by default, but Jetty and Undertow are supported).
- You don't install Tomcat on your host machine.
- You compile your app into a fat `.jar` file that *contains* Tomcat.
- When you execute `java -jar myapp.jar`, the application starts the server internally.
- This is the fundamental reason Spring Boot is the standard for **Microservices**—each service runs independently with its own embedded server.

### D. Spring Boot Actuator
A built-in module that provides production-ready monitoring and management endpoints.
- `/actuator/health`: Checks if the DB, message brokers, and app are healthy.
- `/actuator/metrics`: Exposes memory, CPU, and request statistics.
- It integrates seamlessly with monitoring tools like Prometheus and Grafana.

## 5. The @SpringBootApplication Annotation
Every Spring Boot app starts with a main class annotated with `@SpringBootApplication`. This is actually a shortcut that combines three critical annotations:
1. **`@EnableAutoConfiguration`:** Triggers the auto-configuration mechanism.
2. **`@ComponentScan`:** Tells Spring to scan the current package and its sub-packages for `@Component`, `@Service`, `@Controller`, etc.
3. **`@Configuration`:** Allows the class to define beans using `@Bean` methods.

## 6. Interview Questions

### Q. What is the difference between Spring and Spring Boot?
#### Short Interview Answer
Spring is a comprehensive, low-level framework focused on Dependency Injection, AOP, and infrastructure. Spring Boot is an opinionated layer built on top of Spring designed for rapid development. Spring requires manual configuration and `.war` deployment, whereas Spring Boot provides auto-configuration, starter dependencies, and embedded servers to create standalone `.jar` applications instantly.

### Q. How does Auto-Configuration work in Spring Boot?
#### Short Interview Answer
Auto-configuration, triggered by `@EnableAutoConfiguration`, works by evaluating the classpath at startup. Spring Boot looks at the `.jar` files present in the dependencies and automatically creates and registers the necessary Spring Beans. For example, if it detects a database driver and Spring Data JPA, it will automatically wire up a `DataSource`, `EntityManager`, and transaction management without any explicit XML or Java configuration from the developer.

### Q. How can you change the default embedded server in Spring Boot?
#### Short Interview Answer
By default, `spring-boot-starter-web` includes Tomcat. To switch to Jetty or Undertow, you simply exclude the Tomcat dependency from the web starter in your `pom.xml` or `build.gradle`, and add the corresponding starter dependency for Jetty (`spring-boot-starter-jetty`) or Undertow. Spring Boot's auto-configuration will detect the new server on the classpath and start it automatically.

---

[← Previous: Bean Lifecycle and AOP](../03-spring-core/03-bean-lifecycle-and-aop.md) | [Next: DevTools & Actuator →](04-devtools-and-actuator.md)
