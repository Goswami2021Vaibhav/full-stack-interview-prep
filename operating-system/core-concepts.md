# Core Concepts

_Part of [Operating System](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is an operating system, and what are its main functions?](#1-what-is-an-operating-system-and-what-are-its-main-functions)
- [2. What is the kernel?](#2-what-is-the-kernel)
- [3. What's the difference between a process and a program?](#3-whats-the-difference-between-a-process-and-a-program)
- [4. What is a system call?](#4-what-is-a-system-call)

**🟡 Medium**
- [5. What's the difference between user mode and kernel mode?](#5-whats-the-difference-between-user-mode-and-kernel-mode)
- [6. What are multitasking, multiprogramming, and multiprocessing?](#6-what-are-multitasking-multiprogramming-and-multiprocessing)
- [7. What is an interrupt, and how does the OS handle one?](#7-what-is-an-interrupt-and-how-does-the-os-handle-one)
- [8. What's the difference between monolithic and microkernel architectures?](#8-whats-the-difference-between-monolithic-and-microkernel-architectures)
- [9. What is a shell, and how does it relate to the kernel?](#9-what-is-a-shell-and-how-does-it-relate-to-the-kernel)

**🔴 Hard**
- [10. What is a context switch, and what's its overhead?](#10-what-is-a-context-switch-and-whats-its-overhead)
- [11. What's the difference between a real-time OS and a general-purpose OS?](#11-whats-the-difference-between-a-real-time-os-and-a-general-purpose-os)
- [12. What happens during an OS's boot process, at a high level?](#12-what-happens-during-an-oss-boot-process-at-a-high-level)
- [13. Why is the kernel/user mode separation important for system stability and security?](#13-why-is-the-kerneluser-mode-separation-important-for-system-stability-and-security)
- [14. What's the difference between cooperative and preemptive multitasking?](#14-whats-the-difference-between-cooperative-and-preemptive-multitasking)

---

### 1. What is an operating system, and what are its main functions? 🟢

- Software that manages a computer's hardware resources and provides a common platform for applications to run on — its core functions: process management, memory management, file system management, device/I/O management, and providing a user interface (shell/GUI).

[↑ Back to top](#table-of-contents)

---

### 2. What is the kernel? 🟢

- The core part of the OS that runs with full hardware privileges — manages memory, schedules processes, and handles communication between software and hardware. Everything else (shells, applications) runs on top of, and ultimately depends on, the kernel.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between a process and a program? 🟢

- A **program** is a static set of instructions stored on disk (an executable file) — it does nothing on its own.
- A **process** is a program **in execution** — an active instance with its own memory, state, and resources, created when the program is run.

[↑ Back to top](#table-of-contents)

---

### 4. What is a system call? 🟢

- The mechanism by which a user-mode program **requests a service** from the kernel (reading a file, allocating memory, creating a process) — since user-mode code can't directly access hardware/privileged operations, it must go through this controlled gateway into kernel mode.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between user mode and kernel mode? 🟡

- **User mode**: restricted privilege level — regular applications run here, with no direct access to hardware or critical memory regions, preventing a buggy/malicious program from crashing the whole system.
- **Kernel mode**: full privilege level — the OS kernel itself runs here, with unrestricted access to hardware and memory. Switching between the two (via a system call) is deliberately controlled and adds a small overhead.

[↑ Back to top](#table-of-contents)

---

### 6. What are multitasking, multiprogramming, and multiprocessing? 🟡

- **Multiprogramming**: multiple programs reside in memory at once, so the CPU can switch to another whenever one is waiting on I/O, keeping the CPU busy.
- **Multitasking**: extends multiprogramming with **time-slicing**, giving the illusion of multiple programs running simultaneously on a single CPU by rapidly switching between them.
- **Multiprocessing**: uses **multiple physical CPUs/cores**, allowing genuinely simultaneous execution of multiple processes, not just an illusion via time-slicing.

[↑ Back to top](#table-of-contents)

---

### 7. What is an interrupt, and how does the OS handle one? 🟡

- A signal from hardware (or software) that requires the CPU's **immediate attention**, pausing whatever it's currently doing — the CPU saves its current state, runs a dedicated **interrupt handler** to deal with the event (a key press, a completed disk read), then resumes the original task. This is how the OS responds to asynchronous hardware events without constantly polling for them.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between monolithic and microkernel architectures? 🟡

- **Monolithic kernel**: most OS services (file system, device drivers, networking) run together in kernel space — faster (fewer mode-switch boundaries), but a bug in any component can crash the entire kernel.
- **Microkernel**: only the bare minimum (basic scheduling, IPC, memory management) runs in kernel space — other services run as separate user-space processes, communicating via message passing. More fault-isolated and modular, at the cost of performance overhead from the added communication.

[↑ Back to top](#table-of-contents)

---

### 9. What is a shell, and how does it relate to the kernel? 🟡

- A user-facing program (command-line or graphical) that interprets user commands and translates them into system calls/program executions — it's a layer **on top of** the kernel, not part of it; you can swap shells (`bash`, `zsh`, `fish`) without touching the kernel underneath.

[↑ Back to top](#table-of-contents)

---

### 10. What is a context switch, and what's its overhead? 🔴

- Saving the complete state (registers, program counter, memory mappings) of a currently-running process/thread, and restoring a different one's saved state so it can resume execution — necessary every time the CPU switches which process/thread it's executing. Overhead includes the save/restore work itself, plus indirect costs like CPU cache and TLB invalidation (the new process's memory access patterns "cool down" the cache the previous one had warmed up), which is why excessive context switching can noticeably hurt overall throughput.

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between a real-time OS and a general-purpose OS? 🔴

- **General-purpose OS** (Windows, Linux desktop): optimizes for overall throughput and fairness across many tasks — occasional unpredictable delays are acceptable.
- **Real-time OS (RTOS)**: guarantees a task completes within a **strict, predictable deadline** — used in embedded/safety-critical systems (industrial control, avionics) where a late response (even by milliseconds) can be a failure, prioritizing deterministic timing over overall average throughput.

[↑ Back to top](#table-of-contents)

---

### 12. What happens during an OS's boot process, at a high level? 🔴

1. **Firmware/BIOS-UEFI**: performs hardware checks (POST), then locates a bootable device.
2. **Bootloader**: a small program loaded from disk that locates and loads the actual OS kernel into memory.
3. **Kernel initialization**: the kernel sets up core data structures, device drivers, and memory management.
4. **Init process**: the kernel starts the first user-space process (`init`/`systemd` on Linux), which in turn starts the rest of the system's services.

[↑ Back to top](#table-of-contents)

---

### 13. Why is the kernel/user mode separation important for system stability and security? 🔴

- Without it, **any** user program could directly manipulate hardware, overwrite other processes' memory, or disable critical OS protections — a single bug or malicious program could crash or compromise the entire system. The separation confines user code to a sandboxed mode where it must go through the controlled, validated system call interface to request anything privileged, letting the kernel enforce checks (permissions, resource limits) before honoring the request.

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between cooperative and preemptive multitasking? 🔴

- **Cooperative multitasking**: a running process must **voluntarily yield** control back to the OS/other processes — a single misbehaving or infinite-looping process can freeze the entire system, since nothing forces it to give up the CPU.
- **Preemptive multitasking** (used by virtually all modern general-purpose OSes): the OS can **forcibly interrupt** a running process after its allotted time slice, regardless of whether it's finished — ensures no single process can monopolize the CPU indefinitely.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why did early operating systems favor cooperative multitasking despite its obvious risk, and what changed to make preemptive multitasking the universal standard?

[↑ Back to top](#table-of-contents)

---
