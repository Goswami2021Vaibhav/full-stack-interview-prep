# HTTP & HTTPS

_Part of [Networking](README.md) interview notes._

> REST-API-specific HTTP usage (methods, status codes, headers for API design) is covered in [REST API › HTTP Methods & Status Codes](../rest-api/http-methods-and-status-codes.md). This file covers HTTP/HTTPS as network protocols.

## Table of Contents

**🟢 Easy**
- [1. What is HTTP?](#1-what-is-http)
- [2. What is HTTPS, and how does it differ from HTTP?](#2-what-is-https-and-how-does-it-differ-from-http)
- [3. What is a URL?](#3-what-is-a-url)

**🟡 Medium**
- [4. What is TLS/SSL, and what role does it play in HTTPS?](#4-what-is-tlsssl-and-what-role-does-it-play-in-https)
- [5. What's the difference between HTTP/1.1 and HTTP/2?](#5-whats-the-difference-between-http11-and-http2)
- [6. What is a TLS handshake?](#6-what-is-a-tls-handshake)
- [7. What is HTTP keep-alive?](#7-what-is-http-keep-alive)
- [8. What are the default ports for HTTP and HTTPS?](#8-what-are-the-default-ports-for-http-and-https)

**🔴 Hard**
- [9. What is HTTP/3, and how does it differ from HTTP/2?](#9-what-is-http3-and-how-does-it-differ-from-http2)
- [10. What is head-of-line blocking, and how did HTTP/2 and HTTP/3 address it?](#10-what-is-head-of-line-blocking-and-how-did-http2-and-http3-address-it)
- [11. What is a certificate authority, and how does it relate to HTTPS trust?](#11-what-is-a-certificate-authority-and-how-does-it-relate-to-https-trust)
- [12. What is HSTS (HTTP Strict Transport Security)?](#12-what-is-hsts-http-strict-transport-security)
- [13. What's the difference between symmetric and asymmetric encryption, and how does TLS use both?](#13-whats-the-difference-between-symmetric-and-asymmetric-encryption-and-how-does-tls-use-both)
- [14. What is SNI (Server Name Indication), and why is it needed?](#14-what-is-sni-server-name-indication-and-why-is-it-needed)

---

### 1. What is HTTP? 🟢

- **H**yper**T**ext **T**ransfer **P**rotocol — the application-layer protocol used for requesting and transferring web content between clients and servers, built on top of TCP.

[↑ Back to top](#table-of-contents)

---

### 2. What is HTTPS, and how does it differ from HTTP? 🟢

- HTTP layered on top of **TLS/SSL encryption** — the request/response semantics are identical to plain HTTP, but the entire exchange is encrypted in transit, preventing eavesdropping/tampering by anyone intercepting the connection.

[↑ Back to top](#table-of-contents)

---

### 3. What is a URL? 🟢

- A **U**niform **R**esource **L**ocator — a structured address identifying a specific resource and how to access it: scheme (`https`), host, port, path, and query string.

```
https://example.com:443/users?id=42
└scheme┘└──host────┘└port┘└path┘└query┘
```

[↑ Back to top](#table-of-contents)

---

### 4. What is TLS/SSL, and what role does it play in HTTPS? 🟡

- **TLS** (Transport Layer Security, the modern successor to the older **SSL**) is the cryptographic protocol that encrypts the connection — it handles the handshake (Q6), establishes a shared encryption key, and verifies the server's identity via a certificate, all **before** any actual HTTP data is exchanged.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between HTTP/1.1 and HTTP/2? 🟡

- **HTTP/1.1**: text-based, one request **per connection** at a time without pipelining issues being fully solved in practice — browsers work around this by opening multiple parallel TCP connections.
- **HTTP/2**: binary protocol, supports **multiplexing** — multiple requests/responses interleaved over a **single** TCP connection simultaneously, header compression, and server push — significantly reducing the overhead of opening many separate connections.

[↑ Back to top](#table-of-contents)

---

### 6. What is a TLS handshake? 🟡

- The initial exchange that establishes a secure connection before any encrypted data flows: the client and server agree on a TLS version and cipher suite, the server presents its certificate (proving its identity), and both sides derive a shared **session key** used to encrypt the actual communication that follows.

[↑ Back to top](#table-of-contents)

---

### 7. What is HTTP keep-alive? 🟡

- Reuses a single TCP connection for **multiple** sequential HTTP requests/responses, instead of opening a new connection (with its own TCP and TLS handshake overhead) for every single request — a significant performance optimization, and the default behavior in HTTP/1.1.

[↑ Back to top](#table-of-contents)

---

### 8. What are the default ports for HTTP and HTTPS? 🟡

- HTTP: port **80**. HTTPS: port **443**.

[↑ Back to top](#table-of-contents)

---

### 9. What is HTTP/3, and how does it differ from HTTP/2? 🔴

- Built on **QUIC** (a transport protocol over **UDP**, not TCP) instead of TCP — solves TCP-level head-of-line blocking (Q10) at the transport layer itself, has faster connection establishment (combining the transport and TLS handshake into fewer round trips), and handles network changes (e.g. switching from Wi-Fi to mobile data) more gracefully without dropping the connection.

[↑ Back to top](#table-of-contents)

---

### 10. What is head-of-line blocking, and how did HTTP/2 and HTTP/3 address it? 🔴

- When one **slow or lost** piece of data blocks all the data queued **behind** it from being processed, even if that later data is otherwise ready. HTTP/1.1 suffers from this at the application level (requests on one connection are serialized). HTTP/2's multiplexing solves it at the **application** layer, but its multiple streams still share one **TCP** connection — so a single lost TCP packet can still stall every stream waiting on retransmission (TCP-level head-of-line blocking). HTTP/3's QUIC (over UDP) solves this more completely, since each stream's loss/retransmission is handled **independently**, not blocking unrelated streams.

[↑ Back to top](#table-of-contents)

---

### 11. What is a certificate authority, and how does it relate to HTTPS trust? 🔴

- A trusted third-party organization that **issues and signs** digital certificates verifying a domain's identity — browsers/OSes ship with a built-in list of trusted CAs, and trust a server's certificate because it was signed by one of them (a "chain of trust"). Without this, anyone could claim to be any website, since there'd be no independent verification of "this public key really belongs to example.com."

[↑ Back to top](#table-of-contents)

---

### 12. What is HSTS (HTTP Strict Transport Security)? 🔴

- A response header telling the browser "always use HTTPS for this domain, never plain HTTP, for the next N seconds" — protects against an attacker intercepting an initial **plain HTTP** request (before any redirect to HTTPS could even happen) by ensuring the browser never attempts an insecure connection to that domain again, once it's seen this header at least once.

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between symmetric and asymmetric encryption, and how does TLS use both? 🔴

- **Symmetric encryption**: the **same** key encrypts and decrypts — fast, but both parties need to somehow already share that secret key safely.
- **Asymmetric encryption**: a **public/private key pair** — anyone can encrypt with the public key, but only the private key holder can decrypt; slower, but solves the key-distribution problem.
- TLS uses **asymmetric** encryption during the handshake specifically to safely establish a shared secret (without ever transmitting it in the clear), then switches to **symmetric** encryption for the actual bulk data transfer, since symmetric encryption is far faster for large amounts of data.

[↑ Back to top](#table-of-contents)

---

### 14. What is SNI (Server Name Indication), and why is it needed? 🔴

- An extension to the TLS handshake where the client tells the server **which hostname** it's trying to reach, **before** the server picks which certificate to present — necessary because a single server/IP often hosts **multiple** different domains (each with its own certificate), and without SNI, the server would have no way to know which certificate to send during the handshake, before it even sees the actual HTTP request (which is what historically carries the `Host` header, but that comes *after* the TLS handshake already needs to be complete).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does SNI itself being sent in plaintext during the handshake raise a privacy concern, and what newer mechanism (Encrypted Client Hello) attempts to address it?

[↑ Back to top](#table-of-contents)

---
