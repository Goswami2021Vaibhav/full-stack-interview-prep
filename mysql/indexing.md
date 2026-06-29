# Indexing

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is an index, and why does it speed up queries?](#1-what-is-an-index-and-why-does-it-speed-up-queries)
- [2. How do you create an index?](#2-how-do-you-create-an-index)
- [3. What index is created automatically on a primary key?](#3-what-index-is-created-automatically-on-a-primary-key)

**🟡 Medium**
- [4. What data structure does MySQL (InnoDB) use for indexes?](#4-what-data-structure-does-mysql-innodb-use-for-indexes)
- [5. What is a composite (multi-column) index?](#5-what-is-a-composite-multi-column-index)
- [6. What is the leftmost-prefix rule for composite indexes?](#6-what-is-the-leftmost-prefix-rule-for-composite-indexes)
- [7. What's the tradeoff of adding more indexes to a table?](#7-whats-the-tradeoff-of-adding-more-indexes-to-a-table)
- [8. How do you check if a query is using an index?](#8-how-do-you-check-if-a-query-is-using-an-index)

**🔴 Hard**
- [9. What is a clustered index in InnoDB specifically?](#9-what-is-a-clustered-index-in-innodb-specifically)
- [10. What is a covering index, and why is it especially fast?](#10-what-is-a-covering-index-and-why-is-it-especially-fast)
- [11. Why doesn't a leading wildcard `LIKE '%term'` use an index?](#11-why-doesnt-a-leading-wildcard-like-term-use-an-index)
- [12. What is index cardinality, and why does it matter?](#12-what-is-index-cardinality-and-why-does-it-matter)
- [13. Why might MySQL choose a full table scan over an existing index?](#13-why-might-mysql-choose-a-full-table-scan-over-an-existing-index)

---

### 1. What is an index, and why does it speed up queries? 🟢

- A separate data structure that lets MySQL **locate** rows matching a condition without scanning every row in the table — similar to a book's index letting you jump to a page instead of reading the whole book to find a topic.

[↑ Back to top](#table-of-contents)

---

### 2. How do you create an index? 🟢

```sql
CREATE INDEX idx_users_email ON users(email);
```

[↑ Back to top](#table-of-contents)

---

### 3. What index is created automatically on a primary key? 🟢

- A unique, clustered index (in InnoDB) — every table's primary key is automatically indexed, requiring no explicit `CREATE INDEX` statement.

[↑ Back to top](#table-of-contents)

---

### 4. What data structure does MySQL (InnoDB) use for indexes? 🟡

- A **B+ tree** — a balanced tree structure where all actual data/pointers live at the leaf level, and internal nodes only guide the search — giving `O(log n)` lookups, and efficient range scans since leaf nodes are linked in sorted order.

[↑ Back to top](#table-of-contents)

---

### 5. What is a composite (multi-column) index? 🟡

- An index spanning **multiple** columns together, useful when queries commonly filter/sort on more than one column at once.

```sql
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the leftmost-prefix rule for composite indexes? 🟡

- A composite index can efficiently support queries filtering on a **leading prefix** of its columns (just the first, or the first and second, etc.) — but not on a later column alone, skipping earlier ones.

```sql
-- Index on (user_id, created_at) efficiently supports:
WHERE user_id = 5
WHERE user_id = 5 AND created_at > '2026-01-01'
-- but NOT efficiently: WHERE created_at > '2026-01-01' alone
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the tradeoff of adding more indexes to a table? 🟡

- Each index **speeds up reads** that can use it, but **slows down writes** — every `INSERT`/`UPDATE`/`DELETE` must also update every index on the affected row, and each index consumes additional disk/memory. Index choices should be driven by actual query patterns, not added speculatively for every column.

[↑ Back to top](#table-of-contents)

---

### 8. How do you check if a query is using an index? 🟡

- Prefix the query with `EXPLAIN` — check the `type` column (`ref`/`range`/`const` are good, `ALL` means a full table scan) and `key` (which index, if any, was actually used).

```sql
EXPLAIN SELECT * FROM users WHERE email = 'v@x.com';
```

[↑ Back to top](#table-of-contents)

---

### 9. What is a clustered index in InnoDB specifically? 🔴

- InnoDB **always** organizes a table's actual row data physically in primary-key order — the primary key **is** the clustered index, with no separate structure needed. If no primary key is declared, InnoDB will use the first unique non-null index, or otherwise generate a hidden internal row ID to cluster by — every table is always clustered by something.

[↑ Back to top](#table-of-contents)

---

### 10. What is a covering index, and why is it especially fast? 🔴

- An index that contains **every** column a query needs (both filtered and selected) — MySQL can satisfy the entire query directly from the index structure itself, without a secondary lookup back to the actual table row (avoiding the extra I/O that a normal secondary index lookup requires).

```sql
CREATE INDEX idx_covering ON users(email, name);
SELECT name FROM users WHERE email = 'v@x.com'; -- fully covered, never touches the row itself
```

[↑ Back to top](#table-of-contents)

---

### 11. Why doesn't a leading wildcard `LIKE '%term'` use an index? 🔴

- B+ tree indexes are sorted, letting MySQL efficiently jump to where a **prefix** match would begin (`LIKE 'term%'` can use the index this way) — but a leading `%` means the match could start **anywhere** within the string, which the sorted index structure has no way to narrow down, forcing a full scan regardless of the index's existence.

[↑ Back to top](#table-of-contents)

---

### 12. What is index cardinality, and why does it matter? 🔴

- The number of **distinct** values a column has relative to its total rows — a high-cardinality column (e.g. `email`, mostly unique) makes a very effective index, since each lookup narrows down to very few rows. A low-cardinality column (e.g. a `boolean` flag with only two possible values) makes a poor index candidate, since the lookup still has to sift through roughly half the table either way — the optimizer may simply ignore such an index in favor of a full scan (Q13).

[↑ Back to top](#table-of-contents)

---

### 13. Why might MySQL choose a full table scan over an existing index? 🔴

- The optimizer estimates the **cost** of each available plan and picks the cheapest — if a query would match a large enough fraction of the table anyway (low selectivity, often due to low cardinality, Q12), reading the table sequentially can actually be cheaper than the overhead of jumping around via an index and then fetching each matched row individually. An index existing doesn't guarantee it'll be used; it has to actually be the cheaper option for that specific query.

> [!IMPORTANT]
> **Follow-up questions:**
> - If `EXPLAIN` shows MySQL ignoring an index you expected it to use, what stats/tools would you check to understand why?

[↑ Back to top](#table-of-contents)

---
