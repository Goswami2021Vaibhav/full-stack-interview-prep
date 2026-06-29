# Replication & Sharding

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is replication in MongoDB?](#1-what-is-replication-in-mongodb)
- [2. What is a replica set?](#2-what-is-a-replica-set)
- [3. What is sharding, and why is it needed?](#3-what-is-sharding-and-why-is-it-needed)

**🟡 Medium**
- [4. What's the difference between a primary and a secondary in a replica set?](#4-whats-the-difference-between-a-primary-and-a-secondary-in-a-replica-set)
- [5. How does failover work in a replica set?](#5-how-does-failover-work-in-a-replica-set)
- [6. What is a shard key, and why is choosing it carefully important?](#6-what-is-a-shard-key-and-why-is-choosing-it-carefully-important)
- [7. What is a `mongos` router?](#7-what-is-a-mongos-router)
- [8. What is a config server in a sharded cluster?](#8-what-is-a-config-server-in-a-sharded-cluster)

**🔴 Hard**
- [9. What is read preference, and what options does it support?](#9-what-is-read-preference-and-what-options-does-it-support)
- [10. What is write concern, and how does it relate to durability?](#10-what-is-write-concern-and-how-does-it-relate-to-durability)
- [11. What is chunk migration, and why can it cause performance issues?](#11-what-is-chunk-migration-and-why-can-it-cause-performance-issues)
- [12. What is a "hot shard," and what causes it?](#12-what-is-a-hot-shard-and-what-causes-it)
- [13. What is hashed sharding, and how does it differ from ranged sharding?](#13-what-is-hashed-sharding-and-how-does-it-differ-from-ranged-sharding)
- [14. How do you choose between scaling vertically, adding more replica set secondaries, or sharding?](#14-how-do-you-choose-between-scaling-vertically-adding-more-replica-set-secondaries-or-sharding)

---

### 1. What is replication in MongoDB? 🟢

- Maintaining **multiple copies** of the same data across different servers, so the database stays available (and data isn't lost) if one server fails — also enables distributing read load across copies.

[↑ Back to top](#table-of-contents)

---

### 2. What is a replica set? 🟢

- A group of MongoDB servers (`mongod` instances) holding the same data — one **primary** that accepts all writes, and one or more **secondaries** that replicate from it and can serve reads.

[↑ Back to top](#table-of-contents)

---

### 3. What is sharding, and why is it needed? 🟢

- **Horizontal scaling**: splitting a collection's data across multiple servers (**shards**), each holding only a **portion** of the total data — needed once a dataset/workload grows beyond what a single server (even a powerful one) can handle, whether due to storage size or write throughput.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between a primary and a secondary in a replica set? 🟡

- **Primary**: the only member that accepts **write** operations.
- **Secondary**: continuously replicates the primary's data (by tailing its oplog, see [Core Concepts](core-concepts.md#15-what-is-the-oplog-and-what-role-does-it-play-internally)) and can optionally serve **read** operations, but never accepts writes directly.

[↑ Back to top](#table-of-contents)

---

### 5. How does failover work in a replica set? 🟡

- If the primary becomes unreachable, the remaining members hold an **election** and promote one secondary to become the new primary automatically — minimizing downtime without manual intervention, though there's a brief window where the replica set has no primary and can't accept writes.

[↑ Back to top](#table-of-contents)

---

### 6. What is a shard key, and why is choosing it carefully important? 🟡

- The field (or fields) MongoDB uses to determine **which shard** a given document lives on. A poorly chosen shard key (e.g. low cardinality, or one that concentrates writes on a narrow range of values) leads to uneven data distribution and hot shards (Q12) — and the shard key is **difficult to change** after the fact, making this one of the most consequential early decisions in a sharded deployment.

[↑ Back to top](#table-of-contents)

---

### 7. What is a `mongos` router? 🟡

- The query router applications actually connect to in a sharded cluster — it doesn't store any data itself, but figures out **which shard(s)** a given query needs to hit and routes/merges the results accordingly, presenting a single unified interface so the application doesn't need to know about sharding at all.

[↑ Back to top](#table-of-contents)

---

### 8. What is a config server in a sharded cluster? 🟡

- Stores the cluster's **metadata** — which shard owns which range of data (chunks), cluster configuration — `mongos` routers consult config servers to know how to route each query correctly.

[↑ Back to top](#table-of-contents)

---

### 9. What is read preference, and what options does it support? 🔴

- Controls **which replica set members** a read operation is allowed to be routed to. Common modes: `primary` (default — always the most current data), `secondary`/`secondaryPreferred` (offload reads to secondaries, accepting potentially slightly stale data), `nearest` (lowest network latency, regardless of role).

```js
db.collection.find().readPref('secondaryPreferred');
```

[↑ Back to top](#table-of-contents)

---

### 10. What is write concern, and how does it relate to durability? 🔴

- Specifies how many replica set members must **acknowledge** a write before it's considered successful. `w: 1` (just the primary — fast, but a primary failure right after could lose the write before it replicates), `w: 'majority'` (a majority of members — much safer, the write survives even if the primary fails immediately after, at the cost of slightly higher write latency).

```js
db.orders.insertOne({ ... }, { writeConcern: { w: 'majority' } });
```

[↑ Back to top](#table-of-contents)

---

### 11. What is chunk migration, and why can it cause performance issues? 🔴

- Sharded collections are divided into **chunks** (contiguous ranges of shard key values); as data grows unevenly, the balancer **migrates** chunks between shards to keep the distribution roughly even. Migration itself consumes CPU/network/disk I/O on the shards involved, and can transiently impact query performance on the affected shards while it's in progress — a reason to schedule/monitor balancing carefully on production clusters under heavy load.

[↑ Back to top](#table-of-contents)

---

### 12. What is a "hot shard," and what causes it? 🔴

- A shard that receives a disproportionate share of traffic/data compared to the others, typically caused by a poorly chosen shard key — e.g. a monotonically increasing field (a timestamp, an auto-incrementing ID) as the shard key concentrates **all new writes** onto whichever single shard currently owns the highest range, since new values always fall at the "end."

[↑ Back to top](#table-of-contents)

---

### 13. What is hashed sharding, and how does it differ from ranged sharding? 🔴

- **Ranged sharding**: chunks are contiguous ranges of the actual shard key values — efficient for range queries on that key, but prone to hot shards if writes cluster around one end of the range (Q12).
- **Hashed sharding**: the shard key's value is hashed before determining chunk placement — distributes writes much more evenly (since hashes scatter even sequential input values randomly across the range), at the cost of making efficient **range** queries on that field harder (a range of original values maps to scattered, non-contiguous hash values).

[↑ Back to top](#table-of-contents)

---

### 14. How do you choose between scaling vertically, adding more replica set secondaries, or sharding? 🔴

- **Vertical scaling** (bigger server): simplest, no architecture change — sufficient until you hit a single machine's practical ceiling.
- **More secondaries**: helps when the bottleneck is **read** throughput — doesn't help with write throughput or total storage capacity, since all writes still go through one primary.
- **Sharding**: needed when **write** throughput or total **data size** itself exceeds what one primary/machine can handle — the most operationally complex option, reserved for when the simpler approaches genuinely aren't enough.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why doesn't adding more secondaries to a replica set help at all with write-heavy workloads?

[↑ Back to top](#table-of-contents)

---
