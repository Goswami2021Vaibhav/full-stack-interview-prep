# Common Protocols

_Part of [Networking](README.md) interview notes._

> WebSocket usage from a JavaScript/browser perspective is covered in [JavaScript › Browser APIs & Storage](../javascript/browser-apis-and-storage.md). This file covers the protocols themselves.

## Table of Contents

**🟢 Easy**
- [1. What is FTP?](#1-what-is-ftp)
- [2. What is SMTP?](#2-what-is-smtp)
- [3. What is SSH?](#3-what-is-ssh)

**🟡 Medium**
- [4. What is WebSocket, and how does it differ from HTTP?](#4-what-is-websocket-and-how-does-it-differ-from-http)
- [5. What is MQTT, and where is it used?](#5-what-is-mqtt-and-where-is-it-used)
- [6. What is DHCP, and what problem does it solve?](#6-what-is-dhcp-and-what-problem-does-it-solve)
- [7. What is ARP, and what does it do?](#7-what-is-arp-and-what-does-it-do)
- [8. What's the difference between POP3 and IMAP for email?](#8-whats-the-difference-between-pop3-and-imap-for-email)

**🔴 Hard**
- [9. What is gRPC, and how does it relate to HTTP/2?](#9-what-is-grpc-and-how-does-it-relate-to-http2)
- [10. What is SNMP, and what is it used for?](#10-what-is-snmp-and-what-is-it-used-for)
- [11. What is NTP, and why does clock synchronization matter for distributed systems?](#11-what-is-ntp-and-why-does-clock-synchronization-matter-for-distributed-systems)
- [12. What is ICMP, and what is it commonly used for?](#12-what-is-icmp-and-what-is-it-commonly-used-for)
- [13. What's the difference between a stateful and a stateless protocol?](#13-whats-the-difference-between-a-stateful-and-a-stateless-protocol)

---

### 1. What is FTP? 🟢

- **F**ile **T**ransfer **P**rotocol — a protocol for transferring files between a client and server, traditionally over a separate control connection (commands) and data connection (the actual file transfer). Largely superseded for many uses by simpler HTTP-based file transfer or more secure alternatives (SFTP).

[↑ Back to top](#table-of-contents)

---

### 2. What is SMTP? 🟢

- **S**imple **M**ail **T**ransfer **P**rotocol — the protocol used for **sending** email between mail servers (and from a client to its outgoing mail server). Receiving/reading email typically uses a different protocol (POP3/IMAP, Q8).

[↑ Back to top](#table-of-contents)

---

### 3. What is SSH? 🟢

- **S**ecure **SH**ell — a protocol for securely accessing and controlling a remote machine's command line over an encrypted connection, replacing older unencrypted protocols (Telnet) that sent credentials/data in plaintext.

[↑ Back to top](#table-of-contents)

---

### 4. What is WebSocket, and how does it differ from HTTP? 🟡

- A protocol providing a **persistent, full-duplex** connection between client and server — after an initial HTTP-based handshake (an "upgrade" request), the connection switches to the WebSocket protocol, allowing **either side** to send messages at any time, unlike plain HTTP's strict request-then-response model. Used for real-time features (chat, live updates) where the server needs to push data without waiting for the client to ask.

[↑ Back to top](#table-of-contents)

---

### 5. What is MQTT, and where is it used? 🟡

- A lightweight publish-subscribe messaging protocol designed for constrained devices and unreliable networks — widely used in **IoT** (sensors, smart home devices) where minimal bandwidth/power usage and small message overhead matter far more than the richer features of something like HTTP.

[↑ Back to top](#table-of-contents)

---

### 6. What is DHCP, and what problem does it solve? 🟡

- **D**ynamic **H**ost **C**onfiguration **P**rotocol — automatically assigns an IP address (and other network configuration like DNS servers) to a device when it joins a network, removing the need to manually configure networking on every single device individually.

[↑ Back to top](#table-of-contents)

---

### 7. What is ARP, and what does it do? 🟡

- **A**ddress **R**esolution **P**rotocol — resolves a known IP address to its corresponding **MAC address** on the local network, since Layer 2 (Data Link) delivery actually needs the MAC address, not the IP address, to send a frame to the right physical device (see [ARP spoofing](network-security.md#12-what-is-arp-spoofing) for the related attack).

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between POP3 and IMAP for email? 🟡

- **POP3**: downloads email **to the client** and (traditionally) removes it from the server — email is tied to whichever single device downloaded it.
- **IMAP**: keeps email **stored on the server**, with the client just viewing/syncing a reflection of it — supports accessing the same mailbox consistently across multiple devices, which is why IMAP is the more common modern default.

[↑ Back to top](#table-of-contents)

---

### 9. What is gRPC, and how does it relate to HTTP/2? 🔴

- A high-performance RPC (Remote Procedure Call) framework built on top of **HTTP/2**, using Protocol Buffers for compact, strongly-typed binary serialization — leverages HTTP/2's multiplexing and streaming support natively, making it well suited for efficient service-to-service communication (see [REST API › Core Concepts](../rest-api/core-concepts.md#11-whats-the-difference-between-rest-and-grpc) for the comparison against REST).

[↑ Back to top](#table-of-contents)

---

### 10. What is SNMP, and what is it used for? 🔴

- **S**imple **N**etwork **M**anagement **P**rotocol — used to monitor and manage network devices (routers, switches, servers) remotely, allowing a central management system to query device status/metrics and, in some configurations, push configuration changes.

[↑ Back to top](#table-of-contents)

---

### 11. What is NTP, and why does clock synchronization matter for distributed systems? 🔴

- **N**etwork **T**ime **P**rotocol — synchronizes a device's clock with accurate reference time servers over the network. Clock synchronization matters for distributed systems because many things depend on consistent timestamps across machines — ordering events correctly, certificate/token expiry checks (a clock too far off can cause valid TLS certificates to appear expired or not-yet-valid), and log correlation across multiple servers when debugging an issue.

[↑ Back to top](#table-of-contents)

---

### 12. What is ICMP, and what is it commonly used for? 🔴

- **I**nternet **C**ontrol **M**essage **P**rotocol — used for network diagnostic and error-reporting messages, rather than carrying application data itself. The `ping` command uses ICMP Echo Request/Reply messages to test reachability and measure round-trip latency to another host; it's also used to report errors like "destination unreachable" or signal needed packet fragmentation (relevant to [MTU](core-concepts.md#13-what-is-mtu-maximum-transmission-unit-and-what-happens-when-a-packet-exceeds-it)).

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between a stateful and a stateless protocol? 🔴

- **Stateful** (FTP's control connection, a raw TCP session): the protocol/server retains context about prior interactions in the same session.
- **Stateless** (HTTP, by design): each request is **self-contained**, with the server retaining no memory of previous requests by default — any session-like behavior (logins, shopping carts) must be built **on top of** HTTP explicitly (cookies, tokens), since the underlying protocol itself doesn't provide it (see [REST API › Core Concepts](../rest-api/core-concepts.md#5-what-does-stateless-mean-in-rest-and-why-does-it-matter) for why this matters for API design).

> [!IMPORTANT]
> **Follow-up questions:**
> - Given that HTTP is stateless, how does a website "remember" that you're logged in across multiple requests?

[↑ Back to top](#table-of-contents)

---
