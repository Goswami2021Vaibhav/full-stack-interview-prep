# Virtualization

_Part of [Operating System](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is virtualization?](#1-what-is-virtualization)
- [2. What is a virtual machine (VM)?](#2-what-is-a-virtual-machine-vm)
- [3. What is a hypervisor?](#3-what-is-a-hypervisor)

**🟡 Medium**
- [4. What's the difference between Type 1 and Type 2 hypervisors?](#4-whats-the-difference-between-type-1-and-type-2-hypervisors)
- [5. What's the difference between a VM and a container?](#5-whats-the-difference-between-a-vm-and-a-container)
- [6. What is paravirtualization vs. full virtualization?](#6-what-is-paravirtualization-vs-full-virtualization)
- [7. What is OS-level virtualization?](#7-what-is-os-level-virtualization)
- [8. What's the difference between a guest OS and a host OS?](#8-whats-the-difference-between-a-guest-os-and-a-host-os)

**🔴 Hard**
- [9. How does a container achieve isolation without a full guest OS?](#9-how-does-a-container-achieve-isolation-without-a-full-guest-os)
- [10. What is hardware-assisted virtualization?](#10-what-is-hardware-assisted-virtualization)
- [11. What are the performance tradeoffs between VMs and containers?](#11-what-are-the-performance-tradeoffs-between-vms-and-containers)
- [12. Why have containers become more popular than VMs for microservices deployment?](#12-why-have-containers-become-more-popular-than-vms-for-microservices-deployment)

---

### 1. What is virtualization? 🟢

- Creating a **virtual (software-based)** version of something that's normally physical hardware — a virtual CPU, virtual disk, virtual network interface — letting multiple isolated environments share the same underlying physical hardware.

[↑ Back to top](#table-of-contents)

---

### 2. What is a virtual machine (VM)? 🟢

- A fully isolated, software-emulated computer running its **own complete operating system**, on top of a host machine's physical hardware — behaves to its own software just like a real, separate physical machine would.

[↑ Back to top](#table-of-contents)

---

### 3. What is a hypervisor? 🟢

- The software layer that creates and manages VMs — allocates and isolates the underlying physical hardware (CPU, memory, disk, network) among the VMs running on top of it.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between Type 1 and Type 2 hypervisors? 🟡

- **Type 1 (bare-metal)**: runs **directly** on the host's hardware, with no underlying host OS beneath it (VMware ESXi, Hyper-V) — better performance, typically used in production/data center environments.
- **Type 2 (hosted)**: runs **as an application** on top of an existing host OS (VirtualBox, VMware Workstation) — more convenient for desktop/development use, but with an extra layer of overhead since it goes through the host OS.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between a VM and a container? 🟡

- **VM**: virtualizes the **hardware**, running a full separate guest OS (its own kernel) — strong isolation, but heavier (each VM duplicates an entire OS).
- **Container**: virtualizes at the **OS level**, sharing the **host's kernel** while isolating processes/filesystem/network at the OS level (namespaces, cgroups on Linux) — much lighter weight (no duplicated OS), starts almost instantly, but isolation is comparatively less strong than a full VM's hardware-level separation.

[↑ Back to top](#table-of-contents)

---

### 6. What is paravirtualization vs. full virtualization? 🟡

- **Full virtualization**: the guest OS runs completely **unmodified**, unaware it's virtualized — the hypervisor transparently intercepts and emulates all hardware access, which carries some performance overhead.
- **Paravirtualization**: the guest OS is **modified** to be aware it's running virtualized, and calls the hypervisor directly for certain operations instead of going through full hardware emulation — better performance, at the cost of requiring a specially modified guest OS.

[↑ Back to top](#table-of-contents)

---

### 7. What is OS-level virtualization? 🟡

- Virtualization implemented **within a single OS kernel**, isolating multiple user-space environments (containers) from each other without running separate kernels at all — this is what Docker/Linux containers fundamentally are, distinct from VM-based virtualization (Q5).

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between a guest OS and a host OS? 🟡

- **Host OS**: the operating system running directly on the **physical** machine.
- **Guest OS**: an operating system running **inside** a VM, managed by the hypervisor — from the guest OS's own perspective, it believes it's running on real hardware, unaware it's actually virtualized (unless paravirtualized, Q6).

[↑ Back to top](#table-of-contents)

---

### 9. How does a container achieve isolation without a full guest OS? 🔴

- Using Linux kernel features directly: **namespaces** isolate what a process can *see* (its own view of process IDs, network interfaces, mounted filesystems, hostname — each container believes it has the whole machine to itself), and **cgroups** (control groups) limit/account for what a process can *use* (CPU, memory, I/O limits) — together providing process-level isolation and resource control **without** running a separate kernel per container, all containers share the host's single kernel.

[↑ Back to top](#table-of-contents)

---

### 10. What is hardware-assisted virtualization? 🔴

- CPU features (Intel VT-x, AMD-V) specifically designed to support virtualization **in hardware** — allowing a hypervisor to run guest OS code with much less software-based emulation/interception overhead than older, purely software-based virtualization techniques required, significantly improving VM performance.

[↑ Back to top](#table-of-contents)

---

### 11. What are the performance tradeoffs between VMs and containers? 🔴

- **VMs**: higher overhead (a full duplicated OS per VM, slower boot times measured in tens of seconds/minutes), but stronger isolation — a kernel-level exploit in one VM's guest OS doesn't directly threaten the host or other VMs, since each has its own separate kernel.
- **Containers**: minimal overhead (sharing the host kernel, near-instant startup, much higher density of containers per physical machine) — but weaker isolation boundary, since a kernel vulnerability could potentially be exploited across containers sharing that same kernel.

[↑ Back to top](#table-of-contents)

---

### 12. Why have containers become more popular than VMs for microservices deployment? 🔴

- Microservices (see [System Design › Microservices vs. Monolith](../system-design/microservices-vs-monolith.md)) involve deploying **many small, independent services**, often scaled up/down frequently — containers' near-instant startup time and much lower per-instance resource overhead make this practical at scale in a way that spinning up a full VM per service instance generally wouldn't (each VM's overhead would dominate at that granularity), even though VMs remain preferred where the strongest possible isolation is required (e.g. running genuinely untrusted, multi-tenant workloads).

> [!IMPORTANT]
> **Follow-up questions:**
> - If container isolation is weaker than a VM's, why is it still considered acceptable for most production microservices deployments?

[↑ Back to top](#table-of-contents)

---
