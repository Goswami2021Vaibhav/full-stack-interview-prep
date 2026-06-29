# Design Principles

_Part of [REST API](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How should you name REST API endpoints/resources?](#1-how-should-you-name-rest-api-endpointsresources)
- [2. Should URLs use nouns or verbs?](#2-should-urls-use-nouns-or-verbs)
- [3. What's the difference between a collection and a single-resource endpoint?](#3-whats-the-difference-between-a-collection-and-a-single-resource-endpoint)

**🟡 Medium**
- [4. How do you represent relationships between resources in URLs?](#4-how-do-you-represent-relationships-between-resources-in-urls)
- [5. What's the tradeoff between flat and nested URL structures?](#5-whats-the-tradeoff-between-flat-and-nested-url-structures)
- [6. How should you handle actions that don't map cleanly to CRUD?](#6-how-should-you-handle-actions-that-dont-map-cleanly-to-crud)
- [7. What is content negotiation?](#7-what-is-content-negotiation)
- [8. How do you design a consistent response structure across endpoints?](#8-how-do-you-design-a-consistent-response-structure-across-endpoints)

**🔴 Hard**
- [9. What is the Richardson Maturity Model?](#9-what-is-the-richardson-maturity-model)
- [10. How do you handle long-running operations in a REST API?](#10-how-do-you-handle-long-running-operations-in-a-rest-api)
- [11. What's the tradeoff between a chatty API and a coarse-grained API?](#11-whats-the-tradeoff-between-a-chatty-api-and-a-coarse-grained-api)
- [12. How would you design idempotent POST requests (e.g. for payments)?](#12-how-would-you-design-idempotent-post-requests-eg-for-payments)
- [13. How do you decide between exposing many small resources vs. a few large composite ones?](#13-how-do-you-decide-between-exposing-many-small-resources-vs-a-few-large-composite-ones)

---

### 1. How should you name REST API endpoints/resources? 🟢

- Use **plural nouns** for collections (`/users`, not `/user`), keep names consistent and lowercase, and use hyphens rather than underscores or camelCase in URLs (`/user-profiles`, not `/user_profiles` or `/userProfiles`).

[↑ Back to top](#table-of-contents)

---

### 2. Should URLs use nouns or verbs? 🟢

- **Nouns** — the resource itself, not the action. The HTTP **method** already conveys the action (`GET`, `POST`, `DELETE`), so `/users` + `DELETE` is correct; `/deleteUser` is not RESTful.

```
✅ DELETE /users/42
❌ POST /deleteUser?id=42
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between a collection and a single-resource endpoint? 🟢

- `/users` (collection): `GET` lists all users, `POST` creates a new one.
- `/users/42` (single resource): `GET` fetches user 42, `PUT`/`PATCH` updates it, `DELETE` removes it.

[↑ Back to top](#table-of-contents)

---

### 4. How do you represent relationships between resources in URLs? 🟡

- Nest the related resource under its parent when it's logically owned by it: `/users/42/orders` (orders belonging to user 42). For resources that exist somewhat independently, use a flat structure with a query/filter parameter instead: `/orders?userId=42`.

[↑ Back to top](#table-of-contents)

---

### 5. What's the tradeoff between flat and nested URL structures? 🟡

- **Nested** (`/users/42/orders/7`): clearly expresses ownership/hierarchy, but gets unwieldy beyond 2 levels deep, and the same resource (an order) may need multiple different nested paths depending on context.
- **Flat** (`/orders/7`, with `userId` as a property/filter): simpler, more flexible, avoids deep nesting — generally preferred once relationships go beyond one level.

[↑ Back to top](#table-of-contents)

---

### 6. How should you handle actions that don't map cleanly to CRUD? 🟡

- Model the action itself as a sub-resource, typically created via `POST`: `POST /users/42/activate` rather than inventing a non-standard HTTP verb. Think of it as "creating an activation event" rather than "activating," which keeps it noun-based.

```
POST /orders/7/cancel
POST /users/42/password-reset-requests
```

[↑ Back to top](#table-of-contents)

---

### 7. What is content negotiation? 🟡

- The client tells the server what response format it wants via the `Accept` header (e.g. `application/json` vs `application/xml`), and the server responds in that format (or `406 Not Acceptable` if it can't). Most modern APIs only support JSON, but the mechanism still applies for versioning (media-type versioning) or rare XML/JSON dual-support cases.

[↑ Back to top](#table-of-contents)

---

### 8. How do you design a consistent response structure across endpoints? 🟡

- Pick one envelope shape and apply it everywhere — e.g. always wrap data under a `data` key, errors under an `error` key, and metadata (pagination, etc.) under `meta` — so clients can write generic response-handling code instead of special-casing each endpoint.

```json
{ "data": { "id": 42, "name": "Vaibhav" }, "meta": {} }
```

[↑ Back to top](#table-of-contents)

---

### 9. What is the Richardson Maturity Model? 🔴

- A 4-level model (0–3) describing how "RESTful" an API really is:
  - **Level 0**: one endpoint, one method (e.g. SOAP-style RPC-over-HTTP) — barely REST at all.
  - **Level 1**: multiple resource-based URLs, but still mostly one HTTP method (often just `POST`).
  - **Level 2**: proper use of HTTP methods and status codes per resource — most real-world "REST APIs" sit here.
  - **Level 3**: adds HATEOAS — full hypermedia-driven navigation.
- Most production APIs stop at **Level 2**, which is generally considered "good enough" REST in practice.

[↑ Back to top](#table-of-contents)

---

### 10. How do you handle long-running operations in a REST API? 🔴

- Respond immediately with **202 Accepted** and a status URL the client can poll (or a webhook callback), rather than holding the HTTP connection open until the operation finishes.

```http
POST /reports/export
→ 202 Accepted
  Location: /reports/export/jobs/123

GET /reports/export/jobs/123
→ { "status": "processing" } or { "status": "done", "downloadUrl": "..." }
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the tradeoff between a chatty API and a coarse-grained API? 🔴

- **Chatty** (many small, granular endpoints): flexible, each call does one clear thing, but a client needing related data makes many round trips — costly over high-latency networks (mobile).
- **Coarse-grained** (fewer endpoints returning larger composite payloads): fewer round trips, but less flexible, and clients may receive more data than they actually need (over-fetching) — this exact tension is part of why GraphQL exists as an alternative.

[↑ Back to top](#table-of-contents)

---

### 12. How would you design idempotent POST requests (e.g. for payments)? 🔴

- Since `POST` isn't idempotent by default, require the client to supply a unique **idempotency key** (often a header) — the server stores the result of the first request under that key, and any retry with the same key returns the **original** result instead of re-processing (e.g. charging the card twice).

```http
POST /payments
Idempotency-Key: 7d8f9e2a-...
```

[↑ Back to top](#table-of-contents)

---

### 13. How do you decide between exposing many small resources vs. a few large composite ones? 🔴

- Base it on actual client usage patterns: if clients consistently need several related pieces of data together, a composite endpoint avoids the chatty-API problem (Q11); if needs vary a lot per client, smaller granular resources stay more flexible and reusable. There's no universal answer — it's a tradeoff informed by real consumer needs, not a purity rule.

> [!IMPORTANT]
> **Follow-up questions:**
> - How does GraphQL's design directly address the chatty-vs-coarse-grained tradeoff that plain REST struggles with?

[↑ Back to top](#table-of-contents)

---
