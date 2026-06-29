# Process Management

_Part of [Operating System](README.md) interview notes._

> Node.js-specific child process/worker thread mechanics are covered in [Node.js › Child Processes](../nodejs/child-processes.md). This file covers the underlying OS concepts those APIs build on.

## Table of Contents

**🟢 Easy**
- [1. What is a process?](#1-what-is-a-process)
- [2. What is a thread?](#2-what-is-a-thread)
- [3. What's the difference between a process and a thread?](#3-whats-the-difference-between-a-process-and-a-thread)

**🟡 Medium**
- [4. What are the states in a process's lifecycle?](#4-what-are-the-states-in-a-processs-lifecycle)
- [5. What is a Process Control Block (PCB)?](#5-what-is-a-process-control-block-pcb)
- [6. What happens when a process forks a child process?](#6-what-happens-when-a-process-forks-a-child-process)
- [7. What is an orphan process, and what is a zombie process?](#7-what-is-an-orphan-process-and-what-is-a-zombie-process)
- [8. What's the difference between multithreading and multiprocessing for an application?](#8-whats-the-difference-between-multithreading-and-multiprocessing-for-an-application)

**🔴 Hard**
- [9. What is inter-process communication (IPC), and what mechanisms support it?](#9-what-is-inter-process-communication-ipc-and-what-mechanisms-support-it)
- [10. What's the difference between user-level threads and kernel-level threads?](#10-whats-the-difference-between-user-level-threads-and-kernel-level-threads)
- [11. What is thread pooling, and why is it useful?](#11-what-is-thread-pooling-and-why-is-it-useful)
- [12. How does copy-on-write optimize process forking?](#12-how-does-copy-on-write-optimize-process-forking)
- [13. What's the overhead difference between creating a thread vs. a process?](#13-whats-the-overhead-difference-between-creating-a-thread-vs-a-process)

---

### 1. What is a process? 🟢

- An independent, running instance of a program — with its **own** memory space, file handles, and execution state, isolated from other processes by the OS.

[↑ Back to top](#table-of-contents)

---

### 2. What is a thread? 🟢

- A unit of execution **within** a process — multiple threads in the same process **share** that process's memory space, but each has its own program counter, stack, and registers.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between a process and a thread? 🟢

- A **process** has its own isolated memory; communicating between processes requires explicit IPC (Q9). A **thread** shares memory with other threads in the same process, so threads can communicate simply by reading/writing shared variables — convenient, but requires careful synchronization to avoid race conditions (see [Concurrency & Synchronization](concurrency-and-synchronization.md)).

[↑ Back to top](#table-of-contents)

---

### 4. What are the states in a process's lifecycle? 🟡

- **New**: being created.
- **Ready**: waiting for CPU time, able to run.
- **Running**: actively executing on a CPU.
- **Waiting/Blocked**: waiting on an event (I/O completion) before it can proceed.
- **Terminated**: finished execution.

[↑ Back to top](#table-of-contents)

---

### 5. What is a Process Control Block (PCB)? 🟡

- A data structure the OS maintains **per process**, holding everything needed to manage and resume it — process state, program counter, register values, memory allocation info, and scheduling priority. It's what gets saved/restored during a context switch.

[↑ Back to top](#table-of-contents)

---

### 6. What happens when a process forks a child process? 🟡

- The OS creates a **new** process that's initially a near-identical copy of the parent (same memory contents, same open file descriptors) — historically a full memory copy, though modern systems optimize this via copy-on-write (Q12). The child then typically calls `exec()` to replace its memory image with a different program entirely (the common fork-then-exec pattern for launching a new program).

[↑ Back to top](#table-of-contents)

---

### 7. What is an orphan process, and what is a zombie process? 🟡

- **Orphan process**: a child process whose **parent terminated** before it did — typically "adopted" by the OS's init process so it still has a parent to report back to.
- **Zombie process**: a process that has **finished executing**, but its entry still exists in the process table because its parent hasn't yet read its exit status (via `wait()`) — it's "dead" but not yet fully cleaned up.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between multithreading and multiprocessing for an application? 🟡

- **Multithreading**: multiple threads within **one** process — lighter weight, fast communication via shared memory, but a crash in one thread can take down the entire process, and threads must be carefully synchronized.
- **Multiprocessing**: multiple **separate** processes — heavier weight to create/communicate between, but fully isolated, so one process crashing doesn't directly affect the others (see [Node.js › Child Processes](../nodejs/child-processes.md#4-what-are-worker-threads-and-how-do-they-differ-from-child-processes) for this same tradeoff applied in Node.js specifically).

[↑ Back to top](#table-of-contents)

---

### 9. What is inter-process communication (IPC), and what mechanisms support it? 🔴

- Mechanisms allowing **separate processes** (with isolated memory) to exchange data — since they can't simply share variables like threads can. Common mechanisms: **pipes** (one-directional byte stream between related processes), **message queues**, **shared memory** (an explicitly-mapped region both processes can access, fastest but requires manual synchronization), and **sockets** (works even across different machines, not just within one).

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between user-level threads and kernel-level threads? 🔴

- **User-level threads**: managed entirely by a runtime library in user space, with the kernel unaware of their existence (it only sees one process) — fast to create/switch between, but if one thread makes a blocking system call, the **whole process** can block, since the kernel doesn't know there are other runnable threads inside it.
- **Kernel-level threads**: managed and scheduled directly by the OS kernel — the kernel can run other threads in the same process even if one blocks, but creation/context-switching is more expensive since it requires kernel involvement.

[↑ Back to top](#table-of-contents)

---

### 11. What is thread pooling, and why is it useful? 🔴

- Maintaining a **pre-created pool** of reusable threads, rather than creating/destroying a new thread for every single task — avoids the (non-trivial) overhead of thread creation/teardown for short-lived tasks, and caps the total number of concurrently running threads, preventing resource exhaustion under heavy load.

[↑ Back to top](#table-of-contents)

---

### 12. How does copy-on-write optimize process forking? 🔴

- Rather than immediately duplicating the parent's **entire** memory for the child at fork time, both processes initially **share** the same physical memory pages, marked read-only — only when **either** process actually tries to **write** to a shared page does the OS step in and make a real, separate copy of just that page. This makes `fork()` dramatically cheaper in the common case where the child immediately calls `exec()` (replacing its memory anyway) without ever needing most of that copied memory at all.

[↑ Back to top](#table-of-contents)

---

### 13. What's the overhead difference between creating a thread vs. a process? 🔴

- Creating a **process** requires the OS to set up an entirely new, isolated address space, file descriptor table, and other per-process resources — comparatively expensive. Creating a **thread** within an existing process just needs a new stack and execution context, **reusing** the same address space and resources the process already has — significantly cheaper and faster, which is why thread pools (Q11) and multithreaded designs are favored when communication overhead between concurrent units of work needs to be minimal.

> [!IMPORTANT]
> **Follow-up questions:**
> - Given that threads share memory and processes don't, why would you still sometimes choose multiple processes over multiple threads despite the higher creation overhead?

[↑ Back to top](#table-of-contents)

---
