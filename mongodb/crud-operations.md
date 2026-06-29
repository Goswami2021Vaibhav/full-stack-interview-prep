# CRUD Operations

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you insert a document into a collection?](#1-how-do-you-insert-a-document-into-a-collection)
- [2. How do you query documents in MongoDB?](#2-how-do-you-query-documents-in-mongodb)
- [3. How do you update a document?](#3-how-do-you-update-a-document)
- [4. How do you delete documents?](#4-how-do-you-delete-documents)

**🟡 Medium**
- [5. What's the difference between `find()` and `findOne()`?](#5-whats-the-difference-between-find-and-findone)
- [6. What's the difference between `updateOne()`, `updateMany()`, and `replaceOne()`?](#6-whats-the-difference-between-updateone-updatemany-and-replaceone)
- [7. What are common query operators?](#7-what-are-common-query-operators)
- [8. What does the `upsert` option do?](#8-what-does-the-upsert-option-do)
- [9. How do you project (select) only specific fields from a query?](#9-how-do-you-project-select-only-specific-fields-from-a-query)
- [10. What are common update operators?](#10-what-are-common-update-operators)

**🔴 Hard**
- [11. What's the difference between `deleteOne()`, `deleteMany()`, and `drop()`?](#11-whats-the-difference-between-deleteone-deletemany-and-drop)
- [12. How do you query inside arrays and nested documents?](#12-how-do-you-query-inside-arrays-and-nested-documents)
- [13. What is `bulkWrite()`, and why would you use it?](#13-what-is-bulkwrite-and-why-would-you-use-it)
- [14. What's the difference between `findOneAndUpdate()` and `updateOne()`?](#14-whats-the-difference-between-findoneandupdate-and-updateone)
- [15. How do cursors work, and why does it matter for large result sets?](#15-how-do-cursors-work-and-why-does-it-matter-for-large-result-sets)

---

### 1. How do you insert a document into a collection? 🟢

```js
db.users.insertOne({ name: 'Vaibhav', email: 'v@x.com' });
db.users.insertMany([{ name: 'A' }, { name: 'B' }]);
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you query documents in MongoDB? 🟢

```js
db.users.find({ status: 'active' });          // all matching documents
db.users.find({ age: { $gt: 18 } });            // with a query operator
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you update a document? 🟢

```js
db.users.updateOne({ _id: id }, { $set: { status: 'active' } });
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you delete documents? 🟢

```js
db.users.deleteOne({ _id: id });
db.users.deleteMany({ status: 'inactive' });
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `find()` and `findOne()`? 🟡

- `find()`: returns a **cursor** over all matching documents (lazily iterated, see Q15).
- `findOne()`: returns a **single** document directly (the first match), or `null` if none found — no cursor handling needed by the caller.

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between `updateOne()`, `updateMany()`, and `replaceOne()`? 🟡

- `updateOne()`: applies update operators (`$set`, etc.) to the **first** matching document.
- `updateMany()`: applies them to **all** matching documents.
- `replaceOne()`: replaces the **entire** matched document with a new one (except `_id`) — like `PUT` vs. `PATCH` semantics in REST.

[↑ Back to top](#table-of-contents)

---

### 7. What are common query operators? 🟡

- Comparison: `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`.
- Logical: `$and`, `$or`, `$not`, `$nor`.
- Array: `$in`, `$nin`, `$all`, `$elemMatch`.
- Existence/type: `$exists`, `$type`.

```js
db.products.find({ price: { $gte: 100, $lte: 500 }, category: { $in: ['electronics', 'books'] } });
```

[↑ Back to top](#table-of-contents)

---

### 8. What does the `upsert` option do? 🟡

- If no document matches the filter, **insert** a new one using the filter + update data combined, instead of doing nothing — useful for "create if missing, otherwise update" logic in a single atomic operation.

```js
db.users.updateOne({ email: 'v@x.com' }, { $set: { lastLogin: new Date() } }, { upsert: true });
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you project (select) only specific fields from a query? 🟡

```js
db.users.find({ status: 'active' }, { name: 1, email: 1, _id: 0 }); // include name/email, exclude _id
```

[↑ Back to top](#table-of-contents)

---

### 10. What are common update operators? 🟡

- `$set` (set a field's value), `$unset` (remove a field), `$inc` (increment a number), `$push`/`$pull` (add/remove from an array), `$addToSet` (add to an array only if not already present).

```js
db.users.updateOne({ _id: id }, { $inc: { loginCount: 1 }, $push: { history: 'login' } });
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between `deleteOne()`, `deleteMany()`, and `drop()`? 🔴

- `deleteOne()`/`deleteMany()`: remove specific **documents** matching a filter, the collection itself remains.
- `drop()`: removes the **entire collection** (and its indexes) in one operation — much faster than deleting every document individually if you genuinely want the whole collection gone.

[↑ Back to top](#table-of-contents)

---

### 12. How do you query inside arrays and nested documents? 🔴

```js
db.posts.find({ 'author.name': 'Vaibhav' });          // nested field, dot notation
db.posts.find({ tags: 'mongodb' });                     // array contains this value
db.posts.find({ comments: { $elemMatch: { author: 'A', flagged: true } } }); // array element matching multiple conditions together
```

- `$elemMatch` matters specifically when you need **multiple conditions to match the same array element** — without it, MongoDB would accept conditions matched across *different* elements of the array.

[↑ Back to top](#table-of-contents)

---

### 13. What is `bulkWrite()`, and why would you use it? 🔴

- Sends a **batch** of mixed write operations (inserts, updates, deletes) to the server in a single round trip, instead of issuing each as a separate network call — significantly more efficient for high-volume write workloads.

```js
db.users.bulkWrite([
  { insertOne: { document: { name: 'A' } } },
  { updateOne: { filter: { name: 'B' }, update: { $set: { active: true } } } },
  { deleteOne: { filter: { name: 'C' } } },
]);
```

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between `findOneAndUpdate()` and `updateOne()`? 🔴

- `updateOne()`: returns metadata about the operation (matched/modified counts), **not** the document itself.
- `findOneAndUpdate()`: atomically updates **and returns** the document in one operation (the pre-update version by default, or post-update with `{ returnDocument: 'after' }`) — useful when you need the updated data immediately without a second round-trip query.

[↑ Back to top](#table-of-contents)

---

### 15. How do cursors work, and why does it matter for large result sets? 🔴

- `find()` doesn't fetch every matching document immediately — it returns a **cursor**, which fetches results from the server **in batches** as you iterate, rather than loading the entire result set into memory at once. This matters for large collections: iterating a cursor over millions of documents stays memory-efficient, while calling `.toArray()` on it forces everything into memory upfront, which can be dangerous for very large result sets.

> [!IMPORTANT]
> **Follow-up questions:**
> - What would happen (memory-wise) if you called `.toArray()` on a query matching 10 million documents instead of iterating the cursor directly?

[↑ Back to top](#table-of-contents)

---
