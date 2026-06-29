# Performance & Optimization

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is `EXPLAIN`, and what does it show?](#1-what-is-explain-and-what-does-it-show)
- [2. What is a slow query log?](#2-what-is-a-slow-query-log)
- [3. How does connection pooling help MySQL performance?](#3-how-does-connection-pooling-help-mysql-performance)

**🟡 Medium**
- [4. What is the query cache, and why was it removed in MySQL 8.0?](#4-what-is-the-query-cache-and-why-was-it-removed-in-mysql-80)
- [5. How does `LIMIT` with a large `OFFSET` hurt performance?](#5-how-does-limit-with-a-large-offset-hurt-performance)
- [6. Why should you avoid `SELECT *` in production queries?](#6-why-should-you-avoid-select--in-production-queries)
- [7. What is the N+1 query problem, and how do you fix it?](#7-what-is-the-n1-query-problem-and-how-do-you-fix-it)
- [8. How does batching inserts improve write performance?](#8-how-does-batching-inserts-improve-write-performance)

**🔴 Hard**
- [9. What is the InnoDB buffer pool, and why is its size critical for performance?](#9-what-is-the-innodb-buffer-pool-and-why-is-its-size-critical-for-performance)
- [10. How do you read an `EXPLAIN` output's `type` column to judge query efficiency?](#10-how-do-you-read-an-explain-outputs-type-column-to-judge-query-efficiency)
- [11. What is replication lag, and how does it affect read scaling with read replicas?](#11-what-is-replication-lag-and-how-does-it-affect-read-scaling-with-read-replicas)
- [12. How would you optimize a query that's doing a full table scan on a large table?](#12-how-would-you-optimize-a-query-thats-doing-a-full-table-scan-on-a-large-table)
- [13. What's the difference between vertical scaling, read replicas, and sharding for MySQL?](#13-whats-the-difference-between-vertical-scaling-read-replicas-and-sharding-for-mysql)

---

### 1. What is `EXPLAIN`, and what does it show? 🟢

- Prefixing a query reveals MySQL's **execution plan** for it — which indexes (if any) it would use, the estimated number of rows examined, and the join strategy — without actually running the query.

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 42;
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a slow query log? 🟢

- A log MySQL can be configured to write queries to once they exceed a configured execution-time threshold — the standard starting point for finding which specific queries need optimization in a production system.

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- log queries slower than 1 second
```

[↑ Back to top](#table-of-contents)

---

### 3. How does connection pooling help MySQL performance? 🟢

- Reuses already-open connections instead of establishing a new TCP/authentication handshake per request — opening a fresh connection has real, avoidable overhead, especially at high request volume.

[↑ Back to top](#table-of-contents)

---

### 4. What is the query cache, and why was it removed in MySQL 8.0? 🟡

- A feature that cached the **exact result** of a `SELECT` query, returning it instantly if the same query ran again **before** the underlying table changed. Removed in 8.0 because it scaled poorly under concurrency — a single global lock protected the cache, and **any** write to a table invalidated all cached queries referencing it, making it actively counterproductive on busy, frequently-written-to systems (the cost of maintaining/invalidating it outweighed its benefit).

[↑ Back to top](#table-of-contents)

---

### 5. How does `LIMIT` with a large `OFFSET` hurt performance? 🟡

- MySQL still has to **scan and discard** every row up to the offset before returning the requested page — for a large offset (deep pagination), this gets progressively slower, mirroring the same issue covered for offset-based pagination generally (see [REST API › Pagination](../rest-api/pagination-filtering-sorting.md#5-what-are-the-drawbacks-of-offset-based-pagination-at-scale)). Keyset pagination (filtering by `WHERE id > lastSeenId`) avoids this entirely.

[↑ Back to top](#table-of-contents)

---

### 6. Why should you avoid `SELECT *` in production queries? 🟡

- Pulls **every** column, even ones you don't need — wastes network bandwidth and memory, and prevents the query from being satisfied by a covering index (see [Indexing](indexing.md#10-what-is-a-covering-index-and-why-is-it-especially-fast)), since the index alone almost never contains every column in the table. Explicitly listing needed columns also makes queries more resilient to schema changes (a new column added later won't silently bloat existing queries).

[↑ Back to top](#table-of-contents)

---

### 7. What is the N+1 query problem, and how do you fix it? 🟡

- Fetching a list of N parent records, then running **one additional query per record** to fetch related data (often from an ORM lazily loading associations) — N+1 total round trips instead of a constant number. Fix by eagerly fetching the related data in **one** query upfront, typically via a `JOIN` or a single `WHERE id IN (...)` query covering all the related rows at once.

```sql
-- N+1: one query per user to fetch their orders
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2; -- ...repeated per user

-- Fixed: one query for all of them
SELECT * FROM orders WHERE user_id IN (1, 2, 3, ...);
```

[↑ Back to top](#table-of-contents)

---

### 8. How does batching inserts improve write performance? 🟡

- A single multi-row `INSERT` avoids the per-statement overhead (parsing, round trip, transaction bookkeeping) of issuing many separate single-row inserts — significantly faster for bulk data loading.

```sql
INSERT INTO users (name, email) VALUES ('A', 'a@x.com'), ('B', 'b@x.com'), ('C', 'c@x.com');
```

[↑ Back to top](#table-of-contents)

---

### 9. What is the InnoDB buffer pool, and why is its size critical for performance? 🔴

- InnoDB's main in-memory cache for table/index data — reads/writes that hit data already in the buffer pool avoid disk I/O entirely. If the buffer pool is too small relative to the active working set, MySQL must repeatedly read from disk (far slower than memory), making buffer pool size one of the single most impactful tuning parameters (`innodb_buffer_pool_size`) for a production MySQL instance.

[↑ Back to top](#table-of-contents)

---

### 10. How do you read an `EXPLAIN` output's `type` column to judge query efficiency? 🔴

- Roughly from best to worst: `const`/`system` (at most one matching row, e.g. a primary key lookup), `eq_ref`/`ref` (an index lookup matching a small set of rows), `range` (an index range scan), `index` (scans the whole index, but not the table), `ALL` (a full table scan — generally the one to investigate and fix via better indexing).

[↑ Back to top](#table-of-contents)

---

### 11. What is replication lag, and how does it affect read scaling with read replicas? 🔴

- The delay between a write committing on the **primary** and that same write becoming visible on a **replica** — under replication load or heavy write traffic, this lag can grow. An application reading from a replica immediately after writing to the primary may not see its own just-written data yet ("read-after-write" inconsistency) — a common gotcha when scaling reads via replicas, often worked around by routing read-your-own-write scenarios back to the primary.

[↑ Back to top](#table-of-contents)

---

### 12. How would you optimize a query that's doing a full table scan on a large table? 🔴

- Run `EXPLAIN` to confirm exactly what's happening and why (Q10), check whether a suitable index exists on the filtered/joined columns (and whether its cardinality, Q12 of [Indexing](indexing.md#12-what-is-index-cardinality-and-why-does-it-matter), actually makes it useful), rewrite the query to avoid patterns that defeat indexes (a function applied to the indexed column, a leading wildcard `LIKE`), and consider whether the query is even selective enough to benefit from an index at all versus genuinely needing most of the table.

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between vertical scaling, read replicas, and sharding for MySQL? 🔴

- **Vertical scaling**: a bigger server — simplest, no architecture change, but has a ceiling.
- **Read replicas**: offload **read** traffic to replicated copies — doesn't help write throughput, since all writes still go through one primary, and introduces replication lag (Q11) to manage.
- **Sharding**: split data **horizontally** across multiple independent MySQL instances, each owning a subset of the data — needed once write throughput or total data size exceeds a single primary's capacity, at the cost of significant application-level complexity (cross-shard joins/transactions become hard), similar in spirit to [MongoDB's sharding tradeoffs](../mongodb/replication-and-sharding.md#14-how-do-you-choose-between-scaling-vertically-adding-more-replica-set-secondaries-or-sharding).

> [!IMPORTANT]
> **Follow-up questions:**
> - If a system is read-heavy but rarely write-heavy, which of these three options addresses the actual bottleneck most directly, and why?

[↑ Back to top](#table-of-contents)

---
