# Core Concepts

_Part of [Networking](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is computer networking?](#1-what-is-computer-networking)
- [2. What is an IP address?](#2-what-is-an-ip-address)
- [3. What is a MAC address?](#3-what-is-a-mac-address)
- [4. What is a port number?](#4-what-is-a-port-number)

**🟡 Medium**
- [5. What's the difference between IPv4 and IPv6?](#5-whats-the-difference-between-ipv4-and-ipv6)
- [6. What is a subnet, and what is subnetting?](#6-what-is-a-subnet-and-what-is-subnetting)
- [7. What's the difference between a public and a private IP address?](#7-whats-the-difference-between-a-public-and-a-private-ip-address)
- [8. What is NAT (Network Address Translation)?](#8-what-is-nat-network-address-translation)
- [9. What is a socket?](#9-what-is-a-socket)

**🔴 Hard**
- [10. What's the difference between a router, a switch, and a hub?](#10-whats-the-difference-between-a-router-a-switch-and-a-hub)
- [11. What are latency, bandwidth, and throughput in a networking context?](#11-what-are-latency-bandwidth-and-throughput-in-a-networking-context)
- [12. What is packet switching, and how does it differ from circuit switching?](#12-what-is-packet-switching-and-how-does-it-differ-from-circuit-switching)
- [13. What is MTU (Maximum Transmission Unit), and what happens when a packet exceeds it?](#13-what-is-mtu-maximum-transmission-unit-and-what-happens-when-a-packet-exceeds-it)

---

### 1. What is computer networking? 🟢

- The practice of connecting multiple computing devices so they can communicate and share resources/data with each other, using agreed-upon rules (protocols) for how that communication happens.

[↑ Back to top](#table-of-contents)

---

### 2. What is an IP address? 🟢

- A numerical address identifying a device on a network, used to route data to it — the network-layer equivalent of a postal address.

```
192.168.1.10        (IPv4)
2001:0db8::1         (IPv6)
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a MAC address? 🟢

- A hardware-burned, globally unique identifier assigned to a network interface (a physical NIC, a Wi-Fi chip) — used for addressing on the **local** network segment (Layer 2), unlike an IP address which can change and routes across networks.

[↑ Back to top](#table-of-contents)

---

### 4. What is a port number? 🟢

- A number identifying a **specific application/service** on a device, alongside the IP address — lets a single machine run multiple network services simultaneously (e.g. a web server on port 80, an SSH server on port 22) while still being reachable at one IP.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between IPv4 and IPv6? 🟡

- **IPv4**: 32-bit addresses (~4.3 billion possible addresses) — the original, still widely used standard, but its address space has effectively run out given the number of internet-connected devices today.
- **IPv6**: 128-bit addresses — vastly larger address space, designed specifically to solve IPv4 exhaustion, plus some built-in improvements (simpler header, no need for NAT in many cases) — adoption has been gradual due to the need for backward compatibility with the existing IPv4 internet.

[↑ Back to top](#table-of-contents)

---

### 6. What is a subnet, and what is subnetting? 🟡

- A **subnet** is a logically segmented portion of a larger network, identified by a range of IP addresses. **Subnetting** is the act of dividing a network into these smaller segments — useful for organizing traffic, improving security (isolating segments), and using IP address space efficiently.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between a public and a private IP address? 🟡

- **Public IP**: globally unique and routable across the internet — assigned by an ISP/registry.
- **Private IP**: only meaningful **within** a local network (reserved ranges like `192.168.x.x`, `10.x.x.x`) — not routable on the public internet directly, and commonly shared by many private devices via NAT (Q8).

[↑ Back to top](#table-of-contents)

---

### 8. What is NAT (Network Address Translation)? 🟡

- Translates **private** IP addresses used inside a local network into a single (or small pool of) **public** IP address(es) when communicating with the outside internet, and translates responses back to the correct internal device — lets many devices on a home/office network share one public IP, which has also been a major practical mitigant for IPv4 address exhaustion.

[↑ Back to top](#table-of-contents)

---

### 9. What is a socket? 🟡

- The combination of an **IP address and a port number**, representing one endpoint of a network connection — a TCP connection, for instance, is uniquely identified by the pair of sockets (source and destination) involved.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between a router, a switch, and a hub? 🔴

- **Hub**: the simplest — broadcasts incoming data to **every** connected device, with no awareness of addressing (mostly obsolete now).
- **Switch**: operates at Layer 2, learns which device (by MAC address) is on which port, and forwards data **only** to the intended recipient within the same local network.
- **Router**: operates at Layer 3, connects **different** networks together and forwards data between them based on IP addresses, making routing decisions about the best path to a destination.

[↑ Back to top](#table-of-contents)

---

### 11. What are latency, bandwidth, and throughput in a networking context? 🔴

- **Latency**: the time for a single piece of data to travel from sender to receiver (often measured as round-trip time).
- **Bandwidth**: the theoretical **maximum** data transfer capacity of a connection (e.g. 1 Gbps).
- **Throughput**: the **actual** amount of data successfully transferred in practice, which is often lower than the theoretical bandwidth due to congestion, overhead, or other limiting factors.

[↑ Back to top](#table-of-contents)

---

### 12. What is packet switching, and how does it differ from circuit switching? 🔴

- **Packet switching**: data is broken into small, independently-routed **packets**, each potentially taking a different path to the destination and reassembled there — the foundation of how the modern internet works, since it efficiently shares network links among many simultaneous communications.
- **Circuit switching** (traditional telephone networks): a **dedicated** path is established and reserved for the entire duration of a communication session — guarantees consistent performance for that session, but wastes capacity when the dedicated circuit isn't being fully used.

[↑ Back to top](#table-of-contents)

---

### 13. What is MTU (Maximum Transmission Unit), and what happens when a packet exceeds it? 🔴

- The largest size a single packet/frame is allowed to be on a given network link (commonly 1500 bytes for Ethernet). If a packet exceeds the MTU of a link it needs to cross, it must be **fragmented** into smaller pieces (or, if fragmentation is disallowed, dropped with an error sent back to the sender) — excessive fragmentation adds overhead and can hurt performance, which is part of why protocols try to discover and respect the path's MTU upfront.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can packet fragmentation specifically cause performance problems or even total connectivity failures in certain network configurations (hint: "Path MTU Discovery" and ICMP being blocked)?

[↑ Back to top](#table-of-contents)

---
