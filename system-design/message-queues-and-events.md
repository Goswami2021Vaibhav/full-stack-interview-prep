# Message Queues & Events

_Part of [System Design](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a message queue?](#1-what-is-a-message-queue)
- [2. Why use a message queue instead of a direct synchronous call?](#2-why-use-a-message-queue-instead-of-a-direct-synchronous-call)
- [3. What's the difference between a message queue and a pub/sub system?](#3-whats-the-difference-between-a-message-queue-and-a-pubsub-system)

**🟡 Medium**
- [4. What's the difference between at-least-once, at-most-once, and exactly-once delivery?](#4-whats-the-difference-between-at-least-once-at-most-once-and-exactly-once-delivery)
- [5. What is a dead-letter queue?](#5-what-is-a-dead-letter-queue)
- [6. Why is message ordering hard to guarantee at scale?](#6-why-is-message-ordering-hard-to-guarantee-at-scale)
- [7. What's the difference between Kafka and a traditional message queue like RabbitMQ?](#7-whats-the-difference-between-kafka-and-a-traditional-message-queue-like-rabbitmq)
- [8. What is backpressure in the context of a message queue/consumer?](#8-what-is-backpressure-in-the-context-of-a-message-queueconsumer)

**🔴 Hard**
- [9. How do you achieve idempotent message processing on the consumer side?](#9-how-do-you-achieve-idempotent-message-processing-on-the-consumer-side)
- [10. What is event sourcing?](#10-what-is-event-sourcing)
- [11. What is the outbox pattern, and what problem does it solve?](#11-what-is-the-outbox-pattern-and-what-problem-does-it-solve)
- [12. How would you design a notification system handling millions of events per day?](#12-how-would-you-design-a-notification-system-handling-millions-of-events-per-day)

---

### 1. What is a message queue? 🟢

- A component that holds messages sent by **producers** until they're picked up and processed by **consumers** — decouples the sender from the receiver in both time and direct dependency, so the producer doesn't need the consumer to be available *right now*.

[↑ Back to top](#table-of-contents)

---

### 2. Why use a message queue instead of a direct synchronous call? 🟢

- A direct call **blocks** the caller until the callee responds, and fails entirely if the callee is temporarily down. Routing through a queue lets the producer continue immediately, and the work gets processed **whenever** a consumer is available — improving resilience and letting consumers process at their own sustainable pace (smoothing out traffic spikes).

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between a message queue and a pub/sub system? 🟢

- **Message queue**: each message is typically consumed by **exactly one** consumer (from a pool) — good for distributing work.
- **Pub/sub**: a message published to a topic is delivered to **every** subscriber — good for broadcasting an event to multiple independent interested parties (e.g. "order placed" notifying both the email service and the analytics service).

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between at-least-once, at-most-once, and exactly-once delivery? 🟡

- **At-least-once**: a message might be delivered/processed **more than once** (e.g. after a retry following an ack that got lost) — consumers must handle duplicates (Q9).
- **At-most-once**: a message is delivered **zero or one** times — never duplicated, but might be lost entirely if something fails before delivery completes.
- **Exactly-once**: delivered and processed **exactly** one time — the ideal, but genuinely difficult/expensive to guarantee end-to-end in a distributed system; most real systems achieve an at-least-once delivery guarantee combined with idempotent processing to **effectively** behave like exactly-once.

[↑ Back to top](#table-of-contents)

---

### 5. What is a dead-letter queue? 🟡

- A separate queue where messages that **repeatedly fail** processing (after a configured number of retries) get routed instead of being retried forever or silently dropped — lets you inspect/debug failed messages later without blocking the main queue's processing of healthy messages.

[↑ Back to top](#table-of-contents)

---

### 6. Why is message ordering hard to guarantee at scale? 🟡

- Scaling a queue typically means **parallelizing** across multiple partitions/consumers for throughput — but ordering is naturally only well-defined **within** a single partition, processed by a single consumer; messages across different partitions can be processed concurrently and complete in any relative order. Guaranteeing global order generally requires giving up that parallelism (a single partition/consumer), trading throughput for strict ordering.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between Kafka and a traditional message queue like RabbitMQ? 🟡

- **Kafka**: a distributed **log** — messages are retained (for a configurable period) and consumers track their own read position, so multiple consumer groups can independently re-read the same stream of events; built for high-throughput event streaming.
- **RabbitMQ** (traditional queue): messages are typically **removed** once consumed/acknowledged — built more around per-message routing/task distribution rather than retained, replayable event streams.

[↑ Back to top](#table-of-contents)

---

### 8. What is backpressure in the context of a message queue/consumer? 🟡

- When a consumer can't keep up with the rate messages arrive, the **queue's backlog grows** — backpressure refers to mechanisms (rate-limited consumption, the queue itself filling and applying limits, or signaling producers to slow down) that prevent this from spiraling into unbounded memory growth or producers blindly overwhelming consumers indefinitely.

[↑ Back to top](#table-of-contents)

---

### 9. How do you achieve idempotent message processing on the consumer side? 🔴

- Since most real systems provide at-least-once delivery (Q4), consumers must handle the **same** message arriving more than once without it causing duplicate side effects (e.g. charging a customer twice). Common approach: track a unique message/idempotency ID per processed message (in a database or dedicated dedup store), and **skip** processing if that ID has already been recorded as handled.

[↑ Back to top](#table-of-contents)

---

### 10. What is event sourcing? 🔴

- Instead of storing only the **current state** of an entity, store the full, immutable **sequence of events** that led to that state (e.g. "OrderCreated," "ItemAdded," "OrderShipped") — current state is derived by replaying events. Gives a complete audit trail and the ability to reconstruct state as of any point in time, at the cost of more complex querying (you often need a separate read-optimized projection, rather than querying raw events directly).

[↑ Back to top](#table-of-contents)

---

### 11. What is the outbox pattern, and what problem does it solve? 🔴

- Solves the problem of **atomically** updating a database **and** publishing a corresponding event — doing both as two separate operations risks one succeeding and the other failing (e.g. the DB write commits, but the message broker is briefly unreachable, silently losing the event). The outbox pattern writes the event into an **"outbox" table within the same database transaction** as the actual data change, then a separate background process reliably reads from that outbox table and publishes to the message broker — making the combined "update data + record intent to publish" atomic, even though the actual publish happens slightly later.

[↑ Back to top](#table-of-contents)

---

### 12. How would you design a notification system handling millions of events per day? 🔴

- Decouple event **producers** (any service triggering a notification) from **delivery** via a message queue, route events to channel-specific workers (email, push, SMS) that can scale independently based on their own throughput characteristics, batch where possible (digest emails instead of one email per event), apply per-user rate limiting/preference filtering before actually sending, and use a dead-letter queue (Q5) plus retries with backoff for transient delivery failures (a bounced email, a temporarily unreachable push service).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why would you want notification delivery to be decoupled by channel (separate email/push/SMS workers) rather than one single worker handling all channels?

[↑ Back to top](#table-of-contents)

---
