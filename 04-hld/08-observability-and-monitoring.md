# Observability and Monitoring

As distributed systems grow, understanding their internal state and debugging failures becomes exponentially harder. This chapter covers metrics collection, structured logging, distributed tracing (OpenTelemetry), and the SRE Golden Signals.

## Table of Contents
- [The Three Pillars of Observability](#the-three-pillars-of-observability)
- [Metrics Collection: Push vs Pull](#metrics-collection-push-vs-pull)
- [Distributed Tracing: Spans and Context Propagation](#distributed-tracing-spans-and-context-propagation)
- [Structured Logging and Aggregation](#structured-logging-and-aggregation)
- [SRE Golden Signals & Alerting (SLI, SLO, SLA)](#sre-golden-signals--alerting-sli-slo-sla)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## The Three Pillars of Observability

Observability is the ability to infer the internal state of a system based on its external outputs.

```
                  ┌─────────────────────────────────┐
                  │   Three Pillars of Observability│
                  └──────┬──────────┬──────────┬────┘
                         │          │          │
             ┌───────────▼──┐ ┌─────▼────┐ ┌───▼───────────┐
             │   Metrics    │ │   Logs   │ │    Traces     │
             │ (Numerical)  │ │ (Events) │ │ (Request Flow)│
             └──────────────┘ └──────────┘ └───────────────┘
```

1. **Metrics:** Aggregated numeric data points representing system state over time (e.g., CPU load, QPS). Extremely cheap to store, excellent for alerting.
2. **Logs:** Discrete timestamped text records of specific events (e.g., "User registration failed for ID 42"). Expensive to store, critical for root-cause analysis.
3. **Traces:** Representation of a request's journey through multiple microservices. Critical for locating bottlenecks in distributed architectures.

---

## Metrics Collection: Push vs Pull

How metrics flow from application nodes to the monitoring database (e.g., Prometheus, InfluxDB).

### 1. The Pull Model (e.g., Prometheus)
The metrics database pulls metrics periodically by scraping `/metrics` endpoints exposed by each service.
- **Pros:** No push overload on DB; simple service discovery; if a service dies, the scraper instantly detects it.
- **Cons:** Hard to pull from short-lived serverless jobs (requires a push gateway); requires open inbound ports on services.

### 2. The Push Model (e.g., Datadog, StatsD)
Services push metrics directly to a central collector agent or database.
- **Pros:** Great for serverless/ephemeral tasks; works easily with private subnets.
- **Cons:** High traffic spikes can overwhelm the metrics receiver; services must bundle push-agent libraries.

---

## Distributed Tracing: Spans and Context Propagation

Distributed tracing allows developers to track a request as it crosses network boundaries.

```
[ Gateway ] ────(Trace ID: 100, Span ID: 1)────► [ Order Service ]
                                                      │
                                            (Trace ID: 100, Span ID: 2)
                                                      │
                                                      ▼
                                              [ Payment Service ]
```

### 1. Terminology
- **Trace:** The complete lifecycle of an execution path (composed of multiple spans).
- **Span:** A single unit of work (e.g., SQL execution, HTTP call). Has a start time, duration, parent ID, and metadata.
- **Context Propagation:** Passing the `Trace ID` across HTTP/gRPC headers (e.g., using W3C Trace Context header `traceparent`) so downstream services can tag their spans under the same trace.

---

## Structured Logging and Aggregation

Logging plain text (e.g., `logger.info("Order processed for user " + id)`) is an anti-pattern in distributed systems because parsers cannot easily query it.

### 1. Structured Logging (JSON)
Format logs as JSON objects containing key-value pairs:
```json
{
  "timestamp": "2026-06-13T14:15:00Z",
  "level": "INFO",
  "service": "order-service",
  "traceId": "100abc",
  "userId": "42",
  "message": "Order processed successfully",
  "orderTotal": 150.00
}
```

### 2. Aggregation (ELK / PLG Stack)
Instead of ssh-ing into individual instances, logs are forwarded (via Filebeat/Fluentbit) to a central store:
- **Elasticsearch / Loki:** Indexes and stores log data.
- **Kibana / Grafana:** Dashboard UI for querying logs by attributes.

---

## SRE Golden Signals & Alerting (SLI, SLO, SLA)

System monitoring must focus on customer-facing metrics. SRE defines the **Four Golden Signals**:

1. **Latency:** Time taken to service a request (split successful vs failed).
2. **Traffic:** Demand placed on the system (e.g., HTTP QPS, active sessions).
3. **Errors:** Rate of requests that fail (e.g., 500 error counts, database errors).
4. **Saturation:** Measure of resource usage constraints (e.g., CPU, Memory, DB connection pool utilization).

### SLI vs SLO vs SLA
- **SLI (Indicator):** The metric itself. E.g., "Latency of `/checkout` over 5 minutes."
- **SLO (Objective):** The target goal we set. E.g., "99% of checkout requests must return in <200ms."
- **SLA (Agreement):** The legal contract with clients. E.g., "If system availability drops below 99.9% in a month, we refund 10% billing." (SLO is always tighter than SLA).

---

## Interview Questions

### Q: Why is tracing expensive, and how do we manage its storage cost?
Tracing requires injecting headers, generating UUIDs, and sending trace data to collectors on every HTTP call, consuming CPU and massive storage.
To manage this, we use **Sampling**:
- **Head-based sampling:** Decide at the entry gateway (e.g., sample exactly 5% of all incoming requests). Reduces tracing overhead immediately.
- **Tail-based sampling:** Buffers all spans of a trace and decides at the end (e.g., keep 100% of traces that returned a `500 Error` or latency `>1s`, but discard 99% of successful checkouts). Best for debugging, but computationally expensive.

### Q: What is the difference between a histogram and a gauge metric?
- **Gauge:** Represents a single numerical value that can go up and down arbitrarily. E.g., current CPU utilization (65% → 42% → 80%), active memory, queue size.
- **Histogram:** Measures the distribution of values (typically duration or sizes) across pre-defined buckets. E.g., tracking API latencies into buckets (`<50ms`, `<100ms`, `<500ms`, `>500ms`) to calculate percentiles like p95 or p99.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Three pillars | Metrics (numerical), Logs (events), Traces (request flow) |
| 2 | Prometheus model | Pull model (scrapes endpoints periodically) |
| 3 | Push metrics benefit | Best for short-lived serverless tasks |
| 4 | Trace ID | Unique ID shared across all microservices for a single request |
| 5 | Context propagation | Injecting Trace ID into network headers (e.g., W3C headers) |
| 6 | Golden Signals | Latency, Traffic, Errors, Saturation |
| 7 | SLI vs SLO | SLI: metric; SLO: target we aim for |
| 8 | Tail-based sampling | Keep traces that contain errors or high latency; discard others |
| 9 | Structured logging | Outputting logs in JSON format for indexing/querying |
| 10 | Histogram utility | Groups data into buckets; required to calculate p99 latencies |

---

*Next: [HLD Case Study — URL Shortener](09-hld-design-url-shortener.md)*
