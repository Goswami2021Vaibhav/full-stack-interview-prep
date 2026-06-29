# Core Concepts

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is MongoDB, and how is it different from a relational database?](#1-what-is-mongodb-and-how-is-it-different-from-a-relational-database)
- [2. What is a document, and what is a collection?](#2-what-is-a-document-and-what-is-a-collection)
- [3. What is BSON, and how does it differ from JSON?](#3-what-is-bson-and-how-does-it-differ-from-json)
- [4. What is the `_id` field?](#4-what-is-the-_id-field)

**🟡 Medium**
- [5. What does "schema-less" actually mean in MongoDB?](#5-what-does-schema-less-actually-mean-in-mongodb)
- [6. What is `ObjectId`, and what information does it encode?](#6-what-is-objectid-and-what-information-does-it-encode)
- [7. What's the difference between embedding and referencing documents?](#7-whats-the-difference-between-embedding-and-referencing-documents)
- [8. What is the MongoDB document size limit, and why does it exist?](#8-what-is-the-mongodb-document-size-limit-and-why-does-it-exist)
- [9. What is the `mongosh`/MongoDB shell used for?](#9-what-is-the-mongoshmongodb-shell-used-for)

**🔴 Hard**
- [10. What is WiredTiger, and what role does it play in MongoDB?](#10-what-is-wiredtiger-and-what-role-does-it-play-in-mongodb)
- [11. What's the difference between SQL and NoSQL databases, and when would you choose one over the other?](#11-whats-the-difference-between-sql-and-nosql-databases-and-when-would-you-choose-one-over-the-other)
- [12. What is CAP theorem, and where does MongoDB fall within it?](#12-what-is-cap-theorem-and-where-does-mongodb-fall-within-it)
- [13. What's the difference between strong consistency and eventual consistency in MongoDB's read behavior?](#13-whats-the-difference-between-strong-consistency-and-eventual-consistency-in-mongodbs-read-behavior)
- [14. What is a capped collection, and when would you use one?](#14-what-is-a-capped-collection-and-when-would-you-use-one)
- [15. What is the oplog, and what role does it play internally?](#15-what-is-the-oplog-and-what-role-does-it-play-internally)

---

### 1. What is MongoDB, and how is it different from a relational database? 🟢

- A **document-oriented NoSQL** database — stores data as flexible, JSON-like documents (BSON) grouped into collections, instead of rows in fixed-schema tables. Related data is commonly **embedded** within a single document rather than normalized across multiple joined tables, which fits naturally with how most application objects are already structured in code.

[↑ Back to top](#table-of-contents)

---

### 2. What is a document, and what is a collection? 🟢

- A **document** is a single record, stored as a BSON object with key-value fields (roughly analogous to a row).
- A **collection** is a group of documents (roughly analogous to a table) — but unlike a table, documents within the same collection don't need to share an identical structure.

```js
// A document inside the "users" collection
{ _id: ObjectId("..."), name: "Vaibhav", email: "v@x.com", roles: ["admin"] }
```

[↑ Back to top](#table-of-contents)

---

### 3. What is BSON, and how does it differ from JSON? 🟢

- **B**inary **JSON** — the binary-encoded format MongoDB actually stores documents in. Compared to plain JSON text, BSON adds extra types JSON lacks natively (`Date`, `ObjectId`, `Binary`, distinct integer/float types), and is faster to parse/traverse since it's binary rather than text that needs parsing.

[↑ Back to top](#table-of-contents)

---

### 4. What is the `_id` field? 🟢

- Every document's **primary key**, unique within its collection — automatically generated as an `ObjectId` if not explicitly provided, and automatically indexed.

[↑ Back to top](#table-of-contents)

---

### 5. What does "schema-less" actually mean in MongoDB? 🟡

- MongoDB doesn't **enforce** a rigid structure at the database engine level by default — different documents in the same collection can have different fields/types. In practice, applications still maintain an **implicit schema** in their code (and often an explicit one via [Schema Validation](schema-design.md)/an ORM like Mongoose) — "schema-less" describes flexibility at the storage layer, not an absence of structure in how the application actually uses the data.

[↑ Back to top](#table-of-contents)

---

### 6. What is `ObjectId`, and what information does it encode? 🟡

- A 12-byte identifier: a 4-byte timestamp (seconds since epoch), a 5-byte random value (unique per machine+process), and a 3-byte incrementing counter — this structure makes `ObjectId`s **roughly sortable by creation time** and generatable client-side without coordinating with the server.

```js
ObjectId("507f1f77bcf86cd799439011")
// 507f1f77 -> timestamp | bcf86cd79943 -> random | 9011 -> counter
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between embedding and referencing documents? 🟡

- **Embedding**: nest related data directly inside the parent document — fast reads (one query gets everything), but can lead to data duplication and unbounded document growth if the embedded array grows without limit.
- **Referencing**: store just an `_id` reference to a document in another collection (similar to a foreign key) — avoids duplication, but requires a separate query (or `$lookup`) to join the data. See [Schema Design](schema-design.md) for the deeper tradeoff and decision criteria.

[↑ Back to top](#table-of-contents)

---

### 8. What is the MongoDB document size limit, and why does it exist? 🟡

- **16MB** per document — chosen to keep documents efficient to transmit over the network and hold in memory/RAM during processing; it also discourages unbounded embedding of ever-growing arrays (e.g. embedding every comment ever made on a post) within a single document, nudging schema design toward referencing for that kind of unbounded data.

[↑ Back to top](#table-of-contents)

---

### 9. What is the `mongosh`/MongoDB shell used for? 🟡

- An interactive JavaScript-based shell for connecting to a MongoDB instance directly — running ad-hoc queries, administering the database (creating indexes, users), and scripting operations, similar in spirit to `psql` for PostgreSQL or `mysql` for MySQL.

[↑ Back to top](#table-of-contents)

---

### 10. What is WiredTiger, and what role does it play in MongoDB? 🔴

- MongoDB's default **storage engine** since version 3.2 — responsible for how data is actually written to and read from disk, including document-level locking (rather than collection/database-level locks used by older engines), compression, and a checkpoint-based durability mechanism. Most performance/concurrency characteristics discussed for modern MongoDB are really WiredTiger's behavior under the hood.

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between SQL and NoSQL databases, and when would you choose one over the other? 🔴

- **SQL (relational)**: fixed schema, strong relational integrity (foreign keys, joins), mature multi-row ACID transactions — best when data is highly structured, relationships are complex/important, and consistency is critical (financial records).
- **NoSQL (e.g. MongoDB)**: flexible schema, naturally horizontally scalable, documents map closely to how application objects already look — best when the schema evolves frequently, data is naturally document-shaped (a user profile, a product catalog), or you need to scale writes across many machines easily.

[↑ Back to top](#table-of-contents)

---

### 12. What is CAP theorem, and where does MongoDB fall within it? 🔴

- **C**onsistency, **A**vailability, **P**artition tolerance — a distributed system can only fully guarantee **two** of the three during a network partition. MongoDB (via replica sets) is generally tuned toward **CP** by default (a replica set with no reachable primary becomes unavailable for writes rather than risking inconsistent data), though read/write concern settings let you tune this tradeoff per-operation toward more availability if desired.

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between strong consistency and eventual consistency in MongoDB's read behavior? 🔴

- **Strong consistency**: reading from the **primary** always reflects the latest acknowledged write.
- **Eventual consistency**: reading from a **secondary** (common for read scaling) may return slightly stale data, since replication to secondaries isn't instantaneous — the secondary will *eventually* catch up, but isn't guaranteed to be current at the moment of read. Controlled via `readPreference` and `readConcern` settings.

[↑ Back to top](#table-of-contents)

---

### 14. What is a capped collection, and when would you use one? 🔴

- A fixed-size collection that automatically **overwrites its oldest documents** once it reaches its size limit, maintaining insertion order — useful for high-throughput logging or caching use cases where you only care about the most recent N entries and want old data to age out automatically without manual cleanup.

[↑ Back to top](#table-of-contents)

---

### 15. What is the oplog, and what role does it play internally? 🔴

- The **operations log** — a special capped collection that records every write operation applied to the primary, in order. Secondaries continuously **tail** the oplog and replay those operations to stay in sync (the actual mechanism behind replication — see [Replication & Sharding](replication-and-sharding.md)), and it's also what change streams are built on top of.

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens to replication if a secondary falls behind so far that the operations it needs have already been overwritten in the oplog (since it's capped)?

[↑ Back to top](#table-of-contents)

---
