# SQL Basics

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are the main categories of SQL commands?](#1-what-are-the-main-categories-of-sql-commands)
- [2. How do you filter rows with `WHERE`?](#2-how-do-you-filter-rows-with-where)
- [3. How do you sort results?](#3-how-do-you-sort-results)
- [4. What's the difference between `WHERE` and `HAVING`?](#4-whats-the-difference-between-where-and-having)

**🟡 Medium**
- [5. What does `GROUP BY` do?](#5-what-does-group-by-do)
- [6. What's the difference between `DISTINCT` and `GROUP BY`?](#6-whats-the-difference-between-distinct-and-group-by)
- [7. What are the common aggregate functions?](#7-what-are-the-common-aggregate-functions)
- [8. What's the difference between `IN`, `BETWEEN`, and `LIKE`?](#8-whats-the-difference-between-in-between-and-like)
- [9. How does `NULL` behave differently from a normal value in comparisons?](#9-how-does-null-behave-differently-from-a-normal-value-in-comparisons)
- [10. What is a subquery?](#10-what-is-a-subquery)

**🔴 Hard**
- [11. What's the difference between a correlated and a non-correlated subquery?](#11-whats-the-difference-between-a-correlated-and-a-non-correlated-subquery)
- [12. What is a Common Table Expression (CTE), and why use one over a subquery?](#12-what-is-a-common-table-expression-cte-and-why-use-one-over-a-subquery)
- [13. What are window functions, and how do they differ from `GROUP BY`?](#13-what-are-window-functions-and-how-do-they-differ-from-group-by)
- [14. What's the difference between `UNION` and `UNION ALL`?](#14-whats-the-difference-between-union-and-union-all)
- [15. What is the logical order of execution of a SQL query's clauses?](#15-what-is-the-logical-order-of-execution-of-a-sql-querys-clauses)

---

### 1. What are the main categories of SQL commands? 🟢

- **DDL** (Data Definition Language): `CREATE`, `ALTER`, `DROP` — defines structure.
- **DML** (Data Manipulation Language): `SELECT`, `INSERT`, `UPDATE`, `DELETE` — works with data.
- **DCL** (Data Control Language): `GRANT`, `REVOKE` — permissions.
- **TCL** (Transaction Control Language): `COMMIT`, `ROLLBACK` — transaction management.

[↑ Back to top](#table-of-contents)

---

### 2. How do you filter rows with `WHERE`? 🟢

```sql
SELECT * FROM users WHERE age > 18 AND status = 'active';
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you sort results? 🟢

```sql
SELECT * FROM users ORDER BY created_at DESC;
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between `WHERE` and `HAVING`? 🟢

- `WHERE`: filters individual rows **before** grouping happens.
- `HAVING`: filters **groups** after `GROUP BY` has aggregated them — needed when filtering on an aggregate result (like `COUNT(*) > 5`), which doesn't exist yet at the row level `WHERE` operates on.

```sql
SELECT department, COUNT(*) FROM employees GROUP BY department HAVING COUNT(*) > 5;
```

[↑ Back to top](#table-of-contents)

---

### 5. What does `GROUP BY` do? 🟡

- Groups rows sharing the same value(s) in specified column(s), so aggregate functions (`COUNT`, `SUM`, `AVG`) compute **per group** rather than across the whole table.

```sql
SELECT department, AVG(salary) FROM employees GROUP BY department;
```

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between `DISTINCT` and `GROUP BY`? 🟡

- `DISTINCT`: simply removes duplicate rows from the result.
- `GROUP BY`: groups rows **and** typically combines it with aggregate functions to compute per-group values — `SELECT DISTINCT department` and `SELECT department FROM employees GROUP BY department` return the same rows, but only `GROUP BY` lets you also pull in an aggregate like `COUNT(*)` per group in the same query.

[↑ Back to top](#table-of-contents)

---

### 7. What are the common aggregate functions? 🟡

- `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` — each operates across a set of rows (the whole table, or each group under `GROUP BY`) and returns a single summary value.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `IN`, `BETWEEN`, and `LIKE`? 🟡

- `IN`: matches against a **list** of explicit values.
- `BETWEEN`: matches within an inclusive **range**.
- `LIKE`: pattern matching on strings, using `%` (any sequence) and `_` (single character) wildcards.

```sql
WHERE status IN ('active', 'pending')
WHERE age BETWEEN 18 AND 65
WHERE email LIKE '%@gmail.com'
```

[↑ Back to top](#table-of-contents)

---

### 9. How does `NULL` behave differently from a normal value in comparisons? 🟡

- `NULL` represents "unknown," not a value — so `NULL = NULL` evaluates to `NULL` (not `TRUE`), and any comparison involving `NULL` (`=`, `<`, `>`) yields `NULL`, which is treated as falsy in a `WHERE` clause. You must use `IS NULL`/`IS NOT NULL` explicitly to check for it.

```sql
WHERE email IS NULL       -- correct
WHERE email = NULL         -- always returns no rows, even for NULL emails
```

[↑ Back to top](#table-of-contents)

---

### 10. What is a subquery? 🟡

- A query **nested inside** another query, used as a value, a filtering condition, or a derived table.

```sql
SELECT name FROM users WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between a correlated and a non-correlated subquery? 🔴

- **Non-correlated**: runs **independently**, computing its result once, regardless of the outer query's rows.
- **Correlated**: references a column from the **outer** query, so it must logically re-run **once per outer row** — generally slower, since the database can't simply compute it once and reuse the result.

```sql
-- Correlated: references e.department from the outer row, runs per row
SELECT name FROM employees e
WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);
```

[↑ Back to top](#table-of-contents)

---

### 12. What is a Common Table Expression (CTE), and why use one over a subquery? 🔴

- A named, temporary result set defined with `WITH`, usable within the main query as if it were a table — improves readability for complex queries (especially ones reused multiple times in the same query) compared to deeply nested subqueries, and supports recursion (`WITH RECURSIVE`), which plain subqueries can't do at all.

```sql
WITH high_earners AS (
  SELECT * FROM employees WHERE salary > 100000
)
SELECT department, COUNT(*) FROM high_earners GROUP BY department;
```

[↑ Back to top](#table-of-contents)

---

### 13. What are window functions, and how do they differ from `GROUP BY`? 🔴

- Compute a value **across a set of related rows** (a "window") **without collapsing them** into a single grouped row — each original row stays in the result, now with an added computed column (a running total, a rank, a value from the previous row).

```sql
SELECT name, salary, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
-- Unlike GROUP BY, every employee row is still present, now annotated with its rank
```

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between `UNION` and `UNION ALL`? 🔴

- `UNION`: combines result sets from two queries and **removes duplicate rows** — requires an extra deduplication step.
- `UNION ALL`: combines them **without** removing duplicates — faster, since it skips the deduplication pass; use it whenever you know there won't be duplicates, or don't care about them.

[↑ Back to top](#table-of-contents)

---

### 15. What is the logical order of execution of a SQL query's clauses? 🔴

- Despite the order you **write** them, clauses are logically processed as: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`. This is exactly why `HAVING` can filter on aggregates (computed by the time it runs) while `WHERE` cannot, and why you can't reference a `SELECT`-aliased column in `WHERE` (it doesn't exist yet at that point in execution) but you **can** in `ORDER BY`.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does `SELECT name, COUNT(*) AS total FROM ... WHERE total > 5` fail, while the same condition works fine in a `HAVING` clause?

[↑ Back to top](#table-of-contents)

---
