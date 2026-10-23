# 💻 Software System Design Guide
### The Complete Interview & Production Reference — DSA → Design Patterns → LLD → HLD

> **The living reference for software engineering interviews and production systems.** Deeply covered topics, Java code examples, and real-world case studies.

---

## 📚 Quick Navigation

| I want to... | Start here |
|--------------|------------|
| **Crack DSA interviews** | [Big-O](01-dsa/01-complexity-and-big-o.md) → [Arrays](01-dsa/02-arrays-and-strings.md) → [Patterns](01-dsa/12-dsa-patterns-for-interviews.md) |
| **Master design patterns** | [Creational](02-design-patterns/01-creational-patterns.md) → [Structural](02-design-patterns/02-structural-patterns.md) → [Behavioral](02-design-patterns/03-behavioral-patterns.md) |
| **Ace LLD interviews** | [SOLID & OOP](03-lld/01-solid-and-oop-design.md) → [Parking Lot](03-lld/02-lld-parking-lot.md) → [Elevator](03-lld/03-lld-elevator-system.md) |
| **Ace HLD interviews** | [Foundations](04-hld/01-hld-foundations.md) → [Databases](04-hld/02-databases-sql-nosql-newsql.md) → [URL Shortener](04-hld/09-hld-design-url-shortener.md) |
| **Prepare for interviews** | [Question Bank](00-interview-prep/01-question-bank.md) → [Answer Frameworks](00-interview-prep/02-answer-frameworks.md) |
| **Practice on whiteboard** | [Whiteboard Exercises](00-interview-prep/04-whiteboard-exercises.md) |
| **Find best courses** | [Recommended Courses](COURSES.md) |

---

## 🎯 Why This Guide

**Interview prep that goes beyond surface level.** This guide is structured for the same depth interviewers expect at senior levels — concrete patterns, real tradeoffs, and production failure modes.

| This Guide | Typical Prep Resources |
|------------|----------------------|
| Java code examples throughout | Pseudocode only |
| Full LLD with class diagrams | High-level sketches |
| HLD case studies with capacity estimates | Generic diagrams |
| Flashcard summaries for quick revision | No revision aids |
| SOLID, design patterns, CAP — connected together | Siloed topics |

---

## 📖 Guide Structure

```
software-system-design/
│
├── 00-interview-prep/          # Question bank, frameworks, pitfalls, exercises
├── 01-dsa/                     # Complexity, arrays, trees, graphs, DP, sorting
├── 02-design-patterns/         # GoF patterns + concurrency + real-world usage
├── 03-lld/                     # OOP, UML, SOLID + 8 full LLD case studies
└── 04-hld/                     # Foundations, DBs, caches, queues, APIs, microservices, security, observability, case studies
│
├── GLOSSARY.md                 # Every term defined
├── PATTERNS.md                 # Pattern catalog quick reference
└── COURSES.md                  # Recommended learning paths
```

---

## 📖 Reading Order

```
START HERE
    │
    ▼
00-interview-prep/   ← Calibrate what interviewers look for
    │
    ▼
01-dsa/              ← Foundation: data structures & algorithms
    │
    ▼
02-design-patterns/  ← Reusable solutions to recurring problems
    │
    ▼
03-lld/              ← Apply patterns to design real classes & systems
    │
    ▼
04-hld/              ← Scale those systems for millions of users
    │
    ▼
INTERVIEW READY ✅
```

---

## 🔥 Featured LLD Case Studies

| Case Study | What You'll Design | Key Patterns |
|------------|-------------------|--------------|
| [Parking Lot](03-lld/02-lld-parking-lot.md) | Multi-level parking system | Strategy, Observer, Factory |
| [Elevator System](03-lld/03-lld-elevator-system.md) | Multi-elevator controller | State, Strategy, Command |
| [Movie Ticket Booking](03-lld/04-lld-movie-ticket-booking.md) | Concurrent ticket booking with seat locking | Strategy, Observer, Transaction Lock |
| [Rate Limiter](03-lld/05-lld-rate-limiter.md) | Concurrency-safe API rate limiter | Token Bucket, Sliding Window Log |
| [Food Delivery](03-lld/06-lld-food-delivery.md) | Driver matching & order state tracking | Strategy, State, Observer |
| [ATM Machine](03-lld/07-lld-atm-machine.md) | Cash dispenser & transaction states | State, Chain of Responsibility |
| [Chess Game](03-lld/08-lld-chess-game.md) | Full interactive chess logic & history logs | Command (for undo/redo), Factory |

---

## 🔥 Featured HLD Case Studies

| Case Study | Scale | Key Components |
|------------|-------|---------------|
| [URL Shortener](04-hld/09-hld-design-url-shortener.md) | 100M URLs | Hashing, KV store, CDN |
| [Twitter](04-hld/10-hld-design-twitter.md) | 500M users | Fan-out, Timeline, Kafka |
| [WhatsApp](04-hld/11-hld-design-whatsapp.md) | 2B users | WebSockets, E2E encryption |
| [YouTube](04-hld/12-hld-design-youtube.md) | 2B users | Video streaming, CDN |
| [Uber](04-hld/13-hld-design-uber.md) | 100M rides/day | Geo-spatial, WebSockets |

---

## 🎓 For Interview Prep

LLD and HLD system design interviews ask questions like:

> "Design a parking lot system. Walk me through the classes, relationships, and design patterns."

> "Design Twitter. How do you handle 500 million users and fan-out for celebrity accounts?"

This guide gives you **concrete Java implementations**, **real tradeoffs**, and **production-level thinking** — the depth interviewers expect at senior levels.

➡️ Start with [Interview Prep](00-interview-prep/)

---

## 🔄 Living Guide

This guide tracks:
- Updated Java idioms and best practices
- Emerging patterns in distributed systems
- New LLD problem types in FAANG interviews
- Deprecations and anti-patterns

---

*Last updated: June 2026*
