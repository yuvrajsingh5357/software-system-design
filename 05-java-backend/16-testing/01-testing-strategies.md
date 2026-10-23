# Backend Testing Strategies

## 1. Why Do We Need Testing?
In backend engineering, testing guarantees that business logic works, prevents regressions when refactoring, and ensures the application behaves correctly under edge cases without requiring manual verification.

## 2. The Testing Pyramid
1. **Unit Tests (Fast, Isolated, Many):** Test a single class or method in isolation. Dependencies are mocked.
2. **Integration Tests (Slower, Integrated, Fewer):** Test how multiple units interact together, or how the application interacts with a real database or message broker.
3. **End-to-End / API Tests (Slow, Full System, Very Few):** Test the entire HTTP request lifecycle from Controller to Database and back.

## 3. Spring Boot Testing Tools
- **JUnit 5:** The core testing framework (assertions, test execution).
- **Mockito:** Framework to create fake objects (mocks) to isolate the unit being tested.
- **`@SpringBootTest`:** Loads the *entire* Spring Application Context. Used for heavy integration tests.
- **`@WebMvcTest`:** Loads *only* the web layer (Controllers, Filters, Interceptors). Does not load Services or Repositories. Used to test HTTP responses, JSON serialization, and validation quickly.
- **`@DataJpaTest`:** Loads *only* the JPA/Hibernate layer and configures an in-memory database (H2) by default to test Repository queries.
- **Testcontainers:** A library that spins up actual Docker containers (like PostgreSQL or Kafka) during the test phase to test against real infrastructure instead of in-memory mocks.

## 4. Unit Testing Example (JUnit + Mockito)
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock
    private PaymentRepository paymentRepo;

    @InjectMocks
    private OrderService orderService; // Injects the mock repo into this service

    @Test
    void shouldProcessOrder() {
        // Arrange
        when(paymentRepo.process(any())).thenReturn(true);
        
        // Act
        boolean result = orderService.checkout(new Order());
        
        // Assert
        assertTrue(result);
        verify(paymentRepo, times(1)).process(any()); // Verify the mock was called
    }
}
```

## 5. Interview Questions
### Q. When would you use @Mock vs @MockBean?
**Short Answer:** `@Mock` is a pure Mockito annotation used in fast, isolated Unit Tests that do not require the Spring context. `@MockBean` is a Spring Boot annotation used in Integration Tests (`@SpringBootTest` or `@WebMvcTest`). It tells Spring to replace a real bean inside the Application Context with a Mockito mock.

### Q. Why is using H2 for integration testing sometimes discouraged?
**Short Answer:** H2 is an in-memory database that doesn't fully replicate the specific behaviors, syntax, or locking mechanisms of production databases like PostgreSQL or MySQL. An integration test might pass on H2 but fail in production. Using Testcontainers to spin up a real PostgreSQL Docker image during tests is the modern best practice for ensuring exact environment parity.

---

[← Previous: Docker Fundamentals for Java Backends](../12-docker/01-docker-fundamentals.md) | [Next: Observability (Logs, Metrics, Traces) →](../17-observability/01-observability-metrics.md)
