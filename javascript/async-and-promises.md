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

- **Synchronous** code runs one line at a time, in order, and each line must finish before the next one starts. If a line takes a long time (e.g. a heavy calculation), everything else — including the UI — has to wait, because JavaScript normally runs on a single thread (one "worker" doing one thing at a time).
- **Asynchronous** code lets a slow operation (a timer, a network request, reading a file) start in the background without freezing the rest of the program. The code after it keeps running immediately, and the result of the slow operation is handled later — via a callback function, a Promise, or `await` — once it's ready.
- This is why JavaScript can stay responsive (e.g. a web page doesn't freeze) even though it only does one thing at a time: it hands off slow work and comes back to it later instead of sitting and waiting.

```js
console.log('1');
setTimeout(() => console.log('2'), 0); // scheduled to run later, even with a 0ms delay
console.log('3');
// logs: 1, 3, 2 — '3' runs before '2' because setTimeout's callback is deferred
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a callback, and what is "callback hell"? 🟢

- A **callback** is simply a function you pass as an argument to another function, with the instruction "call this once you're done" — it's the original, most basic way to handle the result of an asynchronous operation, before Promises existed.
- **Callback hell** happens when you need several async steps to happen in sequence, and each one depends on the result of the previous one. Since each step's callback has to be nested inside the previous one, the code grows sideways into a deep, hard-to-read pyramid shape (sometimes literally called the "pyramid of doom").
- The core problems are readability (hard to follow the flow) and error handling (each level needs its own error check). This is exactly the problem Promises and `async`/`await` were designed to solve — they let you write the same sequence of steps in a flat, top-to-bottom style.

```js
getUser(id, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      console.log(comments); // nesting keeps growing to the right with each new step →
    });
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a Promise, and what states can it be in? 🟢

- A Promise is an object that acts as a **placeholder for a value you don't have yet**, but will (or won't) get at some point in the future — like a ticket/receipt for something being prepared.
- It can be in exactly one of three states at any time:
  - **pending**: still waiting, no result yet.
  - **fulfilled**: the operation succeeded, and the promise now holds the resulting value.
  - **rejected**: the operation failed, and the promise now holds a reason/error explaining why.
- Once a promise moves from `pending` to either `fulfilled` or `rejected`, it's called **settled** — and a settled promise can never change state again or produce a different value. This one-way transition is what makes promises predictable and safe to hand around your code.

[↑ Back to top](#table-of-contents)

---

### 4. How do you create and consume a basic Promise? 🟢

- Create a Promise with `new Promise(executor)`. The `executor` function runs immediately and receives two functions as arguments: `resolve` (call this when the work succeeds, passing the result) and `reject` (call this when it fails, passing an error/reason).
- Consume it with `.then(onFulfilled)` to handle a successful result, and `.catch(onRejected)` to handle a failure. These can be chained onto the promise because `.then()` itself returns a new promise.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    success ? resolve('Done!') : reject('Failed!');
  }, 1000);
});

promise
  .then((result) => console.log(result))   // runs if resolve() was called
  .catch((error) => console.error(error)); // runs if reject() was called
