# Core Concepts

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is Node.js?](#1-what-is-nodejs)
- [2. What is the Node.js process model?](#2-what-is-the-nodejs-process-model)
- [3. What are the key features of Node.js?](#3-what-are-the-key-features-of-nodejs)
- [4. What is the V8 engine, and how does Node.js use it?](#4-what-is-the-v8-engine-and-how-does-nodejs-use-it)
- [5. What is the REPL?](#5-what-is-the-repl)

**🟡 Medium**
- [6. What is libuv, and why does Node.js need it?](#6-what-is-libuv-and-why-does-nodejs-need-it)
- [7. What is the Reactor Pattern in Node.js?](#7-what-is-the-reactor-pattern-in-nodejs)
- [8. What are global objects in Node.js?](#8-what-are-global-objects-in-nodejs)
- [9. What's the difference between `npm` and `npx`?](#9-whats-the-difference-between-npm-and-npx)
- [10. What is the purpose of `package.json`?](#10-what-is-the-purpose-of-packagejson)
- [11. What is `package-lock.json`, and why is it important?](#11-what-is-package-lockjson-and-why-is-it-important)
- [12. What is semantic versioning?](#12-what-is-semantic-versioning)
- [13. What is `NODE_ENV`, and how do you use it?](#13-what-is-node_env-and-how-do-you-use-it)

**🔴 Hard**
- [14. What's the difference between Node.js and a browser's JavaScript runtime?](#14-whats-the-difference-between-nodejs-and-a-browsers-javascript-runtime)
- [15. What is JIT compilation, and how does it relate to Node.js performance?](#15-what-is-jit-compilation-and-how-does-it-relate-to-nodejs-performance)
- [16. What does "runtime environment" mean in the context of Node.js?](#16-what-does-runtime-environment-mean-in-the-context-of-nodejs)
- [17. What are Node.js's core/built-in modules?](#17-what-are-nodejss-corebuilt-in-modules)
- [18. What's the difference between a process and a thread in Node.js?](#18-whats-the-difference-between-a-process-and-a-thread-in-nodejs)

---

### 1. What is Node.js? 🟢

- A JavaScript **runtime** built on Chrome's V8 engine that lets JS run **outside the browser** — on servers, CLIs, and desktop apps — adding APIs for file systems, networking, and processes that browsers don't expose.

[↑ Back to top](#table-of-contents)

---

### 2. What is the Node.js process model? 🟢

- A **single main thread** runs your JS code via an event loop, while I/O work (file/network operations) is offloaded to the system or a background thread pool (via libuv) — the main thread is never blocked waiting on I/O, it just gets notified via a callback once the work completes.

[↑ Back to top](#table-of-contents)

---

### 3. What are the key features of Node.js? 🟢

- Single-threaded, event-driven, non-blocking I/O; built on V8 for fast execution; huge package ecosystem (npm); cross-platform; well suited for I/O-heavy, real-time apps (APIs, chat, streaming).

[↑ Back to top](#table-of-contents)

---

### 4. What is the V8 engine, and how does Node.js use it? 🟢

- Google's open-source JavaScript engine (also used in Chrome) — compiles JS to machine code via JIT compilation. Node.js embeds V8 to execute JS, then layers its own APIs (`fs`, `http`, etc.) and the event loop (via libuv) on top, since V8 alone only understands JavaScript, not file systems or networking.

[↑ Back to top](#table-of-contents)

---

### 5. What is the REPL? 🟢

- **R**ead **E**val **P**rint **L**oop — an interactive shell (`node` with no file argument) that reads a JS expression, evaluates it, prints the result, then loops — useful for quickly testing snippets without creating a file.

[↑ Back to top](#table-of-contents)

---

### 6. What is libuv, and why does Node.js need it? 🟡

- A C library that provides the **event loop** itself, plus a thread pool for operations that can't be done asynchronously at the OS level (some file system calls, DNS lookups, crypto). Node.js needs it because V8 alone has no concept of an event loop, I/O, or threading — libuv is what actually makes Node's non-blocking I/O model work under the hood.

[↑ Back to top](#table-of-contents)

---

### 7. What is the Reactor Pattern in Node.js? 🟡

- The design pattern behind Node's concurrency model: instead of blocking on each I/O operation, you register a **callback (handler)** for an event, and a central event loop ("reactor") dispatches that callback once the corresponding I/O operation completes — letting one thread handle many concurrent operations without blocking on any of them.

[↑ Back to top](#table-of-contents)

---

### 8. What are global objects in Node.js? 🟡

- Values available in every module without an explicit `require`/`import`: `process`, `console`, `__dirname`, `__filename`, `Buffer`, `setTimeout`/`setInterval`, and `global` itself (Node's equivalent of the browser's `window`).

```js
console.log(__dirname);  // absolute path of the current module's directory
console.log(process.version); // current Node.js version
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `npm` and `npx`? 🟡

- `npm`: the package manager — installs and manages dependencies.
- `npx`: executes a package's binary **without** needing it permanently installed — downloads it temporarily (or uses a local install if present) and runs it, useful for one-off CLI tools.

```bash
npm install create-react-app  # installs it permanently
npx create-react-app my-app    # runs it once, no permanent install needed
```

[↑ Back to top](#table-of-contents)

---

### 10. What is the purpose of `package.json`? 🟡

- The manifest file for a Node.js project — declares dependencies, scripts, metadata (name, version, entry point), and configuration for various tools, all in one place.

[↑ Back to top](#table-of-contents)

---

### 11. What is `package-lock.json`, and why is it important? 🟡

- Records the **exact** resolved version (and dependency tree) of every installed package, including transitive dependencies — ensures everyone on the team (and CI/production) installs the **identical** dependency tree, instead of `package.json`'s looser version ranges potentially resolving differently over time.

[↑ Back to top](#table-of-contents)

---

### 12. What is semantic versioning? 🟡

- A version numbering convention: `MAJOR.MINOR.PATCH` — `MAJOR` for breaking changes, `MINOR` for backward-compatible new features, `PATCH` for backward-compatible bug fixes. npm's `^`/`~` prefixes use this to determine which updates are considered "safe" to auto-install.

```
^1.2.3  → accepts 1.x.x (any minor/patch update, not 2.0.0)
~1.2.3  → accepts 1.2.x (only patch updates)
```

[↑ Back to top](#table-of-contents)

---

### 13. What is `NODE_ENV`, and how do you use it? 🟡

- A conventional environment variable indicating the current environment (`development`, `production`, `test`) — many libraries (Express, etc.) check it to toggle behavior (e.g. disabling verbose error pages in production).

```bash
NODE_ENV=production node server.js
```

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between Node.js and a browser's JavaScript runtime? 🔴

- Both use a JS engine + event loop, but expose **different global APIs**: Node.js has no `window`/`document`/DOM, but adds `fs`, `process`, `Buffer`, and module systems (CommonJS/ESM) browsers don't have. Browsers add DOM/Web APIs (`fetch`, `localStorage`) Node doesn't natively have (though some are now polyfilled in Node).
- They also differ in **module resolution** (browsers fetch over HTTP per `<script type="module">`; Node resolves from the local filesystem/`node_modules`) and security model (browsers sandbox JS away from the OS; Node JS has full filesystem/network access by default).

[↑ Back to top](#table-of-contents)

---

### 15. What is JIT compilation, and how does it relate to Node.js performance? 🔴

- **J**ust-**I**n-**T**ime compilation: V8 compiles JS to machine code **at runtime**, profiling which functions are "hot" (called often) and re-compiling those with heavier optimizations — rather than purely interpreting line-by-line. This is what gives Node.js (and V8 generally) much of its performance compared to naive interpretation, but also means warm-up time matters, and inconsistent code shapes can cause de-optimization (see [JavaScript › Core Concepts](../javascript/core-concepts.md#12-what-is-the-javascript-engine-and-how-does-v8-optimize-execution)).

[↑ Back to top](#table-of-contents)

---

### 16. What does "runtime environment" mean in the context of Node.js? 🔴

- The complete environment in which JS code executes — not just the language engine (V8), but everything around it: the event loop, the available global APIs (`fs`, `http`, `process`), the module system, and how the OS-level resources (files, sockets, processes) are exposed to JS code. "Node.js is a runtime environment" emphasizes that it's more than a library — it's the whole execution context.

[↑ Back to top](#table-of-contents)

---

### 17. What are Node.js's core/built-in modules? 🔴

- Modules shipped **with** Node itself, usable via `require('name')`/`import 'node:name'` with no install needed: `fs` (file system), `http`/`https` (servers/requests), `path` (path utilities), `os` (OS info), `crypto` (hashing/encryption), `events` (EventEmitter), `stream`, `util`, `child_process`, `cluster`, `dns`, `url`, `querystring`, `assert`, `vm`, among others.

[↑ Back to top](#table-of-contents)

---

### 18. What's the difference between a process and a thread in Node.js? 🔴

- A **process** is an independent running instance of the Node.js program, with its own memory space — `cluster`/`child_process` create separate processes that don't share memory directly (communicate via IPC/message-passing).
- A **thread** runs within a process, sharing the same memory — Node's main JS execution is single-threaded, but `worker_threads` lets you run JS on **additional threads within the same process**, which (unlike separate processes) can share memory via `SharedArrayBuffer`.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might you choose `worker_threads` over spawning a whole new process for CPU-bound work?

[↑ Back to top](#table-of-contents)

---
