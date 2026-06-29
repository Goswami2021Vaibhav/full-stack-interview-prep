# Database Design & Sharding

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you decide between SQL and NoSQL for a system design problem?](#1-how-do-you-decide-between-sql-and-nosql-for-a-system-design-problem)
- [2. What is database replication?](#2-what-is-database-replication)
- [3. What is database sharding?](#3-what-is-database-sharding)

**🟡 Medium**
- [4. What are common sharding strategies?](#4-what-are-common-sharding-strategies)
- [5. What is a hot partition/hot shard, and how do you avoid one?](#5-what-is-a-hot-partitionhot-shard-and-how-do-you-avoid-one)
- [6. What's the difference between vertical partitioning and horizontal partitioning (sharding)?](#6-whats-the-difference-between-vertical-partitioning-and-horizontal-partitioning-sharding)
- [7. How do you handle joins/queries across shards?](#7-how-do-you-handle-joinsqueries-across-shards)
- [8. How does replication lag affect application design?](#8-how-does-replication-lag-affect-application-design)

**🔴 Hard**
- [9. How would you re-shard a database without downtime?](#9-how-would-you-re-shard-a-database-without-downtime)
- [10. What is a composite shard key, and why might you need one?](#10-what-is-a-composite-shard-key-and-why-might-you-need-one)
- [11. How do you design a globally unique ID generator for a sharded system?](#11-how-do-you-design-a-globally-unique-id-generator-for-a-sharded-system)
- [12. What's the tradeoff between normalization and denormalization in a system design context?](#12-whats-the-tradeoff-between-normalization-and-denormalization-in-a-system-design-context)
- [13. How would you design the data storage layer for a system with a billion users?](#13-how-would-you-design-the-data-storage-layer-for-a-system-with-a-billion-users)

---

### 1. How do you decide between SQL and NoSQL for a system design problem? 🟢

- Favor **SQL** when data is highly relational, consistency/transactions matter (financial records), and the schema is relatively stable. Favor **NoSQL** when the schema evolves frequently, data is naturally document/key-value shaped, or you need to scale writes horizontally more easily than a single relational primary typically allows (see [MongoDB › Core Concepts](../mongodb/core-concepts.md#11-whats-the-difference-between-sql-and-nosql-databases-and-when-would-you-choose-one-over-the-other)).

[↑ Back to top](#table-of-contents)

---

### 2. What is database replication? 🟢

- Maintaining multiple copies of the same data across different servers — improves read scalability (read replicas) and availability (a failover target if the primary goes down).

[↑ Back to top](#table-of-contents)

---

### 3. What is database sharding? 🟢

- Splitting a dataset **horizontally** across multiple database instances, each holding only a subset of the rows — needed once a single instance can no longer handle the total write throughput or storage size, even after vertical scaling and read replicas.

[↑ Back to top](#table-of-contents)

---

### 4. What are common sharding strategies? 🟡

- **Key-based (hash) sharding**: hash a shard key (e.g. user ID) to determine which shard owns a row — distributes data evenly, but makes range queries and resharding harder.
- **Range-based sharding**: assign contiguous ranges of the shard key to each shard — easy range queries, but prone to hot shards if writes cluster at one end of the range.
- **Directory-based sharding**: a separate lookup service maps each key to its shard explicitly — most flexible (easy to rebalance), but adds a dependency on that lookup service for every query.

[↑ Back to top](#table-of-contents)

---

### 5. What is a hot partition/hot shard, and how do you avoid one? 🟡

- A shard receiving disproportionately more traffic/data than others, usually from a poorly chosen shard key (e.g. sharding by a monotonically increasing timestamp, concentrating all new writes onto the shard owning the latest range). Avoid by choosing a shard key with high cardinality and an even, unpredictable distribution (e.g. a hashed user ID rather than a raw sequential ID).

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between vertical partitioning and horizontal partitioning (sharding)? 🟡

- **Vertical partitioning**: splitting a table by **columns** — e.g. moving rarely-accessed large text fields into a separate table, keeping the frequently-queried core table smaller and faster.
- **Horizontal partitioning (sharding)**: splitting a table by **rows** across multiple database instances — each shard has the full schema, but only a subset of the rows.

[↑ Back to top](#table-of-contents)

---

### 7. How do you handle joins/queries across shards? 🟡

- Generally **avoid them** by designing the shard key so that data needing to be joined/queried together lives on the **same** shard (e.g. shard an e-commerce database by `customer_id`, so a customer's orders are always co-located with their profile). When a cross-shard query is genuinely unavoidable, it must be handled at the application layer — querying each relevant shard separately and merging results — since the database itself has no native cross-shard join capability.

[↑ Back to top](#table-of-contents)

---

### 8. How does replication lag affect application design? 🟡

- If reads are served from a **replica** that hasn't yet caught up with the primary's latest writes, a user might not see their own just-made change (a "read-your-own-writes" inconsistency) — commonly worked around by routing reads immediately following a user's own write back to the **primary** for a short window, or reading from the primary for any operation where staleness would be confusing/incorrect.

[↑ Back to top](#table-of-contents)

---

### 9. How would you re-shard a database without downtime? 🔴

- A common approach: **dual-write** to both the old and new sharding scheme during a transition period, **backfill** historical data into the new scheme in the background, verify consistency between both, then **cut over** reads to the new scheme once confidently in sync, and finally stop writing to the old scheme. Doing this without downtime requires careful, incremental migration rather than a single big-bang switch.

[↑ Back to top](#table-of-contents)

---

### 10. What is a composite shard key, and why might you need one? 🔴

- A shard key made of **multiple** fields combined (e.g. `(tenant_id, user_id)`) rather than a single column — useful when a single field alone doesn't provide good distribution or doesn't align with how data is actually accessed together (e.g. sharding a multi-tenant system primarily by `tenant_id` alone could create huge, uneven shards for large tenants; combining with a secondary field can spread a large tenant's data more evenly while still keeping logically related data colocated).

[↑ Back to top](#table-of-contents)

---

### 11. How do you design a globally unique ID generator for a sharded system? 🔴

- A single auto-incrementing ID (like a traditional primary key) doesn't work once data is spread across independent shards — common approaches: **UUIDs** (simple, fully decentralized, but not naturally sortable by creation time and larger to index), or a **Snowflake-style** ID (encoding a timestamp + machine/shard ID + a per-machine counter into one integer) — giving roughly time-sortable, globally unique IDs generated independently by each node with no central coordination needed.

[↑ Back to top](#table-of-contents)

---

### 12. What's the tradeoff between normalization and denormalization in a system design context? 🔴

- **Normalized**: less data duplication, easier to keep consistent, but reads often require joins across multiple tables/services — expensive at scale, especially across shards (Q7).
- **Denormalized**: duplicates data to make the most common reads fast and join-free, at the cost of needing to keep duplicates in sync on every write. At scale, most systems lean toward **deliberate, targeted denormalization** for their hottest read paths, since join-heavy queries become impractical once data is sharded.

[↑ Back to top](#table-of-contents)

---

### 13. How would you design the data storage layer for a system with a billion users? 🔴

- Shard the primary data store by a well-distributed key (Q5), use read replicas to absorb read traffic within each shard, cache aggressively for hot read paths (see [Caching](caching.md)), denormalize data that's read far more often than it's written, and consider separating genuinely distinct data needs into purpose-built stores (a relational store for core account data, a wide-column/document store for activity logs, a search index for full-text search) rather than forcing every kind of data into one system.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might using multiple different types of databases (polyglot persistence) for different parts of the same system be the right call at this scale, rather than picking one database for everything?

[↑ Back to top](#table-of-contents)

---
