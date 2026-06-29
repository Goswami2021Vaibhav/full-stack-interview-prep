# Scalability & Load Balancing

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a load balancer?](#1-what-is-a-load-balancer)
- [2. What is horizontal scaling, applied to a web tier?](#2-what-is-horizontal-scaling-applied-to-a-web-tier)
- [3. What is a reverse proxy?](#3-what-is-a-reverse-proxy)

**🟡 Medium**
- [4. What load balancing algorithms are commonly used?](#4-what-load-balancing-algorithms-are-commonly-used)
- [5. What's the difference between Layer 4 and Layer 7 load balancing?](#5-whats-the-difference-between-layer-4-and-layer-7-load-balancing)
- [6. What is session affinity (sticky sessions), and what tradeoff does it introduce?](#6-what-is-session-affinity-sticky-sessions-and-what-tradeoff-does-it-introduce)
- [7. What is auto-scaling?](#7-what-is-auto-scaling)
- [8. How do load balancers perform health checks?](#8-how-do-load-balancers-perform-health-checks)

**🔴 Hard**
- [9. How do you scale a stateful service (e.g. WebSocket connections) behind a load balancer?](#9-how-do-you-scale-a-stateful-service-eg-websocket-connections-behind-a-load-balancer)
- [10. What is DNS-based load balancing, and what are its limitations?](#10-what-is-dns-based-load-balancing-and-what-are-its-limitations)
- [11. How would you avoid the load balancer itself becoming a single point of failure?](#11-how-would-you-avoid-the-load-balancer-itself-becoming-a-single-point-of-failure)
- [12. Why is scaling writes generally harder than scaling reads?](#12-why-is-scaling-writes-generally-harder-than-scaling-reads)
- [13. What is the "thundering herd" problem, and how do you mitigate it?](#13-what-is-the-thundering-herd-problem-and-how-do-you-mitigate-it)

---

### 1. What is a load balancer? 🟢

- A component that distributes incoming requests across multiple backend server instances — improves throughput (work is spread out), availability (one instance failing doesn't take down the whole service), and allows horizontal scaling by adding more instances behind it.

[↑ Back to top](#table-of-contents)

---

### 2. What is horizontal scaling, applied to a web tier? 🟢

- Running multiple identical instances of your application server, each capable of handling any incoming request, with a load balancer distributing traffic across them — adding capacity means adding more instances, rather than upgrading one machine's specs.

[↑ Back to top](#table-of-contents)

---

### 3. What is a reverse proxy? 🟢

- A server that sits in **front** of backend servers, forwarding client requests to them and returning the response back to the client — clients only ever talk to the proxy, never directly to a backend instance. Load balancers are commonly implemented as a type of reverse proxy.

[↑ Back to top](#table-of-contents)

---

### 4. What load balancing algorithms are commonly used? 🟡

- **Round robin**: cycles through instances in order.
- **Least connections**: routes to whichever instance currently has the fewest active connections.
- **Weighted round robin**: like round robin, but some instances (more powerful machines) get a proportionally larger share.
- **IP hash**: routes based on a hash of the client's IP, giving a consistent instance per client (a simple form of session affinity, Q6).

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between Layer 4 and Layer 7 load balancing? 🟡

- **Layer 4** (transport layer): routes based purely on IP/port information, without inspecting the actual request content — faster, simpler, protocol-agnostic.
- **Layer 7** (application layer): inspects the actual HTTP request (URL path, headers, cookies) to make smarter routing decisions (e.g. routing `/api/*` to one service and `/static/*` to another) — more flexible, at the cost of more processing overhead per request.

[↑ Back to top](#table-of-contents)

---

### 6. What is session affinity (sticky sessions), and what tradeoff does it introduce? 🟡

- Routes a given client **consistently** to the same backend instance across requests (often via a cookie) — useful when a service holds in-memory session state. The tradeoff: it undermines clean load distribution (an instance with many "stuck" long-lived clients can get overloaded while others sit idle) and complicates scaling, which is why externalizing session state (Redis) instead of relying on sticky sessions is generally preferred for new designs.

[↑ Back to top](#table-of-contents)

---

### 7. What is auto-scaling? 🟡

- Automatically adding or removing server instances based on real-time load metrics (CPU usage, request rate, queue depth) — handles variable traffic (a flash sale, a viral spike) without manually provisioning for peak capacity 24/7, saving cost during low-traffic periods.

[↑ Back to top](#table-of-contents)

---

### 8. How do load balancers perform health checks? 🟡

- Periodically pinging a lightweight endpoint (e.g. `/health`) on each backend instance — instances that fail to respond (or respond with an error) within a timeout are automatically removed from the pool of instances receiving traffic, until they recover.

[↑ Back to top](#table-of-contents)

---

### 9. How do you scale a stateful service (e.g. WebSocket connections) behind a load balancer? 🔴

- A WebSocket connection is long-lived and tied to a **specific** server instance (unlike a stateless HTTP request) — scaling requires either sticky routing at the load balancer (so reconnects land on the same instance holding that connection's state) **or**, more robustly, a **pub/sub backplane** (Redis Pub/Sub, a message broker) so any instance can publish a message that reaches a client connected to a **different** instance, decoupling "who's connected to whom" from "which instance needs to deliver this message."

[↑ Back to top](#table-of-contents)

---

### 10. What is DNS-based load balancing, and what are its limitations? 🔴

- Configuring DNS to return **multiple** IP addresses (or different IPs to different regions) for the same domain — clients connect to whichever IP they receive. Limitations: DNS responses are **cached** (by browsers, ISPs) for their TTL, so removing an unhealthy server from rotation doesn't take effect immediately everywhere; it also can't make fine-grained, real-time routing decisions the way an actual load balancer in the request path can.

[↑ Back to top](#table-of-contents)

---

### 11. How would you avoid the load balancer itself becoming a single point of failure? 🔴

- Run **multiple** load balancer instances (often behind a floating/virtual IP, or DNS round-robin across them), with health checks and automatic failover between them — at large scale, this is layered: a DNS or anycast-routed layer distributes traffic across multiple independent load balancer instances/regions, each of which then load-balances across application instances.

[↑ Back to top](#table-of-contents)

---

### 12. Why is scaling writes generally harder than scaling reads? 🔴

- Reads can be trivially scaled by adding more **read replicas** (each independently serving read traffic from a copy of the data) — but all **writes** must still ultimately be coordinated against a single source of truth to avoid conflicting/inconsistent updates. Scaling writes requires actually partitioning the data (sharding) so different writes can go to different machines, which is architecturally far more involved than simply adding read replicas.

[↑ Back to top](#table-of-contents)

---

### 13. What is the "thundering herd" problem, and how do you mitigate it? 🔴

- When a large number of clients/instances **simultaneously** retry or restart at once (e.g. after a brief outage, or when a popular cache entry expires), overwhelming the backend with a sudden traffic spike right when it's least able to absorb it. Mitigations: **jittered/randomized retry backoff** (so retries spread out over time instead of synchronizing), staggered auto-scaling/restart rollouts, and request coalescing (e.g. a cache lock so only one request regenerates an expired value while others wait, rather than all of them hitting the database at once — see [Caching](caching.md#9-what-is-cache-stampede-and-how-do-you-prevent-it)).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does adding random jitter to retry delays help more than just adding a fixed delay before retrying?

[↑ Back to top](#table-of-contents)

---
