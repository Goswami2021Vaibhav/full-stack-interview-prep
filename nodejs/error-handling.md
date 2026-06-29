# Error Handling

_Part of [Node.js](README.md) interview notes._

> General JS error-handling mechanics (`try`/`catch`, custom Error classes) are covered in [JavaScript › Error Handling](../javascript/error-handling.md). This file focuses on Node.js process-level error handling and memory issues.

## Table of Contents

**🟢 Easy**
- [1. What's the preferred method of resolving unhandled exceptions in Node.js?](#1-whats-the-preferred-method-of-resolving-unhandled-exceptions-in-nodejs)
- [2. What's the difference between operational and programmer errors?](#2-whats-the-difference-between-operational-and-programmer-errors)
- [3. How do you create a custom error type in Node.js?](#3-how-do-you-create-a-custom-error-type-in-nodejs)

**🟡 Medium**
- [4. What are the different error-handling approaches in Node.js?](#4-what-are-the-different-error-handling-approaches-in-nodejs)
- [5. How do you handle errors in async/await code?](#5-how-do-you-handle-errors-in-asyncawait-code)
- [6. What are the types of memory leaks in Node.js?](#6-what-are-the-types-of-memory-leaks-in-nodejs)
- [7. How do you prevent memory leaks in Node.js?](#7-how-do-you-prevent-memory-leaks-in-nodejs)

**🔴 Hard**
- [8. How does garbage collection work in Node.js?](#8-how-does-garbage-collection-work-in-nodejs)
- [9. How would you solve a "process out of memory" exception?](#9-how-would-you-solve-a-process-out-of-memory-exception)
- [10. How do you detect memory leaks in a running Node.js process?](#10-how-do-you-detect-memory-leaks-in-a-running-nodejs-process)
- [11. How should you handle uncaught exceptions vs. unhandled promise rejections?](#11-how-should-you-handle-uncaught-exceptions-vs-unhandled-promise-rejections)
- [12. How would you design a process-level error-handling strategy for a Node.js API?](#12-how-would-you-design-a-process-level-error-handling-strategy-for-a-nodejs-api)

---

### 1. What's the preferred method of resolving unhandled exceptions in Node.js? 🟢

- **Prevent** them in the first place (validate input, wrap risky calls in `try`/`catch`, attach `.catch()`/error listeners) rather than relying on process-level `uncaughtException` handlers as a primary strategy — by the time that fires, the process is in an **unknown state** and Node's own docs recommend treating it as a signal to log and restart, not to "recover" and keep running indefinitely.

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between operational and programmer errors? 🟢

- **Operational errors**: expected failures from the outside world a correctly-written program should anticipate (a failed network request, invalid user input, a file that doesn't exist) — handle these gracefully, return a proper error response.
- **Programmer errors**: actual bugs (calling a function with the wrong type, a typo, referencing `undefined`) — these indicate the code itself needs fixing, and trying to "handle" them at runtime often just masks a deeper problem.

[↑ Back to top](#table-of-contents)

---

### 3. How do you create a custom error type in Node.js? 🟢

```js
class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NotFoundError';
    this.statusCode = 404;
  }
}

throw new NotFoundError('User not found');
```

[↑ Back to top](#table-of-contents)

---

### 4. What are the different error-handling approaches in Node.js? 🟡

- **Callbacks**: error-first convention — check `err` as the first parameter.
- **Promises**: `.catch()` on the chain.
- **`async`/`await`**: wrap in `try`/`catch`.
- **Process-level safety nets**: `process.on('uncaughtException')`/`process.on('unhandledRejection')` for logging + controlled shutdown — a last resort, not a substitute for the above.

[↑ Back to top](#table-of-contents)

---

### 5. How do you handle errors in async/await code? 🟡

```js
async function getUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`Request failed: ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error('Failed to fetch user:', err);
    throw err; // re-throw or handle, depending on the caller's needs
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What are the types of memory leaks in Node.js? 🟡

- **Global variables** accumulating data unintentionally.
- **Forgotten timers/intervals** (`setInterval` never cleared) keeping their closure's data alive.
- **Event listeners never removed**, especially on long-lived emitters (`process`, shared singletons).
- **Closures** capturing large objects longer than necessary.
- **Caches with no eviction policy**, growing unbounded over time.

[↑ Back to top](#table-of-contents)

---

### 7. How do you prevent memory leaks in Node.js? 🟡

- Always `clearInterval`/`clearTimeout` when no longer needed; remove event listeners (`removeListener`) when an object/connection is done; use bounded caches (LRU with a max size, or a TTL); avoid storing unbounded growing data in module-level/global variables.

[↑ Back to top](#table-of-contents)

---

### 8. How does garbage collection work in Node.js? 🔴

- Handled by V8's garbage collector, using a generational **mark-and-sweep** approach: short-lived objects are collected quickly from a small "young generation" heap; objects that survive multiple collection cycles get promoted to the "old generation," collected less frequently (since scanning it is more expensive). An object becomes eligible for collection once it's no longer **reachable** from any root reference.

[↑ Back to top](#table-of-contents)

---

### 9. How would you solve a "process out of memory" exception? 🔴

- Identify the cause first via heap snapshots/profiling (Q10) rather than guessing. Common fixes: raise V8's heap size limit (`--max-old-space-size`) **as a stopgap**, fix actual leaks (unbounded caches, lingering listeners), process large datasets via streams instead of loading them fully into memory, and paginate/batch large queries instead of loading everything at once.

```bash
node --max-old-space-size=4096 app.js
```

[↑ Back to top](#table-of-contents)

---

### 10. How do you detect memory leaks in a running Node.js process? 🔴

- Take **heap snapshots** at different points in time (via `--inspect` + Chrome DevTools, or the `heapdump` package) while repeating the same operation, and compare them — objects/counts that keep growing across snapshots without ever being freed point to the leak's source. `process.memoryUsage()` gives a quick high-level read (heap used, RSS) for monitoring trends over time.

[↑ Back to top](#table-of-contents)

---

### 11. How should you handle uncaught exceptions vs. unhandled promise rejections? 🔴

- `process.on('uncaughtException', ...)`: catches synchronous errors that escaped every `try`/`catch` — log it and **terminate the process** (the docs explicitly recommend not trying to resume normal operation, since internal state may be corrupted); ideally let a process manager (PM2, Kubernetes) restart it.
- `process.on('unhandledRejection', ...)`: catches promise rejections with no `.catch()` anywhere — log it similarly; in modern Node, unhandled rejections increasingly **also** terminate the process by default (behavior has tightened across Node versions).

```js
process.on('uncaughtException', (err) => {
  logger.error(err);
  process.exit(1); // let the process manager restart a fresh instance
});
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you design a process-level error-handling strategy for a Node.js API? 🔴

- Validate input at the boundary (API routes), use a consistent custom error hierarchy with status codes (Q3), centralize formatting of error responses in one Express error-handling middleware, log operational errors without crashing, and only let genuinely unexpected/programmer errors trigger `uncaughtException`-driven process restarts — combined with a process manager that automatically respawns the crashed instance with minimal downtime.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is it considered bad practice to keep a Node.js process running indefinitely after an `uncaughtException`, even if you "handle" it?

[↑ Back to top](#table-of-contents)

---
