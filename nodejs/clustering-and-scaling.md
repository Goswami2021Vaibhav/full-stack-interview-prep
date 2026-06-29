# Clustering & Scaling

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Since Node.js is single-threaded, how do you make use of all CPU cores?](#1-since-nodejs-is-single-threaded-how-do-you-make-use-of-all-cpu-cores)
- [2. How does the `cluster` module work?](#2-how-does-the-cluster-module-work)

**🟡 Medium**
- [3. What cluster methods does Node.js support?](#3-what-cluster-methods-does-nodejs-support)
- [4. What is a load balancer, and how does it relate to clustering?](#4-what-is-a-load-balancer-and-how-does-it-relate-to-clustering)
- [5. How do you build a microservices architecture with Node.js?](#5-how-do-you-build-a-microservices-architecture-with-nodejs)

**🔴 Hard**
- [6. How do microservices communicate with each other?](#6-how-do-microservices-communicate-with-each-other)
- [7. How do you use PM2 for process management in production?](#7-how-do-you-use-pm2-for-process-management-in-production)
- [8. How do you gracefully shut down a Node.js server?](#8-how-do-you-gracefully-shut-down-a-nodejs-server)
- [9. How would you scale a Node.js app beyond a single machine's CPU cores?](#9-how-would-you-scale-a-nodejs-app-beyond-a-single-machines-cpu-cores)

---

### 1. Since Node.js is single-threaded, how do you make use of all CPU cores? 🟢

- Run **multiple Node.js processes** — one per core — each handling a share of incoming requests, via the built-in `cluster` module or a process manager like PM2's cluster mode.

[↑ Back to top](#table-of-contents)

---

### 2. How does the `cluster` module work? 🟢

- A **primary** process forks multiple **worker** processes (typically one per CPU core), each running a full copy of your app and listening on the same port — the primary distributes incoming connections across workers (commonly round-robin on most platforms), so requests are spread across all cores.

```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
} else {
  require('./server'); // each worker runs the actual app
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What cluster methods does Node.js support? 🟡

- `cluster.fork()`: spawn a new worker process.
- `cluster.isPrimary` / `cluster.isWorker`: check which role the current process has.
- `worker.send()` / `process.on('message')`: IPC messaging between primary and workers.
- `cluster.on('exit', ...)`: detect a worker dying (e.g. to automatically respawn it).

[↑ Back to top](#table-of-contents)

---

### 4. What is a load balancer, and how does it relate to clustering? 🟡

- A component that distributes incoming requests across multiple backend instances. Node's `cluster` module acts as a simple **built-in** load balancer across processes on a single machine; in production at larger scale, an **external** load balancer (Nginx, a cloud load balancer) typically distributes traffic across multiple **machines**, each possibly running its own Node cluster internally.

[↑ Back to top](#table-of-contents)

---

### 5. How do you build a microservices architecture with Node.js? 🟡

- Split a large application into multiple small, independently deployable services, each owning a specific business capability (e.g. a `users` service, an `orders` service) with its own data store, communicating over the network (REST/gRPC/message queues) rather than sharing in-process memory or a single database.

[↑ Back to top](#table-of-contents)

---

### 6. How do microservices communicate with each other? 🔴

- **Synchronous**: REST/HTTP or gRPC calls — simple, but the caller waits and is coupled to the other service's availability.
- **Asynchronous**: a message queue/broker (RabbitMQ, Kafka, SQS) — services publish/consume events without waiting on each other directly, improving resilience and decoupling, at the cost of added eventual-consistency complexity.

[↑ Back to top](#table-of-contents)

---

### 7. How do you use PM2 for process management in production? 🔴

- PM2 is a production process manager: runs your app in **cluster mode** (auto-forking one process per core), automatically restarts crashed processes, supports zero-downtime reloads, and provides built-in monitoring/logging.

```bash
pm2 start app.js -i max     # cluster mode, one instance per CPU core
pm2 list                     # view running processes
pm2 reload app                # zero-downtime restart
pm2 logs                      # view logs
```

[↑ Back to top](#table-of-contents)

---

### 8. How do you gracefully shut down a Node.js server? 🔴

- On receiving a termination signal (`SIGTERM`/`SIGINT`), stop accepting **new** connections, let **in-flight** requests finish, close database connections, then exit — instead of abruptly killing the process mid-request.

```js
process.on('SIGTERM', () => {
  server.close(() => {
    db.disconnect().then(() => process.exit(0));
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 9. How would you scale a Node.js app beyond a single machine's CPU cores? 🔴

- Once a single machine's cores are saturated via `cluster`/PM2, scale **horizontally**: run multiple separate machine/container instances behind a load balancer, store session state externally (Redis, not in-process memory) so any instance can handle any request, and consider splitting into microservices if different parts of the app have very different scaling needs.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does storing session state in-process (rather than in Redis/a shared store) break once you scale to multiple machines?

[↑ Back to top](#table-of-contents)

---