```

[↑ Back to top](#table-of-contents)

---

### 5. What does `async`/`await` do? 🟢

- Putting `async` in front of a function declaration does two things: it makes the function always return a Promise (even if you just `return` a plain value, it gets automatically wrapped in a resolved Promise), and it allows you to use `await` inside that function.
- `await` can only be used inside an `async` function. It pauses that function's execution at that exact line until the Promise being awaited settles — if it fulfills, `await` gives you back the resolved value directly (no `.then()` needed); if it rejects, `await` throws the error, which you can catch with a normal `try`/`catch`.
- Importantly, `await` only pauses the `async` function it's inside of — it does **not** block the rest of the program. Other code elsewhere keeps running normally.
- The big benefit: async code that reads top-to-bottom like plain synchronous code, instead of being wrapped in `.then()` chains.

```js
async function getData() {
  const response = await fetch('/api/user'); // waits for the network response
  const data = await response.json();        // waits for the body to be parsed
  return data; // automatically wrapped in a resolved Promise
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the event loop? 🟡

- JavaScript itself runs on a single thread (the **call stack** — where function calls execute one at a time). But browsers/Node.js provide extra pieces alongside that thread: Web/Node APIs (timers, network requests) and task queues that hold callbacks waiting to run.
- The **event loop** is the mechanism that connects these pieces: it constantly checks "is the call stack empty?" As soon as it is, the event loop takes the next callback waiting in a queue and pushes it onto the stack so it can run.
- The repeating cycle is: run all synchronous code currently on the stack → once the stack is empty, drain (run) **all** pending microtasks → run just **one** macrotask → drain microtasks again → repeat. (See Q7 for what "microtask" and "macrotask" mean.)
- This is the trick that lets a single-threaded language handle timers, network calls, and UI events without freezing: slow operations run outside the call stack, and their callbacks get slotted back in only when the stack is free.

> [!TIP]
> **Real-life example:** a `fetch()` request doesn't freeze the page — the event loop keeps the UI responsive while the network call runs in the background, then schedules the `.then()` callback once it resolves.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between the microtask queue and the macrotask queue? 🟡

- Both are queues that hold callbacks waiting for the call stack to be empty, but they have different priority levels.
- **Microtasks** — things like Promise `.then()`/`.catch()` callbacks and `queueMicrotask()` — have higher priority. Whenever the call stack empties, the event loop runs **every** microtask currently waiting (including new ones added while draining the queue) before it's allowed to touch anything else, even before the browser repaints the screen.
- **Macrotasks** — things like `setTimeout`, `setInterval`, I/O, and UI events — have lower priority. The event loop only takes **one** macrotask per cycle, then goes back to fully draining microtasks again before taking the next macrotask.
- Practical effect: Promise-based code (microtasks) will always run before a `setTimeout` callback (a macrotask), even if the `setTimeout` delay is `0`.

```js
console.log('1');
setTimeout(() => console.log('2'), 0);   // macrotask — lower priority, waits its turn
Promise.resolve().then(() => console.log('3')); // microtask — higher priority, runs first
console.log('4');
// logs: 1, 4, 3, 2 — microtask (3) always beats macrotask (2), regardless of order written
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `Promise.all()`, `Promise.race()`, `Promise.any()`, and `Promise.allSettled()`? 🟡

All four take an array of promises and return a single combined promise, but they differ in when they settle and how they treat failures:

- `Promise.all()`: waits for **every** promise to fulfill, and resolves with an array of all their values, in order. But if even **one** promise rejects, `Promise.all()` immediately rejects with that error — it doesn't wait for the others. Use this when you need everything to succeed, and any single failure should be treated as a total failure.
- `Promise.allSettled()`: waits for **every** promise to finish, no matter whether each one succeeds or fails, and resolves with an array describing each outcome (`{ status: 'fulfilled', value }` or `{ status: 'rejected', reason }`). It never rejects itself. Use this when you want to know the outcome of every task, even if some fail.
- `Promise.race()`: resolves or rejects as soon as the **first** promise in the list settles (whichever finishes first, success or failure "wins"). Use this for things like timeouts — race a real request against a timer promise.
- `Promise.any()`: resolves as soon as the **first** promise fulfills (ignores rejections unless everything rejects). It only rejects if **all** of them fail. Use this when you just need any one success out of several attempts (e.g. querying several mirror servers).

```js
await Promise.all([p1, p2]);       // fails fast — rejects on the very first rejection
await Promise.allSettled([p1, p2]); // always resolves, with a status per promise, never throws
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you handle errors in `async`/`await` code? 🟡

- Wrap the `await` call in a normal `try`/`catch` block, exactly like you would for synchronous code that might throw. When a promise being `await`ed rejects, it behaves as if that line threw an error — execution jumps straight to the `catch` block.
- This is one of the biggest advantages of `async`/`await` over `.then()`/`.catch()` chains: you get to reuse the same familiar `try`/`catch` error-handling pattern used everywhere else in JS, instead of a separate `.catch()` handler tacked onto a chain.
- Note that `fetch()` itself only rejects on network failures — a `404` or `500` response is still considered a "successful" fetch as far as the Promise is concerned, so you typically need to manually check `res.ok` and `throw` yourself, as shown below.

```js
async function getUser() {
  try {
    const res = await fetch('/api/user');
    if (!res.ok) throw new Error('Request failed'); // fetch() doesn't reject on 404/500 by itself
    return await res.json();
  } catch (err) {
    // Catches both network failures and the manually thrown error above
    console.error('Failed to fetch user:', err);
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `async`/`await` and using `.then()`/`.catch()` directly? 🟡

- They're functionally equivalent under the hood — `async`/`await` doesn't replace Promises, it's just a different, more readable **syntax** for working with the same Promise mechanism ("syntactic sugar").
- With `.then()`/`.catch()`, each async step is a new function passed to `.then()`, and multiple dependent steps end up chained together, which can get visually noisy, especially once error handling and conditionals are added.
- With `async`/`await`, the same sequence of steps can be written as plain top-to-bottom statements, and errors are handled with an ordinary `try`/`catch` — which most people find easier to read and reason about, especially when one async step depends on the result of the previous one.

> [!IMPORTANT]
> **Follow-up questions:**
> - When might `.then()` chaining still be preferable over `async`/`await`?
> - How would you run two independent `await` calls concurrently instead of sequentially?

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between `setTimeout`, `setImmediate`, and `process.nextTick()`? 🔴

All three schedule a function to run later, but with very different timing and priority:

- `process.nextTick()` (Node.js only): the highest priority of the three. Its queue is drained completely **before** the event loop is allowed to move on to the next phase — even before Promise microtasks get a turn, in most Node versions. Use sparingly, since overusing it can starve the event loop of other work.
- `setTimeout(fn, 0)`: schedules `fn` to run in the **timers** phase of the event loop, after at least the given delay has elapsed (`0` just means "as soon as possible," not "instantly"). Browsers enforce a minimum delay of about 4ms once you're several levels deep in nested timers, so it's never truly zero.
- `setImmediate()` (Node.js only): schedules `fn` to run in the **check** phase, which generally happens after I/O callbacks complete in the current pass of the loop. In practice, `setImmediate()` often — but not always — runs before a `setTimeout(fn, 0)` scheduled in the same tick, depending on the surrounding context (e.g. whether you're inside an I/O callback).
- Rough priority order in Node.js: `process.nextTick()` > Promise microtasks > `setImmediate()`/`setTimeout(fn, 0)` (whose relative order can vary).

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement a basic Promise from scratch? 🔴

- A minimal Promise needs three things: a **state** (`pending`, `fulfilled`, or `rejected`) that can only change once, a stored **value/reason** once settled, and a list of **callbacks** to run once settlement happens (needed because `.then()` might be called before the promise has settled yet).
- The constructor immediately runs the `executor` function you pass in, giving it `resolve`/`reject` functions that transition the internal state exactly once (further calls are ignored, since a settled promise can't change — see Q3).
- `.then()` needs to handle two cases: if the promise has **already settled**, run the appropriate callback right away; if it's **still pending**, save the callback to run later once `#settle()` is eventually called.

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

- **Serial** (one at a time, in order): `await` each task inside a normal loop. Because `await` pauses the function until the current task finishes, the next task doesn't start until the previous one is done — useful when each task depends on the previous one, or you must limit load to exactly one at a time.
- **Parallel** (all at once): start every task immediately (without awaiting each one individually) and use `Promise.all()` to wait for them all to finish together. This is fastest when tasks are independent of each other, but can overwhelm a server or API if there are many tasks.
- **Limited concurrency** (a compromise): allow up to `limit` tasks to run at the same time, and start a new one only as an existing one finishes. This avoids overloading a resource while still getting most of the speed benefit of parallelism.

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

- Normally, if a promise rejects and nothing is listening for that rejection (no `.catch()`, no surrounding `try`/`catch` around an `await`), the error doesn't just vanish silently — the JS runtime notices and fires a special event: `unhandledrejection` in browsers, or `unhandledRejection` in Node.js.
- Unlike a synchronous thrown error, this doesn't immediately crash the program at the point of the error. However, in modern Node.js, if nothing handles an `unhandledRejection` event, Node will log it and by default **terminate the process** — treating it similarly to an uncaught exception, since silently swallowed async errors are considered a serious bug risk.
- Best practice: always attach a `.catch()` to any promise chain you're not otherwise awaiting/returning, or wrap `await` calls in `try`/`catch`, so failures are handled deliberately instead of leaking as unhandled rejections.

[↑ Back to top](#table-of-contents)

---
