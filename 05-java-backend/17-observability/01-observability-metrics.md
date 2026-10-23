# Observability and Telemetry

## 1. What is Observability?
Observability is the ability to understand the internal state of a system simply by looking at its external outputs. In distributed microservices, a single user request might travel through 5 different services. If it fails or is slow, traditional debugging (setting breakpoints) is impossible. You need Observability.

## 2. The Three Pillars of Observability
1. **Logs:** Immutable records of discrete events that happened over time. (e.g., `ERROR 2023-10-01 Payment failed for user 123`).
2. **Metrics:** Aggregated numeric representations of system health over intervals of time. (e.g., CPU is at 80%, JVM Heap is 90% full, 500 HTTP requests per second).
3. **Traces:** A representation of a series of causally related distributed events that encode the end-to-end request flow.

## 3. Correlation IDs and Distributed Tracing
When Service A calls Service B, how do you link their logs together to trace a single request?
- **Correlation ID (Trace ID):** A unique UUID generated at the API Gateway when a request enters the system. This ID is passed in HTTP headers (`X-B3-TraceId`) to every downstream service.
- Every microservice includes this Trace ID in every log statement.
- **Span ID:** A sub-ID representing a specific unit of work (e.g., a database query) within the larger Trace.

**Spring Implementation:** Use `micrometer-tracing` and `logbook`. Spring will automatically generate Trace IDs and Span IDs, inject them into SLF4J/Logback logs (MDC - Mapped Diagnostic Context), and propagate them via `RestTemplate` or `WebClient`.

## 4. Metrics and Monitoring
- **Spring Boot Actuator:** Provides endpoints like `/actuator/metrics` exposing hundreds of internal JVM and application metrics.
- **Micrometer:** The metrics collection facade in Spring (like SLF4J for logging). It formats Actuator metrics into formats readable by external systems.
- **Prometheus:** A time-series database that continuously "scrapes" the `/actuator/prometheus` endpoint to store historical metric data.
- **Grafana:** A visualization dashboard that queries Prometheus to draw graphs and trigger alerts (e.g., send a Slack message if Error Rate > 5%).

## 5. Interview Questions
### Q. How do you find the root cause of a 500 Internal Server Error in a system with 20 microservices?
**Short Answer:** I would rely on distributed tracing. I would search my centralized logging system (like ELK or Splunk) for the HTTP 500 error to find the specific log entry. Because we use Micrometer Tracing, that log entry will contain a Trace ID. I would then query the logging system or a tracing tool (like Zipkin or Jaeger) using that Trace ID. This will output the entire distributed call graph, showing me exactly which of the 20 services threw the original exception and what the stack trace was.

---

[← Previous: Testing Strategies (Unit, Integration, Testcontainers)](../16-testing/01-testing-strategies.md) | [Next: Production Debugging (CPU, Memory, Deadlocks) →](../13-production-debugging/01-cpu-memory-debugging.md)
