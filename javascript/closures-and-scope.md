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

- A closure is what you get when a function "remembers" and keeps access to the variables from the scope it was originally defined in — even after that outer scope has technically finished running and would normally be gone.
- Closures are created **every single time** a function is defined, not just in special cases. A function always keeps a live link back to its surrounding (lexical) scope — see Q2 for what "lexical" means — not a one-time snapshot/copy of the values at that moment. This is why a closure can see later changes to those variables too (see Q11).
- In the example below, `inner` is defined inside `outer`, so it forms a closure over `outer`'s variables. Normally, `count` would be destroyed once `outer()` finishes running — but because `inner` still references it and gets returned out, the JS engine keeps `count` alive in memory instead of discarding it.

```js
function outer() {
  let count = 0;
  return function inner() {
    return ++count; // `inner` "closes over" `count`, keeping it alive
  };
}
const counter = outer(); // outer() has already finished running here
counter(); // 1
counter(); // 2 — `count` persisted between calls, it wasn't reset or recreated
```

> [!TIP]
> **Real-life example:** a "like" button component where each instance keeps its own private `count`, without using a global variable.

[↑ Back to top](#table-of-contents)

---

### 2. What is lexical scope? 🟢

- "Lexical" just means "related to where the code is physically written." Lexical scope means that which variables a piece of code can access is decided by **where that code is positioned in the source file**, fixed at the time the code is written — not by where or how a function later gets called at runtime.
- Practically: a function can always see variables from the function(s) it's nested inside, just by looking at how the code is structured/indented. This nesting relationship is sometimes called the "scope chain" — a chain of enclosing scopes a variable lookup walks outward through until it finds a match.
- Access only flows **inward-out**: an inner function can read variables from an outer function that contains it, but the outer function can never reach into a variable declared only inside the inner function.

```js
function outer() {
  const a = 1;
  function inner() {
    console.log(a); // can read `a` because `inner` is lexically (physically) nested inside `outer`
  }
  inner();
  // console.log(b) here would fail — outer can't see inner's variables
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What are the types of scope in JavaScript? 🟢

- **Global scope**: anything declared outside of any function or block. It's accessible from literally anywhere else in the program, which also makes it easy to accidentally cause naming collisions in larger codebases.
- **Function scope**: variables declared with `var` belong to the nearest enclosing **function**, not to whatever `{}` block they happen to be written inside. This means a `var` declared inside an `if` block or a `for` loop is actually still visible outside that block, as long as it's within the same function.
- **Block scope**: variables declared with `let`/`const` belong only to the nearest enclosing pair of curly braces `{}` — whether that's an `if`, a `for`/`while` loop, or just a bare `{ }` block on its own. Once execution leaves that block, the variable is gone.
- In practice, this is the main reason modern code prefers `let`/`const` over `var` — block scoping is more predictable and avoids variables unintentionally "leaking" outside the block they were meant for.

```js
if (true) {
  var x = 1;   // function/global-scoped — leaks outside the `if` block
  let y = 2;   // block-scoped — only exists inside this `if` block
}
console.log(x); // 1 — still accessible, var ignored the block boundary
console.log(y); // ReferenceError — y doesn't exist outside its block
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between scope and a closure? 🟢

- **Scope** is a set of rules, fixed by how the code is structured/written, that determines *where in the code* a given variable can be seen and used. It's a static concept — you can figure out a variable's scope just by reading the source code, without running it.
- A **closure** is what happens at runtime when a function actually uses those scope rules to keep a link to its outer variables alive, even after the code that originally created them has finished running.
- A simple way to separate the two: scope is the *rulebook* that says which variables a function is allowed to see. A closure is what happens when a specific function *keeps using* that rulebook to hang onto variables long after you'd otherwise expect them to be gone.

[↑ Back to top](#table-of-contents)

---

### 5. What is variable shadowing? 🟢

- Shadowing happens when you declare a variable in an inner scope using the **exact same name** as a variable that already exists in an outer scope. Inside that inner scope, referring to the name now refers to the new, inner variable — it temporarily "shadows" (hides/blocks access to) the outer one, as if the outer variable didn't exist there.
- Importantly, shadowing doesn't overwrite or destroy the outer variable — it's still perfectly intact once you leave the inner scope; it was just inaccessible by that name while inside it.

```js
let value = 'outer';
function test() {
  let value = 'inner'; // creates a brand-new variable that shadows the outer `value`
  console.log(value);  // "inner" — refers to the local one inside this function
}
test();
console.log(value); // "outer" — the outer variable was never touched
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What is "illegal shadowing," and when does it throw an error?

[↑ Back to top](#table-of-contents)

---

### 6. What's the classic closure-in-a-loop bug, and how do you fix it? 🟡

- The classic bug: you write a `for` loop with `var` that schedules some async callback (like `setTimeout`) on each iteration, expecting each callback to remember "its" loop value — but instead, every callback logs the **same final value**.
- Why it happens: `var` is function-scoped, not block-scoped (see Q3), so there's really only **one** `i` variable shared across the entire loop, not a fresh one per iteration. The loop runs to completion almost immediately (synchronously), updating that single shared `i` each time, and only *afterward* do the `setTimeout` callbacks actually run — by which point `i` already holds its final value (`3` in the example below, since that's the value that made the loop condition `i < 3` false and exit).
- **Fix 1 (modern, preferred)**: swap `var` for `let`. Because `let` is block-scoped, the `for` loop actually creates a **brand-new `i` binding for each iteration** behind the scenes — so each callback's closure captures its own separate copy of `i`, holding whatever value it had during that specific iteration.
- **Fix 2 (pre-ES6 style)**: wrap the loop body in an IIFE (Immediately Invoked Function Expression — a function that runs itself right away) and pass `i` in as an argument. Since each IIFE call creates its own new scope, the parameter effectively captures the current value at that point in time.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// logs 3, 3, 3 — all three callbacks share the one and only `i`, which is 3 by the time they run

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// logs 0, 1, 2 — `let` gives each iteration its own separate `j`
```

[↑ Back to top](#table-of-contents)

---

### 7. How would you use a closure to create a private counter (module pattern)? 🟡

- The technique: write an outer function that declares some local variable(s), and have it `return` an **object of methods** instead of the variable itself. Those returned methods form closures over the outer function's local variables, so they can read and update them — but code outside the function has no direct way to reach in and touch those variables, since they were never exposed.
- This gives you real "private" state without needing any special private-field syntax — the privacy comes entirely from closures and scope, which is why it's called the (closure-based) **module pattern**.

```js
function createCounter() {
  let count = 0; // private — only reachable through the methods below, never directly
  return {
    increment: () => ++count,
    decrement: () => --count,
    value: () => count,
  };
}
const counter = createCounter();
counter.increment();
counter.increment();
counter.value(); // 2 — read through the exposed method
counter.count;   // undefined — there is no direct property called `count`, truly private
```

> [!TIP]
> **Real-life example:** this is exactly how the pre-ES6 "module pattern" simulated private fields, before real private class fields (`#field`) existed.

[↑ Back to top](#table-of-contents)

---

### 8. Can closures cause memory leaks? How? 🟡

- Normally, JavaScript's garbage collector automatically frees memory used by variables once nothing can reach them anymore. But a closure keeps a live reference to its **entire** surrounding scope for as long as the closure itself still exists somewhere reachable — sometimes including variables the closure doesn't even actually use, depending on the engine.
- This becomes a memory leak specifically when a closure that captures something large (a big object, dataset, or DOM tree) is kept alive for much longer than intended — commonly because it's attached as an event listener that's never removed, or stored in some long-lived cache. As long as that closure is reachable, the large object it references can't be garbage-collected, even if you no longer actually need it.
- In the example below, as long as the click listener is attached, `largeData` stays alive in memory — even after the code that originally needed it is long done — simply because the listener's callback still references it.

```js
function attachHandler(largeData) {
  document.getElementById('btn').addEventListener('click', () => {
    console.log(largeData.length); // keeps largeData alive in memory as long as this listener exists
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

- Normally, a function's local variables live only as long as that function is running — once it `return`s, those variables would ordinarily be discarded/garbage-collected since nothing needs them anymore.
- The exception: if an **inner function** defined inside references those variables, and that inner function itself gets returned, stored in a variable, or passed elsewhere, then something still needs access to the outer scope's variables. The JS engine detects this and keeps that outer scope alive in memory, instead of destroying it, for as long as the inner function itself remains reachable.
- This is exactly why closures appear to "remember" values: the variable isn't recreated or copied anywhere — it's the exact same variable, just kept alive longer than it normally would be, because something (the inner function) still holds a live reference to it.

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement a `memoize` function using closures? 🔴

- **Memoization** means caching the result of a function call so that if you call it again with the same arguments, you get the cached answer back instantly instead of redoing the (possibly expensive) work.
- The implementation relies on a closure: `memoize` creates a `cache` (a `Map` from arguments to results) and returns a new wrapper function. That wrapper function forms a closure over `cache`, so every call to the wrapped function can check and update the same cache — the cache persists across calls because the closure keeps it alive.
- On each call: turn the arguments into a lookup key (here, via `JSON.stringify`), check if that key is already in the cache. If yes, return the cached result immediately without recomputing. If no, actually call the original function, store the result under that key, then return it.

```js
function memoize(fn) {
  const cache = new Map(); // persists across calls thanks to the closure below
  return function (...args) {
    const key = JSON.stringify(args); // turn arguments into a comparable cache key
    if (cache.has(key)) return cache.get(key); // cache hit — skip recomputation
    const result = fn(...args); // cache miss — do the real work once
    cache.set(key, result);
    return result;
  };
}

const slowSquare = (n) => { for (let i = 0; i < 1e6; i++); return n * n; };
const fastSquare = memoize(slowSquare);
fastSquare(5); // computed the slow way the first time
fastSquare(5); // returned instantly from cache the second time, same input
```

[↑ Back to top](#table-of-contents)

---

### 11. Does a closure capture a variable's value or a reference to it? 🔴

- A closure captures a **live reference** to the variable itself (sometimes called a "binding"), not a frozen snapshot of whatever value that variable happened to hold at the moment the closure was created.
- This means if the variable's value changes later — even before the closure function is ever actually called — the closure will see the **updated** value when it eventually runs, because it's reading the same variable, not a copy. This is exactly why the classic loop bug from Q6 happens: with `var`, every callback closes over the same one shared `i`, so they all see its final value rather than each remembering "its own" iteration's value.

```js
function makeLogger() {
  let msg = 'first';
  const log = () => console.log(msg); // captures a live reference to `msg`, not its current value
  msg = 'second'; // mutated before `log` is ever called
  return log;
}
makeLogger()(); // "second" — not "first", because `log` reads whatever `msg` holds when called
```

[↑ Back to top](#table-of-contents)

---

### 12. How do closures work under the hood? 🔴

- Every time a function is called, the JS engine creates an **execution context** for that call — internal bookkeeping that tracks things like local variables and where to resume afterward. Attached to this is a **lexical environment**: essentially a record mapping each local variable name to its current value, plus a hidden pointer back to the *parent* environment (the scope it was defined inside).
- Normally, once a function call finishes, its execution context and lexical environment are simply discarded — nothing else needs them, so the garbage collector reclaims that memory.
- However, if an **inner function** defined during that call gets returned out, stored in a variable, or passed somewhere else, that inner function keeps its own hidden pointer back to the lexical environment it was created in. As long as *anything* still holds a reference to that inner function, the engine cannot discard the lexical environment it depends on — so it stays alive in memory. If there are multiple levels of nesting, this pointer chain extends outward through however many parent environments the inner function actually needs, forming what's called the **scope chain**.
- The takeaway: closures aren't a special separate feature bolted onto JS — they're simply what naturally falls out of combining lexical scoping (fixed by where code is written) with garbage collection based on reachability (anything still referenced doesn't get freed).

[↑ Back to top](#table-of-contents)

---
