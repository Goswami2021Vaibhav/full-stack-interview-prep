# TCP vs UDP

_Part of [Networking](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is TCP?](#1-what-is-tcp)
- [2. What is UDP?](#2-what-is-udp)
- [3. What's the main difference between TCP and UDP?](#3-whats-the-main-difference-between-tcp-and-udp)

**🟡 Medium**
- [4. What is the TCP three-way handshake?](#4-what-is-the-tcp-three-way-handshake)
- [5. How does TCP guarantee reliable delivery?](#5-how-does-tcp-guarantee-reliable-delivery)
- [6. Why is UDP faster than TCP?](#6-why-is-udp-faster-than-tcp)
- [7. What are common use cases for UDP despite it being unreliable?](#7-what-are-common-use-cases-for-udp-despite-it-being-unreliable)
- [8. What is TCP's flow control mechanism?](#8-what-is-tcps-flow-control-mechanism)

**🔴 Hard**
- [9. What is TCP congestion control, and why is it needed?](#9-what-is-tcp-congestion-control-and-why-is-it-needed)
- [10. What is the TCP connection termination (four-way) process?](#10-what-is-the-tcp-connection-termination-four-way-process)
- [11. What is a TCP retransmission, and what triggers one?](#11-what-is-a-tcp-retransmission-and-what-triggers-one)
- [12. Why do real-time applications often prefer UDP over TCP despite needing some reliability?](#12-why-do-real-time-applications-often-prefer-udp-over-tcp-despite-needing-some-reliability)

---

### 1. What is TCP? 🟢

- **T**ransmission **C**ontrol **P**rotocol — a connection-oriented transport protocol guaranteeing reliable, ordered, error-checked delivery of data between two endpoints.

[↑ Back to top](#table-of-contents)

---

### 2. What is UDP? 🟢

- **U**ser **D**atagram **P**rotocol — a connectionless transport protocol that sends data without any guarantee of delivery, ordering, or error correction — minimal overhead, maximal speed.

[↑ Back to top](#table-of-contents)

---

### 3. What's the main difference between TCP and UDP? 🟢

- **TCP**: reliable, ordered, connection-based — but with overhead (handshakes, acknowledgments, retransmissions).
- **UDP**: unreliable, unordered, connectionless — minimal overhead, lower latency, but the application itself must handle any reliability it needs (if any).

[↑ Back to top](#table-of-contents)

---

### 4. What is the TCP three-way handshake? 🟡

- The process establishing a TCP connection before any data is sent: **SYN** (client requests a connection), **SYN-ACK** (server acknowledges and agrees), **ACK** (client confirms) — both sides now agree the connection is open and have synchronized initial sequence numbers.

```
Client -> Server: SYN
Server -> Client: SYN-ACK
Client -> Server: ACK
```

[↑ Back to top](#table-of-contents)

---

### 5. How does TCP guarantee reliable delivery? 🟡

- Every byte sent is assigned a **sequence number**; the receiver sends back **acknowledgments** confirming what it received. If the sender doesn't get an ack within an expected time, it **retransmits** (Q11) the unacknowledged data — combined with checksums to detect corruption, this is what gives TCP its reliability guarantee.

[↑ Back to top](#table-of-contents)

---

### 6. Why is UDP faster than TCP? 🟡

- It skips the connection setup handshake (Q4), doesn't wait for acknowledgments, doesn't retransmit lost data, and doesn't enforce ordering — each packet is simply sent and forgotten, eliminating all the bookkeeping overhead TCP's reliability guarantees require.

[↑ Back to top](#table-of-contents)

---

### 7. What are common use cases for UDP despite it being unreliable? 🟡

- **DNS** (a single small query/response, where retrying at the application level is simpler than TCP's full overhead), **video/audio streaming and calls** (a dropped frame is better tolerated than waiting for a retransmission that arrives too late to matter), **online gaming** (similarly, stale retransmitted data is often worse than just moving on), and **DHCP**.

[↑ Back to top](#table-of-contents)

---

### 8. What is TCP's flow control mechanism? 🟡

- Prevents a fast sender from **overwhelming a slower receiver** — the receiver advertises a **window size** (how much data it can currently buffer), and the sender won't send more than that amount before getting an acknowledgment freeing up more window space. Distinct from congestion control (Q9), which protects the **network**, not just the receiver.

[↑ Back to top](#table-of-contents)

---

### 9. What is TCP congestion control, and why is it needed? 🔴

- Prevents a sender from overwhelming the **network itself** (routers/links between sender and receiver, not just the receiver) — TCP starts sending conservatively (slow start) and gradually increases its sending rate, backing off sharply when it detects signs of congestion (packet loss, increased delay), to avoid contributing to (and avoid suffering from) network-wide congestion collapse.

[↑ Back to top](#table-of-contents)

---

### 10. What is the TCP connection termination (four-way) process? 🔴

- Each side independently signals it's done sending: **FIN** (one side says "I'm finished sending"), **ACK** (acknowledged), then the **other** side does the same in reverse (**FIN**, **ACK**) — needs four steps (rather than three, like the handshake) since each direction of the connection must be closed independently, allowing one side to keep sending data even after the other has finished.

[↑ Back to top](#table-of-contents)

---

### 11. What is a TCP retransmission, and what triggers one? 🔴

- Resending data the sender believes was **lost** — triggered either by a **timeout** (no acknowledgment received within an expected window) or by receiving **duplicate acknowledgments** (a signal, via "fast retransmit," that a specific segment was likely lost, without waiting for the full timeout). Excessive retransmissions are usually a symptom of network congestion or an unreliable link.

[↑ Back to top](#table-of-contents)

---

### 12. Why do real-time applications often prefer UDP over TCP despite needing some reliability? 🔴

- TCP's reliability comes at the cost of **ordering and retransmission delays** — if a packet is lost, TCP blocks delivery of everything **after** it until the lost packet is retransmitted and arrives (head-of-line blocking at the transport level, similar in spirit to [HTTP's head-of-line blocking](http-and-https.md#10-what-is-head-of-line-blocking-and-how-did-http2-and-http3-address-it)). For real-time audio/video, a late-arriving retransmitted frame is often **useless** by the time it arrives — applications build their own lightweight, latency-tolerant reliability on top of UDP (e.g. simply dropping/skipping a lost frame and moving on) rather than accepting TCP's strict in-order delivery guarantee.

> [!IMPORTANT]
> **Follow-up questions:**
> - How does QUIC (HTTP/3's transport, built on UDP) manage to provide TCP-like reliability guarantees per-stream while still avoiding this head-of-line blocking problem?

[↑ Back to top](#table-of-contents)

---
