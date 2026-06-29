# Network Security

_Part of [Networking](README.md) interview notes._

> Application-level web security (XSS, CSRF, CORS) is covered in [JavaScript › Browser APIs & Storage](../javascript/browser-apis-and-storage.md) and [Express.js › Security](../expressjs/security.md). This file covers network-layer security concepts.

## Table of Contents

**🟢 Easy**
- [1. What is a firewall?](#1-what-is-a-firewall)
- [2. What is a VPN?](#2-what-is-a-vpn)
- [3. What is encryption?](#3-what-is-encryption)

**🟡 Medium**
- [4. What's the difference between symmetric and asymmetric encryption?](#4-whats-the-difference-between-symmetric-and-asymmetric-encryption)
- [5. What is a man-in-the-middle attack?](#5-what-is-a-man-in-the-middle-attack)
- [6. What is a DDoS attack?](#6-what-is-a-ddos-attack)
- [7. What is a proxy server, and how does it differ from a VPN?](#7-what-is-a-proxy-server-and-how-does-it-differ-from-a-vpn)
- [8. What is port scanning, and why is it a security concern?](#8-what-is-port-scanning-and-why-is-it-a-security-concern)

**🔴 Hard**
- [9. What is a VLAN, and how does it improve network security?](#9-what-is-a-vlan-and-how-does-it-improve-network-security)
- [10. What is an IDS/IPS (Intrusion Detection/Prevention System)?](#10-what-is-an-idsips-intrusion-detectionprevention-system)
- [11. How does a firewall decide whether to allow or block traffic?](#11-how-does-a-firewall-decide-whether-to-allow-or-block-traffic)
- [12. What is ARP spoofing?](#12-what-is-arp-spoofing)
- [13. How does a VPN actually establish a secure tunnel over an untrusted network?](#13-how-does-a-vpn-actually-establish-a-secure-tunnel-over-an-untrusted-network)

---

### 1. What is a firewall? 🟢

- A network security device/software that monitors and filters incoming/outgoing traffic based on a set of **rules** — blocking traffic that doesn't match allowed criteria (a specific port, source IP, protocol), reducing exposure to unwanted/malicious connections.

[↑ Back to top](#table-of-contents)

---

### 2. What is a VPN? 🟢

- A **V**irtual **P**rivate **N**etwork — creates an encrypted tunnel over a public/untrusted network (the internet), letting traffic appear to originate from the VPN's private network and protecting it from eavesdropping along the way.

[↑ Back to top](#table-of-contents)

---

### 3. What is encryption? 🟢

- Transforming data into an unreadable form using a key, such that only someone with the correct corresponding key can reverse the transformation (decrypt) and read the original data.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between symmetric and asymmetric encryption? 🟡

- **Symmetric**: one shared secret key encrypts and decrypts — fast, but requires safely distributing that key beforehand.
- **Asymmetric**: a public/private key pair — solves the key distribution problem, at the cost of being computationally slower (see [HTTP & HTTPS](http-and-https.md#13-whats-the-difference-between-symmetric-and-asymmetric-encryption-and-how-does-tls-use-both) for how TLS combines both).

[↑ Back to top](#table-of-contents)

---

### 5. What is a man-in-the-middle attack? 🟡

- An attacker secretly **intercepts** communication between two parties who believe they're talking directly to each other — able to eavesdrop or even alter the data in transit. HTTPS/TLS (verifying the server's identity via a certificate) is the primary defense against this for web traffic.

[↑ Back to top](#table-of-contents)

---

### 6. What is a DDoS attack? 🟡

- A **D**istributed **D**enial of **S**ervice attack — overwhelming a target with traffic from **many** different sources simultaneously (often a botnet of compromised devices), exhausting its bandwidth/resources so it can't serve legitimate requests. "Distributed" distinguishes it from a simpler single-source DoS attack, and makes it much harder to mitigate by simply blocking one source.

[↑ Back to top](#table-of-contents)

---

### 7. What is a proxy server, and how does it differ from a VPN? 🟡

- A **proxy** forwards specific application-level traffic (often just HTTP) on a client's behalf, typically without encrypting the connection between client and proxy by default.
- A **VPN** operates at a lower level, encrypting and tunneling **all** of a device's network traffic (not just one application's), and typically provides built-in encryption as a core feature rather than an optional add-on.

[↑ Back to top](#table-of-contents)

---

### 8. What is port scanning, and why is it a security concern? 🟡

- Systematically probing a target's ports to discover which ones are **open** (and what services are listening on them) — often a reconnaissance step before an attack, since open/unnecessary ports represent potential attack surface. Defenses include closing unused ports and firewall rules limiting which ports are reachable at all.

[↑ Back to top](#table-of-contents)

---

### 9. What is a VLAN, and how does it improve network security? 🔴

- A **V**irtual **L**AN — logically segments a single physical network into multiple **isolated** broadcast domains, even though devices share the same physical switches/cabling — traffic on one VLAN doesn't reach devices on another VLAN unless explicitly routed. Improves security by isolating sensitive segments (e.g. a separate VLAN for internal servers vs. guest Wi-Fi) without needing entirely separate physical infrastructure.

[↑ Back to top](#table-of-contents)

---

### 10. What is an IDS/IPS (Intrusion Detection/Prevention System)? 🔴

- **IDS**: monitors network traffic for suspicious/known-malicious patterns and **alerts** when found, without blocking it itself.
- **IPS**: does the same monitoring, but can **actively block/drop** the malicious traffic in real time. An IPS is more proactive, but carries more risk of false positives disrupting legitimate traffic, compared to an IDS's purely observational role.

[↑ Back to top](#table-of-contents)

---

### 11. How does a firewall decide whether to allow or block traffic? 🔴

- **Stateless** firewalls evaluate each packet **independently** against a fixed rule set (source/destination IP, port, protocol) — simple and fast, but can't reason about whether a packet is part of an already-established, legitimate conversation.
- **Stateful** firewalls track the state of **active connections**, and can make smarter decisions (e.g. automatically allowing return traffic for a connection that was legitimately initiated from inside the network, without needing an explicit rule for every possible response) — the standard approach in modern firewalls.

[↑ Back to top](#table-of-contents)

---

### 12. What is ARP spoofing? 🔴

- An attacker sends forged **ARP** (Address Resolution Protocol — maps IP addresses to MAC addresses on a local network) messages, tricking other devices on the local network into associating the attacker's MAC address with a legitimate IP (e.g. the network's gateway) — effectively positioning the attacker for a man-in-the-middle attack on local network traffic. Mitigated by static ARP entries for critical devices, or switches with dynamic ARP inspection.

[↑ Back to top](#table-of-contents)

---

### 13. How does a VPN actually establish a secure tunnel over an untrusted network? 🔴

- The client and VPN server perform a handshake (similar in spirit to TLS, see [HTTP & HTTPS](http-and-https.md#6-what-is-a-tls-handshake)) to authenticate each other and agree on encryption keys — once established, every packet the client sends is **encapsulated** inside an encrypted outer packet addressed to the VPN server, which decrypts it and forwards the original packet to its real destination on the client's behalf (and reverses the process for the response) — to anyone observing the network in between, only encrypted traffic to the VPN server is visible, never the actual underlying content or true destination.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does using a VPN hide your browsing destinations from your local ISP, but not necessarily from the VPN provider itself?

[↑ Back to top](#table-of-contents)

---
