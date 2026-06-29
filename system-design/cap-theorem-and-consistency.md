# CAP Theorem & Consistency

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is CAP theorem?](#1-what-is-cap-theorem)
- [2. What is consistency in a distributed system?](#2-what-is-consistency-in-a-distributed-system)
- [3. What is availability in a distributed system?](#3-what-is-availability-in-a-distributed-system)

**🟡 Medium**
- [4. What is partition tolerance, and why is it non-negotiable in practice?](#4-what-is-partition-tolerance-and-why-is-it-non-negotiable-in-practice)
- [5. What's the difference between strong consistency and eventual consistency?](#5-whats-the-difference-between-strong-consistency-and-eventual-consistency)
- [6. What is BASE, and how does it contrast with ACID?](#6-what-is-base-and-how-does-it-contrast-with-acid)
- [7. What's a real-world example of a CP system vs. an AP system?](#7-whats-a-real-world-example-of-a-cp-system-vs-an-ap-system)
- [8. What is quorum-based consistency?](#8-what-is-quorum-based-consistency)

**🔴 Hard**
- [9. Why is CAP theorem often misunderstood as "pick 2 of 3" at all times?](#9-why-is-cap-theorem-often-misunderstood-as-pick-2-of-3-at-all-times)
- [10. What is PACELC, and how does it extend CAP theorem?](#10-what-is-pacelc-and-how-does-it-extend-cap-theorem)
- [11. How does eventual consistency affect application-level design decisions?](#11-how-does-eventual-consistency-affect-application-level-design-decisions)
- [12. How would you choose a consistency model for a specific feature?](#12-how-would-you-choose-a-consistency-model-for-a-specific-feature)

---

### 1. What is CAP theorem? 🟢

- States that a distributed system can only guarantee **two** of the following three properties simultaneously during a network partition: **C**onsistency (every read sees the latest write), **A**vailability (every request gets a response), **P**artition tolerance (the system keeps functioning despite network failures between nodes).

[↑ Back to top](#table-of-contents)

---

### 2. What is consistency in a distributed system? 🟢

- Every read returns the **most recent** write, regardless of which node/replica serves it — all nodes present the same, up-to-date view of the data at any given moment.

[↑ Back to top](#table-of-contents)

---

### 3. What is availability in a distributed system? 🟢

- Every request to a non-failing node receives a **response** (success or failure) within a reasonable time — the system doesn't simply hang or refuse to answer.

[↑ Back to top](#table-of-contents)

---

### 4. What is partition tolerance, and why is it non-negotiable in practice? 🟡

- The system continues operating even when network communication **between nodes** is disrupted (a partition). In any real distributed system spanning multiple machines/data centers, network partitions **will** eventually happen — so partition tolerance isn't really an optional design choice; the actual choice CAP forces is between consistency and availability **during** such a partition.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between strong consistency and eventual consistency? 🟡

- **Strong consistency**: a read is **guaranteed** to reflect the most recent write immediately, everywhere.
- **Eventual consistency**: replicas may temporarily diverge, but **will** converge to the same value eventually (once replication catches up) — trades a temporary consistency gap for better availability/performance, an explicit choice for systems where brief staleness is an acceptable cost.

[↑ Back to top](#table-of-contents)

---

### 6. What is BASE, and how does it contrast with ACID? 🟡

- **BASE**: **B**asically **A**vailable, **S**oft state, **E**ventually consistent — describes the design philosophy of many distributed/NoSQL systems that prioritize availability and tolerate temporary inconsistency.
- **ACID** (see [MySQL › Transactions](../mysql/transactions-and-acid.md#2-what-does-acid-stand-for)): prioritizes strict consistency and isolation, typically at some cost to availability/performance under failure. The two represent largely opposite philosophies on the consistency/availability tradeoff.

[↑ Back to top](#table-of-contents)

---

### 7. What's a real-world example of a CP system vs. an AP system? 🟡

- **CP** (consistency over availability during a partition): a system like a traditional relational database configured for strong consistency (e.g. a MongoDB replica set with a majority write/read concern, see [MongoDB › Core Concepts](../mongodb/core-concepts.md#12-what-is-cap-theorem-and-where-does-mongodb-fall-within-it)) — will refuse to serve a request rather than risk returning stale/conflicting data.
- **AP** (availability over consistency): a system like Cassandra or DynamoDB in their default configurations — always responds, even during a partition, accepting that different nodes might briefly disagree.

[↑ Back to top](#table-of-contents)

---

### 8. What is quorum-based consistency? 🟡

- Requires a **majority** (a quorum) of replicas to acknowledge a read or write before it's considered successful — e.g. with 5 replicas, a write quorum of 3 ensures any subsequent read quorum of 3 will overlap with at least one node that has the latest write, giving a tunable middle ground between strict consistency and full availability.

[↑ Back to top](#table-of-contents)

---

### 9. Why is CAP theorem often misunderstood as "pick 2 of 3" at all times? 🔴

- CAP only describes the tradeoff **during an actual network partition** — outside of a partition, a well-designed system can be both consistent and available simultaneously. The "pick 2 of 3" framing oversimplifies it into a permanent, static choice, when it's really a choice about **behavior specifically when something has already gone wrong** (a partition), not a constant operating mode.

[↑ Back to top](#table-of-contents)

---

### 10. What is PACELC, and how does it extend CAP theorem? 🔴

- **P**artition: **A**vailability vs. **C**onsistency (the original CAP tradeoff) — **E**lse (when there's no partition): **L**atency vs. **C**onsistency. PACELC points out that CAP alone is incomplete: even **without** any partition, there's a separate, ongoing tradeoff between how strongly consistent a system is and how much latency it incurs achieving that (e.g. waiting for a quorum of replicas to acknowledge adds latency, even when everything is healthy).

[↑ Back to top](#table-of-contents)

---

### 11. How does eventual consistency affect application-level design decisions? 🔴

- The application must account for the **possibility** of briefly stale reads — e.g. handling a "read-your-own-writes" requirement explicitly (routing certain reads to the primary, see [System Design › Database Design](database-design-and-sharding.md#8-how-does-replication-lag-affect-application-design)), designing UI feedback that doesn't assume instant global visibility of a change, and avoiding business logic that silently assumes every replica is always perfectly in sync.

[↑ Back to top](#table-of-contents)

---

### 12. How would you choose a consistency model for a specific feature? 🔴

- Match the model to the feature's actual **tolerance for staleness/conflict**: a bank account balance needs **strong** consistency (an incorrect balance is unacceptable, even briefly) — a "like" counter on a social post can tolerate **eventual** consistency just fine (a momentarily-off count by a few likes is harmless, and the gain in availability/performance is well worth it). Treating every feature with the same blanket consistency requirement either over-engineers the unimportant cases or under-protects the critical ones.

> [!IMPORTANT]
> **Follow-up questions:**
> - For a feature like "available inventory count" on an e-commerce checkout page, would you lean toward strong or eventual consistency, and what's the actual business risk of getting that choice wrong in either direction?

[↑ Back to top](#table-of-contents)

---
