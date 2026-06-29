# Event Loop & Async

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is the event loop, and how does it work?](#1-what-is-the-event-loop-and-how-does-it-work)
- [2. What is asynchronous programming in Node.js?](#2-what-is-asynchronous-programming-in-nodejs)
- [3. What's the difference between asynchronous and non-blocking?](#3-whats-the-difference-between-asynchronous-and-non-blocking)
- [4. What is a callback function?](#4-what-is-a-callback-function)
- [5. What is an error-first callback?](#5-what-is-an-error-first-callback)

**🟡 Medium**
- [6. What is callback hell, and how do you avoid it?](#6-what-is-callback-hell-and-how-do-you-avoid-it)
- [7. What's the difference between Events and Callbacks?](#7-whats-the-difference-between-events-and-callbacks)
- [8. What is `EventEmitter`, and how does it work?](#8-what-is-eventemitter-and-how-does-it-work)
- [9. What are the common `EventEmitter` methods?](#9-what-are-the-common-eventemitter-methods)
- [10. What are the phases of the Node.js event loop?](#10-what-are-the-phases-of-the-nodejs-event-loop)
- [11. What's the difference between `process.nextTick()` and `setImmediate()`?](#11-whats-the-difference-between-processnexttick-and-setimmediate)
- [12. What's the difference between microtasks and macrotasks in Node.js?](#12-whats-the-difference-between-microtasks-and-macrotasks-in-nodejs)
- [13. How do Promises compare to async/await?](#13-how-do-promises-compare-to-asyncawait)
- [14. What's the difference between `Promise.all`, `allSettled`, `race`, and `any`?](#14-whats-the-difference-between-promiseall-allsettled-race-and-any)

**🔴 Hard**
- [15. Is Node.js entirely single-threaded? How does it handle concurrency despite that?](#15-is-nodejs-entirely-single-threaded-how-does-it-handle-concurrency-despite-that)
- [16. What is typically the first argument passed to a Node.js callback handler, and why?](#16-what-is-typically-the-first-argument-passed-to-a-nodejs-callback-handler-and-why)
- [17. What are the timing features of Node.js?](#17-what-are-the-timing-features-of-nodejs)
- [18. How would you implement a `sleep` function in Node.js?](#18-how-would-you-implement-a-sleep-function-in-nodejs)
- [19. How does Node.js prevent blocking code from freezing the whole server?](#19-how-does-nodejs-prevent-blocking-code-from-freezing-the-whole-server)
- [20. What evolution did async code in Node.js go through (callbacks → promises → async/await)?](#20-what-evolution-did-async-code-in-nodejs-go-through-callbacks--promises--asyncawait)

---

### 1. What is the event loop, and how does it work? 🟢

- The mechanism that lets Node's single thread handle many concurrent operations: it continuously checks if the call stack is empty, then pulls the next pending callback (from timers, I/O completions, etc.) and runs it — repeating indefinitely while the process is alive.

[↑ Back to top](#table-of-contents)

---

### 2. What is asynchronous programming in Node.js? 🟢

- Code that doesn't block execution while waiting for an operation (file read, network request, timer) to complete — instead, it registers a callback/Promise to be resolved **later**, letting the rest of the program keep running in the meantime.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between asynchronous and non-blocking? 🟢

- **Asynchronous**: describes the *programming model* — code that doesn't wait, and is notified later via callback/event/Promise.
- **Non-blocking**: describes *I/O behavior at the system level* — an I/O call returns immediately (even if data isn't ready yet) instead of halting the thread until it is.
- In practice in Node.js the two go hand-in-hand, but they describe different layers: async is about your code's structure, non-blocking is about how the underlying I/O call behaves.

[↑ Back to top](#table-of-contents)

---

### 4. What is a callback function? 🟢

- A function passed as an argument to be invoked later, once an operation completes — Node's original (pre-Promise) mechanism for async results.

```js
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

[↑ Back to top](#table-of-contents)

---

### 5. What is an error-first callback? 🟢

- The Node.js convention where a callback's **first parameter** is always reserved for an error (or `null` if none occurred), with subsequent parameters carrying the actual result — lets callers handle failures consistently across the entire ecosystem.

```js
function readConfig(callback) {
  fs.readFile('config.json', (err, data) => {
    if (err) return callback(err);
    callback(null, JSON.parse(data));
  });
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What is callback hell, and how do you avoid it? 🟡

- Deeply nested callbacks (each dependent on the previous result) that become hard to read, debug, and handle errors in consistently.
- Avoided with Promises (flatten via `.then()` chaining) or, better, `async`/`await` for sequential-looking async code.

```js
// Callback hell
getUser(id, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      console.log(comments);
    });
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between Events and Callbacks? 🟡

- A **callback** is typically called **once**, tied to a single specific operation's completion (a file finished reading).
- An **event** (via `EventEmitter`) can fire **multiple times**, and multiple independent listeners can subscribe to the same event — a more general pub-sub mechanism rather than a one-off "do this when done" handler.

[↑ Back to top](#table-of-contents)

---

### 8. What is `EventEmitter`, and how does it work? 🟡

- The core class (`require('events')`) that implements Node's event-driven pattern — objects can `emit()` named events, and other code can `on()` (subscribe) to react when those events fire. Many Node built-ins (`http.Server`, streams) extend `EventEmitter` internally.

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();

emitter.on('greet', (name) => console.log(`Hello, ${name}`));
emitter.emit('greet', 'Vaibhav'); // "Hello, Vaibhav"
```

[↑ Back to top](#table-of-contents)

---

### 9. What are the common `EventEmitter` methods? 🟡

- `.on(event, listener)` — subscribe (can fire multiple times).
- `.once(event, listener)` — subscribe, auto-unsubscribe after the first fire.
- `.emit(event, ...args)` — trigger the event, synchronously calling all listeners.
- `.off(event, listener)` / `.removeListener()` — unsubscribe.
- `.removeAllListeners(event)` — clear all listeners for an event.

[↑ Back to top](#table-of-contents)

---

### 10. What are the phases of the Node.js event loop? 🟡

In order, each loop iteration ("tick") passes through:
1. **Timers** — `setTimeout`/`setInterval` callbacks whose time has elapsed.
2. **Pending callbacks** — certain system-level callbacks deferred from the previous loop.
3. **Poll** — retrieves new I/O events; executes I/O-related callbacks (most of them).
4. **Check** — `setImmediate()` callbacks.
5. **Close callbacks** — e.g. `socket.on('close', ...)`.
- (`process.nextTick()` and Promise microtasks actually run **between every phase**, not as a phase of their own — see Q11/Q12.)

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between `process.nextTick()` and `setImmediate()`? 🟡

- `process.nextTick(fn)`: runs `fn` **immediately after the current operation completes**, before the event loop continues to its next phase — highest priority, runs before any I/O or timers.
- `setImmediate(fn)`: runs `fn` in the **check** phase, after the current poll phase's I/O callbacks.

```js
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
// Output: "nextTick" then "setImmediate" — nextTick always wins
```

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between microtasks and macrotasks in Node.js? 🟡

- **Microtasks** (`process.nextTick` callbacks, resolved Promise `.then()` callbacks): drained **completely** before the event loop moves to the next phase/macrotask.
- **Macrotasks** (`setTimeout`, `setImmediate`, I/O callbacks): each phase processes its macrotasks, then microtasks are drained again before moving on.
- `process.nextTick`'s queue is actually processed **before** the Promise microtask queue, both being higher priority than any macrotask.

[↑ Back to top](#table-of-contents)

---

### 13. How do Promises compare to async/await? 🟡

- `async`/`await` is syntax sugar **over** Promises — functionally equivalent, but reads more like synchronous code and makes sequential error handling (`try`/`catch`) cleaner than chaining multiple `.then()`/`.catch()` calls (see [JavaScript › Async, Promises & Event Loop](../javascript/async-and-promises.md#10-whats-the-difference-between-asyncawait-and-using-thencatch-directly) for the deeper comparison).

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between `Promise.all`, `allSettled`, `race`, and `any`? 🟡

- `Promise.all`: waits for all to fulfill; rejects fast on the first rejection.
- `Promise.allSettled`: waits for all, regardless of outcome — never rejects, gives per-promise status.
- `Promise.race`: settles as soon as the first promise settles, fulfilled or rejected.
- `Promise.any`: settles as soon as the first one **fulfills**; rejects only if all reject.

```js
const results = await Promise.allSettled([fetchA(), fetchB()]);
// [{status:'fulfilled', value:...}, {status:'rejected', reason:...}]
```

[↑ Back to top](#table-of-contents)

---

### 15. Is Node.js entirely single-threaded? How does it handle concurrency despite that? 🔴

- **No** — your JS code runs on a single main thread, but Node uses libuv's **thread pool** (default size 4) under the hood for certain operations (some `fs` calls, DNS lookups, crypto, compression) that can't be done via the OS's native async mechanisms.
- Network I/O is typically handled by the OS's own async/non-blocking primitives (epoll/kqueue/IOCP), not the thread pool at all — so "concurrency" in Node comes from a mix of OS-level async I/O **and** a background thread pool for the rest, all coordinated by the single-threaded event loop.

[↑ Back to top](#table-of-contents)

---

### 16. What is typically the first argument passed to a Node.js callback handler, and why? 🔴

- An `error` object (or `null` if none) — the error-first convention (Q5) — chosen so the calling code is **forced to at least consider** the error case before reaching the actual result, rather than errors being easy to silently ignore.

[↑ Back to top](#table-of-contents)

---

### 17. What are the timing features of Node.js? 🔴

- `setTimeout(fn, ms)` — runs once after at least `ms` milliseconds.
- `setInterval(fn, ms)` — runs repeatedly every `ms` milliseconds.
- `setImmediate(fn)` — runs once, in the check phase of the current loop iteration.
- `process.nextTick(fn)` — runs once, before the event loop proceeds at all.
- None of these guarantee **exact** timing — they guarantee a minimum delay; actual execution depends on what else is queued/running.

[↑ Back to top](#table-of-contents)

---

### 18. How would you implement a `sleep` function in Node.js? 🔴

- There's no built-in blocking sleep (that would defeat Node's non-blocking model) — implement it as a Promise wrapping `setTimeout`, then `await` it.

```js
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function run() {
  console.log('Start');
  await sleep(1000);
  console.log('1 second later');
}
```

[↑ Back to top](#table-of-contents)

---

### 19. How does Node.js prevent blocking code from freezing the whole server? 🔴

- It doesn't, automatically — Node.js **can** be blocked: a long synchronous CPU-bound loop, or a sync `fs` call, occupies the single main thread and stalls **every** other request being handled by that process, since there's no preemptive multitasking on that thread.
- The "prevention" is architectural discipline: use async versions of APIs, offload CPU-heavy work to `worker_threads`/a separate process, and avoid long synchronous loops in request-handling code.

> [!IMPORTANT]
> **Follow-up questions:**
> - What's a concrete symptom you'd see in production logs/metrics if someone accidentally used a blocking sync call in a hot code path?

[↑ Back to top](#table-of-contents)

---

### 20. What evolution did async code in Node.js go through (callbacks → promises → async/await)? 🔴

1. **Callbacks** (original): simple but prone to callback hell and inconsistent error handling.
2. **Promises** (ES6 / `util.promisify`): chainable `.then()`/`.catch()`, flattens nesting, standardizes error propagation via rejection.
3. **`async`/`await`** (ES2017): reads like synchronous code on top of Promises, with `try`/`catch` for errors — the current standard for new code.
- Node's own core APIs have followed this same path — many `fs`/`dns`/etc. modules now expose a `*/promises` variant alongside their original callback-based API.

[↑ Back to top](#table-of-contents)

---
