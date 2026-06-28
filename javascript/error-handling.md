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

- `try`: the block of code that might throw an error.
- `catch (error)`: runs only if the `try` block throws — receives the thrown value.
- `finally`: always runs after, whether an error was thrown or not — used for cleanup (closing a connection, hiding a loading spinner).

```js
try {
  riskyOperation();
} catch (err) {
  console.error('Failed:', err.message);
} finally {
  console.log('Cleanup runs regardless');
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What is the built-in `Error` object, and what are its common subtypes? 🟢

- `Error` has `message`, `name`, and `stack` properties.
- Built-in subtypes for specific situations: `TypeError` (wrong type, e.g. calling a non-function), `RangeError` (value out of allowed range), `ReferenceError` (referencing an undeclared variable), `SyntaxError` (invalid code, usually caught at parse time, not in a `try/catch` around runtime code).

```js
null.foo;          // TypeError
new Array(-1);      // RangeError
nonExistentVar;      // ReferenceError
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you create a custom error type? 🟢

- Extend the built-in `Error` class, call `super(message)` first, then add any extra fields you need.

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

throw new ValidationError('Email is required', 'email');
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between a syntax error, a runtime error, and a logical error? 🟢

- **Syntax error**: invalid code structure — caught before execution even starts (e.g. a missing bracket).
- **Runtime error**: valid syntax, but fails while running (e.g. calling a method on `undefined`).
- **Logical error**: code runs fine with no errors thrown, but produces the **wrong result** — hardest to catch since nothing crashes.

[↑ Back to top](#table-of-contents)

---

### 5. What does `throw` do, and what can you throw? 🟢

- Immediately stops normal execution and hands control to the nearest enclosing `catch` block (or crashes the program/rejects the promise if there isn't one).
- Technically you can throw **any value** (a string, number, object), but throwing an actual `Error` instance is best practice — it captures a `stack` trace automatically.

```js
throw 'Something broke';       // works, but no stack trace, harder to debug
throw new Error('Something broke'); // preferred — has .stack, .message
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you handle errors in asynchronous code? 🟡

- **Callbacks**: error-first convention — check the first argument (`(err, data) => {...}`).
- **Promises**: `.catch()` on the chain.
- **async/await**: wrap the `await` in `try`/`catch`.

```js
// Promise
fetchData().then(handle).catch(handleError);

// async/await
async function run() {
  try {
    const data = await fetchData();
    handle(data);
  } catch (err) {
    handleError(err);
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What is error propagation? 🟡

- If an error isn't caught in the function where it's thrown, it bubbles up the **call stack** to the caller, then that caller's caller, and so on — until some `catch` handles it, or it reaches the top and crashes the program (or rejects the promise, for async code).

```js
function a() { throw new Error('fail'); }
function b() { a(); }       // doesn't catch, error keeps propagating
function c() {
  try { b(); } catch (e) { console.log('Caught at c:', e.message); }
}
c(); // "Caught at c: fail"
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between operational errors and programmer errors? 🟡

- **Operational errors**: expected failures from the outside world that a well-written program should anticipate and handle gracefully (network timeout, invalid user input, file not found).
- **Programmer errors**: actual bugs (calling a function with the wrong type, a typo, referencing `undefined`) — these usually indicate something needs to be **fixed in the code**, not just caught and swallowed.

> [!IMPORTANT]
> **Follow-up questions:**
> - Should you always `catch` and continue, or sometimes let the program crash on purpose?
> - How does this distinction affect how you design error classes in a real app?

[↑ Back to top](#table-of-contents)

---

### 9. How do you catch errors globally? 🟡

- **Browser**: `window.onerror` (for thrown errors) and `window.addEventListener('unhandledrejection', ...)` (for unhandled promise rejections).
- **Node.js**: `process.on('uncaughtException', ...)` and `process.on('unhandledRejection', ...)`.
- These are last-resort safety nets (e.g. for logging to an error-tracking service) — not a substitute for handling errors close to where they happen.

[↑ Back to top](#table-of-contents)

---

### 10. How would you design a centralized error-handling strategy for an app? 🔴

- Define a small hierarchy of custom error classes (e.g. `AppError` base class, with `ValidationError`, `NotFoundError`, `AuthError` subclasses) carrying a `statusCode` or `isOperational` flag.
- In a server, wrap route handlers so thrown/rejected errors are funneled into one centralized error-handling middleware instead of repeating `try`/`catch` everywhere.
- Log/report **programmer errors** differently from expected **operational errors** (e.g. alert on the former, just respond with a 4xx on the latter).

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Express-style centralized handler
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({ message: err.message });
});
```

[↑ Back to top](#table-of-contents)

---

### 11. What happens to the stack trace when you re-throw an error inside a `catch` block? 🔴

- If you `throw err` (the same error object), the original stack trace is **preserved** — it still shows where the error first occurred.
- If you create a **new** `Error` inside the `catch` (e.g. `throw new Error('wrapped: ' + err.message)`), the stack trace now starts from this new throw point, and you lose the original location unless you manually attach it (e.g. via `error.cause`, an ES2022 feature).

```js
try {
  JSON.parse('not json');
} catch (err) {
  throw new Error('Parsing failed', { cause: err }); // preserves original via `cause`
}
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement custom error classes with proper `instanceof` support across subclasses? 🔴

- Extending `Error` directly works in modern JS/TypeScript, but historically (especially when compiling down with older TS/Babel targets) the prototype chain could break, making `instanceof` checks fail for subclasses of `Error`.
- The fix is to explicitly reset the prototype after calling `super()`.

```js
class AppError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
    Object.setPrototypeOf(this, AppError.prototype); // ensures instanceof works correctly
  }
}
class NotFoundError extends AppError {}

const err = new NotFoundError('missing');
err instanceof NotFoundError; // true
err instanceof AppError;       // true
err instanceof Error;          // true
```

[↑ Back to top](#table-of-contents)

---
