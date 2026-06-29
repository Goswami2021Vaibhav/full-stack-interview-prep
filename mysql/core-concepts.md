# Core Concepts

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is MySQL, and what type of database is it?](#1-what-is-mysql-and-what-type-of-database-is-it)
- [2. What is a database, a table, and a schema in MySQL?](#2-what-is-a-database-a-table-and-a-schema-in-mysql)
- [3. What is a primary key?](#3-what-is-a-primary-key)
- [4. What is a foreign key?](#4-what-is-a-foreign-key)

**🟡 Medium**
- [5. What's the difference between a primary key and a unique key?](#5-whats-the-difference-between-a-primary-key-and-a-unique-key)
- [6. What's the difference between `CHAR` and `VARCHAR`?](#6-whats-the-difference-between-char-and-varchar)
- [7. What is a storage engine, and which is MySQL's default?](#7-what-is-a-storage-engine-and-which-is-mysqls-default)
- [8. What's the difference between `InnoDB` and `MyISAM`?](#8-whats-the-difference-between-innodb-and-myisam)
- [9. What is a composite key?](#9-what-is-a-composite-key)

**🔴 Hard**
- [10. What's the difference between a clustered and a non-clustered index, conceptually?](#10-whats-the-difference-between-a-clustered-and-a-non-clustered-index-conceptually)
- [11. What is referential integrity, and how does MySQL enforce it?](#11-what-is-referential-integrity-and-how-does-mysql-enforce-it)
- [12. What's the difference between `DELETE`, `TRUNCATE`, and `DROP`?](#12-whats-the-difference-between-delete-truncate-and-drop)
- [13. What is a view, and what are its limitations?](#13-what-is-a-view-and-what-are-its-limitations)
- [14. What's the difference between a relational database and a document database like MongoDB?](#14-whats-the-difference-between-a-relational-database-and-a-document-database-like-mongodb)

---

### 1. What is MySQL, and what type of database is it? 🟢

- An open-source **relational database management system (RDBMS)** — data is organized into tables with a fixed schema (columns and types defined upfront), and relationships between tables are expressed via foreign keys, queried using SQL.

[↑ Back to top](#table-of-contents)

---

### 2. What is a database, a table, and a schema in MySQL? 🟢

- **Database**: a named collection of related tables.
- **Table**: a structured collection of rows, each following the same set of defined columns.
- **Schema**: the structure itself — column names, types, constraints — that defines what a table (or database) looks like. In MySQL specifically, "schema" and "database" are often used interchangeably.

[↑ Back to top](#table-of-contents)

---

### 3. What is a primary key? 🟢

- A column (or set of columns) that **uniquely identifies** each row in a table — cannot contain `NULL`, and a table can have only one.

```sql
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
```

[↑ Back to top](#table-of-contents)

---

### 4. What is a foreign key? 🟢

- A column referencing the primary key of **another** table, establishing a relationship between the two and enforcing that the referenced value must actually exist (see Q11).

```sql
CREATE TABLE orders (
  id INT PRIMARY KEY,
  user_id INT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between a primary key and a unique key? 🟡

- Both enforce uniqueness, but a table can have only **one** primary key (which also disallows `NULL`), while it can have **multiple** unique keys, and unique keys **do** allow `NULL` (with most databases allowing multiple `NULL`s, since `NULL` isn't considered equal to another `NULL`).

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between `CHAR` and `VARCHAR`? 🟡

- `CHAR(n)`: **fixed-length** — always stores exactly `n` characters, padding shorter values with spaces. Slightly faster for fixed-size data (e.g. a country code).
- `VARCHAR(n)`: **variable-length** — stores only as many characters as actually given (plus a small length prefix), up to a max of `n`. More space-efficient for variable-length text.

[↑ Back to top](#table-of-contents)

---

### 7. What is a storage engine, and which is MySQL's default? 🟡

- The underlying component responsible for how data is actually **stored, indexed, and retrieved** on disk — different engines offer different tradeoffs (transaction support, locking granularity, crash recovery). **InnoDB** has been MySQL's default since version 5.5.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `InnoDB` and `MyISAM`? 🟡

- **InnoDB**: supports transactions (ACID), foreign keys, and row-level locking — the modern default, suitable for almost all use cases.
- **MyISAM**: no transaction support, no foreign keys, only table-level locking (writes block all reads/writes on the whole table) — legacy engine, largely superseded by InnoDB except in rare read-heavy, no-transaction-needed scenarios.

[↑ Back to top](#table-of-contents)

---

### 9. What is a composite key? 🟡

- A primary (or unique) key made up of **more than one column** together — used when no single column alone uniquely identifies a row, but a combination does (e.g. `(student_id, course_id)` in an enrollment table).

```sql
CREATE TABLE enrollments (
  student_id INT, course_id INT,
  PRIMARY KEY (student_id, course_id)
);
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between a clustered and a non-clustered index, conceptually? 🔴

- **Clustered index**: determines the **physical order** rows are stored in on disk — there can be only one per table (in InnoDB, it's built on the primary key automatically). Looking up by it goes straight to the row.
- **Non-clustered (secondary) index**: a separate structure pointing **back** to the row's location (in InnoDB, via the primary key value) — a table can have many. A lookup via a secondary index requires an extra step to fetch the actual row, unless the query is fully covered by the index alone (see [Indexing](indexing.md)).

[↑ Back to top](#table-of-contents)

---

### 11. What is referential integrity, and how does MySQL enforce it? 🔴

- The guarantee that a foreign key value always points to a **row that actually exists** in the referenced table — MySQL (with InnoDB) enforces this automatically: inserting/updating a row with a foreign key value that doesn't exist in the parent table is rejected, and by default, deleting a referenced parent row is also rejected unless `ON DELETE CASCADE`/`SET NULL` is explicitly configured.

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between `DELETE`, `TRUNCATE`, and `DROP`? 🔴

- `DELETE`: removes rows matching a condition (or all rows if none given), row-by-row, logged and (slowly) reversible within a transaction, fires triggers, can be rolled back.
- `TRUNCATE`: removes **all** rows at once by deallocating the table's data pages — much faster, but can't filter with `WHERE`, generally can't be rolled back the same way (DDL-like behavior in many configurations), and resets auto-increment counters.
- `DROP`: removes the **entire table** (structure included), not just its rows.

[↑ Back to top](#table-of-contents)

---

### 13. What is a view, and what are its limitations? 🔴

- A **virtual table** defined by a stored `SELECT` query — looks and queries like a real table, but doesn't store data itself; the underlying query runs each time the view is queried. Limitations: can be slower than a real table for complex underlying queries (no independent indexing of the view itself), and updating through a view is only possible for relatively simple, single-table views.

```sql
CREATE VIEW active_users AS SELECT id, name FROM users WHERE status = 'active';
SELECT * FROM active_users;
```

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between a relational database and a document database like MongoDB? 🔴

- **Relational (MySQL)**: fixed schema enforced upfront, data normalized across related tables, joined via foreign keys — strong consistency guarantees and mature multi-row transactions are core strengths.
- **Document (MongoDB)**: flexible schema, related data commonly **embedded** within a single document rather than joined — see [MongoDB › Core Concepts](../mongodb/core-concepts.md#11-whats-the-difference-between-sql-and-nosql-databases-and-when-would-you-choose-one-over-the-other) for the fuller tradeoff and when to choose each.

> [!IMPORTANT]
> **Follow-up questions:**
> - For an e-commerce order history feature, would you lean toward MySQL or MongoDB, and what about the data's shape/access pattern drives that choice?

[↑ Back to top](#table-of-contents)

---
