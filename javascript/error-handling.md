# Error Handling

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What do `try`, `catch`, and `finally` each do?](#1-what-do-try-catch-and-finally-each-do)
- [2. What is the built-in `Error` object, and what are its common subtypes?](#2-what-is-the-built-in-error-object-and-what-are-its-common-subtypes)
- [3. How do you create a custom error type?](#3-how-do-you-create-a-custom-error-type)
- [4. What's the difference between a syntax error, a runtime error, and a logical error?](#4-whats-the-difference-between-a-syntax-error-a-runtime-error-and-a-logical-error)
- [5. What does `throw` do, and what can you throw?](#5-what-does-throw-do-and-what-can-you-throw)

**🟡 Medium**
- [6. How do you handle errors in asynchronous code?](#6-how-do-you-handle-errors-in-asynchronous-code)
- [7. What is error propagation?](#7-what-is-error-propagation)
- [8. What's the difference between operational errors and programmer errors?](#8-whats-the-difference-between-operational-errors-and-programmer-errors)
- [9. How do you catch errors globally?](#9-how-do-you-catch-errors-globally)

**🔴 Hard**
- [10. How would you design a centralized error-handling strategy for an app?](#10-how-would-you-design-a-centralized-error-handling-strategy-for-an-app)
- [11. What happens to the stack trace when you re-throw an error inside a `catch` block?](#11-what-happens-to-the-stack-trace-when-you-re-throw-an-error-inside-a-catch-block)
- [12. How would you implement custom error classes with proper `instanceof` support across subclasses?](#12-how-would-you-implement-custom-error-classes-with-proper-instanceof-support-across-subclasses)

---

### 1. What do `try`, `catch`, and `finally` each do? 🟢

- `try`/`catch`/`finally` is JS's way of saying "attempt this, and if it blows up, don't crash the whole program — handle it gracefully instead."
- `try`: wraps the block of code that might throw an error. JS runs this code and watches for problems.
- `catch (error)`: only runs if something inside `try` throws. The `error` parameter holds whatever was thrown (usually an `Error` object with useful info like `.message`).
- `finally`: runs **every single time**, whether the `try` succeeded or the `catch` caught an error — even if there's a `return` inside `try` or `catch`. It's the right place for cleanup work that must happen no matter what (closing a file, hiding a loading spinner, releasing a lock).
- If there's no `catch` block that matches, the error keeps going up (see Q7 on error propagation) — but `finally` still runs before it leaves.

```js
try {
  riskyOperation(); // throws an error
} catch (err) {
  console.error('Failed:', err.message); // handle it
} finally {
  console.log('Cleanup runs regardless'); // always runs, error or not
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What is the built-in `Error` object, and what are its common subtypes? 🟢

- `Error` is JavaScript's built-in object for representing "something went wrong." Every error you throw or catch is either an `Error` or something built on top of it.
- It has three useful properties: `message` (a human-readable description of what happened), `name` (the error's type/category, e.g. `"TypeError"`), and `stack` (a trace of the function calls that led to the error — invaluable for debugging, since it shows you *where* in the code it happened).
- JS also ships a handful of built-in **subtypes** of `Error`, each for a specific kind of mistake:
  - `TypeError`: you used a value as the wrong type (e.g. calling something that isn't a function, or reading a property off `null`).
  - `RangeError`: a value is technically the right type but outside the allowed range (e.g. an array with a negative length).
  - `ReferenceError`: you referenced a variable that doesn't exist/isn't declared.
  - `SyntaxError`: the code itself is malformed — this is usually caught by the JS engine *before* your code even runs, so wrapping it in `try/catch` at runtime won't help if the syntax error is in the same file.

```js
null.foo;       // TypeError: Cannot read properties of null
new Array(-1);  // RangeError: Invalid array length
nonExistentVar; // ReferenceError: nonExistentVar is not defined
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you create a custom error type? 🟢

- Sometimes the built-in error types (`TypeError`, `RangeError`, etc.) aren't specific enough — you want your own category, like `ValidationError` or `NotFoundError`, that carries extra details relevant to your app.
- To do this, create a `class` that `extends Error`. Inside the constructor, call `super(message)` **first** — this runs `Error`'s own constructor, which sets up `.message` and `.stack` correctly. Skipping this (or calling it too late) leaves your custom error missing that setup.
- After `super()`, you're free to set `this.name` (so `console.log`/`.toString()` and error reporting tools show your custom type name instead of the generic `"Error"`) and attach any extra fields you need (like which form `field` failed validation).

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);           // sets up this.message and this.stack
    this.name = 'ValidationError'; // shows up in stack traces/logs
    this.field = field;       // extra info specific to this error type
  }
}

throw new ValidationError('Email is required', 'email');
// err.message => 'Email is required'
// err.field   => 'email'
// err.name    => 'ValidationError'
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between a syntax error, a runtime error, and a logical error? 🟢

- These three describe **when** and **how** a problem shows up in your code:
- **Syntax error**: the code is written incorrectly — it breaks the grammar rules of JavaScript (e.g. a missing bracket, an unclosed string). The JS engine catches this while *parsing* the file, before a single line actually runs. Nothing executes at all until it's fixed.
- **Runtime error**: the code is grammatically valid and starts running, but something goes wrong partway through (e.g. calling `.toUpperCase()` on `undefined`). This is what `try`/`catch` is designed to handle.
- **Logical error**: the code is valid, it runs from start to finish with no errors thrown at all — but it produces the **wrong answer** (e.g. using `+` instead of `-` in a calculation). This is the hardest to catch because nothing crashes or complains; you only notice when the output looks wrong, which is why tests and careful code review matter so much.

[↑ Back to top](#table-of-contents)

---

### 5. What does `throw` do, and what can you throw? 🟢

- `throw` immediately stops the normal flow of your code at that exact line — no code after it in the same function runs. Control jumps straight to the nearest enclosing `catch` block that can handle it.
- If there's no `catch` anywhere up the call stack, the program crashes (in Node) or logs an uncaught error (in the browser); for async code, it turns into a rejected Promise instead (see Q6).
- Technically, `throw` accepts **any value** at all — a string, a number, a plain object, even `undefined`. But you should almost always throw an actual `Error` (or a subclass of it), because `Error` automatically captures a `.stack` trace showing exactly where and how the error happened. Throwing a plain string loses that debugging information.

```js
throw 'Something broke';            // works, but no .stack — hard to trace back later
throw new Error('Something broke'); // preferred — has .stack and .message for debugging
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you handle errors in asynchronous code? 🟡

- Asynchronous code (things that finish later, like a network request) can't use a plain `try`/`catch` wrapped around it the way synchronous code can, because the error happens after the surrounding code has already finished running. JS gives you three different patterns depending on which style of async code you're using:
- **Callbacks**: follow the "error-first" convention — the callback's first parameter is reserved for an error (`null` if none happened), so you check it manually: `(err, data) => { if (err) { /* handle it */ } }`.
- **Promises**: attach a `.catch()` to the chain — it runs if the Promise (or any `.then()` before it) rejects.
- **async/await**: since `await` lets asynchronous code *look* synchronous, you can wrap it in a normal `try`/`catch`, and it works exactly as you'd expect — if the awaited Promise rejects, control jumps to `catch`.

```js
// Promise style
fetchData().then(handle).catch(handleError);

// async/await style — reads top-to-bottom like sync code
async function run() {
  try {
    const data = await fetchData(); // if this rejects, jumps to catch
    handle(data);
  } catch (err) {
    handleError(err);
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What is error propagation? 🟡

- The **call stack** is just the chain of functions that called each other to get to where you currently are (function `c` called `b`, which called `a`). Error propagation is what happens when an error travels back up that chain looking for somewhere to be handled.
- If a function doesn't wrap the risky code in `try`/`catch`, an error thrown inside it doesn't just disappear — it "bubbles up" to whoever called that function. That caller gets the same choice: handle it, or let it keep bubbling further up.
- This continues until either some `catch` block along the way handles it, or it reaches the very top of the stack with nothing to catch it — at which point the program crashes (or, for async code, the Promise rejects instead of crashing immediately).
- This is useful because it means you don't have to add a `try`/`catch` around every single function call — you can let errors travel up to one central place that knows how to handle them.

```js
function a() { throw new Error('fail'); }
function b() { a(); }       // doesn't catch — error keeps propagating upward
function c() {
  try { b(); } catch (e) { console.log('Caught at c:', e.message); }
}
c(); // "Caught at c: fail" — the error traveled a -> b -> c before being caught
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between operational errors and programmer errors? 🟡

- This distinction is about **why** an error happened, and it changes how you should react to it.
- **Operational errors**: failures that come from the outside world, which a well-written program should *expect* and handle gracefully, because they're not really "bugs" — they're normal things that can go wrong (a network request times out, a user types invalid input, a file doesn't exist). The right response is usually to catch it and show a friendly message or retry.
- **Programmer errors**: actual mistakes in the code itself (calling a function with the wrong type of argument, a typo in a variable name, forgetting to handle `undefined`). These indicate a bug that needs to be **fixed in the source code** — catching and silently ignoring them just hides the problem and can leave your app in a broken, unpredictable state.
- A simple way to remember it: operational errors are things *users* or the *environment* can cause; programmer errors are things only *you* (the developer) can fix.

> [!IMPORTANT]
> **Follow-up questions:**
> - Should you always `catch` and continue, or sometimes let the program crash on purpose?
> - How does this distinction affect how you design error classes in a real app?

[↑ Back to top](#table-of-contents)

---

### 9. How do you catch errors globally? 🟡

- Sometimes an error slips through without being caught anywhere in your code (maybe a bug you didn't anticipate). Both browsers and Node.js provide a global, catch-all listener as a last line of defense so you at least find out about it rather than silently failing.
- **Browser**: `window.onerror` fires for uncaught thrown errors; `window.addEventListener('unhandledrejection', ...)` fires specifically for Promises that rejected with nobody calling `.catch()` on them.
- **Node.js**: `process.on('uncaughtException', ...)` and `process.on('unhandledRejection', ...)` do the equivalent job.
- Treat these as **safety nets for logging/reporting** (e.g. sending the error to a monitoring service like Sentry so you know it happened), not as your main error-handling strategy. By the time an error reaches here, you've usually lost the context needed to recover gracefully — it's better to catch errors close to where they can actually happen (see Q6).

[↑ Back to top](#table-of-contents)

---

### 10. How would you design a centralized error-handling strategy for an app? 🔴

- The goal of "centralized" error handling is to avoid scattering `try`/`catch` and ad-hoc error-formatting logic all over the codebase, and instead have one consistent place that decides how every error gets logged, reported, and turned into a response.
- **Step 1 — build a small hierarchy of custom error classes.** A base `AppError` class (extending `Error`) carries shared fields like `statusCode` (what HTTP status to respond with) and `isOperational` (is this an expected failure or a real bug — see Q8). Specific subclasses like `ValidationError`, `NotFoundError`, `AuthError` extend it with their own defaults.
- **Step 2 — fewer scattered `try`/`catch` blocks.** In a server, wrap route handlers (or use middleware) so that any thrown/rejected error is automatically funneled into **one** centralized error-handling function, instead of writing `try`/`catch` in every single route.
- **Step 3 — respond differently based on the error type.** Expected operational errors (like "user not found") get logged quietly and turned into a clean 4xx response. Unexpected programmer errors get logged loudly (so someone notices and fixes the bug) and typically respond with a generic 500 so internal details aren't leaked to the user.

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode; // e.g. 404, 400
    this.isOperational = true;    // marks this as an expected, "safe" error
  }
}

// Express-style centralized handler — the last stop for every error in the app
app.use((err, req, res, next) => {
  const status = err.statusCode || 500; // default to 500 for unexpected errors
  res.status(status).json({ message: err.message });
});
```

[↑ Back to top](#table-of-contents)

---

### 11. What happens to the stack trace when you re-throw an error inside a `catch` block? 🔴

- "Re-throwing" means catching an error and then throwing it again (often after logging it, or wrapping it with more context) instead of swallowing it silently.
- A `.stack` trace is a record of where in your code the error was created — it's set **once**, the moment an `Error` object is constructed (i.e. when `new Error(...)` runs), not when it's thrown.
- If you re-throw the **exact same error object** with `throw err`, the original `.stack` is untouched — it still points to where the error first happened, since you didn't create a new `Error`.
- If instead you create a **brand-new** `Error` inside the `catch` block (e.g. to add more context, like `throw new Error('wrapped: ' + err.message)`), that new error gets its **own** stack trace starting from this new throw point — you lose the original location entirely unless you preserve it manually.
- The fix is the `cause` option (ES2022): pass the original error as `{ cause: err }` when constructing the new one. This attaches the original error as `.cause` on the new error, so you keep both the added context *and* a reference back to the original problem.

```js
try {
  JSON.parse('not json');
} catch (err) {
  // New error with more context, but the original is preserved via `cause`
  throw new Error('Parsing failed', { cause: err });
}
// later: caughtError.cause === original JSON.parse error
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement custom error classes with proper `instanceof` support across subclasses? 🔴

- `instanceof` checks whether an object was built from a particular class (or one of its parent classes), by walking up the object's **prototype chain** — a linked series of "parent" objects JS uses to look up inherited behavior. For example, `err instanceof AppError` checks if `AppError.prototype` appears somewhere in that chain.
- In plain, modern JS, `class X extends Error {}` sets up this chain correctly out of the box, so `instanceof` works as expected.
- The catch: when TypeScript or Babel compiles your class down to an older JS target (like ES5, which predates the `class` keyword and native `extends` support for built-ins), the compiled output can fail to properly link the prototype chain for subclasses of built-ins like `Error`. The result: `err instanceof NotFoundError` unexpectedly returns `false`, even though `err` was created with `new NotFoundError(...)`.
- The reliable fix is to explicitly reset the prototype right after calling `super()`, using `Object.setPrototypeOf(this, ActualClass.prototype)`. This forces the chain to be correct regardless of how the code was compiled.

```js
class AppError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
    Object.setPrototypeOf(this, AppError.prototype); // forces the prototype chain to be correct
  }
}
class NotFoundError extends AppError {}

const err = new NotFoundError('missing');
err instanceof NotFoundError; // true
err instanceof AppError;      // true — walks up the chain to AppError
err instanceof Error;         // true — and further up to Error
```

[↑ Back to top](#table-of-contents)

---
