# Performance

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you measure function execution time in Node.js?](#1-how-do-you-measure-function-execution-time-in-nodejs)
- [2. What is the purpose of compression middleware?](#2-what-is-the-purpose-of-compression-middleware)
- [3. How does caching improve Node.js application performance?](#3-how-does-caching-improve-nodejs-application-performance)

**🟡 Medium**
- [4. How does connection pooling improve database performance?](#4-how-does-connection-pooling-improve-database-performance)
- [5. How do you debug a Node.js application?](#5-how-do-you-debug-a-nodejs-application)
- [6. How do you use `util.debuglog` for conditional debug logging?](#6-how-do-you-use-utildebuglog-for-conditional-debug-logging)
- [7. What general techniques improve Node.js performance?](#7-what-general-techniques-improve-nodejs-performance)

**🔴 Hard**
- [8. How do you profile CPU performance in a Node.js app?](#8-how-do-you-profile-cpu-performance-in-a-nodejs-app)
- [9. How do you take and compare heap snapshots to find a memory leak?](#9-how-do-you-take-and-compare-heap-snapshots-to-find-a-memory-leak)
- [10. How would you implement caching with Redis in Node.js?](#10-how-would-you-implement-caching-with-redis-in-nodejs)
- [11. How would you implement simple rate limiting in a Node.js app?](#11-how-would-you-implement-simple-rate-limiting-in-a-nodejs-app)
- [12. What's the difference between vertical and horizontal scaling for a Node.js app?](#12-whats-the-difference-between-vertical-and-horizontal-scaling-for-a-nodejs-app)
- [13. How do you identify whether a performance bottleneck is CPU-bound or I/O-bound?](#13-how-do-you-identify-whether-a-performance-bottleneck-is-cpu-bound-or-io-bound)
- [14. How does scaling with `cluster`/PM2 relate to raw per-request performance vs. availability?](#14-how-does-scaling-with-clusterpm2-relate-to-raw-per-request-performance-vs-availability)

---

### 1. How do you measure function execution time in Node.js? 🟢

```js
const start = performance.now();
doExpensiveWork();
console.log(`Took ${performance.now() - start}ms`);

// or, for a quick CLI check:
console.time('work');
doExpensiveWork();
console.timeEnd('work');
```

[↑ Back to top](#table-of-contents)

---

### 2. What is the purpose of compression middleware? 🟢

- Compresses HTTP response bodies (typically with gzip/brotli) before sending them — significantly reduces payload size over the network, especially for text-based responses (HTML/JSON/CSS), at the cost of some CPU time spent compressing.

```js
const compression = require('compression');
app.use(compression());
```

[↑ Back to top](#table-of-contents)

---

### 3. How does caching improve Node.js application performance? 🟢

- Avoids redoing expensive work (a slow DB query, an external API call, heavy computation) by storing and reusing the result for subsequent identical requests — trades a bit of memory/storage for significantly reduced latency and load on downstream systems.

[↑ Back to top](#table-of-contents)

---

### 4. How does connection pooling improve database performance? 🟡

- Opening a new database connection per request is expensive (TCP handshake, auth). A connection **pool** keeps a set of already-open connections ready to be reused across requests — avoiding that per-request setup cost, and capping the total number of concurrent connections to avoid overwhelming the database.

```js
const pool = new Pool({ max: 20 }); // reuse up to 20 open connections
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you debug a Node.js application? 🟡

- Run with `--inspect` (or `--inspect-brk` to pause on the first line) and attach Chrome DevTools or VS Code's debugger — lets you set breakpoints, step through code, and inspect variables, instead of relying purely on `console.log`.

```bash
node --inspect-brk app.js
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you use `util.debuglog` for conditional debug logging? 🟡

- Creates a logger that only actually prints when the corresponding namespace is listed in the `NODE_DEBUG` environment variable — lets you leave detailed debug logging in production code with **zero output** (and minimal overhead) unless explicitly enabled.

```js
const debug = require('util').debuglog('myapp');
debug('Processing request %s', requestId); // silent unless NODE_DEBUG=myapp
```

```bash
NODE_DEBUG=myapp node app.js  # now debug() calls print
```

[↑ Back to top](#table-of-contents)

---

### 7. What general techniques improve Node.js performance? 🟡

- Use async I/O everywhere (avoid sync calls in request paths), cache expensive/repeated computations and queries, use streams for large data, scale across cores with `cluster`/PM2, use compression and connection pooling, and profile **before** optimizing — don't guess at bottlenecks.

[↑ Back to top](#table-of-contents)

---

### 8. How do you profile CPU performance in a Node.js app? 🔴

- Run with `--prof` to generate a V8 log of where CPU time is actually spent, then process it into a readable summary — or use the Chrome DevTools "Performance" tab via `--inspect` for a visual flame graph, which is usually easier to interpret.

```bash
node --prof app.js
node --prof-process isolate-0x*.log > processed.txt
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you take and compare heap snapshots to find a memory leak? 🔴

- Connect Chrome DevTools to a running `--inspect` process, take a heap snapshot, perform the suspected leaking action repeatedly, take another snapshot, then use DevTools' **comparison view** to see which object types grew between the two — trace their retainers to find what's holding onto them.

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement caching with Redis in Node.js? 🔴

```js
const redis = require('redis');
const client = redis.createClient();

async function getUser(id) {
  const cached = await client.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  const user = await db.users.findById(id);
  await client.set(`user:${id}`, JSON.stringify(user), { EX: 3600 }); // expire in 1 hour
  return user;
}
```

[↑ Back to top](#table-of-contents)

---

### 11. How would you implement simple rate limiting in a Node.js app? 🔴

- Track request counts per client (IP/user/API key) within a sliding/fixed time window, using an in-memory store for a single instance or Redis for a distributed setup, and reject requests once the limit is exceeded.

```js
const rateLimit = require('express-rate-limit');
app.use(rateLimit({ windowMs: 60_000, max: 100 })); // 100 requests per minute per IP
```

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between vertical and horizontal scaling for a Node.js app? 🔴

- **Vertical scaling**: giving a single machine more resources (CPU/RAM) — simple, but has a hard ceiling and a single point of failure.
- **Horizontal scaling**: running **more instances** (processes/containers/machines) behind a load balancer — better fault tolerance and effectively unlimited ceiling, but requires the app to be stateless (or externalize state to Redis/a DB) so any instance can serve any request.

[↑ Back to top](#table-of-contents)

---

### 13. How do you identify whether a performance bottleneck is CPU-bound or I/O-bound? 🔴

- **CPU-bound**: the event loop itself is busy/blocked — CPU usage stays high, and even simple unrelated requests slow down because the single thread is occupied computing something. A CPU profile (Q8) will show time concentrated in your own JS functions.
- **I/O-bound**: CPU usage stays low/idle, but requests are slow because they're waiting on external responses (DB, network, disk) — the event loop is mostly idle, just waiting for callbacks to fire. Look at wait times in DB/API call metrics rather than CPU profiles.

[↑ Back to top](#table-of-contents)

---

### 14. How does scaling with `cluster`/PM2 relate to raw per-request performance vs. availability? 🔴

- Clustering doesn't make any **single** request faster — each worker still processes its own requests on one thread, same as before. What it improves is **throughput** (more requests handled concurrently, since they're spread across cores) and **availability** (one worker crashing doesn't take down the whole app, since others keep serving and the primary can respawn it).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why wouldn't adding more cluster workers help if your bottleneck is actually a slow downstream database, not CPU?

[↑ Back to top](#table-of-contents)

---
