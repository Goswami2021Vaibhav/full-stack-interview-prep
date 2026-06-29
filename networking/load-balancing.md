# Load Balancing

_Part of [Networking](README.md) interview notes._

> Load balancing **algorithms**, sticky sessions, and auto-scaling are covered from a system-design perspective in [System Design › Scalability & Load Balancing](../system-design/scalability-and-load-balancing.md). This file focuses on the underlying network-layer mechanics.

## Table of Contents

**🟢 Easy**
- [1. What is load balancing, at the network level?](#1-what-is-load-balancing-at-the-network-level)
- [2. What's the difference between Layer 4 and Layer 7 load balancing?](#2-whats-the-difference-between-layer-4-and-layer-7-load-balancing)

**🟡 Medium**
- [3. What is DNS round robin?](#3-what-is-dns-round-robin)
- [4. What is a load balancer's health check, at the network level?](#4-what-is-a-load-balancers-health-check-at-the-network-level)
- [5. What is anycast routing, and how does it relate to load balancing?](#5-what-is-anycast-routing-and-how-does-it-relate-to-load-balancing)
- [6. What is NAT-based load balancing?](#6-what-is-nat-based-load-balancing)

**🔴 Hard**
- [7. How does a Layer 4 load balancer make routing decisions without inspecting application data?](#7-how-does-a-layer-4-load-balancer-make-routing-decisions-without-inspecting-application-data)
- [8. What is connection draining?](#8-what-is-connection-draining)
- [9. How does global server load balancing (GSLB) work across multiple data centers?](#9-how-does-global-server-load-balancing-gslb-work-across-multiple-data-centers)
- [10. What's the difference between hardware and software load balancers?](#10-whats-the-difference-between-hardware-and-software-load-balancers)
- [11. Why might you combine DNS-based and Layer 7 load balancing in one architecture?](#11-why-might-you-combine-dns-based-and-layer-7-load-balancing-in-one-architecture)

---

### 1. What is load balancing, at the network level? 🟢

- Distributing incoming network traffic across multiple servers so no single one is overwhelmed — implemented at various layers of the network stack, from simple DNS-level tricks up to deep application-aware routing.

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between Layer 4 and Layer 7 load balancing? 🟢

- **Layer 4**: makes routing decisions using only **transport-layer** information (source/destination IP and port) — fast, protocol-agnostic, can't see inside the actual request content.
- **Layer 7**: inspects the actual **application-layer** content (HTTP headers, URL path, cookies) to make smarter, content-aware routing decisions — more flexible, more processing overhead per request.

[↑ Back to top](#table-of-contents)

---

### 3. What is DNS round robin? 🟡

- A simple load-balancing technique where a domain's DNS records list **multiple** IP addresses, and the DNS server returns them in rotating order — clients connect to whichever IP they happen to receive. Crude (no real-time awareness of server health/load) and limited by DNS caching (see [DNS § propagation](dns.md#9-what-is-dns-propagation-and-why-can-it-take-time)), but simple to set up with no dedicated load balancer hardware/software needed.

[↑ Back to top](#table-of-contents)

---

### 4. What is a load balancer's health check, at the network level? 🟡

- Periodic, low-level probes (a TCP connection attempt, an ICMP ping, or a lightweight HTTP request) sent to each backend server to confirm it's still reachable and responding — servers failing these checks are automatically removed from the pool receiving traffic until they recover.

[↑ Back to top](#table-of-contents)

---

### 5. What is anycast routing, and how does it relate to load balancing? 🟡

- A routing technique where the **same** IP address is announced from **multiple, geographically distributed** locations — the underlying internet routing infrastructure automatically directs each client to the **topologically nearest** location advertising that address. Used by CDNs and DNS providers as an effective, infrastructure-level form of load balancing/proximity routing, without any application-level logic needed.

[↑ Back to top](#table-of-contents)

---

### 6. What is NAT-based load balancing? 🟡

- The load balancer presents a single **virtual IP** to clients, and uses NAT to rewrite the destination address of incoming packets to whichever **actual backend server** it selects (and rewrites the response back) — the backend servers and the actual routing decision stay completely hidden from the client, which only ever sees the one virtual IP.

[↑ Back to top](#table-of-contents)

---

### 7. How does a Layer 4 load balancer make routing decisions without inspecting application data? 🔴

- It only looks at the TCP/UDP packet headers (source/destination IP and port) — typically hashing some combination of these (or simply round-robin) to pick a backend, then either rewriting the packet's destination (NAT-based, Q6) or directly relaying the connection — all without ever parsing or understanding the actual HTTP/application payload riding inside those packets, which is what makes it fast but unable to route based on content like a URL path.

[↑ Back to top](#table-of-contents)

---

### 8. What is connection draining? 🔴

- When removing a server from the load balancer's pool (for deployment, scaling down, maintenance), connection draining lets **existing, in-flight** connections to that server finish naturally, while simply not routing any **new** connections to it — avoids abruptly cutting off active users mid-request, which a hard/immediate removal would do.

[↑ Back to top](#table-of-contents)

---

### 9. How does global server load balancing (GSLB) work across multiple data centers? 🔴

- Combines DNS-based routing (directing a client to the nearest/healthiest **data center**, often via GeoDNS or anycast, Q5) with a traditional load balancer **within** each data center (distributing traffic across servers there) — a two-tier approach: DNS/anycast handles the coarse-grained "which region" decision, and a local load balancer handles the fine-grained "which server" decision once traffic arrives.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between hardware and software load balancers? 🔴

- **Hardware**: dedicated physical appliances (e.g. F5 BIG-IP) — historically offered very high raw throughput and low latency via specialized hardware, but expensive, less flexible, and harder to scale elastically.
- **Software** (Nginx, HAProxy, cloud-native load balancers): run as regular software on commodity servers or as a managed cloud service — far more flexible and elastically scalable, and modern hardware/networking has narrowed the raw performance gap enough that software load balancers now dominate most new architectures, especially in cloud environments.

[↑ Back to top](#table-of-contents)

---

### 11. Why might you combine DNS-based and Layer 7 load balancing in one architecture? 🔴

- DNS-based routing operates at a **coarse** granularity (which entire data center/region a client should reach) and is essentially free of per-request overhead, while Layer 7 load balancing provides **fine-grained, content-aware** routing **within** a given location (path-based routing, sticky sessions, request-level health awareness) — neither fully substitutes for the other; DNS-level routing gets traffic to the right general place, and Layer 7 handles the smart decisions once it's there (see GSLB, Q9).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can't anycast alone fully replace the need for a Layer 7 load balancer within a single data center?

[↑ Back to top](#table-of-contents)

---
