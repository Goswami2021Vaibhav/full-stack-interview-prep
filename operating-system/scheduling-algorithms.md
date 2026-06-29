# Scheduling Algorithms

_Part of [Operating System](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is CPU scheduling, and why is it needed?](#1-what-is-cpu-scheduling-and-why-is-it-needed)
- [2. What is a scheduler?](#2-what-is-a-scheduler)
- [3. What is First Come First Served (FCFS) scheduling?](#3-what-is-first-come-first-served-fcfs-scheduling)

**🟡 Medium**
- [4. What is Shortest Job First (SJF) scheduling?](#4-what-is-shortest-job-first-sjf-scheduling)
- [5. What is Round Robin scheduling?](#5-what-is-round-robin-scheduling)
- [6. What is Priority Scheduling, and what problem can it cause?](#6-what-is-priority-scheduling-and-what-problem-can-it-cause)
- [7. What's the difference between preemptive and non-preemptive scheduling?](#7-whats-the-difference-between-preemptive-and-non-preemptive-scheduling)
- [8. What is aging, and how does it solve starvation?](#8-what-is-aging-and-how-does-it-solve-starvation)

**🔴 Hard**
- [9. What is Multilevel Queue scheduling?](#9-what-is-multilevel-queue-scheduling)
- [10. What is Multilevel Feedback Queue scheduling?](#10-what-is-multilevel-feedback-queue-scheduling)
- [11. What metrics are used to evaluate scheduling algorithms?](#11-what-metrics-are-used-to-evaluate-scheduling-algorithms)
- [12. How would you choose a scheduling algorithm for a real-time system vs. a general-purpose OS?](#12-how-would-you-choose-a-scheduling-algorithm-for-a-real-time-system-vs-a-general-purpose-os)

---

### 1. What is CPU scheduling, and why is it needed? 🟢

- The process of deciding **which** ready process/thread gets to run on the CPU **next**, and for how long — needed because there are typically far more runnable processes than available CPU cores, so the OS must decide an order that balances fairness, responsiveness, and overall throughput.

[↑ Back to top](#table-of-contents)

---

### 2. What is a scheduler? 🟢

- The OS component that implements the actual scheduling algorithm/policy — invoked whenever a decision about which process to run next needs to be made (a process finishes, blocks, or its time slice expires).

[↑ Back to top](#table-of-contents)

---

### 3. What is First Come First Served (FCFS) scheduling? 🟢

- Processes are executed strictly in the order they **arrive**, with no preemption — simple to implement, but a long process at the front of the queue makes every process behind it wait, even if they're individually much shorter (the "convoy effect").

[↑ Back to top](#table-of-contents)

---

### 4. What is Shortest Job First (SJF) scheduling? 🟡

- Always picks the **shortest** remaining job next — minimizes average waiting time mathematically (provably optimal for that specific metric), but requires knowing each job's execution time in advance, which is rarely available in practice, and can starve longer jobs if short jobs keep arriving (Q8).

[↑ Back to top](#table-of-contents)

---

### 5. What is Round Robin scheduling? 🟡

- Each process gets a small, fixed **time slice (quantum)** in turn, cycling through the ready queue — if a process doesn't finish within its slice, it's preempted and moved to the back of the queue. Fair and responsive (every process gets CPU time regularly), but the time-slice size matters: too short causes excessive context-switching overhead, too long degrades into behaving like FCFS.

[↑ Back to top](#table-of-contents)

---

### 6. What is Priority Scheduling, and what problem can it cause? 🟡

- Each process is assigned a priority, and the **highest-priority** ready process runs next — can cause **starvation**: a low-priority process might never run at all if higher-priority processes keep arriving continuously.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between preemptive and non-preemptive scheduling? 🟡

- **Non-preemptive**: once a process starts running, it keeps the CPU until it **voluntarily** finishes or blocks (FCFS, non-preemptive SJF).
- **Preemptive**: the scheduler can **forcibly interrupt** a running process (when a higher-priority process arrives, or a time slice expires) to give the CPU to someone else (Round Robin, preemptive Priority Scheduling).

[↑ Back to top](#table-of-contents)

---

### 8. What is aging, and how does it solve starvation? 🟡

- Gradually **increases** the priority of a process the longer it waits in the ready queue — ensures that even an initially low-priority process will **eventually** become high-priority enough to get scheduled, directly solving the starvation problem that plain priority scheduling (Q6) can suffer from.

[↑ Back to top](#table-of-contents)

---

### 9. What is Multilevel Queue scheduling? 🔴

- Separates processes into **multiple distinct queues** based on a fixed characteristic (e.g. system processes, interactive processes, batch processes), each queue with its own scheduling algorithm and (often) its own fixed priority relative to other queues — but a process is permanently assigned to one queue, with no movement between them, which can be inflexible if a process's behavior changes over time.

[↑ Back to top](#table-of-contents)

---

### 10. What is Multilevel Feedback Queue scheduling? 🔴

- Like Multilevel Queue, but processes can **move between queues** based on observed behavior — e.g. a process that uses its full time slice repeatedly (CPU-bound) gets demoted to a lower-priority, longer-time-slice queue, while one that frequently blocks for I/O (interactive) stays in a higher-priority, short-time-slice queue. This adaptiveness is what makes it the most flexible and commonly approximated approach in real general-purpose OS schedulers.

[↑ Back to top](#table-of-contents)

---

### 11. What metrics are used to evaluate scheduling algorithms? 🔴

- **Turnaround time**: total time from a process's arrival to its completion.
- **Waiting time**: total time spent in the ready queue, not running.
- **Response time**: time from arrival until the **first** time it gets CPU time (matters most for interactive systems).
- **Throughput**: number of processes completed per unit time. Different algorithms optimize for different combinations of these, often trading one off against another.

[↑ Back to top](#table-of-contents)

---

### 12. How would you choose a scheduling algorithm for a real-time system vs. a general-purpose OS? 🔴

- **General-purpose OS**: prioritize overall fairness and reasonable average response time across many varied tasks — something like Multilevel Feedback Queue, balancing interactive responsiveness with throughput for batch work.
- **Real-time system**: prioritize **deterministic, guaranteed** deadlines for critical tasks over average-case performance — algorithms like Rate Monotonic Scheduling or Earliest Deadline First, which explicitly reason about task deadlines rather than just fairness/throughput (see [Core Concepts](core-concepts.md#11-whats-the-difference-between-a-real-time-os-and-a-general-purpose-os)).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why would Shortest Job First, despite being mathematically optimal for minimizing average waiting time, rarely be usable as-is in a real production scheduler?

[↑ Back to top](#table-of-contents)

---
