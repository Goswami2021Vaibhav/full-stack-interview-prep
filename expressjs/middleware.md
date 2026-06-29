# Middleware

_Part of [Express.js](README.md) interview notes._

> Error-handling middleware specifically is covered in depth in [Error Handling](error-handling.md). This file covers general middleware concepts.

## Table of Contents

**🟢 Easy**
- [1. What is middleware in Express?](#1-what-is-middleware-in-express)
- [2. What is the signature of a middleware function?](#2-what-is-the-signature-of-a-middleware-function)
- [3. What does `next()` do?](#3-what-does-next-do)

**🟡 Medium**
- [4. What's the difference between application-level, router-level, and built-in middleware?](#4-whats-the-difference-between-application-level-router-level-and-built-in-middleware)
- [5. What is third-party middleware?](#5-what-is-third-party-middleware)
- [6. How do you apply middleware to only specific routes?](#6-how-do-you-apply-middleware-to-only-specific-routes)
- [7. What are `express.json()` and `express.urlencoded()` used for?](#7-what-are-expressjson-and-expressurlencoded-used-for)
- [8. Why does middleware execution order matter?](#8-why-does-middleware-execution-order-matter)

**🔴 Hard**
- [9. How do you write a custom logging middleware?](#9-how-do-you-write-a-custom-logging-middleware)
- [10. How would you implement an async middleware wrapper to catch promise rejections automatically?](#10-how-would-you-implement-an-async-middleware-wrapper-to-catch-promise-rejections-automatically)
- [11. What happens if you forget to call `next()` in a middleware function?](#11-what-happens-if-you-forget-to-call-next-in-a-middleware-function)
- [12. How do you skip remaining non-error middleware and jump straight to error handling?](#12-how-do-you-skip-remaining-non-error-middleware-and-jump-straight-to-error-handling)
- [13. How would you conditionally apply middleware based on runtime configuration?](#13-how-would-you-conditionally-apply-middleware-based-on-runtime-configuration)

---

### 1. What is middleware in Express? 🟢

- A function that sits in the request-handling pipeline, with access to `req`, `res`, and a `next` function — it can inspect/modify the request, end the response, or pass control along to the next middleware/route handler.

[↑ Back to top](#table-of-contents)

---

### 2. What is the signature of a middleware function? 🟢

```js
function middleware(req, res, next) {
  // do something
  next(); // pass control to the next middleware/route
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What does `next()` do? 🟢

- Passes control to the **next** matching middleware/route in the stack. Calling `next(err)` with an argument skips all remaining regular middleware and jumps straight to error-handling middleware (see [Error Handling](error-handling.md)).

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between application-level, router-level, and built-in middleware? 🟡

- **Application-level**: bound via `app.use()`/`app.METHOD()` directly on the app instance.
- **Router-level**: identical syntax, but bound to an `express.Router()` instance instead — scoped to that router's routes.
- **Built-in**: ships with Express itself (`express.static`, `express.json`, `express.urlencoded`) — no extra install needed.

[↑ Back to top](#table-of-contents)

---

### 5. What is third-party middleware? 🟡

- Middleware published as separate npm packages, not built into Express core — e.g. `cors`, `helmet`, `morgan` (logging), `compression`, `cookie-parser`.

```js
const morgan = require('morgan');
app.use(morgan('dev')); // logs each request
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you apply middleware to only specific routes? 🟡

- Pass it as an extra argument before the route handler, or mount it with `app.use(path, middleware)` scoped to that path prefix.

```js
app.get('/admin', requireAuth, (req, res) => { /* ... */ });
app.use('/admin', requireAuth); // applies to everything under /admin
```

[↑ Back to top](#table-of-contents)

---

### 7. What are `express.json()` and `express.urlencoded()` used for? 🟡

- Built-in middleware that parses incoming request bodies — `express.json()` for `Content-Type: application/json`, `express.urlencoded()` for traditional HTML form submissions — populating `req.body` with the parsed result. Without them, `req.body` is `undefined`.

```js
app.use(express.json());
app.post('/users', (req, res) => {
  console.log(req.body); // parsed JSON object
});
```

[↑ Back to top](#table-of-contents)

---

### 8. Why does middleware execution order matter? 🟡

- Middleware runs strictly in **registration order** — a middleware that parses the body (`express.json()`) must be registered **before** any route handler that reads `req.body`, and auth checks must run before the protected route handler they're meant to guard.

```js
app.get('/profile', (req, res) => { /* req.user is undefined here! */ });
app.use(authenticate); // registered too late — already missed the route above

// Fix: register authenticate before the routes that need it
app.use(authenticate);
app.get('/profile', (req, res) => { /* req.user is now set */ });
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you write a custom logging middleware? 🔴

```js
function requestLogger(req, res, next) {
  const start = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.url} ${res.statusCode} - ${Date.now() - start}ms`);
  });
  next();
}
app.use(requestLogger);
```

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement an async middleware wrapper to catch promise rejections automatically? 🔴

- Express doesn't automatically forward a rejected promise from an `async` route handler to error-handling middleware (in versions before Express 5) — wrap handlers so any thrown/rejected error gets passed to `next()` automatically.

```js
const asyncHandler = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await db.findUser(req.params.id); // if this throws, asyncHandler catches it
  res.json(user);
}));
```

[↑ Back to top](#table-of-contents)

---

### 11. What happens if you forget to call `next()` in a middleware function? 🔴

- The request **hangs** — no further middleware/route runs, and no response is ever sent, so the client just waits until it eventually times out. (Unless that middleware itself called `res.send()`/`res.end()`, in which case not calling `next()` is correct and intentional.)

[↑ Back to top](#table-of-contents)

---

### 12. How do you skip remaining non-error middleware and jump straight to error handling? 🔴

- Call `next(err)` with an argument — Express recognizes this and skips ahead to the nearest error-handling middleware (a 4-argument function), bypassing any remaining regular middleware/routes.

```js
app.get('/users/:id', (req, res, next) => {
  if (!isValidId(req.params.id)) {
    return next(new Error('Invalid ID')); // jumps straight to error handler
  }
  // ...
});
```

[↑ Back to top](#table-of-contents)

---

### 13. How would you conditionally apply middleware based on runtime configuration? 🔴

- Wrap the middleware call in a check, or register it conditionally at startup based on `NODE_ENV`/feature flags.

```js
if (process.env.NODE_ENV !== 'production') {
  app.use(morgan('dev')); // verbose logging only outside production
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you make a single middleware function itself conditionally act, rather than conditionally registering it at all?

[↑ Back to top](#table-of-contents)

---
