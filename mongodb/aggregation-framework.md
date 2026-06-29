# Aggregation Framework

_Part of [MongoDB](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is the aggregation framework?](#1-what-is-the-aggregation-framework)
- [2. What is a pipeline, and how does data flow through it?](#2-what-is-a-pipeline-and-how-does-data-flow-through-it)
- [3. What does the `$match` stage do?](#3-what-does-the-match-stage-do)

**🟡 Medium**
- [4. What does the `$group` stage do?](#4-what-does-the-group-stage-do)
- [5. What does the `$project` stage do?](#5-what-does-the-project-stage-do)
- [6. What does the `$sort` stage do, and where should it appear in the pipeline?](#6-what-does-the-sort-stage-do-and-where-should-it-appear-in-the-pipeline)
- [7. What's the difference between `$group` and SQL's `GROUP BY`?](#7-whats-the-difference-between-group-and-sqls-group-by)
- [8. What do `$limit` and `$skip` do, and what's a caveat of using `$skip` for pagination?](#8-what-do-limit-and-skip-do-and-what-s-a-caveat-of-using-skip-for-pagination)

**🔴 Hard**
- [9. What does the `$lookup` stage do, and how does it compare to a SQL join?](#9-what-does-the-lookup-stage-do-and-how-does-it-compare-to-a-sql-join)
- [10. What does the `$unwind` stage do?](#10-what-does-the-unwind-stage-do)
- [11. How do you optimize an aggregation pipeline for performance?](#11-how-do-you-optimize-an-aggregation-pipeline-for-performance)
- [12. What is `$facet`, and when would you use it?](#12-what-is-facet-and-when-would-you-use-it)
- [13. What is the difference between the aggregation pipeline and MapReduce in MongoDB?](#13-what-is-the-difference-between-the-aggregation-pipeline-and-mapreduce-in-mongodb)
- [14. How would you build a pipeline to compute a running total or per-group ranking?](#14-how-would-you-build-a-pipeline-to-compute-a-running-total-or-per-group-ranking)

---

### 1. What is the aggregation framework? 🟢

- MongoDB's mechanism for processing and transforming data **server-side** — computing sums/averages, reshaping documents, joining collections, and more — through a sequence of declarative processing stages, rather than pulling raw data to the application and computing everything in code.

[↑ Back to top](#table-of-contents)

---

### 2. What is a pipeline, and how does data flow through it? 🟢

- An ordered array of **stages**, each taking the **output of the previous stage** as its input — documents flow through stage by stage, getting filtered, reshaped, or grouped along the way, conceptually similar to piping commands together in a Unix shell.

```js
db.orders.aggregate([
  { $match: { status: 'completed' } },
  { $group: { _id: '$customerId', total: { $sum: '$amount' } } },
]);
```

[↑ Back to top](#table-of-contents)

---

### 3. What does the `$match` stage do? 🟢

- Filters documents, just like a `find()` query — typically placed **early** in the pipeline to discard irrelevant documents as soon as possible, reducing the work every later stage has to do.

[↑ Back to top](#table-of-contents)

---

### 4. What does the `$group` stage do? 🟡

- Groups documents by a specified key and computes aggregate values (sum, average, count, min/max) per group — analogous to SQL's `GROUP BY` combined with aggregate functions.

```js
db.orders.aggregate([
  { $group: { _id: '$status', count: { $sum: 1 }, avgAmount: { $avg: '$amount' } } },
]);
```

[↑ Back to top](#table-of-contents)

---

### 5. What does the `$project` stage do? 🟡

- Reshapes each document — including/excluding fields, renaming them, or computing new fields from existing ones — similar to `SELECT` with computed columns in SQL.

```js
db.orders.aggregate([
  { $project: { customer: 1, totalWithTax: { $multiply: ['$amount', 1.18] } } },
]);
```

[↑ Back to top](#table-of-contents)

---

### 6. What does the `$sort` stage do, and where should it appear in the pipeline? 🟡

- Sorts documents by a specified field. Placing it **before** a `$group`/`$limit` (when possible) lets MongoDB potentially use an index to avoid an expensive in-memory sort, and placing `$limit` right after `$sort` lets MongoDB avoid sorting more documents than necessary.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `$group` and SQL's `GROUP BY`? 🟡

- Functionally similar in purpose, but `$group`'s `_id` field **is** the grouping key (it must be explicitly assigned, often as an expression rather than a plain column name) — and since MongoDB documents can be nested/array-shaped, you can group by computed expressions or even multiple fields combined into one object key in ways that map less directly onto flat SQL columns.

```js
{ $group: { _id: { year: { $year: '$date' }, status: '$status' }, total: { $sum: '$amount' } } }
```

[↑ Back to top](#table-of-contents)

---

### 8. What do `$limit` and `$skip` do, and what's a caveat of using `$skip` for pagination? 🟡

- `$limit`: caps the number of documents passed to the next stage.
- `$skip`: discards a number of documents before passing the rest along — but like offset-based pagination generally (see [REST API › Pagination](../rest-api/pagination-filtering-sorting.md#5-what-are-the-drawbacks-of-offset-based-pagination-at-scale)), a large `$skip` value still requires scanning past all skipped documents, getting progressively slower for deep pages.

[↑ Back to top](#table-of-contents)

---

### 9. What does the `$lookup` stage do, and how does it compare to a SQL join? 🔴

- Performs a **left outer join** against another collection within the same database, embedding matched documents from the "foreign" collection into the current pipeline's documents. Conceptually similar to a SQL `LEFT JOIN`, but generally **less performant** at scale — MongoDB isn't optimized around joins the way relational engines are, so schemas are usually designed (via embedding/denormalization, see [Schema Design](schema-design.md)) to minimize reliance on `$lookup` in hot query paths.

```js
db.orders.aggregate([
  { $lookup: { from: 'customers', localField: 'customerId', foreignField: '_id', as: 'customer' } },
]);
```

[↑ Back to top](#table-of-contents)

---

### 10. What does the `$unwind` stage do? 🔴

- Takes a document with an **array** field and outputs **one separate document per array element**, duplicating the rest of the document's fields for each — necessary when you need to group/filter/aggregate based on individual array elements rather than the array as a whole.

```js
// { _id: 1, tags: ['a', 'b'] } -> { _id: 1, tags: 'a' }, { _id: 1, tags: 'b' }
db.posts.aggregate([{ $unwind: '$tags' }, { $group: { _id: '$tags', count: { $sum: 1 } } }]);
```

[↑ Back to top](#table-of-contents)

---

### 11. How do you optimize an aggregation pipeline for performance? 🔴

- Put `$match` (and `$sort`, where index-supported) as **early** as possible to shrink the working set before expensive stages run; ensure fields used in early `$match`/`$sort` stages are indexed (MongoDB can use indexes for the **initial** stages of a pipeline, but not for arbitrary later stages); avoid unnecessary `$lookup`s in hot paths; and use `$project`/`$unset` early to drop fields you don't need, reducing the amount of data flowing through later stages.

[↑ Back to top](#table-of-contents)

---

### 12. What is `$facet`, and when would you use it? 🔴

- Runs **multiple independent aggregation sub-pipelines** against the same input documents in parallel, returning all their results together in one response — useful for something like a product listing page needing both the paginated results **and** separate category/price-range facet counts from the same underlying data, without running multiple separate queries.

```js
db.products.aggregate([
  { $facet: {
      results: [{ $skip: 0 }, { $limit: 20 }],
      categoryCounts: [{ $group: { _id: '$category', count: { $sum: 1 } } }],
  } },
]);
```

[↑ Back to top](#table-of-contents)

---

### 13. What is the difference between the aggregation pipeline and MapReduce in MongoDB? 🔴

- **Aggregation pipeline**: declarative stages, generally faster, easier to read and optimize, and is the actively recommended/maintained approach for virtually all data-processing needs.
- **MapReduce**: an older, more general-purpose JavaScript-based approach for custom map/reduce logic — more flexible for genuinely exotic custom logic, but slower and effectively deprecated in favor of the aggregation pipeline (which now covers the vast majority of use cases MapReduce used to be needed for).

[↑ Back to top](#table-of-contents)

---

### 14. How would you build a pipeline to compute a running total or per-group ranking? 🔴

- Use **window functions** (`$setWindowFields`, available in modern MongoDB versions) — lets you compute values like a running total or rank **within a partition**, without manually self-joining or post-processing in application code.

```js
db.sales.aggregate([
  { $setWindowFields: {
      partitionBy: '$region',
      sortBy: { date: 1 },
      output: { runningTotal: { $sum: '$amount', window: { documents: ['unbounded', 'current'] } } },
  } },
]);
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Before `$setWindowFields` was introduced, how would you have computed a running total — and why was that approach more cumbersome?

[↑ Back to top](#table-of-contents)

---
