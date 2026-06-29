# Normalization

_Part of [MySQL](README.md) interview notes._

> This file covers normalization as applied in MySQL schema design. The deeper theoretical definitions of normal forms (functional dependencies, formal proofs) are covered in [DBMS › Normalization](../dbms/normalization.md).

## Table of Contents

**🟢 Easy**
- [1. What is normalization, and why is it done?](#1-what-is-normalization-and-why-is-it-done)
- [2. What problem does normalization solve?](#2-what-problem-does-normalization-solve)
- [3. What is First Normal Form (1NF)?](#3-what-is-first-normal-form-1nf)

**🟡 Medium**
- [4. What is Second Normal Form (2NF)?](#4-what-is-second-normal-form-2nf)
- [5. What is Third Normal Form (3NF)?](#5-what-is-third-normal-form-3nf)
- [6. What is a partial dependency?](#6-what-is-a-partial-dependency)
- [7. What is a transitive dependency?](#7-what-is-a-transitive-dependency)
- [8. What anomalies does normalization prevent?](#8-what-anomalies-does-normalization-prevent)

**🔴 Hard**
- [9. What is denormalization, and when would you deliberately denormalize a MySQL schema?](#9-what-is-denormalization-and-when-would-you-deliberately-denormalize-a-mysql-schema)
- [10. What is Boyce-Codd Normal Form (BCNF), and how does it differ from 3NF?](#10-what-is-boyce-codd-normal-form-bcnf-and-how-does-it-differ-from-3nf)
- [11. What's the practical tradeoff between a highly normalized schema and a denormalized one?](#11-whats-the-practical-tradeoff-between-a-highly-normalized-schema-and-a-denormalized-one)
- [12. How would you normalize a table that stores comma-separated values in a single column?](#12-how-would-you-normalize-a-table-that-stores-comma-separated-values-in-a-single-column)

---

### 1. What is normalization, and why is it done? 🟢

- The process of organizing tables to **reduce data duplication** and avoid inconsistencies — splitting data into multiple related tables (linked via foreign keys) instead of repeating the same information across many rows of one large table.

[↑ Back to top](#table-of-contents)

---

### 2. What problem does normalization solve? 🟢

- Without it, the same piece of data (e.g. a customer's address) might be duplicated across many rows — updating it means updating every duplicate, and missing one leaves the data **inconsistent**. Normalization stores each fact **once**, in one place.

[↑ Back to top](#table-of-contents)

---

### 3. What is First Normal Form (1NF)? 🟢

- Every column holds a single, **atomic** value (no comma-separated lists, no repeating groups within one field), and each row is uniquely identifiable.

```sql
-- Violates 1NF: multiple values crammed into one column
| id | phones              |
| 1  | '555-1234,555-5678' |

-- 1NF: one phone per row
| id | phone     |
| 1  | 555-1234  |
| 1  | 555-5678  |
```

[↑ Back to top](#table-of-contents)

---

### 4. What is Second Normal Form (2NF)? 🟡

- Builds on 1NF, and additionally requires that every non-key column depends on the **entire** primary key, not just part of it — relevant specifically for tables with a **composite** primary key (Q6).

[↑ Back to top](#table-of-contents)

---

### 5. What is Third Normal Form (3NF)? 🟡

- Builds on 2NF, and additionally requires that non-key columns depend **only** on the primary key, not on **other non-key columns** (no transitive dependencies, Q7). 3NF is the practical target most application schemas aim for — it eliminates the vast majority of redundancy-driven anomalies without the added complexity of stricter normal forms.

[↑ Back to top](#table-of-contents)

---

### 6. What is a partial dependency? 🟡

- When a non-key column depends on only **part** of a composite primary key, rather than the whole thing — violates 2NF.

```sql
-- (student_id, course_id) is the composite PK; course_name depends only on course_id, not the whole key
| student_id | course_id | course_name | grade |

-- Fix: move course_name to its own `courses` table, keyed by course_id alone
```

[↑ Back to top](#table-of-contents)

---

### 7. What is a transitive dependency? 🟡

- When a non-key column depends on **another non-key column**, rather than directly on the primary key — violates 3NF.

```sql
-- zip_code determines city, but city isn't directly dependent on the employee's id (the PK)
| emp_id | zip_code | city     |

-- Fix: move zip_code -> city mapping into its own table, referenced by zip_code
```

[↑ Back to top](#table-of-contents)

---

### 8. What anomalies does normalization prevent? 🟡

- **Update anomaly**: changing a duplicated fact requires updating it in multiple places, risking inconsistency if one is missed.
- **Insert anomaly**: can't add certain data without also having unrelated data available (e.g. can't add a new course without an enrolled student, if course info is only stored within the enrollment table).
- **Delete anomaly**: deleting one row unintentionally erases other, unrelated information that happened to be stored alongside it.

[↑ Back to top](#table-of-contents)

---

### 9. What is denormalization, and when would you deliberately denormalize a MySQL schema? 🔴

- Deliberately **reintroducing** some redundancy (duplicating a value, or pre-joining frequently-needed data into one wider table) to **improve read performance**, at the cost of needing to keep duplicates in sync on writes — done when a normalized schema's joins are a measured, proven bottleneck for a read-heavy, performance-critical path, and the duplicated data changes rarely enough that keeping it in sync is manageable.

[↑ Back to top](#table-of-contents)

---

### 10. What is Boyce-Codd Normal Form (BCNF), and how does it differ from 3NF? 🔴

- A **stricter** version of 3NF: every determinant (a column or set of columns that other columns functionally depend on) must itself be a candidate key — addresses a narrow edge case where a table satisfies 3NF's letter but still has an anomaly-causing dependency from a non-candidate-key determinant. Rare in practical schema design, since most real-world 3NF schemas naturally satisfy BCNF too; it mainly comes up in formal normalization theory and overlapping-candidate-key edge cases (covered more rigorously in [DBMS › Normalization](../dbms/normalization.md)).

[↑ Back to top](#table-of-contents)

---

### 11. What's the practical tradeoff between a highly normalized schema and a denormalized one? 🔴

- **Normalized**: less redundancy, easier to keep consistent on writes, but reads often need multiple joins across tables — can get expensive for complex, read-heavy queries.
- **Denormalized**: faster reads (fewer/no joins needed), at the cost of redundant data that must be kept in sync, and a higher risk of inconsistency if an update path is missed. Most production schemas land somewhere in between — normalized as a baseline (3NF), with deliberate, targeted denormalization only where profiling shows it's actually needed.

[↑ Back to top](#table-of-contents)

---

### 12. How would you normalize a table that stores comma-separated values in a single column? 🔴

- Extract the comma-separated values into their **own table**, with one row per value, linked back via a foreign key — turning a many-values-in-one-field anti-pattern into a proper one-to-many relationship that can be indexed, joined, and queried per individual value.

```sql
-- Before: tags stored as 'sql,database,mysql' in one column
-- After:
CREATE TABLE post_tags (post_id INT, tag VARCHAR(50), FOREIGN KEY (post_id) REFERENCES posts(id));
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does storing comma-separated values make it nearly impossible to efficiently query "all posts tagged with X" using an index?

[↑ Back to top](#table-of-contents)

---
