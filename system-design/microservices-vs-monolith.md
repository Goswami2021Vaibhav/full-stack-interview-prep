# Microservices vs Monolith

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a monolithic architecture?](#1-what-is-a-monolithic-architecture)
- [2. What is a microservices architecture?](#2-what-is-a-microservices-architecture)
- [3. What are the main benefits of microservices?](#3-what-are-the-main-benefits-of-microservices)

**🟡 Medium**
- [4. What are the main downsides/costs of microservices?](#4-what-are-the-main-downsidescosts-of-microservices)
- [5. How do microservices communicate with each other?](#5-how-do-microservices-communicate-with-each-other)
- [6. What is a service mesh?](#6-what-is-a-service-mesh)
- [7. What is an API gateway, and what role does it play?](#7-what-is-an-api-gateway-and-what-role-does-it-play)
- [8. What is the database-per-service pattern, and what problem does it solve?](#8-what-is-the-database-per-service-pattern-and-what-problem-does-it-solve)

**🔴 Hard**
- [9. How do you handle a transaction that spans multiple microservices?](#9-how-do-you-handle-a-transaction-that-spans-multiple-microservices)
- [10. What is the Saga pattern?](#10-what-is-the-saga-pattern)
- [11. How would you decide where to draw service boundaries when decomposing a monolith?](#11-how-would-you-decide-where-to-draw-service-boundaries-when-decomposing-a-monolith)
- [12. What is the strangler fig pattern, and when would you use it?](#12-what-is-the-strangler-fig-pattern-and-when-would-you-use-it)

---

### 1. What is a monolithic architecture? 🟢

- A single, unified application containing all of a system's functionality — one codebase, one deployment unit, typically one database — components communicate via in-process function calls rather than network requests.

[↑ Back to top](#table-of-contents)

---

### 2. What is a microservices architecture? 🟢

- The system is split into multiple **independent**, separately deployable services, each owning a specific business capability and typically its own data store — services communicate over the network (REST, gRPC, message queues).

[↑ Back to top](#table-of-contents)

---

### 3. What are the main benefits of microservices? 🟢

- **Independent deployability** (ship one service without redeploying everything), **independent scaling** (scale just the service under heavy load), **technology flexibility** (each service can use the best-suited language/database), and **fault isolation** (one service crashing doesn't necessarily take down the whole system).

[↑ Back to top](#table-of-contents)

---

### 4. What are the main downsides/costs of microservices? 🟡

- Significantly more **operational complexity** (many services to deploy, monitor, version), **network latency/failure modes** introduced wherever an in-process call became a network call, harder **debugging** (tracing a request across multiple services), and **data consistency** challenges since each service typically owns its own database (Q8/Q9). Microservices trade simplicity for flexibility — not free, and not automatically the right choice for every system or team size.

[↑ Back to top](#table-of-contents)

---

### 5. How do microservices communicate with each other? 🟡

- **Synchronously**: direct REST/gRPC calls — simple, but couples the caller to the callee's availability and response time.
- **Asynchronously**: via a message queue/event bus — decouples services in time (the callee doesn't need to be up *right now*), better resilience, at the cost of eventual-consistency complexity (see [Message Queues & Events](message-queues-and-events.md)).

[↑ Back to top](#table-of-contents)

---

### 6. What is a service mesh? 🟡

- Infrastructure (often deployed as a sidecar proxy alongside each service instance) that handles cross-cutting service-to-service concerns — retries, timeouts, mutual TLS, load balancing, observability — **without** each service needing to implement that logic itself, centralizing it at the infrastructure layer instead of duplicating it across every service's codebase.

[↑ Back to top](#table-of-contents)

---

### 7. What is an API gateway, and what role does it play? 🟡

- A single entry point that sits in front of all microservices, routing each incoming request to the appropriate backend service — also commonly handles cross-cutting concerns centrally (authentication, rate limiting, request logging) so individual services don't each need to reimplement them.

[↑ Back to top](#table-of-contents)

---

### 8. What is the database-per-service pattern, and what problem does it solve? 🟡

- Each microservice owns its **own** database, which no other service accesses directly — enforces true service independence (a service's internal data model can change freely without coordinating with other teams) and avoids the tight coupling that a single shared database across services would create (any service could otherwise be broken by another team's schema change).

[↑ Back to top](#table-of-contents)

---

### 9. How do you handle a transaction that spans multiple microservices? 🔴

- Traditional ACID transactions don't work across separate databases owned by separate services — instead, use the **Saga pattern** (Q10): a sequence of local transactions, each in its own service, with explicit **compensating actions** to undo previous steps if a later step fails, since there's no single database transaction to simply roll back.

[↑ Back to top](#table-of-contents)

---

### 10. What is the Saga pattern? 🔴

- Breaks a multi-service business operation into a sequence of local transactions, each committed independently — if a later step fails, previously completed steps are undone via **compensating transactions** (e.g. "refund the payment" to compensate for an earlier "charge the payment" step), rather than a single atomic rollback. Implemented either via **orchestration** (a central coordinator explicitly tells each service what to do next) or **choreography** (each service reacts to events from the previous step, with no central coordinator).

[↑ Back to top](#table-of-contents)

---

### 11. How would you decide where to draw service boundaries when decomposing a monolith? 🔴

- Align boundaries with **business capabilities/domains** (a "domain-driven design" bounded context) rather than technical layers — e.g. separate `Orders`, `Inventory`, and `Payments` services rather than separate `Database-access-service`/`Business-logic-service` layers. Good boundaries minimize the need for cross-service calls/transactions for common operations, and roughly align with team ownership boundaries (Conway's Law: system structure tends to mirror organizational communication structure).

[↑ Back to top](#table-of-contents)

---

### 12. What is the strangler fig pattern, and when would you use it? 🔴

- An **incremental** migration strategy for moving from a monolith to microservices: route specific pieces of functionality to new, separately-built services **one at a time** (via a gateway/proxy), while the rest of the traffic still flows to the existing monolith — gradually "strangling" the monolith's responsibilities down to nothing, rather than attempting a risky, all-at-once rewrite. Used specifically when a full rewrite is too risky/slow to do as a single big-bang cutover.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is a full rewrite-and-cutover approach generally considered riskier than the strangler fig pattern for migrating a large, business-critical monolith?

[↑ Back to top](#table-of-contents)

---
