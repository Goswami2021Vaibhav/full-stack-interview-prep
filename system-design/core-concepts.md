# Core Concepts

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is system design, and why is it asked in interviews?](#1-what-is-system-design-and-why-is-it-asked-in-interviews)
- [2. What's the difference between high-level design (HLD) and low-level design (LLD)?](#2-whats-the-difference-between-high-level-design-hld-and-low-level-design-lld)
- [3. What are functional vs. non-functional requirements?](#3-what-are-functional-vs-non-functional-requirements)
- [4. What is latency vs. throughput?](#4-what-is-latency-vs-throughput)

**🟡 Medium**
- [5. What is availability, and how is it typically measured?](#5-what-is-availability-and-how-is-it-typically-measured)
- [6. What's the difference between vertical and horizontal scaling?](#6-whats-the-difference-between-vertical-and-horizontal-scaling)
- [7. What is a single point of failure (SPOF)?](#7-what-is-a-single-point-of-failure-spof)
- [8. What's the difference between a stateless and a stateful service?](#8-whats-the-difference-between-a-stateless-and-a-stateful-service)
- [9. Why do interviewers ask for back-of-the-envelope estimation?](#9-why-do-interviewers-ask-for-back-of-the-envelope-estimation)

**🔴 Hard**
- [10. What's the difference between throughput and bandwidth?](#10-whats-the-difference-between-throughput-and-bandwidth)
- [11. How do you approach a system design interview question structurally?](#11-how-do-you-approach-a-system-design-interview-question-structurally)
- [12. How do you reason about capacity planning (estimating QPS, storage, bandwidth)?](#12-how-do-you-reason-about-capacity-planning-estimating-qps-storage-bandwidth)
- [13. What is a bottleneck, and how do you identify one in a system design?](#13-what-is-a-bottleneck-and-how-do-you-identify-one-in-a-system-design)
- [14. Why does "it depends on the requirements" matter so much in system design answers?](#14-why-does-it-depends-on-the-requirements-matter-so-much-in-system-design-answers)

---

### 1. What is system design, and why is it asked in interviews? 🟢

- The process of defining the architecture, components, and data flow needed to satisfy a system's functional and non-functional requirements — interviewers use it to assess how a candidate handles ambiguity, makes and justifies tradeoffs, and reasons about systems at scale, rather than just whether they know a "correct" answer (there usually isn't one single correct answer).

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between high-level design (HLD) and low-level design (LLD)? 🟢

- **HLD**: the big picture — major components (services, databases, caches, queues), how they connect, and the overall data flow.
- **LLD**: the details within a single component — class/interface design, specific algorithms, database schema fields. Most "system design interview" rounds focus on HLD; LLD shows up more in object-oriented design rounds.

[↑ Back to top](#table-of-contents)

---

### 3. What are functional vs. non-functional requirements? 🟢

- **Functional**: *what* the system should do — "users can post a tweet," "users can follow other users."
- **Non-functional**: *how well* it should do it — latency targets, availability targets, scalability needs, consistency requirements. Both must be clarified upfront, since non-functional requirements (e.g. "must handle 1M requests/sec") drive most of the actual architectural decisions.

[↑ Back to top](#table-of-contents)

---

### 4. What is latency vs. throughput? 🟢

- **Latency**: how long a **single** operation takes (response time).
- **Throughput**: how **many** operations a system can handle in a given time period. The two aren't always correlated — a system can have low latency but low throughput (a single fast machine) or higher latency but high throughput (a batching pipeline that processes many requests together).

[↑ Back to top](#table-of-contents)

---

### 5. What is availability, and how is it typically measured? 🟡

- The percentage of time a system is operational and able to serve requests, typically expressed in "nines" — 99.9% ("three nines") allows ~8.7 hours of downtime per year, while 99.99% ("four nines") allows only ~52 minutes per year. Higher availability targets cost significantly more in redundancy and complexity.

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between vertical and horizontal scaling? 🟡

- **Vertical scaling**: adding more resources (CPU, RAM) to a **single** machine — simple, but has a hard ceiling and remains a single point of failure.
- **Horizontal scaling**: adding **more machines** to share the load — effectively unlimited ceiling and better fault tolerance, but requires the application to be designed to run across multiple instances (often meaning it needs to be stateless, see Q8).

[↑ Back to top](#table-of-contents)

---

### 7. What is a single point of failure (SPOF)? 🟡

- Any component whose failure brings down the **entire** system, because nothing else can take over for it — a key thing to identify and eliminate (via redundancy, replication, failover) when designing for high availability.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between a stateless and a stateful service? 🟡

- **Stateless**: doesn't retain any client-specific data between requests — any instance can handle any request, making horizontal scaling and load balancing trivial.
- **Stateful**: retains some per-client state (an in-memory session, an open connection) — scaling it requires either routing a client consistently back to the same instance (sticky sessions) or externalizing the state (to Redis, a database) so any instance can serve any client.

[↑ Back to top](#table-of-contents)

---

### 9. Why do interviewers ask for back-of-the-envelope estimation? 🟡

- Rough numbers (expected users, requests per second, storage growth per day) directly determine architectural decisions — whether you need sharding, how much cache capacity is needed, whether a single database can handle the write load at all. Skipping estimation often leads candidates toward over-engineering a system that doesn't actually need that complexity, or under-engineering one that does.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between throughput and bandwidth? 🔴

- **Throughput**: the actual rate of **requests/operations** a system processes (e.g. queries per second).
- **Bandwidth**: the maximum rate of **data transfer** a network connection can carry (e.g. megabits per second) — a system can be throughput-limited (CPU/DB-bound) well below its available bandwidth, or vice versa for data-heavy workloads (video streaming) where bandwidth is the binding constraint rather than request count.

[↑ Back to top](#table-of-contents)

---

### 11. How do you approach a system design interview question structurally? 🔴

- A common structure: (1) clarify functional and non-functional requirements explicitly, (2) do rough capacity estimation (Q12), (3) sketch a high-level architecture/API design, (4) deep-dive into the most interesting/critical components (data model, a specific bottleneck), (5) discuss tradeoffs and how the design would evolve under further scale or failure scenarios. The structure matters less than demonstrating you're driving the conversation deliberately rather than jumping straight to a final diagram.

[↑ Back to top](#table-of-contents)

---

### 12. How do you reason about capacity planning (estimating QPS, storage, bandwidth)? 🔴

- Start from a rough user count and usage pattern (e.g. "100M daily active users, each posts twice a day") and derive: **QPS** (total daily actions ÷ seconds in a day, adjusted for peak traffic multipliers), **storage** (size per record × records created per day × retention period), and **bandwidth** (average payload size × QPS). The goal isn't precision — it's getting numbers in the right **order of magnitude** to inform whether you need sharding, caching, or a CDN at all.

[↑ Back to top](#table-of-contents)

---

### 13. What is a bottleneck, and how do you identify one in a system design? 🔴

- The component that limits the **entire** system's capacity, regardless of how much you scale everything else — identified by reasoning about which resource (CPU, disk I/O, network, a single database's write throughput) saturates first under the estimated load (Q12). Scaling components that **aren't** the actual bottleneck wastes effort without improving overall system capacity.

[↑ Back to top](#table-of-contents)

---

### 14. Why does "it depends on the requirements" matter so much in system design answers? 🔴

- Nearly every system design decision is a **tradeoff** (consistency vs. availability, normalized vs. denormalized data, SQL vs. NoSQL) — the "right" choice depends entirely on the specific system's actual requirements (read/write ratio, consistency needs, scale). A strong answer explicitly **names the tradeoff** and justifies the choice against the stated requirements, rather than presenting one architecture as universally correct.

> [!IMPORTANT]
> **Follow-up questions:**
> - If an interviewer doesn't give you explicit numbers (users, QPS, data size) upfront, what should you do before designing anything?

[↑ Back to top](#table-of-contents)

---
