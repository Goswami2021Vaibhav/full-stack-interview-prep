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

- A **high-level** language — you don't manage memory or talk to hardware directly, the engine handles that for you.
- **Dynamically-typed** — you don't declare a variable's type up front; it's decided at runtime based on whatever value is currently stored in it, and can change later.
- Originally created to make static web pages interactive (form validation, animations, responding to clicks), but its use has grown far beyond that.
- Today it runs almost everywhere: in the browser (manipulating the page via the DOM), on servers (Node.js), in mobile apps (React Native), on desktops (Electron), and even on embedded devices.
- It's **multi-paradigm** — you can write it in a procedural style (step-by-step instructions), object-oriented style (using prototypes instead of classical classes under the hood), or functional style (passing functions around as values).

[↑ Back to top](#table-of-contents)

---

### 2. Is JavaScript compiled or interpreted? 🟢

- Technically **both**. It's not "read and executed one line at a time" the way people often imagine — that mental model is outdated.
- Modern engines (like V8, which powers Chrome and Node.js) use **JIT (Just-In-Time) compilation**:
  1. Your source code is **parsed** into a tree-like structure (an AST — Abstract Syntax Tree) that represents its grammar.
  2. That tree is turned into **bytecode**, a lower-level set of instructions an interpreter can run quickly.
  3. While the code runs, the engine watches for "hot" code — functions or loops called over and over — and **compiles those specific parts into optimized machine code** so they run much faster the next time.
- So there's an interpreter step (fast startup, no long upfront compile wait) *and* a compiler step (speed for repeated code) working together, rather than one or the other.

[↑ Back to top](#table-of-contents)

---

### 3. What does it mean that JavaScript is single-threaded? 🟢

- A "thread" is a sequence of instructions a computer executes. Being single-threaded means JavaScript has only **one call stack** (think of it as one "worker" and one to-do list) — it can run only one piece of your JS code at any given instant. Two of your functions never truly run at the exact same time.
- This does **not** mean JS can't handle things like timers, network requests, or file reads without freezing the page. Those asynchronous tasks are handed off to the browser (or Node) to manage in the background, and the **event loop** is the mechanism that brings their results back to your single thread's call stack once it's free, one at a time.
- Under the hood, the browser/Node runtime does use extra threads (e.g. for downloading a file over the network, or reading from disk) — but none of *your JavaScript code* ever executes on more than one thread simultaneously. Only the coordination happens in the background; your logic still runs one step at a time.
- Analogy: imagine a single cashier (the call stack) at a store. They can only serve one customer at a time. But when a customer needs something looked up in the back room (an async task), the cashier hands that off to a stockroom worker (the browser/Node APIs) and moves on to the next customer. When the stockroom worker comes back with the item, they join the back of the line (the queue) and get served when the cashier is free — they don't interrupt whoever's currently being served.

> [!IMPORTANT]
> **Follow-up questions:**
> - If JS is single-threaded, how does `setTimeout` or `fetch` not block the main thread?
> - What is Web Workers, and how does it relate to this?

[↑ Back to top](#table-of-contents)

---

### 4. What is the difference between `==` and `===`? 🟢

- `===` (**strict equality**): compares the value **and** the type. Both sides must already be the same type, or it returns `false` — no conversion happens behind the scenes. This is predictable and the recommended default.
- `==` (**loose equality**): if the two operands have different types, JavaScript first tries to **coerce** (automatically convert) one or both to a common type before comparing. This "type coercion" is a common source of confusing bugs because the conversion rules aren't always obvious.
- Rule of thumb for interviews and real code: always prefer `===`/`!==` unless you have a specific, well-understood reason to use `==` (like checking for both `null` and `undefined` at once).

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

- `undefined` is a real value that JavaScript itself assigns to a variable that has been **declared** but hasn't been given a value yet. The variable exists in memory — it just has no meaningful content assigned to it.
- "Not defined" is what you get (as a `ReferenceError`) when you try to use a variable name that was **never declared** anywhere the engine can see — in the current scope or any outer scope it checks. The engine has no record of that name at all.
- In short: `undefined` means "I know this name, but it has no value yet." A `ReferenceError` means "I don't know this name at all."

```js
let a;
console.log(a); // undefined

console.log(b); // ReferenceError: b is not defined
```

[↑ Back to top](#table-of-contents)

---

### 6. What is hoisting? 🟡

- Before running your code, the JS engine scans the current scope (function or block) and registers all the variable/function **declarations** it finds — as if they were "moved" to the top. This is called **hoisting**. Only the declaration (the name) is hoisted, not the assignment (the value) — the value is still set only when execution actually reaches that line.
- The three ways of declaring a variable behave differently once hoisted:
  - `function` **declarations** (`function greet() {}`) are hoisted completely, including their body — so you can call the function *before* the line it's written on.
  - `var` is hoisted and automatically initialized to `undefined`, so reading it before its assignment line doesn't error, it just gives you `undefined`.
  - `let` and `const` are hoisted too (the engine knows the name exists), but they are **not initialized** — they stay in a "locked" state called the Temporal Dead Zone until their declaration line actually runs, so accessing them early throws an error (see next question).
- Why it exists: it's a side effect of how the engine sets up scope in two passes (first registering names, then executing code) rather than a feature you should rely on deliberately — in practice, always declare variables before using them for clarity.

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

- The **Temporal Dead Zone** is the stretch of code between the start of a scope (e.g. the top of a function or block) and the line where a `let`/`const` variable is actually declared. During that window, the variable technically exists (the engine already knows its name, due to hoisting) but it is **uninitialized** — you cannot read or write to it.
- "Temporal" just means "related to time" here — it's about *when* in the execution order you're standing, not *where* in the code.
- Trying to access the variable while inside this zone throws a `ReferenceError`, which is actually a helpful safety net: it stops you from accidentally using a variable before it's properly set up, unlike `var`, which would silently hand you `undefined` and let a bug slip through quietly.

```js
{
  console.log(x); // ReferenceError (TDZ)
  let x = 10;
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the scope chain? 🟡

- A "scope" is simply the region of code where a variable is accessible. Every function or block creates its own scope, and scopes can be nested inside each other, like Russian nesting dolls.
- When your code references a variable, the engine first checks the **current** (innermost) scope. If it isn't found there, it steps **outward** to the scope that contains it, then the one containing that, and so on — all the way out to the **global scope** if necessary. This ordered path of "where to look next" is the **scope chain**.
- This is called **lexical scoping** because the chain is determined by *where functions are physically written in the code*, not by which function called which at runtime.
- If the variable isn't found anywhere along the entire chain, including the global scope, you get a `ReferenceError`.
- This outward-lookup mechanism is exactly what makes **closures** possible: an inner function keeps access to its outer function's variables (via the scope chain) even after the outer function has finished running.

> [!TIP]
> **Real-life example:** a `config` value read inside a deeply nested callback without being passed explicitly — it's resolved by walking up the scope chain to where it was originally declared.

[↑ Back to top](#table-of-contents)

---

### 9. How does the `this` keyword work? 🟡

- `this` is a special keyword that refers to "the object the current code is running in the context of." The key thing that confuses people: `this` is **not** decided by *where* a regular function is written — it's decided by *how* the function is **called** (with one exception: arrow functions, see below).
- The same function can have a different `this` each time it's called, depending on the call site:
  - **Plain function call** (`greet()`) → `this` is `undefined` in strict mode (an error if you try to use a property on it), or the global object (`window` in browsers) in non-strict/legacy code.
  - **Method call** (`obj.method()`) → `this` is whatever is to the left of the dot — `obj` in this case.
  - **Arrow function** → doesn't get its own `this` at all. Instead, it "inherits" `this` from whichever scope it was written inside (its enclosing lexical scope) — the same rule used for looking up any other variable.
  - **`call`/`apply`/`bind`** → you explicitly tell the function what `this` should be, overriding the normal rules.
  - **`new Constructor()`** → `this` is a brand-new, empty object that the constructor then sets properties on.

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

- Strict mode is an opt-in, stricter variant of JavaScript, turned on by adding the literal string `"use strict";` at the very top of a file or function. It changes how certain mistakes are handled — from being silently ignored (or causing confusing behavior) to throwing an actual error, so bugs surface immediately instead of hiding.
- Concretely, it catches things like:
  - Assigning to a variable you never declared (e.g. `x = 5;` with no `let`/`const`/`var`) throws instead of silently creating a global variable.
  - Duplicate parameter names in a function (`function f(a, a) {}`) become a syntax error instead of being silently allowed.
  - `this` in a plain function call is `undefined` instead of quietly defaulting to the global object, which helps catch accidental global-object usage.
- You usually don't need to add it manually anymore: **ES6 modules and classes are automatically in strict mode**, so most modern code gets these protections for free.

[↑ Back to top](#table-of-contents)

---

### 11. What is an IIFE and why is it useful? 🟡

- **IIFE** stands for **I**mmediately **I**nvoked **F**unction **E**xpression — you define a function and call it in the very same statement, so it runs right away rather than being saved for later use.
- The classic reason to use one: JavaScript variables declared with `var` don't have block scope, only function scope — so wrapping code in a function creates a private, throwaway scope. Anything declared inside the IIFE stays inside it and can't leak out or collide with other scripts' global variables.
- This was especially important before ES6 introduced real modules (with `import`/`export`) and block-scoped `let`/`const` — the IIFE was the standard way to fake a "private module" in plain JavaScript.

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

- A **JavaScript engine** is the program that actually reads and executes your JS code. V8 is Google's engine, used in Chrome and Node.js (other engines exist too, like SpiderMonkey in Firefox).
- V8's pipeline, step by step:
  1. Your source code is parsed into an **AST** (Abstract Syntax Tree — a structured tree representing the code's grammar, e.g. "this is a function call with these arguments").
  2. An interpreter called **Ignition** turns the AST into **bytecode** (simple, generic instructions) and starts executing it immediately — this gets your code running fast without waiting for heavy optimization up front.
  3. While running, V8 **profiles** the code — it watches which functions get called a lot ("hot" functions/loops) and what shapes of data they tend to receive.
  4. Those hot functions get handed over to **TurboFan**, a JIT (Just-In-Time) compiler, which turns them into highly optimized machine code — but only under the *assumption* that the argument types and object shapes it observed will keep being the same.
- **Hidden classes**: normally, looking up a property on an object is like a dictionary lookup (search by key name, which is relatively slow). V8 instead creates internal "hidden classes" — shared blueprints — for objects that have the same set of properties added in the same order, so property access becomes more like accessing a fixed slot in a struct (a fast, predictable memory offset) instead of searching a dictionary.
- If code later violates those assumptions — e.g. an object's shape changes unexpectedly, or a function that used to always receive numbers suddenly receives a string — V8 **deoptimizes**: it throws away the optimized machine code and falls back to the slower, generic bytecode path until it can safely re-optimize.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does adding properties to an object in a different order than other instances hurt performance?
> - What's the difference between Ignition and TurboFan in V8?

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between the call stack, heap, and event loop? 🔴

- **Call stack**: keeps track of which functions are currently running. Every time a function is called, a new "frame" is pushed onto the stack; when that function returns, its frame is popped off. It's **LIFO** (Last In, First Out) — the most recently called function finishes first, just like a stack of plates where you can only take from the top.
- **Heap**: a large, mostly unstructured region of memory where objects, arrays, and closures actually live. The call stack holds small, fixed-size values and *references* (pointers) to data in the heap, rather than the actual object contents.
- **Event loop**: the mechanism enabling async behavior on a single thread. It continuously checks: "Is the call stack empty?" Once it is, it takes the next waiting callback from a queue (there are separate queues — the **microtask queue** for promises, and the **macrotask/task queue** for things like `setTimeout`) and pushes it onto the call stack to run. Microtasks are always fully drained before the next macrotask runs.
- Putting it together: the call stack executes code, the heap stores the data that code works with, and the event loop is the traffic controller that feeds waiting async callbacks back onto the call stack once it's free.

> [!TIP]
> **Real-life example:** a `fetch()` call doesn't block the stack — it's handed off, and its `.then()` callback waits in the microtask queue until the stack is empty, then runs.

[↑ Back to top](#table-of-contents)

---

### 14. How does dynamic typing affect JavaScript's performance compared to statically typed languages? 🔴

- In a statically-typed, compiled language (like Java or C++), the compiler knows every variable's exact type before the program even runs, so it can generate highly efficient machine code upfront, with no need to double-check types while running.
- JavaScript is **dynamically typed** — a variable's type is only known at runtime, and it can even change from one type to another during execution. This means the engine can't fully optimize code ahead of time; it has to add runtime checks (e.g. "is this actually still a number right now?") that a statically-typed language wouldn't need.
- V8 compensates for this with two main tricks:
  - **Hidden classes** (covered in Q12): grouping objects with the same shape so property lookups are fast.
  - **Inline caching**: at a specific place in the code where a property is accessed (a "call site"), V8 remembers the result/shape from the last time it ran that line, and reuses that shortcut next time — as long as the shape hasn't changed. This approximates the speed of static typing for code that behaves consistently.
- Performance degrades when a single call site sees many different object shapes or argument types over time (called **megamorphic** code) — the inline cache can't settle on one shortcut, so the engine falls back to slower, more general lookups.

[↑ Back to top](#table-of-contents)

---
