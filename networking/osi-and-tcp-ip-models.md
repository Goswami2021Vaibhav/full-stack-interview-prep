# OSI & TCP/IP Models

_Part of [Networking](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is the OSI model?](#1-what-is-the-osi-model)
- [2. What are the 7 layers of the OSI model?](#2-what-are-the-7-layers-of-the-osi-model)
- [3. What is the TCP/IP model?](#3-what-is-the-tcpip-model)

**🟡 Medium**
- [4. How does the TCP/IP model map onto the OSI model's 7 layers?](#4-how-does-the-tcpip-model-map-onto-the-osi-models-7-layers)
- [5. What happens at the Network layer (Layer 3)?](#5-what-happens-at-the-network-layer-layer-3)
- [6. What happens at the Transport layer (Layer 4)?](#6-what-happens-at-the-transport-layer-layer-4)
- [7. What happens at the Application layer (Layer 7)?](#7-what-happens-at-the-application-layer-layer-7)
- [8. What is encapsulation in the context of these layered models?](#8-what-is-encapsulation-in-the-context-of-these-layered-models)

**🔴 Hard**
- [9. Why do we still teach the OSI model when most real systems use the simpler TCP/IP model?](#9-why-do-we-still-teach-the-osi-model-when-most-real-systems-use-the-simpler-tcpip-model)
- [10. What's an example of how a single HTTP request maps through each layer?](#10-whats-an-example-of-how-a-single-http-request-maps-through-each-layer)
- [11. Why is layering important for protocol design?](#11-why-is-layering-important-for-protocol-design)
- [12. What happens at the Data Link layer (Layer 2), and what is a frame?](#12-what-happens-at-the-data-link-layer-layer-2-and-what-is-a-frame)

---

### 1. What is the OSI model? 🟢

- A 7-layer conceptual framework describing how network communication is broken into distinct responsibilities, from raw physical transmission up to application-level data — used as a common reference/vocabulary for discussing networking, even though no real system implements it exactly as 7 strict, separate layers.

[↑ Back to top](#table-of-contents)

---

### 2. What are the 7 layers of the OSI model? 🟢

1. **Physical** — raw bits over a physical medium.
2. **Data Link** — framing, MAC addressing, local delivery.
3. **Network** — IP addressing, routing between networks.
4. **Transport** — end-to-end delivery (TCP/UDP), ports.
5. **Session** — managing communication sessions.
6. **Presentation** — data format/encoding/encryption.
7. **Application** — the actual application-level protocol (HTTP, etc.).

[↑ Back to top](#table-of-contents)

---

### 3. What is the TCP/IP model? 🟢

- A simpler, **practically-used** 4-layer model (Link, Internet, Transport, Application) that the actual modern internet is built on — predates the OSI model's full formalization and is what real protocol stacks (Linux's networking stack, etc.) are actually organized around.

[↑ Back to top](#table-of-contents)

---

### 4. How does the TCP/IP model map onto the OSI model's 7 layers? 🟡

- **Link**: roughly OSI's Physical + Data Link.
- **Internet**: roughly OSI's Network layer.
- **Transport**: directly maps to OSI's Transport layer.
- **Application**: roughly OSI's Session + Presentation + Application combined into one layer.

[↑ Back to top](#table-of-contents)

---

### 5. What happens at the Network layer (Layer 3)? 🟡

- Handles **logical addressing** (IP addresses) and **routing** — determining the best path for a packet to travel from source to destination across potentially many intermediate networks. Routers operate primarily at this layer.

[↑ Back to top](#table-of-contents)

---

### 6. What happens at the Transport layer (Layer 4)? 🟡

- Provides **end-to-end** communication between specific applications (via ports) on two hosts — handles reliability (TCP's acknowledgments/retransmission), ordering, and flow control, or deliberately skips all of that for speed (UDP). See [TCP vs UDP](tcp-vs-udp.md).

[↑ Back to top](#table-of-contents)

---

### 7. What happens at the Application layer (Layer 7)? 🟡

- Where actual application-level protocols operate — HTTP, DNS, SMTP, FTP — defining the **format and meaning** of the data being exchanged for a specific purpose, built on top of the lower layers' job of just getting bytes from one place to another reliably.

[↑ Back to top](#table-of-contents)

---

### 8. What is encapsulation in the context of these layered models? 🟡

- Each layer wraps the data from the layer above it with its **own header** (and sometimes trailer) before passing it down — e.g. application data gets a TCP header, then an IP header, then a frame header, with each layer only caring about its own header and treating everything inside as an opaque payload. The receiving side reverses this process, peeling off headers as data moves up the stack.

[↑ Back to top](#table-of-contents)

---

### 9. Why do we still teach the OSI model when most real systems use the simpler TCP/IP model? 🔴

- The OSI model provides a more **granular, vendor-neutral vocabulary** for discussing networking concepts precisely (e.g. distinguishing "Layer 2 switch" from "Layer 3 router" clearly) — even though no real-world stack implements all 7 layers as cleanly separate, the terminology and conceptual separation of concerns remains genuinely useful for teaching and for precisely describing where in the stack a given device/problem operates.

[↑ Back to top](#table-of-contents)

---

### 10. What's an example of how a single HTTP request maps through each layer? 🔴

1. **Application**: the browser constructs an HTTP request.
2. **Transport**: wrapped in a TCP segment (adds source/destination port).
3. **Network**: wrapped in an IP packet (adds source/destination IP).
4. **Data Link**: wrapped in a frame (adds source/destination MAC address for the local hop).
5. **Physical**: transmitted as actual electrical/optical/radio signals.
- Each layer on the receiving end reverses this, stripping its corresponding header until the original HTTP request is delivered to the destination application.

[↑ Back to top](#table-of-contents)

---

### 11. Why is layering important for protocol design? 🔴

- Each layer can **evolve independently** without requiring changes to the layers above or below it, as long as the interface between them stays consistent — e.g. you can switch from Wi-Fi to Ethernet (a Layer 1/2 change) without anything at the Application layer needing to know or care, since IP (Layer 3) provides a consistent abstraction regardless of the physical medium underneath.

[↑ Back to top](#table-of-contents)

---

### 12. What happens at the Data Link layer (Layer 2), and what is a frame? 🔴

- Handles communication **within a single local network segment** — framing raw bits into structured units (**frames**), each tagged with source/destination **MAC addresses**, and managing access to the shared physical medium. Switches operate primarily at this layer, using MAC addresses (rather than IP addresses) to decide where to forward each frame.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does a packet need both a MAC address (Layer 2) and an IP address (Layer 3) — what would break if only one of the two existed?

[↑ Back to top](#table-of-contents)

---
