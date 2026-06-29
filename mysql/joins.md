# Joins

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a `JOIN`?](#1-what-is-a-join)
- [2. What is an `INNER JOIN`?](#2-what-is-an-inner-join)
- [3. What is a `LEFT JOIN`?](#3-what-is-a-left-join)

**🟡 Medium**
- [4. What is a `RIGHT JOIN`, and why is it rarely used in practice?](#4-what-is-a-right-join-and-why-is-it-rarely-used-in-practice)
- [5. What is a `FULL OUTER JOIN`, and how do you simulate it in MySQL?](#5-what-is-a-full-outer-join-and-how-do-you-simulate-it-in-mysql)
- [6. What is a `CROSS JOIN`?](#6-what-is-a-cross-join)
- [7. What is a self-join, and when would you use one?](#7-what-is-a-self-join-and-when-would-you-use-one)
- [8. What's the difference between an explicit `JOIN ... ON` and an implicit comma join?](#8-whats-the-difference-between-an-explicit-join--on-and-an-implicit-comma-join)

**🔴 Hard**
- [9. What happens to non-matching rows in each join type?](#9-what-happens-to-non-matching-rows-in-each-join-type)
- [10. How do you find rows in one table that have no matching row in another?](#10-how-do-you-find-rows-in-one-table-that-have-no-matching-row-in-another)
- [11. How does join order/strategy affect query performance?](#11-how-does-join-orderstrategy-affect-query-performance)
- [12. What's the difference between a `JOIN` and a subquery for combining data from two tables?](#12-whats-the-difference-between-a-join-and-a-subquery-for-combining-data-from-two-tables)
- [13. How would you join three or more tables together?](#13-how-would-you-join-three-or-more-tables-together)

---

### 1. What is a `JOIN`? 🟢

- Combines rows from two or more tables based on a related column between them — the core mechanism for querying across normalized, relational tables.

[↑ Back to top](#table-of-contents)

---

### 2. What is an `INNER JOIN`? 🟢

- Returns only rows that have a **matching** value in both tables — rows with no match on either side are excluded entirely.

```sql
SELECT u.name, o.total FROM users u INNER JOIN orders o ON u.id = o.user_id;
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a `LEFT JOIN`? 🟢

- Returns **all** rows from the left table, plus matching rows from the right table — if there's no match, the right table's columns come back as `NULL`.

```sql
SELECT u.name, o.total FROM users u LEFT JOIN orders o ON u.id = o.user_id;
-- Includes every user, even those with zero orders (o.total would be NULL)
```

[↑ Back to top](#table-of-contents)

---

### 4. What is a `RIGHT JOIN`, and why is it rarely used in practice? 🟡

- The mirror of `LEFT JOIN` — returns all rows from the **right** table, with unmatched left-table columns as `NULL`. Rarely used because any `RIGHT JOIN` can be rewritten as a `LEFT JOIN` by simply swapping the table order — most style guides standardize on always using `LEFT JOIN` for consistency/readability.

[↑ Back to top](#table-of-contents)

---

### 5. What is a `FULL OUTER JOIN`, and how do you simulate it in MySQL? 🟡

- Returns all rows from **both** tables, matched where possible, with `NULL`s filling in unmatched columns on either side. MySQL doesn't support `FULL OUTER JOIN` natively — simulate it with a `LEFT JOIN UNION RIGHT JOIN` (or `UNION ALL` plus filtering to avoid double-counting matched rows).

```sql
SELECT * FROM a LEFT JOIN b ON a.id = b.a_id
UNION
SELECT * FROM a RIGHT JOIN b ON a.id = b.a_id;
```

[↑ Back to top](#table-of-contents)

---

### 6. What is a `CROSS JOIN`? 🟡

- Returns the **Cartesian product** — every row from the first table paired with every row from the second, with no matching condition at all. Rows = (rows in A) × (rows in B), useful for generating combinations (e.g. every size paired with every color) but dangerous on large tables if used unintentionally.

[↑ Back to top](#table-of-contents)

---

### 7. What is a self-join, and when would you use one? 🟡

- A table joined to **itself**, typically using aliases to distinguish the two "copies" — useful for hierarchical data within one table (e.g. an `employees` table where each row has a `manager_id` referencing another row in the same table).

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e LEFT JOIN employees m ON e.manager_id = m.id;
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between an explicit `JOIN ... ON` and an implicit comma join? 🟡

- `FROM a, b WHERE a.id = b.a_id` (implicit/old-style) is functionally equivalent to `FROM a JOIN b ON a.id = b.a_id` (explicit) for inner joins — but the explicit syntax is universally preferred: it clearly separates the join condition from filtering conditions, supports outer joins (which the comma syntax can't express at all), and is far less error-prone (forgetting the `WHERE` condition in the comma form silently produces a full cross join).

[↑ Back to top](#table-of-contents)

---

### 9. What happens to non-matching rows in each join type? 🔴

- `INNER JOIN`: excluded entirely from the result.
- `LEFT JOIN`: kept from the left table, right-side columns become `NULL`.
- `RIGHT JOIN`: kept from the right table, left-side columns become `NULL`.
- `FULL OUTER JOIN` (simulated): kept from **both** sides where unmatched, with `NULL`s on whichever side has no match.

[↑ Back to top](#table-of-contents)

---

### 10. How do you find rows in one table that have no matching row in another? 🔴

- `LEFT JOIN` against the second table, then filter for rows where the joined column is `NULL` (an "anti-join" pattern).

```sql
SELECT u.* FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL; -- users with zero orders
```

[↑ Back to top](#table-of-contents)

---

### 11. How does join order/strategy affect query performance? 🔴

- MySQL's optimizer generally chooses join order itself (often starting from the table that filters down to the fewest rows first), but **indexing the join columns** matters far more than the order you write tables in — without an index on the joined column, MySQL falls back to a slower nested-loop scan comparing every row pair. Use `EXPLAIN` to check which join algorithm/order is actually chosen and whether indexes are being used.

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between a `JOIN` and a subquery for combining data from two tables? 🔴

- Often functionally interchangeable for the same logical result, but a `JOIN` typically lets the query optimizer reason about and optimize the combined access pattern more effectively (e.g. choosing better indexes across both tables together), whereas a correlated subquery (see [SQL Basics](sql-basics.md#11-whats-the-difference-between-a-correlated-and-a-non-correlated-subquery)) re-executes per outer row and can be significantly slower. As a rule of thumb, prefer `JOIN`s for combining columns from related tables, and reserve subqueries for cases naturally expressed as "a value/condition derived from another query."

[↑ Back to top](#table-of-contents)

---

### 13. How would you join three or more tables together? 🔴

- Chain additional `JOIN` clauses, each with its own `ON` condition — MySQL processes them left to right (logically), though the optimizer is free to reorder for performance as long as the result is equivalent.

```sql
SELECT u.name, o.total, p.name AS product
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

> [!IMPORTANT]
> **Follow-up questions:**
> - If one of the joins in a multi-table chain should be a `LEFT JOIN` instead of `INNER JOIN`, how does that change which rows survive to the final result?

[↑ Back to top](#table-of-contents)

---
