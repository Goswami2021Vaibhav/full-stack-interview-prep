# Concurrency & Synchronization

_Part of [Operating System](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is concurrency?](#1-what-is-concurrency)
- [2. What is a race condition?](#2-what-is-a-race-condition)
- [3. What is a critical section?](#3-what-is-a-critical-section)

**🟡 Medium**
- [4. What is mutual exclusion?](#4-what-is-mutual-exclusion)
- [5. What is a mutex?](#5-what-is-a-mutex)
- [6. What is a semaphore?](#6-what-is-a-semaphore)
- [7. What's the difference between a mutex and a semaphore?](#7-whats-the-difference-between-a-mutex-and-a-semaphore)
- [8. What's the difference between a binary semaphore and a counting semaphore?](#8-whats-the-difference-between-a-binary-semaphore-and-a-counting-semaphore)

**🔴 Hard**
- [9. What is the producer-consumer problem?](#9-what-is-the-producer-consumer-problem)
- [10. What is the readers-writers problem?](#10-what-is-the-readers-writers-problem)
- [11. What is a spinlock, and when is it preferred over a mutex?](#11-what-is-a-spinlock-and-when-is-it-preferred-over-a-mutex)
- [12. What is a monitor, in the concurrency sense?](#12-what-is-a-monitor-in-the-concurrency-sense)
- [13. What's the difference between busy waiting and blocking synchronization?](#13-whats-the-difference-between-busy-waiting-and-blocking-synchronization)

---

### 1. What is concurrency? 🟢

- Multiple tasks making progress over the **same period of time** — either genuinely simultaneously (on multiple cores) or interleaved on a single core via fast context switching, creating the illusion of simultaneity.

[↑ Back to top](#table-of-contents)

---

### 2. What is a race condition? 🟢

- A bug that occurs when multiple threads/processes access **shared data concurrently**, and the final outcome depends on the unpredictable **timing/order** of their execution — e.g. two threads incrementing the same counter without synchronization can lose an update if their reads/writes interleave badly.

[↑ Back to top](#table-of-contents)

---

### 3. What is a critical section? 🟢

- A section of code that accesses **shared resources**, which must not be executed by more than one thread/process at the same time — protecting it (via a mutex/semaphore) is what prevents race conditions (Q2).

[↑ Back to top](#table-of-contents)

---

### 4. What is mutual exclusion? 🟡

- The guarantee that **only one** thread/process can be inside a critical section at any given moment — the core property that synchronization primitives (mutexes, semaphores) exist to enforce.

[↑ Back to top](#table-of-contents)

---

### 5. What is a mutex? 🟡

- A locking mechanism ensuring mutual exclusion — a thread must **acquire** the mutex (lock it) before entering a critical section, and **release** it (unlock) afterward; any other thread attempting to acquire it while locked must wait.

[↑ Back to top](#table-of-contents)

---

### 6. What is a semaphore? 🟡

- A more general synchronization primitive holding an internal **counter** — threads `wait()`/decrement it to "acquire" and `signal()`/increment it to "release"; if the counter reaches zero, further `wait()` calls block until it's incremented again. Can coordinate access among a **limited number of concurrent users**, not just strictly one at a time.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between a mutex and a semaphore? 🟡

- **Mutex**: specifically for mutual exclusion — exactly one owner at a time, and (in most implementations) only the thread that locked it can unlock it.
- **Semaphore**: a more general counter-based mechanism — can allow **multiple** concurrent holders (a counting semaphore, Q8), and any thread can signal it, not necessarily the one that originally waited on it. A mutex is essentially a binary semaphore with stricter ownership rules.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between a binary semaphore and a counting semaphore? 🟡

- **Binary semaphore**: counter only ever 0 or 1 — functions much like a mutex.
- **Counting semaphore**: counter can be any non-negative integer — used to limit access to a **pool** of N identical resources (e.g. a connection pool with 10 connections uses a semaphore initialized to 10).

[↑ Back to top](#table-of-contents)

---

### 9. What is the producer-consumer problem? 🔴

- A classic synchronization scenario: one or more **producer** threads generate data and place it into a shared, bounded buffer, while one or more **consumer** threads remove and process it — synchronization must ensure producers don't write to a full buffer, consumers don't read from an empty one, and concurrent access to the buffer itself doesn't corrupt it. Typically solved using semaphores tracking "empty slots" and "filled slots," plus a mutex protecting the buffer itself.

[↑ Back to top](#table-of-contents)

---

### 10. What is the readers-writers problem? 🔴

- A scenario where multiple threads want to **read** shared data (safe to do concurrently, since reads don't conflict with each other) but writes must be **exclusive** (no other reader or writer may access the data at the same time as a writer). Solutions must balance allowing concurrent reads for performance while still fully protecting writes — naive solutions can starve writers if readers keep arriving continuously (an instance of the general starvation problem).

[↑ Back to top](#table-of-contents)

---

### 11. What is a spinlock, and when is it preferred over a mutex? 🔴

- A lock where a thread **actively polls in a tight loop** ("spins") checking if the lock is free, rather than yielding the CPU and being put to sleep (which a mutex typically does when blocked). Preferred when the **expected wait time is very short** — spinning avoids the overhead of a full context switch (sleep + wake-up), which can be more expensive than just briefly busy-waiting; but spinning for a long wait wastes CPU cycles that could've gone to other work, so it's a poor choice when contention/wait times are long or unpredictable.

[↑ Back to top](#table-of-contents)

---

### 12. What is a monitor, in the concurrency sense? 🔴

- A higher-level synchronization construct that bundles a mutex **together with** the shared data and operations on it, so that mutual exclusion is automatically enforced by the language/runtime any time a monitor's methods are called — reduces the risk of forgetting to lock/unlock manually, since the synchronization is built into the construct itself rather than left to the programmer's discipline. Java's `synchronized` keyword is a practical example of monitor-style synchronization.

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between busy waiting and blocking synchronization? 🔴

- **Busy waiting** (spinlocks, Q11): the waiting thread keeps **actively running**, repeatedly checking a condition — consumes CPU the whole time it waits.
- **Blocking**: the waiting thread is put into a **sleep/waiting state** by the OS, consuming no CPU until it's explicitly woken up once the condition it's waiting for becomes true — better for longer or unpredictable waits, at the cost of the wake-up's context-switch overhead.

> [!IMPORTANT]
> **Follow-up questions:**
> - In a multi-core system, why might spinning very briefly actually outperform blocking, even though blocking seems like the more "efficient" approach in principle?

[↑ Back to top](#table-of-contents)

---
