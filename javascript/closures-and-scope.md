# Closures & Scope

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a closure?](#1-what-is-a-closure)
- [2. What is lexical scope?](#2-what-is-lexical-scope)
- [3. What are the types of scope in JavaScript?](#3-what-are-the-types-of-scope-in-javascript)
- [4. What's the difference between scope and a closure?](#4-whats-the-difference-between-scope-and-a-closure)
- [5. What is variable shadowing?](#5-what-is-variable-shadowing)

**🟡 Medium**
- [6. What's the classic closure-in-a-loop bug, and how do you fix it?](#6-whats-the-classic-closure-in-a-loop-bug-and-how-do-you-fix-it)
- [7. How would you use a closure to create a private counter (module pattern)?](#7-how-would-you-use-a-closure-to-create-a-private-counter-module-pattern)
- [8. Can closures cause memory leaks? How?](#8-can-closures-cause-memory-leaks-how)
- [9. Why do closures still have access to outer variables after the outer function has returned?](#9-why-do-closures-still-have-access-to-outer-variables-after-the-outer-function-has-returned)

**🔴 Hard**
- [10. How would you implement a `memoize` function using closures?](#10-how-would-you-implement-a-memoize-function-using-closures)
- [11. Does a closure capture a variable's value or a reference to it?](#11-does-a-closure-capture-a-variables-value-or-a-reference-to-it)
- [12. How do closures work under the hood?](#12-how-do-closures-work-under-the-hood)

---

### 1. What is a closure? 🟢

- A function that "remembers" the variables from its outer (lexical) scope, even after the outer function has finished executing.
- Created **every time** a function is defined — it keeps a live link to its surrounding scope, not a snapshot copy.

```js
function outer() {
  let count = 0;
  return function inner() {
    return ++count;
  };
}
const counter = outer();
counter(); // 1
counter(); // 2 — `count` persisted between calls
```

> [!TIP]
> **Real-life example:** a "like" button component where each instance keeps its own private `count`, without using a global variable.

[↑ Back to top](#table-of-contents)

---

### 2. What is lexical scope? 🟢

- Scope determined by **where a variable is written** in the source code, not by where/how a function is later called.
- Inner functions can access variables from enclosing functions, but outer functions can't reach into inner ones.

```js
function outer() {
  const a = 1;
  function inner() {
    console.log(a); // can read `a` — lexically inside `outer`
  }
  inner();
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What are the types of scope in JavaScript? 🟢

- **Global scope**: declared outside any function/block, accessible everywhere.
- **Function scope**: `var` is scoped to the nearest enclosing function.
- **Block scope**: `let`/`const` are scoped to the nearest enclosing `{}` (if, for, while, or a bare block).

```js
if (true) {
  var x = 1;   // function/global-scoped — leaks outside the block
  let y = 2;   // block-scoped — only exists inside this `if`
}
console.log(x); // 1
console.log(y); // ReferenceError
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between scope and a closure? 🟢

- **Scope** is the set of rules that determines where a variable is accessible — a static concept defined by code structure.
- A **closure** is the actual mechanism/object that lets a function retain access to its outer scope's variables after that scope has technically exited.
- Put simply: scope is the *rulebook*, closure is a function *using* that rulebook to keep variables alive.

[↑ Back to top](#table-of-contents)

---

### 5. What is variable shadowing? 🟢

- When a variable declared in an inner scope has the **same name** as one in an outer scope — the inner one "shadows" (hides) the outer one within that scope.

```js
let value = 'outer';
function test() {
  let value = 'inner'; // shadows the outer `value`
  console.log(value);  // "inner"
}
test();
console.log(value); // "outer" — unaffected
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What is "illegal shadowing," and when does it throw an error?

[↑ Back to top](#table-of-contents)

---

### 6. What's the classic closure-in-a-loop bug, and how do you fix it? 🟡

- With `var` (function-scoped), all loop iterations share the **same** variable. By the time an async callback (like `setTimeout`) runs, the loop has already finished and the variable holds its final value.
- Fix 1: use `let` instead of `var` — creates a **new binding per iteration** (block-scoped).
- Fix 2 (pre-ES6): wrap the body in an IIFE to capture the current value in a new scope each iteration.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// logs 3, 3, 3 — all callbacks share the same `i`

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// logs 0, 1, 2 — each iteration gets its own `j`
```

[↑ Back to top](#table-of-contents)

---

### 7. How would you use a closure to create a private counter (module pattern)? 🟡

- Return an object of methods from an outer function — the methods close over shared private variables that are inaccessible from outside.

```js
function createCounter() {
  let count = 0; // private — not accessible directly
  return {
    increment: () => ++count,
    decrement: () => --count,
    value: () => count,
  };
}
const counter = createCounter();
counter.increment();
counter.increment();
counter.value(); // 2
counter.count;   // undefined — truly private
```

> [!TIP]
> **Real-life example:** this is exactly how the pre-ES6 "module pattern" simulated private fields, before real private class fields (`#field`) existed.

[↑ Back to top](#table-of-contents)

---

### 8. Can closures cause memory leaks? How? 🟡

- A closure keeps its entire outer scope alive in memory for as long as the closure itself is reachable — even variables the closure doesn't actually use.
- If a long-lived closure (e.g. an event listener never removed) holds a reference to a large object (like a big DOM tree or dataset), that object can't be garbage collected.

```js
function attachHandler(largeData) {
  document.getElementById('btn').addEventListener('click', () => {
    console.log(largeData.length); // keeps largeData alive as long as the listener exists
  });
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is it important to call `removeEventListener` when a component is destroyed?
> - How would you avoid unintentionally capturing large objects in a closure?

[↑ Back to top](#table-of-contents)

---

### 9. Why do closures still have access to outer variables after the outer function has returned? 🟡

- Normally a function's local variables are destroyed once it returns. But if an inner function references them and is itself returned/stored somewhere, the JS engine keeps that outer scope alive in memory (instead of garbage-collecting it) for as long as the inner function exists.
- This is why closures "remember" — the variable isn't re-created, it's the *same* one, kept alive by the reference.

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement a `memoize` function using closures? 🔴

- Use a closure to hold a cache object that persists between calls, keyed by the function's arguments.

```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

const slowSquare = (n) => { for (let i = 0; i < 1e6; i++); return n * n; };
const fastSquare = memoize(slowSquare);
fastSquare(5); // computed
fastSquare(5); // returned instantly from cache
```

[↑ Back to top](#table-of-contents)

---

### 11. Does a closure capture a variable's value or a reference to it? 🔴

- A **reference** (a live binding), not a snapshot value — this is why the loop bug in Q6 happens, and why a closure sees later mutations to a variable.

```js
function makeLogger() {
  let msg = 'first';
  const log = () => console.log(msg);
  msg = 'second'; // mutated before `log` is ever called
  return log;
}
makeLogger()(); // "second" — not "first"
```

[↑ Back to top](#table-of-contents)

---

### 12. How do closures work under the hood? 🔴

- Each function call creates an **execution context** with its own **lexical environment** (a record of local variables + a reference to the parent environment).
- Normally this environment is discarded after the function returns. But if an inner function is returned or stored elsewhere, the engine keeps that environment alive as long as something still references the inner function — forming a chain back to all enclosing scopes it needs.
- This is why closures aren't "magic" — they're just the natural consequence of how scope chains and garbage collection (reachability) work together.

[↑ Back to top](#table-of-contents)

---
