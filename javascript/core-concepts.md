# Core Concepts

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is JavaScript?](#1-what-is-javascript)
- [2. Is JavaScript compiled or interpreted?](#2-is-javascript-compiled-or-interpreted)
- [3. What does it mean that JavaScript is single-threaded?](#3-what-does-it-mean-that-javascript-is-single-threaded)
- [4. What is the difference between `==` and `===`?](#4-what-is-the-difference-between--and-)
- [5. What's the difference between `undefined` and "not defined"?](#5-whats-the-difference-between-undefined-and-not-defined)

**🟡 Medium**
- [6. What is hoisting?](#6-what-is-hoisting)
- [7. What is the Temporal Dead Zone (TDZ)?](#7-what-is-the-temporal-dead-zone-tdz)
- [8. What is the scope chain?](#8-what-is-the-scope-chain)
- [9. How does the `this` keyword work?](#9-how-does-the-this-keyword-work)
- [10. What is strict mode and why use it?](#10-what-is-strict-mode-and-why-use-it)
- [11. What is an IIFE and why is it useful?](#11-what-is-an-iife-and-why-is-it-useful)

**🔴 Hard**
- [12. What is the JavaScript engine, and how does V8 optimize execution?](#12-what-is-the-javascript-engine-and-how-does-v8-optimize-execution)
- [13. What's the difference between the call stack, heap, and event loop?](#13-whats-the-difference-between-the-call-stack-heap-and-event-loop)
- [14. How does dynamic typing affect JavaScript's performance compared to statically typed languages?](#14-how-does-dynamic-typing-affect-javascripts-performance-compared-to-statically-typed-languages)

---

### 1. What is JavaScript? 🟢

- A high-level, dynamically-typed scripting language, originally built to make web pages interactive.
- Today it runs everywhere — browsers (DOM manipulation), servers (Node.js), mobile (React Native), and even embedded devices.
- Multi-paradigm: supports procedural, object-oriented (via prototypes), and functional styles.

[↑ Back to top](#table-of-contents)

---

### 2. Is JavaScript compiled or interpreted? 🟢

- Technically **both** — modern engines (like V8) use **JIT (Just-In-Time) compilation**: code is parsed, converted to bytecode, and hot paths are compiled to optimized machine code at runtime.
- It's not "interpreted line-by-line" the way people often assume — that mental model is outdated.

[↑ Back to top](#table-of-contents)

---

### 3. What does it mean that JavaScript is single-threaded? 🟢

- JS has **one call stack** — it can only execute one piece of code at a time, no true parallel execution of JS code.
- Asynchronous behavior (timers, network calls, promises) is handled by the **event loop**, not by extra threads running your JS.
- The browser/Node still uses background threads internally (e.g. for I/O), but your JS code itself never runs concurrently.

> [!IMPORTANT]
> **Follow-up questions:**
> - If JS is single-threaded, how does `setTimeout` or `fetch` not block the main thread?
> - What is Web Workers, and how does it relate to this?

[↑ Back to top](#table-of-contents)

---

### 4. What is the difference between `==` and `===`? 🟢

- `===` (strict equality): compares value **and** type, no conversion.
- `==` (loose equality): converts operands to the same type before comparing, which can produce surprising results.

```js
0 == '0';   // true  (string coerced to number)
0 === '0';  // false (different types)
null == undefined;  // true
null === undefined; // false
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does `NaN === NaN` evaluate to `false`?
> - What are the coercion rules `==` follows internally?

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `undefined` and "not defined"? 🟢

- `undefined`: the variable **exists** (declared) but has no assigned value yet.
- "Not defined" / `ReferenceError`: the variable was **never declared** in any accessible scope.

```js
let a;
console.log(a); // undefined

console.log(b); // ReferenceError: b is not defined
```

[↑ Back to top](#table-of-contents)

---

### 6. What is hoisting? 🟡

- JavaScript moves **declarations** (not initializations) to the top of their scope during the compile phase, before code runs.
- `function` declarations are hoisted fully (usable before their line). `var` is hoisted but initialized as `undefined`. `let`/`const` are hoisted but stay uninitialized (see TDZ below).

```js
console.log(x); // undefined (not an error)
var x = 5;

greet(); // works, fully hoisted
function greet() { console.log('hi'); }
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does accessing a `let` variable before its declaration throw, while `var` just gives `undefined`?
> - Are function *expressions* hoisted the same way as function *declarations*?

[↑ Back to top](#table-of-contents)

---

### 7. What is the Temporal Dead Zone (TDZ)? 🟡

- The time between entering a scope and the actual `let`/`const` declaration line, during which the variable exists but **cannot be accessed**.
- Accessing it in that window throws a `ReferenceError`, instead of silently returning `undefined` like `var` would.

```js
{
  console.log(x); // ReferenceError (TDZ)
  let x = 10;
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the scope chain? 🟡

- When a variable is referenced, the engine looks for it in the current scope; if not found, it walks **outward** through each enclosing (lexical) scope until it reaches the global scope.
- If it's not found anywhere, you get a `ReferenceError`.
- This outward lookup chain is what makes closures possible.

> [!TIP]
> **Real-life example:** a `config` value read inside a deeply nested callback without being passed explicitly — it's resolved by walking up the scope chain to where it was originally declared.

[↑ Back to top](#table-of-contents)

---

### 9. How does the `this` keyword work? 🟡

- `this` is **not** fixed by where a function is defined — it's determined by **how the function is called** (except for arrow functions).
- Common bindings:
  - Plain function call → `this` is `undefined` (strict mode) or the global object (non-strict).
  - Method call (`obj.method()`) → `this` is `obj`.
  - Arrow function → inherits `this` from the enclosing lexical scope (doesn't have its own).
  - `call`/`apply`/`bind` → `this` is explicitly set.
  - `new Constructor()` → `this` is the newly created instance.

```js
const obj = {
  name: 'Vaibhav',
  greet() { console.log(this.name); },
  greetArrow: () => console.log(this.name),
};
obj.greet();      // "Vaibhav"
obj.greetArrow(); // undefined (this = outer/lexical scope, not obj)
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What does `this` refer to inside a regular function passed as a callback (e.g. to `setTimeout`)?
> - How do `call`, `apply`, and `bind` differ from each other?

[↑ Back to top](#table-of-contents)

---

### 10. What is strict mode and why use it? 🟡

- Enabled via `"use strict"` at the top of a file/function — makes JS parse and run in a stricter variant.
- Catches common mistakes: throws on assigning to undeclared variables, disallows duplicate parameter names, makes `this` `undefined` in plain function calls instead of defaulting to the global object.
- ES6 modules and classes are strict mode by default.

[↑ Back to top](#table-of-contents)

---

### 11. What is an IIFE and why is it useful? 🟡

- **I**mmediately **I**nvoked **F**unction **E**xpression — a function defined and executed right away.
- Classic use: creating a private scope so variables don't leak into the global namespace (pre-ES6 module pattern).

```js
(function () {
  const privateVar = 'hidden';
  console.log(privateVar);
})();
// privateVar is not accessible outside
```

> [!TIP]
> **Real-life example:** older libraries (like early jQuery plugins) wrapped their entire code in an IIFE to avoid polluting `window` with global variables.

[↑ Back to top](#table-of-contents)

---

### 12. What is the JavaScript engine, and how does V8 optimize execution? 🔴

- The JS engine (e.g. V8 in Chrome/Node) parses JS into an AST, generates bytecode via an interpreter (Ignition), and profiles the running code.
- "Hot" functions get recompiled into optimized machine code by a JIT compiler (TurboFan), using assumptions about argument/object shapes.
- **Hidden classes**: V8 creates internal "shapes" for objects with the same structure, so property access can be optimized like a struct lookup instead of a dictionary lookup.
- If those assumptions break (e.g. an object's shape changes unexpectedly), V8 "deoptimizes" back to slower code.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does adding properties to an object in a different order than other instances hurt performance?
> - What's the difference between Ignition and TurboFan in V8?

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between the call stack, heap, and event loop? 🔴

- **Call stack**: tracks function calls currently executing — LIFO; one frame per active function.
- **Heap**: where objects, arrays, and closures are actually allocated in memory.
- **Event loop**: the mechanism that, once the call stack is empty, pulls the next callback from the task/microtask queues and pushes it onto the call stack — this is what enables async behavior on a single thread.

> [!TIP]
> **Real-life example:** a `fetch()` call doesn't block the stack — it's handed off, and its `.then()` callback waits in the microtask queue until the stack is empty, then runs.

[↑ Back to top](#table-of-contents)

---

### 14. How does dynamic typing affect JavaScript's performance compared to statically typed languages? 🔴

- Since variable types can change at runtime, the engine can't always predict a value's shape ahead of time — it must do extra runtime checks and can't apply all the optimizations a statically-typed/compiled language (like Java or C++) could apply upfront.
- V8 compensates using hidden classes and inline caching (caching the result of a property lookup at a call site, assuming the shape stays the same) to approximate static-typing-like speed for consistent code patterns.
- Performance degrades when object shapes or argument types vary unpredictably at the same call site (megamorphic code).

[↑ Back to top](#table-of-contents)

---
