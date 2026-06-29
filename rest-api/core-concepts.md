# Core Concepts

_Part of [REST API](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is REST, and what does "RESTful" mean?](#1-what-is-rest-and-what-does-restful-mean)
- [2. What are the key principles of REST?](#2-what-are-the-key-principles-of-rest)
- [3. What's the difference between REST and SOAP?](#3-whats-the-difference-between-rest-and-soap)
- [4. What is a "resource" in REST?](#4-what-is-a-resource-in-rest)

**🟡 Medium**
- [5. What does "stateless" mean in REST, and why does it matter?](#5-what-does-stateless-mean-in-rest-and-why-does-it-matter)
- [6. What is HATEOAS, and is it commonly implemented in practice?](#6-what-is-hateoas-and-is-it-commonly-implemented-in-practice)
- [7. What is idempotency, and which HTTP methods are idempotent?](#7-what-is-idempotency-and-which-http-methods-are-idempotent)
- [8. What's the difference between safe and unsafe HTTP methods?](#8-whats-the-difference-between-safe-and-unsafe-http-methods)
- [9. What's the difference between a REST API and a plain CRUD API?](#9-whats-the-difference-between-a-rest-api-and-a-plain-crud-api)

**🔴 Hard**
- [10. What's the difference between REST and GraphQL?](#10-whats-the-difference-between-rest-and-graphql)
- [11. What's the difference between REST and gRPC?](#11-whats-the-difference-between-rest-and-grpc)
- [12. How does REST achieve cacheability, and why does it matter for scalability?](#12-how-does-rest-achieve-cacheability-and-why-does-it-matter-for-scalability)
- [13. What are the architectural constraints of REST, and which is most commonly violated in practice?](#13-what-are-the-architectural-constraints-of-rest-and-which-is-most-commonly-violated-in-practice)

---

### 1. What is REST, and what does "RESTful" mean? 🟢

- REST (**Re**presentational **S**tate **T**ransfer) is an architectural style for designing networked APIs, built around resources identified by URLs and manipulated via standard HTTP methods. "RESTful" describes an API that follows REST's conventions/principles.

[↑ Back to top](#table-of-contents)

---

### 2. What are the key principles of REST? 🟢

- **Client-server separation**, **statelessness** (Q5), **cacheability**, a **uniform interface** (consistent use of HTTP methods/URLs), **layered system** (proxies/gateways can sit transparently between client and server), and optionally **code on demand**.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between REST and SOAP? 🟢

- **REST**: an architectural style, typically using JSON over plain HTTP — lightweight, flexible, no strict standard contract.
- **SOAP**: a strict, XML-based **protocol** with a formal contract (WSDL), built-in error handling and security standards — more rigid, more overhead, still used in some enterprise/legacy systems (e.g. banking, telecom).

[↑ Back to top](#table-of-contents)

---

### 4. What is a "resource" in REST? 🟢

- Any entity exposed via the API and addressable by a URL — a user, an order, a collection of products. Operations on a resource happen via HTTP methods (`GET /users/42` reads it, `DELETE /users/42` removes it), not via verbs baked into the URL.

[↑ Back to top](#table-of-contents)

---

### 5. What does "stateless" mean in REST, and why does it matter? 🟡

- Each request must contain **all** the information the server needs to process it — the server doesn't store any client session state between requests. This lets any server instance handle any request (critical for horizontal scaling and load balancing), and makes the API easier to reason about and cache.

[↑ Back to top](#table-of-contents)

---

### 6. What is HATEOAS, and is it commonly implemented in practice? 🟡

- **H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate — responses include links to related actions/resources, so clients navigate the API dynamically rather than hardcoding URL structures.

```json
{ "id": 42, "name": "Vaibhav", "links": { "self": "/users/42", "orders": "/users/42/orders" } }
```

- In practice, **most** real-world REST APIs skip full HATEOAS — it adds complexity for limited practical benefit when clients are tightly coupled to the API anyway (mobile apps, internal services). It's more of an ideal from Fielding's original definition than a widely-adopted convention.

[↑ Back to top](#table-of-contents)

---

### 7. What is idempotency, and which HTTP methods are idempotent? 🟡

- An operation is idempotent if making the **same** request multiple times produces the same result as making it once (no extra side effects on repeats).
- `GET`, `PUT`, `DELETE`, `HEAD`, `OPTIONS` are idempotent. `POST` and `PATCH` are generally **not** (a repeated `POST` typically creates a second resource).

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between safe and unsafe HTTP methods? 🟡

- **Safe**: doesn't change server state at all — `GET`, `HEAD`, `OPTIONS` (read-only, can be cached/prefetched freely).
- **Unsafe**: may modify state — `POST`, `PUT`, `PATCH`, `DELETE`. (Every safe method is also idempotent, but not every idempotent method is safe — `DELETE` is idempotent but not safe.)

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between a REST API and a plain CRUD API? 🟡

- A plain CRUD API just maps create/read/update/delete to endpoints, without necessarily following REST's broader principles (statelessness, proper use of status codes/methods, cacheability, uniform interface). Most "REST APIs" in practice are really CRUD APIs that loosely follow REST conventions — true strict REST (with HATEOAS, full uniform interface) is rarer (see Richardson Maturity Model in [Design Principles](design-principles.md)).

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between REST and GraphQL? 🔴

- **REST**: multiple fixed endpoints, each returning a fixed shape — can lead to over-fetching (unused fields) or under-fetching (needing multiple round trips for related data).
- **GraphQL**: a single endpoint where the client specifies exactly what fields/relations it needs in the query — avoids over/under-fetching, at the cost of more complex server-side query resolution and harder HTTP-level caching (since every query is typically a POST to the same URL).

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between REST and gRPC? 🔴

- **REST**: text-based (typically JSON over HTTP/1.1), human-readable, broadly compatible with browsers/tooling.
- **gRPC**: binary protocol (Protocol Buffers) over HTTP/2, supports streaming in both directions, significantly faster/more compact — but needs generated client/server code from `.proto` definitions and isn't natively browser-friendly, making it better suited to internal service-to-service communication than public-facing APIs.

[↑ Back to top](#table-of-contents)

---

### 12. How does REST achieve cacheability, and why does it matter for scalability? 🔴

- Responses can declare themselves cacheable via standard HTTP headers (`Cache-Control`, `ETag`, `Last-Modified`) — clients, proxies, and CDNs can then reuse a cached response instead of hitting the origin server again. This reduces server load and latency dramatically at scale, which is why REST's reliance on **standard HTTP semantics** (rather than a custom protocol) is a deliberate design advantage — it gets caching infrastructure "for free."

[↑ Back to top](#table-of-contents)

---

### 13. What are the architectural constraints of REST, and which is most commonly violated in practice? 🔴

- Fielding's dissertation defines six: client-server, statelessness, cacheability, uniform interface, layered system, and (optional) code-on-demand.
- **HATEOAS** (part of the uniform interface constraint) is the one most commonly skipped in real-world "REST" APIs (Q6) — most APIs that call themselves RESTful are, strictly speaking, just stateless HTTP/JSON APIs rather than fully compliant with Fielding's original definition.

> [!IMPORTANT]
> **Follow-up questions:**
> - Does skipping HATEOAS meaningfully hurt a typical API's usability in practice, or is it mostly a theoretical purity concern?

[↑ Back to top](#table-of-contents)

---
