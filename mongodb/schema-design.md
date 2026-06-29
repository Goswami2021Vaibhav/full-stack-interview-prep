# Schema Design

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is "schema design" in a schema-less database?](#1-what-is-schema-design-in-a-schema-less-database)
- [2. What is the core principle behind designing a MongoDB schema?](#2-what-is-the-core-principle-behind-designing-a-mongodb-schema)
- [3. When should you embed data vs. reference it?](#3-when-should-you-embed-data-vs-reference-it)

**🟡 Medium**
- [4. What is the "one-to-few" relationship pattern?](#4-what-is-the-one-to-few-relationship-pattern)
- [5. What is the "one-to-many" relationship pattern?](#5-what-is-the-one-to-many-relationship-pattern)
- [6. What is the "one-to-squillions" (or "one-to-zillions") pattern?](#6-what-is-the-one-to-squillions-or-one-to-zillions-pattern)
- [7. How do you model a many-to-many relationship in MongoDB?](#7-how-do-you-model-a-many-to-many-relationship-in-mongodb)
- [8. What is schema validation, and how do you enforce it?](#8-what-is-schema-validation-and-how-do-you-enforce-it)

**🔴 Hard**
- [9. What is the "unbounded array" anti-pattern, and why is it dangerous?](#9-what-is-the-unbounded-array-anti-pattern-and-why-is-it-dangerous)
- [10. What is the Subset Pattern?](#10-what-is-the-subset-pattern)
- [11. What is the Extended Reference Pattern?](#11-what-is-the-extended-reference-pattern)
- [12. What is the Bucket Pattern, and when is it useful?](#12-what-is-the-bucket-pattern-and-when-is-it-useful)
- [13. What is the Computed Pattern?](#13-what-is-the-computed-pattern)
- [14. How do you handle schema evolution/migrations in a flexible-schema database?](#14-how-do-you-handle-schema-evolutionmigrations-in-a-flexible-schema-database)

---

### 1. What is "schema design" in a schema-less database? 🟢

- Even though MongoDB doesn't enforce a rigid structure, you still have to decide **how to shape your documents** — what to embed, what to reference, how to structure arrays — and that decision has a real, lasting impact on query performance and complexity. "Schema-less" means flexible storage, not "no design needed."

[↑ Back to top](#table-of-contents)

---

### 2. What is the core principle behind designing a MongoDB schema? 🟢

- **Design for your queries, not for your data's "natural" relational shape.** Unlike relational normalization (which optimizes for avoiding data duplication), MongoDB schema design optimizes for how the application will actually **read** the data — shaping documents so the most common queries are fast and require minimal joins (`$lookup`).

[↑ Back to top](#table-of-contents)

---

### 3. When should you embed data vs. reference it? 🟢

- **Embed** when the related data is usually read **together** with the parent, is bounded in size, and doesn't need to be queried independently very often.
- **Reference** when the related data is large/unbounded, needs to be queried independently, or is shared across many parent documents (duplicating it everywhere would be wasteful and hard to keep in sync).

[↑ Back to top](#table-of-contents)

---

### 4. What is the "one-to-few" relationship pattern? 🟡

- When a document has a **small, bounded** number of related items (a person's few addresses, a few phone numbers) — embed them directly as an array within the parent document, since they're small and almost always accessed together.

```js
{ name: 'Vaibhav', addresses: [{ city: 'Mumbai' }, { city: 'Pune' }] }
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the "one-to-many" relationship pattern? 🟡

- When the related items number in the dozens/hundreds and **could** grow further (a customer's orders) — typically reference them instead, storing the parent's `_id` on each child document, and query the children separately when needed rather than embedding a growing list in the parent.

```js
// orders collection, each referencing its customer
{ _id: ObjectId(), customerId: ObjectId('...'), total: 250 }
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the "one-to-squillions" (or "one-to-zillions") pattern? 🟡

- When the related items are **unbounded and potentially huge** (every log entry for a server, every sensor reading for a device) — store a reference to the **parent** on each child document (the inverse of one-to-many) so the parent document never grows, and query children by that reference, often combined with pagination.

```js
// sensor readings, NOT embedded in the device document
{ deviceId: ObjectId('...'), value: 23.5, timestamp: ISODate() }
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you model a many-to-many relationship in MongoDB? 🟡

- Commonly via an array of references on **both** sides (e.g. a `studentIds` array on courses, a `courseIds` array on students), or via a separate "join" collection if the relationship itself carries its own data (e.g. an enrollment date) — similar in spirit to a junction table in a relational schema.

[↑ Back to top](#table-of-contents)

---

### 8. What is schema validation, and how do you enforce it? 🟡

- MongoDB supports **optional**, declarative validation rules (via JSON Schema) applied at the collection level — rejecting writes that don't conform, giving you some of the structural guarantees of a relational schema while keeping flexibility for fields not covered by the rules.

```js
db.createCollection('users', {
  validator: {
    $jsonSchema: {
      required: ['email'],
      properties: { email: { bsonType: 'string' } },
    },
  },
});
```

[↑ Back to top](#table-of-contents)

---

### 9. What is the "unbounded array" anti-pattern, and why is it dangerous? 🔴

- Embedding an array that grows **without limit** (e.g. every comment ever made on a post, embedded inside the post document) — the document keeps growing toward the 16MB limit, write performance degrades as WiredTiger has to rewrite the whole document on every array update, and most queries don't actually need the *entire* array every time anyway. Fix by referencing instead, or applying the Bucket Pattern (Q12) / Subset Pattern (Q10).

[↑ Back to top](#table-of-contents)

---

### 10. What is the Subset Pattern? 🔴

- Embed only the **most relevant/recent subset** of a large related collection directly in the parent (e.g. a product's 10 most recent reviews), and keep the **full** set in a separate referenced collection — gives fast access to the commonly-needed subset without the unbounded-growth problem (Q9) of embedding everything.

[↑ Back to top](#table-of-contents)

---

### 11. What is the Extended Reference Pattern? 🔴

- Instead of storing just an `_id` reference to a related document, **duplicate a few frequently-accessed fields** from it directly alongside the reference — e.g. an order document stores the customer's `_id` **plus** their `name` and `email`, so common reads avoid a join entirely, accepting a small amount of duplication (and the need to update those duplicated fields if the source changes) in exchange for read performance.

```js
{ orderId: '...', customer: { _id: ObjectId('...'), name: 'Vaibhav', email: 'v@x.com' } }
```

[↑ Back to top](#table-of-contents)

---

### 12. What is the Bucket Pattern, and when is it useful? 🔴

- Groups multiple related, time-ordered data points into a single document **bucket** (e.g. one document per sensor per hour, containing an array of that hour's readings) rather than one document per individual reading — reduces the total number of documents (and associated per-document overhead/index entries) for high-volume time-series-style data, while keeping each bucket's array bounded (e.g. capped at one hour's worth) to avoid the unbounded-array problem.

```js
{ deviceId: '...', hour: ISODate('2026-06-29T10:00:00Z'), readings: [{ t: '10:00', v: 23.1 }, { t: '10:01', v: 23.3 }] }
```

[↑ Back to top](#table-of-contents)

---

### 13. What is the Computed Pattern? 🔴

- Precompute and store a derived/aggregated value (a running total, a view count, an average rating) directly on a document **at write time**, rather than recalculating it from scratch on every single read — trades a small amount of write-time cost for significantly faster, cheaper reads on values that are read far more often than the underlying data changes.

```js
// Pre-stored, updated incrementally on each new review, instead of averaging on every product page view
{ productId: '...', avgRating: 4.6, reviewCount: 128 }
```

[↑ Back to top](#table-of-contents)

---

### 14. How do you handle schema evolution/migrations in a flexible-schema database? 🔴

- Since there's no enforced schema to "alter" like a relational `ALTER TABLE`, the common approaches are: write application code that tolerates **both** old and new document shapes during a transition (checking for a field's presence, providing defaults for missing ones), or run a **background migration script** that updates existing documents to the new shape over time — often paired with a `schemaVersion` field on each document to track which shape it's currently in.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is "just add the new field to all documents in one big migration" sometimes impractical for a very large, high-traffic collection?

[↑ Back to top](#table-of-contents)

---
