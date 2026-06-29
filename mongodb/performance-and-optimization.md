# Performance & Optimization

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is `explain()`, and what does it show?](#1-what-is-explain-and-what-does-it-show)
- [2. What is a full collection scan, and why is it usually undesirable?](#2-what-is-a-full-collection-scan-and-why-is-it-usually-undesirable)
- [3. How does connection pooling improve MongoDB performance?](#3-how-does-connection-pooling-improve-mongodb-performance)

**🟡 Medium**
- [4. How do you identify slow queries in MongoDB?](#4-how-do-you-identify-slow-queries-in-mongodb)
- [5. What is the working set, and why does it matter for performance?](#5-what-is-the-working-set-and-why-does-it-matter-for-performance)
- [6. How does denormalization improve read performance in MongoDB?](#6-how-does-denormalization-improve-read-performance-in-mongodb)
- [7. What's the impact of large documents on performance?](#7-whats-the-impact-of-large-documents-on-performance)
- [8. How do you use projection to improve query performance?](#8-how-do-you-use-projection-to-improve-query-performance)

**🔴 Hard**
- [9. What does the query planner do, and how does it choose an execution plan?](#9-what-does-the-query-planner-do-and-how-does-it-choose-an-execution-plan)
- [10. What is index intersection, and when does MongoDB use it?](#10-what-is-index-intersection-and-when-does-mongodb-use-it)
- [11. How do you diagnose and fix a collection suffering from lock contention?](#11-how-do-you-diagnose-and-fix-a-collection-suffering-from-lock-contention)
- [12. How would you design a caching layer in front of MongoDB?](#12-how-would-you-design-a-caching-layer-in-front-of-mongodb)
- [13. What strategies exist for archiving old data to keep a collection performant?](#13-what-strategies-exist-for-archiving-old-data-to-keep-a-collection-performant)
- [14. How do you monitor MongoDB performance in production?](#14-how-do-you-monitor-mongodb-performance-in-production)

---

### 1. What is `explain()`, and what does it show? 🟢

- A method that reveals **how** MongoDB actually executed (or would execute) a given query — which indexes were used (or not), how many documents were scanned vs. returned, and the time taken — essential for diagnosing slow queries.

```js
db.users.find({ email: 'v@x.com' }).explain('executionStats');
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a full collection scan, and why is it usually undesirable? 🟢

- When MongoDB has to examine **every single document** in a collection to find matches, because no suitable index exists for the query (`COLLSCAN` in `explain()` output) — gets progressively slower as the collection grows, unlike an indexed lookup which stays fast regardless of collection size.

[↑ Back to top](#table-of-contents)

---

### 3. How does connection pooling improve MongoDB performance? 🟢

- Reusing a pool of already-established connections instead of opening a new one per request — avoids the repeated overhead of TCP/TLS handshakes for every database operation. MongoDB drivers handle this automatically by default.

[↑ Back to top](#table-of-contents)

---

### 4. How do you identify slow queries in MongoDB? 🟡

- Enable the **database profiler** to log operations exceeding a time threshold, or check `mongod`'s slow query log directly — both record execution time, query shape, and whether an index was used, pointing directly at what to optimize.

```js
db.setProfilingLevel(1, { slowms: 100 }); // log queries slower than 100ms
db.system.profile.find().sort({ ts: -1 }).limit(5);
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the working set, and why does it matter for performance? 🟡

- The portion of data (and indexes) actively being accessed, which ideally fits entirely in **RAM**. If the working set exceeds available memory, WiredTiger has to repeatedly read from disk instead of memory — disk I/O is orders of magnitude slower, so a working set that doesn't fit in RAM is one of the most common root causes of unexplained slowness at scale.

[↑ Back to top](#table-of-contents)

---

### 6. How does denormalization improve read performance in MongoDB? 🟡

- Embedding/duplicating related data (see [Schema Design](schema-design.md#11-what-is-the-extended-reference-pattern)) avoids needing a separate query or `$lookup` to assemble a complete view — trading some write-time complexity (keeping duplicated copies in sync) and storage for significantly faster, simpler reads, which is usually the right tradeoff since most applications read far more often than they write.

[↑ Back to top](#table-of-contents)

---

### 7. What's the impact of large documents on performance? 🟡

- Larger documents take more time/bandwidth to transfer over the network, consume more memory when loaded, and (under WiredTiger) any update typically rewrites the whole document — so a workload doing many small updates to a few large documents pays a bigger cost per update than the same updates spread across smaller documents. This is part of why the unbounded-array anti-pattern (see [Schema Design](schema-design.md#9-what-is-the-unbounded-array-anti-pattern-and-why-is-it-dangerous)) hurts performance over time.

[↑ Back to top](#table-of-contents)

---

### 8. How do you use projection to improve query performance? 🟡

- Only request the fields you actually need (`{ name: 1, email: 1 }`) instead of fetching the entire document — reduces the amount of data MongoDB has to read, serialize, and transfer over the network, especially impactful on documents with large fields (embedded arrays, long text) you don't need for a given query.

[↑ Back to top](#table-of-contents)

---

### 9. What does the query planner do, and how does it choose an execution plan? 🔴

- For a given query shape, the planner generates **multiple candidate plans** (using different available indexes, or none), runs them briefly in parallel against the actual data, and picks the one that performs best empirically — then **caches** that winning plan choice for future queries of the same shape, re-evaluating periodically or when index changes invalidate the cached choice.

[↑ Back to top](#table-of-contents)

---

### 10. What is index intersection, and when does MongoDB use it? 🔴

- When no single compound index covers a query's full filter, MongoDB can sometimes combine results from **two separate single-field indexes** instead of falling back to a full collection scan — a useful fallback, but generally **less efficient** than one well-designed compound index covering the actual query pattern directly, so it shouldn't be relied on as a primary indexing strategy.

[↑ Back to top](#table-of-contents)

---

### 11. How do you diagnose and fix a collection suffering from lock contention? 🔴

- Check `db.currentOp()` for long-running or queued operations, and look at WiredTiger's per-document locking behavior — contention usually stems from many concurrent writes targeting the **same document** (a frequently-incremented counter, a hot single record) or very long-running operations holding resources. Fixes include sharding the hot document's workload differently, batching updates, or reducing transaction/operation duration.

[↑ Back to top](#table-of-contents)

---

### 12. How would you design a caching layer in front of MongoDB? 🔴

- Cache frequently-read, infrequently-changed data (e.g. a product catalog) in something like Redis, with a clear invalidation strategy (TTL-based expiry, or explicit invalidation on write) — reduces load on MongoDB for "hot" reads, but introduces the classic cache-consistency tradeoff: a TTL-based cache risks briefly serving stale data, while explicit invalidation needs careful coordination on every write path that touches the cached data.

[↑ Back to top](#table-of-contents)

---

### 13. What strategies exist for archiving old data to keep a collection performant? 🔴

- Move old/rarely-accessed documents to a separate **archive collection** (or cold storage) on a schedule, keeping the "hot" collection smaller and its working set more likely to fit in RAM; alternatively, use a [TTL index](indexing.md#8-what-is-a-ttl-time-to-live-index) if the data should simply expire outright rather than being preserved elsewhere.

[↑ Back to top](#table-of-contents)

---

### 14. How do you monitor MongoDB performance in production? 🔴

- Track key metrics continuously: query latency/throughput, replication lag between primary and secondaries, cache hit ratio (working set fitting in RAM), connection counts, and lock/queue wait times — via MongoDB's own `mongostat`/`mongotop`, the built-in profiler, or a dedicated monitoring tool (MongoDB Atlas's monitoring, Datadog, Prometheus exporters) rather than waiting for users to report slowness first.

> [!IMPORTANT]
> **Follow-up questions:**
> - What would rising replication lag on a secondary specifically suggest is going wrong, and how would you start investigating it?

[↑ Back to top](#table-of-contents)

---
