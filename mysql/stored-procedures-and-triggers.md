# Stored Procedures & Triggers

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a stored procedure?](#1-what-is-a-stored-procedure)
- [2. What is a trigger?](#2-what-is-a-trigger)
- [3. What is a function, and how does it differ from a stored procedure?](#3-what-is-a-function-and-how-does-it-differ-from-a-stored-procedure)

**🟡 Medium**
- [4. What are the advantages of using stored procedures?](#4-what-are-the-advantages-of-using-stored-procedures)
- [5. What are the disadvantages/downsides of stored procedures?](#5-what-are-the-disadvantagesdownsides-of-stored-procedures)
- [6. What are the trigger timing/event combinations available in MySQL?](#6-what-are-the-trigger-timingevent-combinations-available-in-mysql)
- [7. How do you pass parameters into a stored procedure?](#7-how-do-you-pass-parameters-into-a-stored-procedure)
- [8. What is the `NEW` and `OLD` keyword inside a trigger?](#8-what-is-the-new-and-old-keyword-inside-a-trigger)

**🔴 Hard**
- [9. What's a common pitfall of using triggers for business logic?](#9-whats-a-common-pitfall-of-using-triggers-for-business-logic)
- [10. Can a trigger call a stored procedure, and what restrictions apply?](#10-can-a-trigger-call-a-stored-procedure-and-what-restrictions-apply)
- [11. How do you handle errors within a stored procedure?](#11-how-do-you-handle-errors-within-a-stored-procedure)
- [12. When would you choose application-level logic over a stored procedure/trigger for the same task?](#12-when-would-you-choose-application-level-logic-over-a-stored-proceduretrigger-for-the-same-task)

---

### 1. What is a stored procedure? 🟢

- A named, precompiled block of SQL statements stored **in the database** itself, which can be invoked by name with parameters — like a function defined inside the database rather than in application code.

```sql
DELIMITER //
CREATE PROCEDURE GetUserById(IN userId INT)
BEGIN
  SELECT * FROM users WHERE id = userId;
END //
DELIMITER ;

CALL GetUserById(42);
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a trigger? 🟢

- A block of SQL that automatically executes in response to a specific event (`INSERT`, `UPDATE`, `DELETE`) on a table — runs implicitly as part of that event, without the application explicitly calling anything.

```sql
CREATE TRIGGER before_user_insert
BEFORE INSERT ON users FOR EACH ROW
SET NEW.created_at = NOW();
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a function, and how does it differ from a stored procedure? 🟢

- A stored **function** must return exactly one value and can be used directly inside a SQL expression (e.g. in a `SELECT`) — a stored **procedure** can return zero, one, or multiple result sets/output parameters, but can't be embedded inside an expression the way a function can.

```sql
SELECT CalculateTax(amount) FROM orders; -- function used inline
```

[↑ Back to top](#table-of-contents)

---

### 4. What are the advantages of using stored procedures? 🟡

- Reduced network round trips (multiple statements execute server-side in one call), centralized/reusable logic shared across multiple applications hitting the same database, and a layer of abstraction that can let you change underlying table structure without changing every application's queries.

[↑ Back to top](#table-of-contents)

---

### 5. What are the disadvantages/downsides of stored procedures? 🟡

- Business logic split between application code and the database makes the system **harder to reason about, version, and test** as a whole; database-specific stored procedure syntax (MySQL's procedural SQL dialect) reduces portability to a different database; and they're harder to unit test and debug than equivalent application-layer code with proper tooling.

[↑ Back to top](#table-of-contents)

---

### 6. What are the trigger timing/event combinations available in MySQL? 🟡

- Timing: `BEFORE` or `AFTER`. Event: `INSERT`, `UPDATE`, or `DELETE`. Six combinations total (`BEFORE INSERT`, `AFTER INSERT`, `BEFORE UPDATE`, etc.) — `BEFORE` triggers can modify the row before it's written (e.g. setting a timestamp), `AFTER` triggers react to a change that's already committed to that statement (e.g. writing an audit log entry).

[↑ Back to top](#table-of-contents)

---

### 7. How do you pass parameters into a stored procedure? 🟡

- `IN` (input only, default), `OUT` (output — the procedure sets it, caller reads it after), `INOUT` (both).

```sql
CREATE PROCEDURE GetUserCount(OUT total INT)
BEGIN
  SELECT COUNT(*) INTO total FROM users;
END;
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the `NEW` and `OLD` keyword inside a trigger? 🟡

- `NEW`: refers to the row's value **after** the triggering change (available in `INSERT`/`UPDATE` triggers).
- `OLD`: refers to the row's value **before** the change (available in `UPDATE`/`DELETE` triggers). An `UPDATE` trigger has access to both, letting you compare what changed.

[↑ Back to top](#table-of-contents)

---

### 9. What's a common pitfall of using triggers for business logic? 🔴

- Triggers fire **implicitly** — a developer looking at application code (or even a plain SQL statement) has no visible indication that a trigger will also run as a side effect, making behavior **harder to trace** and debug, especially when multiple triggers/chained triggers interact in non-obvious ways. Overuse of triggers for core business logic tends to make a system's actual behavior scattered and difficult to follow compared to keeping that logic explicit in application code.

[↑ Back to top](#table-of-contents)

---

### 10. Can a trigger call a stored procedure, and what restrictions apply? 🔴

- Yes, but a trigger **cannot** use statements that explicitly start/commit/rollback a transaction (`START TRANSACTION`, `COMMIT`, `ROLLBACK`) within itself, since it's already running as part of the triggering statement's own transaction — and care is needed to avoid recursive trigger chains (a trigger on table A modifying table B, which itself has a trigger modifying table A again) leading to infinite loops or unexpected cascading effects.

[↑ Back to top](#table-of-contents)

---

### 11. How do you handle errors within a stored procedure? 🔴

- Define a `DECLARE ... HANDLER` to catch specific conditions (like a constraint violation) and react accordingly (log it, roll back, re-raise) — without one, an unhandled error simply aborts the procedure.

```sql
DECLARE EXIT HANDLER FOR SQLEXCEPTION
BEGIN
  ROLLBACK;
  RESIGNAL; -- re-raise the error to the caller after rolling back
END;
```

[↑ Back to top](#table-of-contents)

---

### 12. When would you choose application-level logic over a stored procedure/trigger for the same task? 🔴

- Favor application-level logic for most **business logic** — it's easier to version control alongside the rest of the codebase, unit test with standard tooling, and keep portable across different database choices. Reach for stored procedures/triggers mainly for things genuinely tied to data integrity at the database layer itself (auditing every change regardless of which application made it, enforcing invariants that must hold no matter which client touches the data) rather than as a general substitute for application code.

> [!IMPORTANT]
> **Follow-up questions:**
> - If your application has multiple separate services all writing to the same MySQL database, does that change the calculus toward favoring triggers for cross-cutting concerns like auditing?

[↑ Back to top](#table-of-contents)

---
