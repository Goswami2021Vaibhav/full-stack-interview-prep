# DNS

_Part of [Networking](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is DNS, and what problem does it solve?](#1-what-is-dns-and-what-problem-does-it-solve)
- [2. What is a domain name?](#2-what-is-a-domain-name)
- [3. What is an A record?](#3-what-is-an-a-record)

**🟡 Medium**
- [4. What are common DNS record types?](#4-what-are-common-dns-record-types)
- [5. How does DNS resolution work, step by step?](#5-how-does-dns-resolution-work-step-by-step)
- [6. What's the difference between a recursive resolver and an authoritative server?](#6-whats-the-difference-between-a-recursive-resolver-and-an-authoritative-server)
- [7. What is DNS caching, and what role does TTL play?](#7-what-is-dns-caching-and-what-role-does-ttl-play)
- [8. What is a CNAME record, and when would you use one?](#8-what-is-a-cname-record-and-when-would-you-use-one)

**🔴 Hard**
- [9. What is DNS propagation, and why can it take time?](#9-what-is-dns-propagation-and-why-can-it-take-time)
- [10. What is GeoDNS, and how does it relate to load balancing?](#10-what-is-geodns-and-how-does-it-relate-to-load-balancing)
- [11. What is DNSSEC, and what problem does it solve?](#11-what-is-dnssec-and-what-problem-does-it-solve)
- [12. What is DNS cache poisoning, and how is it mitigated?](#12-what-is-dns-cache-poisoning-and-how-is-it-mitigated)

---

### 1. What is DNS, and what problem does it solve? 🟢

- The **D**omain **N**ame **S**ystem translates human-readable domain names (`example.com`) into the IP addresses computers actually use to route traffic — without it, you'd need to memorize and type raw IP addresses for every website.

[↑ Back to top](#table-of-contents)

---

### 2. What is a domain name? 🟢

- A human-readable, hierarchical name identifying a resource on the internet (`www.example.com`) — read right to left in terms of hierarchy: `.com` (top-level domain) → `example` (the registered domain) → `www` (a subdomain).

[↑ Back to top](#table-of-contents)

---

### 3. What is an A record? 🟢

- A DNS record mapping a domain name directly to an **IPv4 address** — the most basic and common record type.

```
example.com.    A    93.184.216.34
```

[↑ Back to top](#table-of-contents)

---

### 4. What are common DNS record types? 🟡

- **A**: maps to an IPv4 address.
- **AAAA**: maps to an IPv6 address.
- **CNAME**: aliases one domain name to another (Q8).
- **MX**: specifies the mail server responsible for a domain.
- **TXT**: arbitrary text, often used for domain verification or email security (SPF/DKIM) records.

[↑ Back to top](#table-of-contents)

---

### 5. How does DNS resolution work, step by step? 🟡

1. The client checks its **local cache** — if found and not expired, done.
2. If not cached, it asks a configured **recursive resolver** (often the ISP's, or a public one like 8.8.8.8).
3. The resolver queries a **root** DNS server, which points it to the right **TLD** server (for `.com`, etc.).
4. The TLD server points it to the domain's **authoritative** name server.
5. The authoritative server returns the actual record (e.g. the A record's IP), which the resolver caches and returns to the client.

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between a recursive resolver and an authoritative server? 🟡

- **Recursive resolver**: does the legwork **on behalf of** the client, querying through the DNS hierarchy (root → TLD → authoritative) until it has a final answer, then caching and returning it.
- **Authoritative server**: holds the **actual, definitive** records for a specific domain — it's the final source of truth the recursive resolver eventually reaches.

[↑ Back to top](#table-of-contents)

---

### 7. What is DNS caching, and what role does TTL play? 🟡

- Resolvers (and browsers/OSes) **cache** DNS answers locally to avoid repeating the full resolution process for every single request — each record's **TTL** (Time To Live) specifies how long that cached answer remains valid before it must be re-queried, balancing freshness (a short TTL reflects changes quickly) against performance/load (a longer TTL reduces repeated lookups).

[↑ Back to top](#table-of-contents)

---

### 8. What is a CNAME record, and when would you use one? 🟡

- An alias mapping one domain name to **another** domain name (not directly to an IP) — e.g. `blog.example.com` → `example.github.io`. Useful when pointing a subdomain at a third-party service whose underlying IP might change, since you alias to their domain name rather than a specific IP that could become stale.

[↑ Back to top](#table-of-contents)

---

### 9. What is DNS propagation, and why can it take time? 🔴

- After updating a DNS record, the change must spread across **every** caching resolver around the world that had the old value cached — since each cached copy only refreshes once its TTL expires, the old answer can keep being served from various caches until their individual TTLs run out, meaning the practical "propagation time" for a change is roughly bounded by the previous record's TTL.

[↑ Back to top](#table-of-contents)

---

### 10. What is GeoDNS, and how does it relate to load balancing? 🔴

- A DNS configuration that returns **different** IP addresses depending on the geographic location (or network) of the requester — routes users to the server/data center closest to them, reducing latency, and forms a simple (though coarse-grained, see [System Design › Scalability](../system-design/scalability-and-load-balancing.md#10-what-is-dns-based-load-balancing-and-what-are-its-limitations)) form of load distribution at the DNS layer, before any request even reaches a specific server.

[↑ Back to top](#table-of-contents)

---

### 11. What is DNSSEC, and what problem does it solve? 🔴

- **D**NS **S**ecurity **E**xtensions — adds cryptographic **signatures** to DNS records, letting a resolver verify that a response genuinely came from the legitimate authoritative source and wasn't tampered with in transit — addresses the fact that plain DNS has no built-in authenticity verification, making it otherwise vulnerable to spoofing/cache poisoning (Q12).

[↑ Back to top](#table-of-contents)

---

### 12. What is DNS cache poisoning, and how is it mitigated? 🔴

- An attack where a malicious actor injects a **forged** DNS response into a resolver's cache, causing it to return an attacker-controlled IP for a legitimate domain (redirecting victims to a malicious server). Mitigated by DNSSEC (Q11, cryptographically verifying authenticity), using randomized query IDs/source ports (making forged responses harder to guess/match), and resolvers validating that responses actually correspond to a query they sent.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does a short DNS TTL help limit the blast radius of a successful cache poisoning attack, even without DNSSEC?

[↑ Back to top](#table-of-contents)

---
