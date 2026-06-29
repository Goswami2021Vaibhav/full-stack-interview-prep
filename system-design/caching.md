# Caching

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is caching, and why is it used?](#1-what-is-caching-and-why-is-it-used)
- [2. What is a cache hit vs. a cache miss?](#2-what-is-a-cache-hit-vs-a-cache-miss)
- [3. What is TTL (Time To Live)?](#3-what-is-ttl-time-to-live)

**🟡 Medium**
- [4. What is cache invalidation, and why is it hard?](#4-what-is-cache-invalidation-and-why-is-it-hard)
- [5. What are common cache eviction policies?](#5-what-are-common-cache-eviction-policies)
- [6. What's the difference between write-through, write-around, and write-back caching?](#6-whats-the-difference-between-write-through-write-around-and-write-back-caching)
- [7. Where can caching be applied across a typical web stack?](#7-where-can-caching-be-applied-across-a-typical-web-stack)
- [8. What is a CDN, and how does it relate to caching?](#8-what-is-a-cdn-and-how-does-it-relate-to-caching)

**🔴 Hard**
- [9. What is cache stampede, and how do you prevent it?](#9-what-is-cache-stampede-and-how-do-you-prevent-it)
- [10. What's the difference between cache-aside and read-through caching?](#10-whats-the-difference-between-cache-aside-and-read-through-caching)
- [11. How do you handle cache consistency across multiple application server instances?](#11-how-do-you-handle-cache-consistency-across-multiple-application-server-instances)
- [12. Why is cache invalidation called one of the "two hard things in computer science"?](#12-why-is-cache-invalidation-called-one-of-the-two-hard-things-in-computer-science)
- [13. How would you design a caching strategy for a social media feed?](#13-how-would-you-design-a-caching-strategy-for-a-social-media-feed)

---

### 1. What is caching, and why is it used? 🟢

- Storing a copy of frequently-accessed or expensive-to-compute data somewhere **faster to read from** than its original source — reduces latency for the caller and load on the underlying system (database, an external API).

[↑ Back to top](#table-of-contents)

---

### 2. What is a cache hit vs. a cache miss? 🟢

- **Cache hit**: the requested data **is** found in the cache — fast path.
- **Cache miss**: the data isn't in the cache — must be fetched from the original (slower) source, typically populating the cache for next time.

[↑ Back to top](#table-of-contents)

---

### 3. What is TTL (Time To Live)? 🟢

- The duration a cached entry is considered valid before it automatically expires — after which it's treated as a miss and re-fetched from the source on the next request.

[↑ Back to top](#table-of-contents)

---

### 4. What is cache invalidation, and why is it hard? 🟡

- The process of removing/updating a cached entry once the underlying data has **changed**, so the cache doesn't keep serving stale data. It's hard because the cache and the source of truth are **two separate copies** that must be kept in sync — every code path that writes to the underlying data must also remember to invalidate the corresponding cache entry, and missing even one path leads to silently stale data.

[↑ Back to top](#table-of-contents)

---

### 5. What are common cache eviction policies? 🟡

- **LRU** (Least Recently Used): evicts the entry that hasn't been accessed in the longest time.
- **LFU** (Least Frequently Used): evicts the entry accessed the fewest times.
- **FIFO**: evicts the oldest-inserted entry, regardless of access pattern. LRU is the most commonly used default, since "recently accessed" is usually a decent proxy for "likely to be accessed again soon."

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between write-through, write-around, and write-back caching? 🟡

- **Write-through**: every write goes to the cache **and** the underlying store simultaneously — always consistent, but every write pays the full latency cost.
- **Write-around**: writes go **only** to the underlying store, bypassing the cache — the cache fills in lazily on subsequent reads; avoids caching data that's written but rarely re-read.
- **Write-back**: writes go to the cache **first**, and are flushed to the underlying store **later** (asynchronously) — fastest writes, but risks data loss if the cache fails before the flush happens.

[↑ Back to top](#table-of-contents)

---

### 7. Where can caching be applied across a typical web stack? 🟡

- **Browser cache** (static assets), **CDN** (Q8, edge-cached content close to users), **reverse proxy/web server cache**, **application-level cache** (Redis/Memcached for computed results, session data), and **database query cache/buffer pool** (the database's own internal caching, see [MySQL › Performance](../mysql/performance-and-optimization.md#9-what-is-the-innodb-buffer-pool-and-why-is-its-size-critical-for-performance)). Each layer caches at a different granularity and lifetime.

[↑ Back to top](#table-of-contents)

---

### 8. What is a CDN, and how does it relate to caching? 🟡

- A **C**ontent **D**elivery **N**etwork — a globally distributed network of edge servers that cache and serve content (images, videos, static files, sometimes API responses) **physically close** to the requesting user, reducing latency from network distance and offloading traffic that would otherwise hit your origin servers directly.

[↑ Back to top](#table-of-contents)

---

### 9. What is cache stampede, and how do you prevent it? 🔴

- When a **popular** cache entry expires (or the cache itself restarts cold), and a large burst of concurrent requests **all** miss simultaneously and hit the underlying database/service at once to regenerate it — potentially overwhelming it. Mitigations: a **lock/mutex** so only one request regenerates the value while others wait briefly for it, staggered/jittered TTLs (so popular entries don't all expire at the exact same moment), or serving slightly stale data while a background refresh happens.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between cache-aside and read-through caching? 🔴

- **Cache-aside**: the **application** explicitly checks the cache first, and on a miss, fetches from the source and writes the result into the cache itself — the application owns the caching logic.
- **Read-through**: the **cache itself** (acting as a layer in front of the data source) handles fetching from the source transparently on a miss — the application just always asks the cache, unaware of whether it was a hit or miss underneath.

[↑ Back to top](#table-of-contents)

---

### 11. How do you handle cache consistency across multiple application server instances? 🔴

- Use a **shared, external cache** (Redis/Memcached) rather than each instance maintaining its **own** separate in-memory cache — with separate in-memory caches, one instance invalidating its local copy doesn't affect the (now stale) copy in every other instance. A shared cache means invalidation happens in exactly one place, visible to all instances immediately.

[↑ Back to top](#table-of-contents)

---

### 12. Why is cache invalidation called one of the "two hard things in computer science"? 🔴

- (The joke: "There are only two hard things in Computer Science: cache invalidation and naming things.") It's hard because correctness depends on **every** write path remembering to invalidate the right cache entries, often across multiple, loosely-coordinated services — a single missed invalidation path leads to subtle, hard-to-reproduce staleness bugs that often only surface in production under real traffic patterns, long after the code was written.

[↑ Back to top](#table-of-contents)

---

### 13. How would you design a caching strategy for a social media feed? 🔴

- Cache each user's **pre-computed feed** (fan-out-on-write: when a followed user posts, push the new post into each follower's cached feed) for fast reads on the extremely common "view my feed" action — accepting some write amplification (one post triggers many cache updates) in exchange for near-instant reads, which is the right tradeoff for a read-heavy feed. For users with very large follower counts ("celebrities"), a hybrid fan-out-on-read approach is often used instead, to avoid an enormous write amplification spike from a single post.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does fan-out-on-write become problematic specifically for accounts with millions of followers, and how does a hybrid approach address that?

[↑ Back to top](#table-of-contents)

---
