# Async, Promises & Event Loop

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between synchronous and asynchronous code?](#1-whats-the-difference-between-synchronous-and-asynchronous-code)
- [2. What is a callback, and what is "callback hell"?](#2-what-is-a-callback-and-what-is-callback-hell)
- [3. What is a Promise, and what states can it be in?](#3-what-is-a-promise-and-what-states-can-it-be-in)
- [4. How do you create and consume a basic Promise?](#4-how-do-you-create-and-consume-a-basic-promise)
- [5. What does `async`/`await` do?](#5-what-does-asyncawait-do)

**🟡 Medium**
- [6. What is the event loop?](#6-what-is-the-event-loop)
- [7. What's the difference between the microtask queue and the macrotask queue?](#7-whats-the-difference-between-the-microtask-queue-and-the-macrotask-queue)
- [8. What's the difference between `Promise.all()`, `Promise.race()`, `Promise.any()`, and `Promise.allSettled()`?](#8-whats-the-difference-between-promiseall-promiserace-promiseany-and-promiseallsettled)
- [9. How do you handle errors in `async`/`await` code?](#9-how-do-you-handle-errors-in-asyncawait-code)
- [10. What's the difference between `async`/`await` and using `.then()`/`.catch()` directly?](#10-whats-the-difference-between-asyncawait-and-using-thencatch-directly)

**🔴 Hard**
- [11. What's the difference between `setTimeout`, `setImmediate`, and `process.nextTick()`?](#11-whats-the-difference-between-settimeout-setimmediate-and-processnexttick)
- [12. How would you implement a basic Promise from scratch?](#12-how-would-you-implement-a-basic-promise-from-scratch)
- [13. How would you run async tasks serially vs. in parallel, with limited concurrency?](#13-how-would-you-run-async-tasks-serially-vs-in-parallel-with-limited-concurrency)
- [14. What happens if a rejected promise is never handled?](#14-what-happens-if-a-rejected-promise-is-never-handled)

---

### 1. What's the difference between synchronous and asynchronous code? 🟢

- **Synchronous**: each line runs to completion before the next one starts — blocking.
- **Asynchronous**: an operation (timer, network request, file read) is started and the rest of the code continues running; the result is handled later via a callback, promise, or `await`.

```js
console.log('1');
setTimeout(() => console.log('2'), 0); // deferred, runs later
console.log('3');
// logs: 1, 3, 2
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a callback, and what is "callback hell"? 🟢

- A **callback** is a function passed to be run once an async operation completes.
- **Callback hell**: deeply nested callbacks (each depending on the previous one's result) that become hard to read and maintain — usually solved with Promises or `async`/`await`.

```js
getUser(id, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      console.log(comments); // nesting keeps growing →
    });
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a Promise, and what states can it be in? 🟢

- An object representing the eventual result of an async operation.
- Three states: **pending** (not yet resolved/rejected), **fulfilled** (succeeded, has a value), **rejected** (failed, has a reason).
- Once fulfilled or rejected, a promise is "settled" and can never change state again.

[↑ Back to top](#table-of-contents)

---

### 4. How do you create and consume a basic Promise? 🟢

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    success ? resolve('Done!') : reject('Failed!');
  }, 1000);
});

promise
  .then((result) => console.log(result))
  .catch((error) => console.error(error));
```

[↑ Back to top](#table-of-contents)

---

### 5. What does `async`/`await` do? 🟢

- `async` marks a function as always returning a Promise.
- `await` pauses execution **inside that function** until the awaited Promise settles, letting you write async code that reads like synchronous code.

```js
async function getData() {
  const response = await fetch('/api/user');
  const data = await response.json();
  return data;
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the event loop? 🟡

- The mechanism that lets single-threaded JavaScript handle async operations: once the call stack is empty, the event loop pulls the next callback from a queue and pushes it onto the stack to run.
- It continuously cycles: run synchronous code → drain all microtasks → run one macrotask → drain microtasks again → repeat.

> [!TIP]
> **Real-life example:** a `fetch()` request doesn't freeze the page — the event loop keeps the UI responsive while the network call runs in the background, then schedules the `.then()` callback once it resolves.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between the microtask queue and the macrotask queue? 🟡

- **Microtasks** (Promise callbacks, `queueMicrotask`): higher priority — **all** pending microtasks run before the next macrotask, even before screen repaint.
- **Macrotasks** (`setTimeout`, `setInterval`, I/O, UI events): lower priority — only **one** runs per event loop tick.

```js
console.log('1');
setTimeout(() => console.log('2'), 0);   // macrotask
Promise.resolve().then(() => console.log('3')); // microtask
console.log('4');
// logs: 1, 4, 3, 2 — microtask (3) beats macrotask (2)
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `Promise.all()`, `Promise.race()`, `Promise.any()`, and `Promise.allSettled()`? 🟡

- `Promise.all()`: waits for **all** to fulfill; rejects immediately if **any one** rejects.
- `Promise.allSettled()`: waits for **all** to finish regardless of outcome, returns each result's status (`fulfilled`/`rejected`) — never rejects itself.
- `Promise.race()`: settles as soon as the **first** promise settles (fulfilled or rejected).
- `Promise.any()`: settles as soon as the **first** one fulfills; only rejects if **all** reject.

```js
await Promise.all([p1, p2]);       // fails fast on first rejection
await Promise.allSettled([p1, p2]); // always resolves, with per-promise status
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you handle errors in `async`/`await` code? 🟡

- Wrap the `await` in a `try`/`catch` block — a rejected awaited promise throws synchronously-looking inside the `async` function.

```js
async function getUser() {
  try {
    const res = await fetch('/api/user');
    if (!res.ok) throw new Error('Request failed');
    return await res.json();
  } catch (err) {
    console.error('Failed to fetch user:', err);
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `async`/`await` and using `.then()`/`.catch()` directly? 🟡

- Functionally equivalent — `async`/`await` is syntax sugar over Promises, not a different mechanism.
- `async`/`await` reads more like synchronous code and makes sequential async steps and error handling (via `try`/`catch`) easier to follow than chained `.then()` calls, especially with multiple dependent steps.

> [!IMPORTANT]
> **Follow-up questions:**
> - When might `.then()` chaining still be preferable over `async`/`await`?
> - How would you run two independent `await` calls concurrently instead of sequentially?

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between `setTimeout`, `setImmediate`, and `process.nextTick()`? 🔴

- `process.nextTick()` (Node.js only): runs **before** the event loop continues to the next phase — even before microtasks/promises in some Node versions; highest priority.
- `setTimeout(fn, 0)`: runs in the **timers** phase, after at least the specified delay (browsers clamp minimum delay to ~4ms when nested).
- `setImmediate()` (Node.js only): runs in the **check** phase, generally after I/O callbacks in the current loop iteration — often (but not always) after `setTimeout(fn, 0)`.

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement a basic Promise from scratch? 🔴

```js
class MyPromise {
  #state = 'pending';
  #value;
  #callbacks = [];

  constructor(executor) {
    const resolve = (value) => this.#settle('fulfilled', value);
    const reject = (reason) => this.#settle('rejected', reason);
    try { executor(resolve, reject); } catch (e) { reject(e); }
  }

  #settle(state, value) {
    if (this.#state !== 'pending') return;
    this.#state = state;
    this.#value = value;
    this.#callbacks.forEach((cb) => cb());
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const run = () => {
        try {
          if (this.#state === 'fulfilled') resolve(onFulfilled?.(this.#value));
          if (this.#state === 'rejected') reject(onRejected?.(this.#value));
        } catch (e) { reject(e); }
      };
      this.#state === 'pending' ? this.#callbacks.push(run) : run();
    });
  }
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you add `.catch()` and `.finally()` to this implementation?
> - How does Promise chaining (`.then().then()`) actually flatten nested promises?

[↑ Back to top](#table-of-contents)

---

### 13. How would you run async tasks serially vs. in parallel, with limited concurrency? 🔴

```js
// Serial — one at a time, in order
async function runSerial(tasks) {
  const results = [];
  for (const task of tasks) results.push(await task());
  return results;
}

// Parallel — all at once
async function runParallel(tasks) {
  return Promise.all(tasks.map((task) => task()));
}

// Limited concurrency — at most `limit` running simultaneously
async function runWithLimit(tasks, limit) {
  const results = [];
  const executing = [];
  for (const task of tasks) {
    const p = task().then((res) => results.push(res));
    executing.push(p);
    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }
  await Promise.all(executing);
  return results;
}
```

[↑ Back to top](#table-of-contents)

---

### 14. What happens if a rejected promise is never handled? 🔴

- The runtime fires an `unhandledrejection` event (browser) or `unhandledRejection` event (Node.js) — it doesn't crash synchronously like a thrown error would, but in newer Node versions an unhandled rejection can terminate the process.
- Always attach a `.catch()` (or wrap in `try`/`catch` with `await`) on any promise chain you don't explicitly return/await elsewhere.

[↑ Back to top](#table-of-contents)

---
