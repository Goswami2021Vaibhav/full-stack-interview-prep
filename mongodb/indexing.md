# Indexing

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is an index, and why does it speed up queries?](#1-what-is-an-index-and-why-does-it-speed-up-queries)
- [2. What index does MongoDB create automatically?](#2-what-index-does-mongodb-create-automatically)
- [3. How do you create an index?](#3-how-do-you-create-an-index)

**🟡 Medium**
- [4. What is a compound index?](#4-what-is-a-compound-index)
- [5. What is the order of fields in a compound index, and why does it matter?](#5-what-is-the-order-of-fields-in-a-compound-index-and-why-does-it-matter)
- [6. What is a multikey index?](#6-what-is-a-multikey-index)
- [7. What is a unique index?](#7-what-is-a-unique-index)
- [8. What is a TTL (Time-To-Live) index?](#8-what-is-a-ttl-time-to-live-index)
- [9. How do you check if a query is using an index?](#9-how-do-you-check-if-a-query-is-using-an-index)

**🔴 Hard**
- [10. What is the ESR (Equality, Sort, Range) rule for compound index field order?](#10-what-is-the-esr-equality-sort-range-rule-for-compound-index-field-order)
- [11. What is a covered query, and why is it especially fast?](#11-what-is-a-covered-query-and-why-is-it-especially-fast)
- [12. What is a text index, and what are its limitations?](#12-what-is-a-text-index-and-what-are-its-limitations)
- [13. What's the tradeoff of having too many indexes on a collection?](#13-whats-the-tradeoff-of-having-too-many-indexes-on-a-collection)
- [14. What is a partial index, and when would you use one?](#14-what-is-a-partial-index-and-when-would-you-use-one)

---

### 1. What is an index, and why does it speed up queries? 🟢

- A separate, ordered data structure (a B-tree) that lets MongoDB **locate** matching documents without scanning every document in the collection (a "full collection scan") — similar in concept to an index in a book letting you jump directly to a topic instead of reading every page.

[↑ Back to top](#table-of-contents)

---

### 2. What index does MongoDB create automatically? 🟢

- A unique index on `_id`, for every collection, automatically — guaranteeing fast lookups by `_id` and enforcing uniqueness on it.

[↑ Back to top](#table-of-contents)

---

### 3. How do you create an index? 🟢

```js
db.users.createIndex({ email: 1 }); // 1 = ascending, -1 = descending
```

[↑ Back to top](#table-of-contents)

---

### 4. What is a compound index? 🟡

- An index on **multiple fields together**, not just one — useful when queries commonly filter/sort by more than one field at once.

```js
db.orders.createIndex({ customerId: 1, createdAt: -1 });
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the order of fields in a compound index, and why does it matter? 🟡

- The index is built as a structure sorted by the **first** field, then the second within each value of the first, and so on — it can efficiently support queries on a **prefix** of that field order (e.g. just the first field, or the first two), but generally can't efficiently support a query that skips the first field and only filters on a later one.

```js
// Index on { a: 1, b: 1, c: 1 } efficiently supports:
{ a: 1 }, { a: 1, b: 1 }, { a: 1, b: 1, c: 1 }
// but NOT efficiently: { b: 1 } alone, or { c: 1 } alone
```

[↑ Back to top](#table-of-contents)

---

### 6. What is a multikey index? 🟡

- An index created on a field that holds an **array** — MongoDB automatically creates a separate index entry for **each element** of the array, allowing queries to efficiently match any individual array value.

```js
db.posts.createIndex({ tags: 1 }); // becomes multikey automatically since `tags` is an array field
```

[↑ Back to top](#table-of-contents)

---

### 7. What is a unique index? 🟡

- Enforces that no two documents in the collection can have the same value for the indexed field(s) — an insert/update violating this throws a duplicate-key error.

```js
db.users.createIndex({ email: 1 }, { unique: true });
```

[↑ Back to top](#table-of-contents)

---

### 8. What is a TTL (Time-To-Live) index? 🟡

- Automatically **deletes** documents once a certain amount of time has passed since a date field's value — used for data that should naturally expire (session tokens, temporary logs, verification codes), without needing a manual cleanup job.

```js
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 }); // auto-deleted after 1 hour
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you check if a query is using an index? 🟡

- Run `.explain()` on the query and inspect the `winningPlan` — look for `IXSCAN` (index scan, good) vs. `COLLSCAN` (full collection scan, usually means a missing/unused index).

```js
db.users.find({ email: 'v@x.com' }).explain('executionStats');
```

[↑ Back to top](#table-of-contents)

---

### 10. What is the ESR (Equality, Sort, Range) rule for compound index field order? 🔴

- A guideline for ordering fields in a compound index: put fields used for **E**quality matches first, then fields used for **S**orting, then fields used for **R**ange queries last. This ordering lets MongoDB narrow down to the smallest possible set of index entries via equality first, use the index's natural order to satisfy the sort without an extra in-memory sort step, and only then scan a range — maximizing how much of the work the index itself can do.

```js
// Query: find status='active', sort by createdAt, filter age between 18-65
// Index following ESR: { status: 1, createdAt: 1, age: 1 }
```

[↑ Back to top](#table-of-contents)

---

### 11. What is a covered query, and why is it especially fast? 🔴

- A query where **every** field it needs (both in the filter and the projection) is present in the index itself — MongoDB can return results directly from the index structure without ever touching the actual documents on disk, drastically reducing I/O.

```js
db.users.createIndex({ email: 1, name: 1 });
db.users.find({ email: 'v@x.com' }, { name: 1, email: 1, _id: 0 }); // covered — never reads the actual document
```

[↑ Back to top](#table-of-contents)

---

### 12. What is a text index, and what are its limitations? 🔴

- Supports basic full-text search across string fields (tokenizing and stemming words) — but only **one** text index is allowed per collection (across however many fields you include in it), it doesn't support relevance ranking as sophisticated as a dedicated search engine, and for serious full-text search needs at scale, a dedicated tool (Elasticsearch, Atlas Search) is generally preferred over MongoDB's built-in text index.

[↑ Back to top](#table-of-contents)

---

### 13. What's the tradeoff of having too many indexes on a collection? 🔴

- Every index **speeds up reads** that use it, but **slows down writes** — each insert/update/delete must also update every index on the collection, and indexes consume additional RAM/disk. Over-indexing (especially with many overlapping or rarely-used indexes) can meaningfully degrade write throughput for marginal or no read benefit; index choices should be driven by actual query patterns, not added speculatively.

[↑ Back to top](#table-of-contents)

---

### 14. What is a partial index, and when would you use one? 🔴

- An index that only includes documents matching a specified **filter expression**, rather than every document in the collection — smaller and cheaper to maintain than a full index, useful when queries (and the index supporting them) only ever care about a subset of documents (e.g. only `active` orders, ignoring the much larger set of archived ones).

```js
db.orders.createIndex({ customerId: 1 }, { partialFilterExpression: { status: 'active' } });
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How does a partial index differ from a sparse index, and when would you reach for one over the other?

[↑ Back to top](#table-of-contents)

---
