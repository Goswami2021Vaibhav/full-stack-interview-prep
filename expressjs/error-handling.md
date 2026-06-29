# Error Handling

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you handle errors in Express route handlers?](#1-how-do-you-handle-errors-in-express-route-handlers)
- [2. What is Express's default error handler?](#2-what-is-expresss-default-error-handler)

**🟡 Medium**
- [3. How do you write a custom error-handling middleware?](#3-how-do-you-write-a-custom-error-handling-middleware)
- [4. How does Express identify error-handling middleware vs. regular middleware?](#4-how-does-express-identify-error-handling-middleware-vs-regular-middleware)
- [5. How do you handle errors thrown inside async route handlers?](#5-how-do-you-handle-errors-thrown-inside-async-route-handlers)
- [6. How do you return consistent JSON error responses across an API?](#6-how-do-you-return-consistent-json-error-responses-across-an-api)

**🔴 Hard**
- [7. How would you implement a centralized error class hierarchy for an Express app?](#7-how-would-you-implement-a-centralized-error-class-hierarchy-for-an-express-app)
- [8. How do you avoid leaking stack traces/sensitive info in production error responses?](#8-how-do-you-avoid-leaking-stack-tracessensitive-info-in-production-error-responses)
- [9. How do you handle 404s distinctly from other errors?](#9-how-do-you-handle-404s-distinctly-from-other-errors)
- [10. How do you avoid repeating try/catch in every async route handler?](#10-how-do-you-avoid-repeating-trycatch-in-every-async-route-handler)
- [11. How does error-handling middleware ordering affect which errors get caught?](#11-how-does-error-handling-middleware-ordering-affect-which-errors-get-caught)
- [12. What's the difference between synchronous errors and propagated async errors in Express?](#12-whats-the-difference-between-synchronous-errors-and-propagated-async-errors-in-express)

---

### 1. How do you handle errors in Express route handlers? 🟢

- For synchronous code, just `throw` — Express catches it automatically. For async code, catch the error and pass it to `next(err)` explicitly (in Express 4) so it reaches error-handling middleware.

```js
app.get('/sync', (req, res) => {
  throw new Error('Boom'); // Express catches this automatically
});

app.get('/async', async (req, res, next) => {
  try {
    await doSomething();
  } catch (err) {
    next(err); // must forward manually in Express 4
  }
});
```

[↑ Back to top](#table-of-contents)

---

### 2. What is Express's default error handler? 🟢

- If no custom error-handling middleware is defined (or none of them sends a response), Express's built-in default handler sends a generic response with the error's status code (or 500) and, in development, the stack trace in the response body — not suitable for production use as-is.

[↑ Back to top](#table-of-contents)

---

### 3. How do you write a custom error-handling middleware? 🟡

- A middleware function with **four** parameters, `(err, req, res, next)` — Express identifies it as an error handler specifically by that arity (Q4).

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.statusCode || 500).json({ error: err.message });
});
```

[↑ Back to top](#table-of-contents)

---

### 4. How does Express identify error-handling middleware vs. regular middleware? 🟡

- Purely by the **number of declared parameters** — a function with 4 params (`err, req, res, next`) is treated as an error handler; one with 3 (`req, res, next`) is treated as regular middleware. Express inspects `fn.length` to decide.

[↑ Back to top](#table-of-contents)

---

### 5. How do you handle errors thrown inside async route handlers? 🟡

- In Express 4, you **must** manually `catch` and call `next(err)` — an unhandled rejection inside an `async` handler does **not** automatically reach error-handling middleware. Use a wrapper (see [Middleware](middleware.md#10-how-would-you-implement-an-async-middleware-wrapper-to-catch-promise-rejections-automatically)) to avoid repeating this everywhere.

[↑ Back to top](#table-of-contents)

---

### 6. How do you return consistent JSON error responses across an API? 🟡

- Centralize response formatting in **one** error-handling middleware at the end of the stack, instead of each route shaping its own error JSON.

```js
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    error: { message: err.message, code: err.code || 'INTERNAL_ERROR' },
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 7. How would you implement a centralized error class hierarchy for an Express app? 🔴

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}
class NotFoundError extends AppError {
  constructor(message = 'Not found') { super(message, 404); }
}
class ValidationError extends AppError {
  constructor(message = 'Invalid input') { super(message, 400); }
}

// route
if (!user) throw new NotFoundError('User not found');

// error middleware
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({ error: err.message });
});
```

[↑ Back to top](#table-of-contents)

---

### 8. How do you avoid leaking stack traces/sensitive info in production error responses? 🔴

- Check `NODE_ENV` (or an explicit flag) in the error-handling middleware, and only include `err.stack`/internal details when **not** in production — always log the full details server-side regardless, just don't expose them to the client.

```js
app.use((err, req, res, next) => {
  logger.error(err); // always log full details
  res.status(err.statusCode || 500).json({
    error: err.message,
    ...(process.env.NODE_ENV !== 'production' && { stack: err.stack }),
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you handle 404s distinctly from other errors? 🔴

- Add a dedicated "not found" middleware **after** all real routes but **before** the error handler — it only runs if nothing else matched, converting the absence of a route into an explicit, structured error.

```js
app.use((req, res, next) => {
  next(new NotFoundError(`Route ${req.originalUrl} not found`));
});

app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({ error: err.message });
});
```

[↑ Back to top](#table-of-contents)

---

### 10. How do you avoid repeating try/catch in every async route handler? 🔴

- Wrap each async handler in a helper that catches rejections and forwards them to `next()` automatically (see [Middleware](middleware.md#10-how-would-you-implement-an-async-middleware-wrapper-to-catch-promise-rejections-automatically)), or use a package like `express-async-errors` that patches Express to do this globally.

[↑ Back to top](#table-of-contents)

---

### 11. How does error-handling middleware ordering affect which errors get caught? 🔴

- Error-handling middleware only catches errors from middleware/routes **registered before it** — it must be the **last** thing added to the stack. If registered too early, errors thrown by routes defined afterward won't reach it (they'll fall through to Express's default handler instead).

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between synchronous errors and propagated async errors in Express? 🔴

- **Synchronous** `throw` inside a route handler: Express's internal try/catch around each handler call catches it automatically, no manual `next(err)` needed.
- **Asynchronous** errors (rejected promises, errors inside a `setTimeout`/callback): Express 4's automatic catching **doesn't** extend into async code — you must explicitly call `next(err)` yourself, or use a wrapper. (Express 5 improves this for returned promises specifically, but the underlying distinction between sync and truly async/deferred errors still matters.)

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can't Express's synchronous try/catch automatically catch an error thrown inside a `.then()` callback or a `setTimeout`?

[↑ Back to top](#table-of-contents)

---
