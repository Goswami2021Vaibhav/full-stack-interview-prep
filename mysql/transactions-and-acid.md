# Transactions & ACID

_Part of [MySQL](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a transaction?](#1-what-is-a-transaction)
- [2. What does ACID stand for?](#2-what-does-acid-stand-for)
- [3. How do you start, commit, and roll back a transaction?](#3-how-do-you-start-commit-and-roll-back-a-transaction)

**🟡 Medium**
- [4. What is atomicity in the context of a transaction?](#4-what-is-atomicity-in-the-context-of-a-transaction)
- [5. What is isolation, and why is it needed?](#5-what-is-isolation-and-why-is-it-needed)
- [6. What is a dirty read?](#6-what-is-a-dirty-read)
- [7. What is a non-repeatable read?](#7-what-is-a-non-repeatable-read)
- [8. What is a phantom read?](#8-what-is-a-phantom-read)

**🔴 Hard**
- [9. What are the four standard transaction isolation levels?](#9-what-are-the-four-standard-transaction-isolation-levels)
- [10. What is MySQL's default isolation level, and what does it guarantee?](#10-what-is-mysqls-default-isolation-level-and-what-does-it-guarantee)
- [11. What is a deadlock, and how does MySQL handle one?](#11-what-is-a-deadlock-and-how-does-mysql-handle-one)
- [12. What is MVCC (Multi-Version Concurrency Control), and how does InnoDB use it?](#12-what-is-mvcc-multi-version-concurrency-control-and-how-does-innodb-use-it)
- [13. What's the difference between optimistic and pessimistic locking?](#13-whats-the-difference-between-optimistic-and-pessimistic-locking)

---

### 1. What is a transaction? 🟢

- A sequence of one or more SQL operations executed as a **single logical unit** — either every operation in it succeeds and is permanently applied, or none of them are, even if multiple separate statements are involved.

[↑ Back to top](#table-of-contents)

---

### 2. What does ACID stand for? 🟢

- **A**tomicity (all-or-nothing), **C**onsistency (the database moves between valid states, never violating its constraints), **I**solation (concurrent transactions don't interfere with each other's intermediate state), **D**urability (once committed, a transaction's changes survive even a crash).

[↑ Back to top](#table-of-contents)

---

### 3. How do you start, commit, and roll back a transaction? 🟢

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;   -- makes both changes permanent
-- or: ROLLBACK; -- undoes everything since START TRANSACTION
```

[↑ Back to top](#table-of-contents)

---

### 4. What is atomicity in the context of a transaction? 🟡

- The guarantee that a transaction's operations are applied **entirely or not at all** — if a transfer involves debiting one account and crediting another, atomicity ensures you never end up in a state where only one of the two happened (e.g. due to a crash mid-transaction).

[↑ Back to top](#table-of-contents)

---

### 5. What is isolation, and why is it needed? 🟡

- Controls how much one transaction's **in-progress, uncommitted** changes are visible to other concurrently running transactions. Without isolation, concurrent transactions could see each other's half-finished work, leading to inconsistent reads and subtle bugs — isolation levels (Q9) let you tune the tradeoff between strict correctness and concurrency/performance.

[↑ Back to top](#table-of-contents)

---

### 6. What is a dirty read? 🟡

- Reading data that another transaction has **modified but not yet committed** — if that other transaction later rolls back, you've read data that, strictly speaking, never actually existed in the database's committed history.

[↑ Back to top](#table-of-contents)

---

### 7. What is a non-repeatable read? 🟡

- Reading the **same row twice** within one transaction and getting **different** values, because another transaction committed an update to that row in between your two reads — the row "changed under you" mid-transaction.

[↑ Back to top](#table-of-contents)

---

### 8. What is a phantom read? 🟡

- Re-running the **same query** within one transaction and getting a **different set of rows** (new rows appearing or previously-matching rows disappearing), because another transaction inserted/deleted rows matching that query's condition in between — distinct from a non-repeatable read, which is about a row's *values* changing, not the *set of rows* matched changing.

[↑ Back to top](#table-of-contents)

---

### 9. What are the four standard transaction isolation levels? 🔴

From least to most strict:
1. **`READ UNCOMMITTED`**: allows dirty reads — almost never used in practice.
2. **`READ COMMITTED`**: prevents dirty reads, but allows non-repeatable reads and phantoms.
3. **`REPEATABLE READ`**: prevents dirty and non-repeatable reads; phantom reads are largely (though not absolutely, in edge cases) prevented too, thanks to InnoDB's MVCC snapshot (Q12).
4. **`SERIALIZABLE`**: fully prevents all three, by effectively making transactions behave as if executed one at a time — strongest guarantee, lowest concurrency.

[↑ Back to top](#table-of-contents)

---

### 10. What is MySQL's default isolation level, and what does it guarantee? 🔴

- **`REPEATABLE READ`** (InnoDB's default) — notably **stricter** than the SQL standard's minimum requirement for that level, since InnoDB's MVCC implementation (Q12) also prevents most phantom reads, which the standard doesn't strictly require at this level.

[↑ Back to top](#table-of-contents)

---

### 11. What is a deadlock, and how does MySQL handle one? 🔴

- Two (or more) transactions each hold a lock the **other** is waiting for, so neither can proceed — a circular wait. InnoDB **detects** this automatically and resolves it by choosing one transaction as the "victim," rolling it back (returning a deadlock error to that transaction's client) so the other can proceed. Applications should catch this error and **retry** the rolled-back transaction.

[↑ Back to top](#table-of-contents)

---

### 12. What is MVCC (Multi-Version Concurrency Control), and how does InnoDB use it? 🔴

- Instead of literally blocking readers while a writer is active, InnoDB keeps **multiple versions** of a row (via an internal undo log) — each transaction reads from a **consistent snapshot** as of when it started, regardless of what other transactions commit afterward. This is what lets `REPEATABLE READ` provide strong consistency **without** readers blocking writers (a major concurrency advantage over naive locking-only approaches).

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between optimistic and pessimistic locking? 🔴

- **Pessimistic locking**: acquire a lock **upfront** (`SELECT ... FOR UPDATE`) before reading data you intend to modify, blocking other transactions from touching it until you're done — safe, but reduces concurrency, especially if held for a long time.
- **Optimistic locking**: don't lock at all upfront — instead, check (typically via a `version` column) at **update time** whether the row changed since you read it, and reject/retry if so. Better concurrency for low-contention scenarios, but requires explicit application logic to detect and handle the conflict.

```sql
-- Optimistic: only updates if version hasn't changed since it was read
UPDATE accounts SET balance = 900, version = version + 1 WHERE id = 1 AND version = 5;
```

> [!IMPORTANT]
> **Follow-up questions:**
> - In a high-contention scenario (many transactions updating the same row constantly), would optimistic locking's "retry on conflict" approach actually perform better or worse than pessimistic locking?

[↑ Back to top](#table-of-contents)

---
