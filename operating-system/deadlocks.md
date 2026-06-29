# Deadlocks

_Part of [Operating System](README.md) interview notes._

> MySQL's specific InnoDB deadlock detection/handling is covered in [MySQL › Transactions & ACID](../mysql/transactions-and-acid.md#11-what-is-a-deadlock-and-how-does-mysql-handle-one). This file covers the general OS-level theory those mechanisms build on.

## Table of Contents

**🟢 Easy**
- [1. What is a deadlock?](#1-what-is-a-deadlock)
- [2. What are the four necessary conditions for deadlock (Coffman conditions)?](#2-what-are-the-four-necessary-conditions-for-deadlock-coffman-conditions)

**🟡 Medium**
- [3. What is deadlock prevention?](#3-what-is-deadlock-prevention)
- [4. What is deadlock avoidance?](#4-what-is-deadlock-avoidance)
- [5. What is deadlock detection?](#5-what-is-deadlock-detection)
- [6. What is deadlock recovery?](#6-what-is-deadlock-recovery)
- [7. What is a resource allocation graph?](#7-what-is-a-resource-allocation-graph)

**🔴 Hard**
- [8. What is the Banker's algorithm?](#8-what-is-the-bankers-algorithm)
- [9. What's the difference between deadlock and starvation?](#9-whats-the-difference-between-deadlock-and-starvation)
- [10. What's the difference between deadlock prevention and deadlock avoidance?](#10-whats-the-difference-between-deadlock-prevention-and-deadlock-avoidance)
- [11. How does breaking the circular wait condition prevent deadlock?](#11-how-does-breaking-the-circular-wait-condition-prevent-deadlock)
- [12. How do real systems (like databases) typically handle deadlocks in practice?](#12-how-do-real-systems-like-databases-typically-handle-deadlocks-in-practice)

---

### 1. What is a deadlock? 🟢

- A state where two or more processes/threads are each waiting for a resource **held by another** in the same set — none of them can proceed, and without external intervention, they'll wait forever.

[↑ Back to top](#table-of-contents)

---

### 2. What are the four necessary conditions for deadlock (Coffman conditions)? 🟢

1. **Mutual exclusion**: at least one resource is held in a non-shareable way.
2. **Hold and wait**: a process holds a resource while waiting for another.
3. **No preemption**: a resource can't be forcibly taken away from a process holding it.
4. **Circular wait**: a closed chain of processes, each waiting for a resource held by the next.

All **four** must hold simultaneously for deadlock to occur — breaking any **one** of them prevents it.

[↑ Back to top](#table-of-contents)

---

### 3. What is deadlock prevention? 🟡

- Designing the system so that at least **one** of the four necessary conditions (Q2) can **never** occur at all — e.g. requiring processes to request all needed resources upfront (eliminating hold-and-wait), or imposing a strict global ordering on resource acquisition (eliminating circular wait, Q11).

[↑ Back to top](#table-of-contents)

---

### 4. What is deadlock avoidance? 🟡

- Doesn't eliminate the possibility structurally, but **dynamically checks** before granting each resource request whether doing so could lead to deadlock, and denies/delays the request if so — requires advance knowledge of each process's maximum resource needs (the Banker's algorithm, Q8, is the classic example).

[↑ Back to top](#table-of-contents)

---

### 5. What is deadlock detection? 🟡

- Allows deadlocks to potentially occur, but periodically (or on-demand) **checks** for them — typically by analyzing a resource allocation graph (Q7) for cycles — and triggers recovery (Q6) once one is found.

[↑ Back to top](#table-of-contents)

---

### 6. What is deadlock recovery? 🟡

- Once a deadlock is detected, breaking it by either **terminating** one or more of the involved processes (releasing their held resources), or **preempting** resources from one of them and rolling it back — both options are disruptive, since the chosen "victim" process loses its progress.

[↑ Back to top](#table-of-contents)

---

### 7. What is a resource allocation graph? 🟡

- A graph with two types of nodes (processes and resources) and directed edges representing either "process is **requesting** resource" or "resource is **allocated to** process" — a **cycle** in this graph indicates a deadlock (in the simple case of single-instance resources).

[↑ Back to top](#table-of-contents)

---

### 8. What is the Banker's algorithm? 🔴

- A deadlock **avoidance** algorithm (Q4): before granting a resource request, it simulates whether the system would remain in a **"safe state"** afterward — meaning there still exists **some** order in which all processes could finish given their declared maximum future needs. If granting a request would lead to an unsafe state (where deadlock becomes possible), the request is denied/delayed instead. Requires knowing each process's maximum resource demand in advance, which limits its practicality outside of theoretical/controlled settings.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between deadlock and starvation? 🔴

- **Deadlock**: a set of processes are **permanently** stuck, each waiting on the others — guaranteed to never proceed without intervention.
- **Starvation**: a process is **repeatedly denied** the resources it needs (e.g. always losing out to higher-priority processes, Q6 of [Scheduling Algorithms](scheduling-algorithms.md#6-what-is-priority-scheduling-and-what-problem-can-it-cause)) — it **might** eventually get to run, just unfairly delayed indefinitely in the worst case, rather than provably stuck forever.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between deadlock prevention and deadlock avoidance? 🔴

- **Prevention**: a **static, structural** guarantee — redesign the system so one of the four necessary conditions can never arise at all, regardless of runtime behavior.
- **Avoidance**: a **dynamic, runtime** check — the system is theoretically still capable of deadlocking, but actively monitors and denies any specific request that would lead toward it, requiring ongoing computation (and advance knowledge of resource needs) at each request.

[↑ Back to top](#table-of-contents)

---

### 11. How does breaking the circular wait condition prevent deadlock? 🔴

- Impose a strict, global **ordering** on resource types (e.g. always acquire resource A before resource B, never the reverse) — if every process must request resources in the same fixed order, a circular chain of "waiting for the next" can **never** form, since it would require some process to violate the agreed ordering. This is one of the most practical, commonly-used real-world deadlock prevention techniques.

[↑ Back to top](#table-of-contents)

---

### 12. How do real systems (like databases) typically handle deadlocks in practice? 🔴

- Most production systems (including InnoDB, see [MySQL › Transactions](../mysql/transactions-and-acid.md#11-what-is-a-deadlock-and-how-does-mysql-handle-one)) favor **detection + recovery** over prevention/avoidance — true prevention/avoidance algorithms (strict lock ordering, Banker's algorithm) are often impractical to enforce across arbitrary application code, so it's generally simpler to let deadlocks occasionally happen, detect them automatically, and resolve by rolling back one of the involved transactions (the "victim"), letting the application retry.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is the Banker's algorithm rarely used in real general-purpose operating systems, despite being a well-known theoretical solution?

[↑ Back to top](#table-of-contents)

---
