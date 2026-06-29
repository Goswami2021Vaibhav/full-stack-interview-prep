# Transactions

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Does MongoDB support transactions?](#1-does-mongodb-support-transactions)
- [2. What is atomicity at the single-document level in MongoDB?](#2-what-is-atomicity-at-the-single-document-level-in-mongodb)
- [3. What is ACID, and which of its guarantees does MongoDB provide?](#3-what-is-acid-and-which-of-its-guarantees-does-mongodb-provide)

**🟡 Medium**
- [4. How do you use a multi-document transaction in MongoDB?](#4-how-do-you-use-a-multi-document-transaction-in-mongodb)
- [5. Why did MongoDB historically rely so heavily on single-document atomicity instead of multi-document transactions?](#5-why-did-mongodb-historically-rely-so-heavily-on-single-document-atomicity-instead-of-multi-document-transactions)
- [6. What atomic update operators can substitute for a transaction in simple cases?](#6-what-atomic-update-operators-can-substitute-for-a-transaction-in-simple-cases)
- [7. What happens if part of a transaction fails?](#7-what-happens-if-part-of-a-transaction-fails)

**🔴 Hard**
- [8. What is a write conflict during a transaction, and how does MongoDB handle it?](#8-what-is-a-write-conflict-during-a-transaction-and-how-does-mongodb-handle-it)
- [9. What's the performance cost of using multi-document transactions?](#9-whats-the-performance-cost-of-using-multi-document-transactions)
- [10. How do transactions interact with sharded clusters?](#10-how-do-transactions-interact-with-sharded-clusters)
- [11. When should you redesign your schema instead of reaching for a multi-document transaction?](#11-when-should-you-redesign-your-schema-instead-of-reaching-for-a-multi-document-transaction)
- [12. What is the default transaction timeout, and why does one exist?](#12-what-is-the-default-transaction-timeout-and-why-does-one-exist)

---

### 1. Does MongoDB support transactions? 🟢

- Yes — **multi-document ACID transactions** have been supported since MongoDB 4.0 (within a replica set) and 4.2 (across sharded clusters). Before that, MongoDB only guaranteed atomicity at the **single-document** level.

[↑ Back to top](#table-of-contents)

---

### 2. What is atomicity at the single-document level in MongoDB? 🟢

- Any update to a **single** document (even one touching multiple nested fields/array elements within it) is always applied **entirely or not at all** — this has been true since MongoDB's earliest versions, regardless of multi-document transaction support.

[↑ Back to top](#table-of-contents)

---

### 3. What is ACID, and which of its guarantees does MongoDB provide? 🟢

- **A**tomicity, **C**onsistency, **I**solation, **D**urability. MongoDB provides full ACID guarantees for single-document operations always, and for **multi-document** operations specifically when wrapped in an explicit transaction (Q1) — outside a transaction, multiple separate writes have no atomicity guarantee across each other.

[↑ Back to top](#table-of-contents)

---

### 4. How do you use a multi-document transaction in MongoDB? 🟡

```js
const session = client.startSession();
try {
  session.startTransaction();
  await accounts.updateOne({ _id: fromId }, { $inc: { balance: -100 } }, { session });
  await accounts.updateOne({ _id: toId }, { $inc: { balance: 100 } }, { session });
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
} finally {
  session.endSession();
}
```

[↑ Back to top](#table-of-contents)

---

### 5. Why did MongoDB historically rely so heavily on single-document atomicity instead of multi-document transactions? 🟡

- Because good MongoDB schema design **embeds** related data that needs to change together into a **single** document (see [Schema Design](schema-design.md)) — if data that must stay consistent together lives in one document, single-document atomicity alone is often enough, without ever needing a multi-document transaction at all. Multi-document transactions exist for the cases where this isn't practical, not as the default way of working.

[↑ Back to top](#table-of-contents)

---

### 6. What atomic update operators can substitute for a transaction in simple cases? 🟡

- `$inc` (atomic increment/decrement), `findOneAndUpdate()` (atomic read-modify-write on one document), and `$set`/array operators combined in a single `updateOne()` call — all atomic on their own **single document**, often sufficient instead of reaching for a full transaction.

```js
// Atomic decrement with a guard condition — no transaction needed
db.inventory.updateOne({ _id: id, stock: { $gte: 1 } }, { $inc: { stock: -1 } });
```

[↑ Back to top](#table-of-contents)

---

### 7. What happens if part of a transaction fails? 🟡

- The **entire** transaction is rolled back — none of its writes are applied, even ones that "succeeded" earlier within the same transaction before the failure occurred. The application must explicitly call `abortTransaction()` (or the driver does so automatically on certain errors) to formally end it.

[↑ Back to top](#table-of-contents)

---

### 8. What is a write conflict during a transaction, and how does MongoDB handle it? 🔴

- Occurs when a transaction tries to modify a document that **another concurrent operation** (inside or outside a transaction) has modified since the transaction started. MongoDB detects this and **aborts** the transaction with a write-conflict error — the application is expected to **retry** the entire transaction from the start, since MongoDB doesn't automatically merge or retry transactions for you.

[↑ Back to top](#table-of-contents)

---

### 9. What's the performance cost of using multi-document transactions? 🔴

- Transactions hold resources (locks, snapshot state) for their **entire duration**, and span at least one network round trip per operation plus the commit — meaningfully more overhead than equivalent single-document atomic operations. Long-running transactions also increase the chance of write conflicts (Q8) with other concurrent operations, since the snapshot stays open longer. They should be used deliberately, for genuinely multi-document needs, not as a default tool.

[↑ Back to top](#table-of-contents)

---

### 10. How do transactions interact with sharded clusters? 🔴

- Supported since MongoDB 4.2, but a transaction spanning **multiple shards** is more expensive than one confined to a single shard — it requires additional cross-shard coordination (two-phase commit-like behavior) to ensure atomicity across machines that don't share any common state. Where possible, schema/shard-key design that keeps related documents needing transactional consistency together on the **same shard** avoids this added cost.

[↑ Back to top](#table-of-contents)

---

### 11. When should you redesign your schema instead of reaching for a multi-document transaction? 🔴

- If you find yourself **frequently** needing a transaction to keep two pieces of data consistent, that's often a signal those two pieces of data should have been **embedded together** in a single document in the first place (back to Q5) — transactions are meant for genuinely cross-entity operations (e.g. a funds transfer between two independent accounts), not as a patch over a schema that should have grouped related data together from the start.

[↑ Back to top](#table-of-contents)

---

### 12. What is the default transaction timeout, and why does one exist? 🔴

- MongoDB transactions have a default execution time limit (`transactionLifetimeLimitSeconds`, default 60 seconds) — without a limit, a long-running or stalled transaction would hold locks/snapshot resources indefinitely, blocking other operations and consuming server resources; the timeout forces it to abort and release those resources if it runs too long.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does a transaction holding a snapshot open for a long time increase the likelihood of write conflicts with other operations, compared to a very short transaction?

[↑ Back to top](#table-of-contents)

---
