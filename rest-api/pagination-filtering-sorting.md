# Pagination, Filtering & Sorting

_Part of [REST API](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Why is pagination necessary for REST APIs?](#1-why-is-pagination-necessary-for-rest-apis)
- [2. What is offset-based pagination?](#2-what-is-offset-based-pagination)
- [3. How do you implement basic filtering via query parameters?](#3-how-do-you-implement-basic-filtering-via-query-parameters)

**🟡 Medium**
- [4. What is cursor-based pagination, and how does it differ from offset-based?](#4-what-is-cursor-based-pagination-and-how-does-it-differ-from-offset-based)
- [5. What are the drawbacks of offset-based pagination at scale?](#5-what-are-the-drawbacks-of-offset-based-pagination-at-scale)
- [6. How do you implement sorting via query parameters?](#6-how-do-you-implement-sorting-via-query-parameters)
- [7. How do you handle multiple filter conditions in a single request?](#7-how-do-you-handle-multiple-filter-conditions-in-a-single-request)

**🔴 Hard**
- [8. How would you design a consistent pagination response envelope?](#8-how-would-you-design-a-consistent-pagination-response-envelope)
- [9. What's the consistency tradeoff between cursor-based and offset-based pagination under concurrent writes?](#9-whats-the-consistency-tradeoff-between-cursor-based-and-offset-based-pagination-under-concurrent-writes)
- [10. What is keyset pagination, and how does it differ from simple cursor pagination?](#10-what-is-keyset-pagination-and-how-does-it-differ-from-simple-cursor-pagination)
- [11. How would you support filtering on nested/related resources without overcomplicating the query string?](#11-how-would-you-support-filtering-on-nestedrelated-resources-without-overcomplicating-the-query-string)
- [12. How do you prevent pagination parameters from being abused for a denial-of-service-style query?](#12-how-do-you-prevent-pagination-parameters-from-being-abused-for-a-denial-of-service-style-query)

---

### 1. Why is pagination necessary for REST APIs? 🟢

- Returning an entire large collection in one response is slow, memory-intensive on both ends, and usually unnecessary — pagination returns a manageable chunk at a time, improving response times and reducing load.

[↑ Back to top](#table-of-contents)

---

### 2. What is offset-based pagination? 🟢

- The client specifies how many items to **skip** and how many to return.

```
GET /users?offset=20&limit=10   // skip the first 20, return the next 10
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you implement basic filtering via query parameters? 🟢

```
GET /users?status=active&role=admin
```

```js
app.get('/users', (req, res) => {
  const { status, role } = req.query;
  const filtered = users.filter(u => (!status || u.status === status) && (!role || u.role === role));
  res.json(filtered);
});
```

[↑ Back to top](#table-of-contents)

---

### 4. What is cursor-based pagination, and how does it differ from offset-based? 🟡

- Instead of an offset/count, the client passes an opaque **cursor** (often an encoded reference to the last item seen) pointing to where the next page should start.

```
GET /users?cursor=eyJpZCI6NDJ9&limit=10
```

- Difference from offset: cursor-based pagination doesn't depend on a numeric position, so it's stable even if items are inserted/removed between requests — offset-based can skip or repeat items if the underlying data changes mid-pagination.

[↑ Back to top](#table-of-contents)

---

### 5. What are the drawbacks of offset-based pagination at scale? 🟡

- The database still has to **scan and discard** all the skipped rows to reach the offset — for a large offset (e.g. page 10,000), this gets progressively slower. It's also inconsistent under concurrent writes (Q9): inserting/deleting rows while paginating can cause items to be skipped or duplicated across pages.

[↑ Back to top](#table-of-contents)

---

### 6. How do you implement sorting via query parameters? 🟡

```
GET /users?sort=createdAt:desc
GET /users?sort=-createdAt          // alternate convention: "-" prefix means descending
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you handle multiple filter conditions in a single request? 🟡

- Combine multiple query parameters, treated as an implicit `AND`; for more complex needs (ranges, `OR` logic), adopt an explicit operator convention.

```
GET /products?category=electronics&price[gte]=100&price[lte]=500
```

[↑ Back to top](#table-of-contents)

---

### 8. How would you design a consistent pagination response envelope? 🔴

- Return the page of data alongside metadata describing the pagination state, in a consistent shape across every paginated endpoint.

```json
{
  "data": [ /* items */ ],
  "meta": { "total": 245, "limit": 10, "offset": 20, "hasMore": true }
}
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the consistency tradeoff between cursor-based and offset-based pagination under concurrent writes? 🔴

- **Offset-based**: if a row is inserted/deleted before the current offset while a client pages through results, every subsequent page shifts — items can be skipped entirely or seen twice.
- **Cursor-based**: anchored to a specific item's position (e.g. "after the row with this ID/timestamp"), so it's immune to that shifting problem — new inserts elsewhere don't affect where the next page starts.

[↑ Back to top](#table-of-contents)

---

### 10. What is keyset pagination, and how does it differ from simple cursor pagination? 🔴

- A specific, **performant** implementation of cursor-based pagination: instead of an opaque token, the cursor is literally the **sort key's value** of the last seen row (e.g. `created_at` + `id` for a tiebreak), and the query directly filters `WHERE (created_at, id) > (lastCreatedAt, lastId)` — this lets the database use an index efficiently, avoiding the "scan and discard" cost of offset-based pagination (Q5) entirely, unlike some generic cursor implementations that still page via offset internally.

```sql
SELECT * FROM users WHERE (created_at, id) > ('2026-01-01', 42) ORDER BY created_at, id LIMIT 10;
```

[↑ Back to top](#table-of-contents)

---

### 11. How would you support filtering on nested/related resources without overcomplicating the query string? 🔴

- Use a clear, documented dot-notation or bracket convention for nested fields, and consider capping how deep filtering can go to avoid an explosion of query complexity — beyond a certain complexity, a dedicated search endpoint (or GraphQL) often becomes a better fit than stretching REST query parameters further.

```
GET /orders?customer.country=IN&customer.tier=premium
```

[↑ Back to top](#table-of-contents)

---

### 12. How do you prevent pagination parameters from being abused for a denial-of-service-style query? 🔴

- Enforce a **maximum** allowed page size server-side (ignore or reject a client-requested `limit` beyond it), and validate/cap offset values too — without this, a client requesting `limit=1000000` (or an enormous offset forcing a huge table scan) could degrade performance for everyone.

```js
const limit = Math.min(parseInt(req.query.limit) || 20, 100); // hard cap at 100
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does keyset pagination's performance advantage over offset-based pagination grow specifically as the table gets larger?

[↑ Back to top](#table-of-contents)

---
