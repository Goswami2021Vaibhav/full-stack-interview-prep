# Case Studies

_Part of [System Design](README.md) interview notes._

> These are walkthrough-style answers, deliberately concise. Each follows the framework from [Core Concepts](core-concepts.md#11-how-do-you-approach-a-system-design-interview-question-structurally): clarify requirements, estimate scale, sketch the architecture, then go deep on the most interesting piece. In a real interview, expect to spend most of the time driving this conversation, not just stating a final answer.

## Table of Contents

**🟢 Easy**
- [1. What's a good general framework to follow when answering a system design case study?](#1-whats-a-good-general-framework-to-follow-when-answering-a-system-design-case-study)

**🟡 Medium**
- [2. How would you design a URL shortener (like bit.ly)?](#2-how-would-you-design-a-url-shortener-like-bitly)
- [3. How would you design a rate limiter?](#3-how-would-you-design-a-rate-limiter)
- [4. How would you design a basic notification system?](#4-how-would-you-design-a-basic-notification-system)

**🔴 Hard**
- [5. How would you design a news feed system (like Twitter/Instagram)?](#5-how-would-you-design-a-news-feed-system-like-twitterinstagram)
- [6. How would you design a chat application (like WhatsApp)?](#6-how-would-you-design-a-chat-application-like-whatsapp)
- [7. How would you design a distributed file storage system (like Dropbox)?](#7-how-would-you-design-a-distributed-file-storage-system-like-dropbox)
- [8. How would you design a ride-sharing matching system (like Uber)?](#8-how-would-you-design-a-ride-sharing-matching-system-like-uber)
- [9. How would you design an autocomplete/typeahead search system?](#9-how-would-you-design-an-autocompletetypeahead-search-system)
- [10. How would you design a distributed unique ID generator service?](#10-how-would-you-design-a-distributed-unique-id-generator-service)

---

### 1. What's a good general framework to follow when answering a system design case study? 🟢

1. **Clarify requirements** — functional (what must it do) and non-functional (scale, latency, consistency needs).
2. **Estimate scale** — rough QPS, storage, bandwidth (see [Core Concepts](core-concepts.md#12-how-do-you-reason-about-capacity-planning-estimating-qps-storage-bandwidth)).
3. **Sketch a high-level architecture** — major components and data flow.
4. **Deep-dive** on the most interesting/critical part (the data model, a specific bottleneck).
5. **Discuss tradeoffs** and how the design evolves under further scale or failure.

[↑ Back to top](#table-of-contents)

---

### 2. How would you design a URL shortener (like bit.ly)? 🟡

- **Core need**: map a short code to a long URL, and redirect on lookup.
- **Generating the short code**: either hash the long URL (truncated, with collision handling) or use a counter/base62-encoded unique ID (see [Database Design § ID generation](database-design-and-sharding.md#11-how-do-you-design-a-globally-unique-id-generator-for-a-sharded-system)) — the counter approach avoids collisions entirely.
- **Storage**: a simple key-value mapping (`short_code -> long_url`) — a perfect fit for a key-value store or a simple indexed SQL table, since lookups are by exact key.
- **Read-heavy**: redirects vastly outnumber creations — cache hot short codes aggressively (see [Caching](caching.md)), since the lookup path is the most performance-critical.
- **Scale**: shard by short code (or its hash) once a single store can't hold/serve the full mapping table.

[↑ Back to top](#table-of-contents)

---

### 3. How would you design a rate limiter? 🟡

- **Core need**: track how many requests a client (by user ID/API key/IP) has made in a time window, and reject once a threshold is exceeded.
- **Algorithm choice**: a **sliding window** or **token bucket** algorithm gives smoother behavior than a naive fixed window (which allows a burst right at the window boundary).
- **Storage**: an in-memory store (Redis) with atomic increment + expiry — needs to be **shared** across all application instances, since a per-instance counter would let a client get a separate limit per instance it happens to be routed to.
- **Where to enforce it**: at the API gateway/edge, so rejected requests never even reach the backend services, and the limiting logic is centralized rather than duplicated per service.

[↑ Back to top](#table-of-contents)

---

### 4. How would you design a basic notification system? 🟡

- **Core need**: trigger a notification (email/push/SMS) in response to events from various services, without those services needing to know delivery details.
- **Architecture**: producers publish events to a **message queue** (decoupling them from delivery timing/failures, see [Message Queues & Events](message-queues-and-events.md#2-why-use-a-message-queue-instead-of-a-direct-synchronous-call)); separate **channel-specific workers** (email, push, SMS) consume and handle actual delivery, each scaling independently.
- **Reliability**: retries with backoff for transient failures, a dead-letter queue for persistent failures, and idempotent processing (see [Message Queues & Events](message-queues-and-events.md#9-how-do-you-achieve-idempotent-message-processing-on-the-consumer-side)) in case the same event gets delivered twice.

[↑ Back to top](#table-of-contents)

---

### 5. How would you design a news feed system (like Twitter/Instagram)? 🔴

- **Core need**: show each user a chronological/ranked feed of posts from people they follow.
- **Fan-out-on-write** (push): when a user posts, immediately push that post into each follower's **pre-computed** feed cache — fast reads (the common case), but writes get expensive for users with huge follower counts.
- **Fan-out-on-read** (pull): a feed is assembled **at read time** by querying all followed users' recent posts and merging — avoids the write amplification problem, but makes reads more expensive.
- **Hybrid (most real systems)**: fan-out-on-write for typical users, fan-out-on-read for "celebrity" accounts with huge follower counts — see [Caching § feed strategy](caching.md#13-how-would-you-design-a-caching-strategy-for-a-social-media-feed) for the deeper tradeoff.

[↑ Back to top](#table-of-contents)

---

### 6. How would you design a chat application (like WhatsApp)? 🔴

- **Core need**: real-time bidirectional message delivery between users, plus message persistence/history.
- **Connections**: clients hold a persistent **WebSocket** connection to a chat server — since these are stateful, long-lived connections, scaling requires a pub/sub backplane (see [Scalability § stateful services](scalability-and-load-balancing.md#9-how-do-you-scale-a-stateful-service-eg-websocket-connections-behind-a-load-balancer)) so a message from user A (connected to server 1) can reach user B (connected to a different server 2).
- **Delivery guarantees**: track per-message delivery/read status; queue messages for offline users and deliver on reconnect.
- **Storage**: messages are typically sharded by conversation ID, since a conversation's messages are almost always read together.

[↑ Back to top](#table-of-contents)

---

### 7. How would you design a distributed file storage system (like Dropbox)? 🔴

- **Core need**: upload, store, sync, and share large files reliably across devices.
- **Storage**: actual file content (blobs) goes into object storage (e.g. S3-like), **not** a traditional database — split large files into **chunks**, both for efficient resumable uploads and to deduplicate identical chunks across files (saving storage).
- **Metadata**: a separate database tracks file metadata (name, owner, chunk list, version history) — this is the part that benefits from a relational/indexed structure, distinct from the actual blob storage.
- **Sync**: clients periodically poll or receive push notifications of changes, then download only the **changed chunks**, not the entire file, to minimize bandwidth.

[↑ Back to top](#table-of-contents)

---

### 8. How would you design a ride-sharing matching system (like Uber)? 🔴

- **Core need**: continuously match nearby available drivers to requesting riders in near real-time.
- **Location indexing**: drivers continuously report their GPS location; index active drivers using a **geospatial structure** (a geohash-based grid, or a quadtree) so "find nearby available drivers" is a fast spatial query rather than scanning every driver.
- **Matching**: when a ride is requested, query nearby drivers, rank by estimated time-to-pickup, and offer the ride — handling the race condition of multiple riders potentially matching the same driver simultaneously needs careful locking/atomic assignment.
- **Real-time updates**: both rider and driver apps need live position updates during the ride — similar real-time delivery challenge as the chat system (Q6).

[↑ Back to top](#table-of-contents)

---

### 9. How would you design an autocomplete/typeahead search system? 🔴

- **Core need**: as a user types, return relevant suggestions with very low latency (typically under ~100ms to feel instant).
- **Data structure**: a **trie** (prefix tree) is the classic structure — each node represents a character, and walking down from the root by the typed prefix quickly narrows to all matching completions, often pre-ranked by popularity/frequency at each node.
- **Scale**: the trie (or a sharded/partitioned version of it) is typically held **in memory** across multiple servers for speed, since disk-based lookups would be too slow for the latency requirement; updated periodically (not necessarily in real-time) from aggregated query logs to refresh popularity rankings.

[↑ Back to top](#table-of-contents)

---

### 10. How would you design a distributed unique ID generator service? 🔴

- **Core need**: generate IDs that are unique across many machines, without a single bottlenecked counter.
- **Approach**: a **Snowflake-style** ID — a single 64-bit integer encoding a timestamp (most significant bits, giving rough time-sortability), a machine/worker ID (to distinguish generators), and a per-machine sequence counter (to disambiguate multiple IDs generated within the same millisecond on the same machine) — see [Database Design § ID generation](database-design-and-sharding.md#11-how-do-you-design-a-globally-unique-id-generator-for-a-sharded-system).
- **Why not just UUIDs**: UUIDs are simpler and need zero coordination, but aren't naturally time-sortable and are larger to index — Snowflake-style IDs are preferred when insertion-order sortability and compact storage matter.

> [!IMPORTANT]
> **Follow-up questions:**
> - For the autocomplete system (Q9), how would you handle a prefix so popular that its candidate list itself becomes large enough to be a bottleneck to rank in real time?

[↑ Back to top](#table-of-contents)

---
