# Routing

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you define a route in Express?](#1-how-do-you-define-a-route-in-express)
- [2. What HTTP methods does Express support route handlers for?](#2-what-http-methods-does-express-support-route-handlers-for)
- [3. How do you handle route parameters?](#3-how-do-you-handle-route-parameters)

**🟡 Medium**
- [4. What is `express.Router()`, and why use it?](#4-what-is-expressrouter-and-why-use-it)
- [5. How do you chain multiple handlers for one route?](#5-how-do-you-chain-multiple-handlers-for-one-route)
- [6. How do you define a route that matches multiple HTTP methods?](#6-how-do-you-define-a-route-that-matches-multiple-http-methods)
- [7. How do you implement a catch-all/404 route?](#7-how-do-you-implement-a-catch-all404-route)

**🔴 Hard**
- [8. How does route matching order affect behavior in Express?](#8-how-does-route-matching-order-affect-behavior-in-express)
- [9. What's the difference between route-level and app-level middleware?](#9-whats-the-difference-between-route-level-and-app-level-middleware)
- [10. How would you implement nested routers (mounting a router on a path prefix)?](#10-how-would-you-implement-nested-routers-mounting-a-router-on-a-path-prefix)
- [11. How do you pass data from a parent router/middleware to a nested router?](#11-how-do-you-pass-data-from-a-parent-routermiddleware-to-a-nested-router)
- [12. How would you implement versioned APIs in Express routing?](#12-how-would-you-implement-versioned-apis-in-express-routing)
- [13. How do you define routes with optional parameters or wildcards?](#13-how-do-you-define-routes-with-optional-parameters-or-wildcards)

---

### 1. How do you define a route in Express? 🟢

```js
app.get('/users', (req, res) => res.json([]));
app.post('/users', (req, res) => res.status(201).json({ created: true }));
```

[↑ Back to top](#table-of-contents)

---

### 2. What HTTP methods does Express support route handlers for? 🟢

- A method for each standard HTTP verb: `app.get()`, `app.post()`, `app.put()`, `app.patch()`, `app.delete()`, plus `app.all()` (matches any method) and `app.options()`.

[↑ Back to top](#table-of-contents)

---

### 3. How do you handle route parameters? 🟢

- Prefix a path segment with `:` — the matched value is available on `req.params`.

```js
app.get('/users/:id/posts/:postId', (req, res) => {
  const { id, postId } = req.params;
});
```

[↑ Back to top](#table-of-contents)

---

### 4. What is `express.Router()`, and why use it? 🟡

- A mini, mountable Express app — lets you group related routes into their own module, keeping `app.js` from becoming one giant file of every route.

```js
// routes/users.js
const router = express.Router();
router.get('/', (req, res) => res.json([]));
module.exports = router;

// app.js
app.use('/users', require('./routes/users'));
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you chain multiple handlers for one route? 🟡

- Pass multiple functions to a route — each can call `next()` to pass control to the next one in the chain (commonly used for per-route validation/auth before the final handler).

```js
app.post('/users', validateUser, authenticate, (req, res) => {
  res.status(201).json({ created: true });
});
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you define a route that matches multiple HTTP methods? 🟡

- Use `app.route(path)` to chain method handlers for the **same** path without repeating it.

```js
app.route('/users/:id')
  .get((req, res) => { /* ... */ })
  .put((req, res) => { /* ... */ })
  .delete((req, res) => { /* ... */ });
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you implement a catch-all/404 route? 🟡

- Add a middleware **after all other routes** with no specific path — anything that fell through unmatched reaches it.

```js
app.use((req, res) => {
  res.status(404).json({ error: 'Not found' });
});
```

[↑ Back to top](#table-of-contents)

---

### 8. How does route matching order affect behavior in Express? 🔴

- Routes/middleware are checked in the **exact order they're registered** — the first matching one runs. A broad route (or catch-all) registered too early will intercept requests meant for more specific routes registered later, since Express doesn't pick "the best match," just the **first** one.

```js
app.get('/users/:id', handlerA); // registered first — always wins for /users/anything
app.get('/users/new', handlerB);  // never reached! :id already matched 'new'

// Fix: register the more specific route first
app.get('/users/new', handlerB);
app.get('/users/:id', handlerA);
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between route-level and app-level middleware? 🔴

- **App-level** (`app.use(fn)`): applies to every request that reaches the app (or every request matching a path prefix).
- **Router-level** (`router.use(fn)`): identical concept, but scoped to a specific `express.Router()` instance — only affects requests routed through that router.
- Both use the same middleware signature; the difference is purely about **scope** (whole app vs. one router's routes).

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement nested routers (mounting a router on a path prefix)? 🔴

```js
// routes/comments.js
const router = express.Router({ mergeParams: true });
router.get('/', (req, res) => res.json({ postId: req.params.postId }));
module.exports = router;

// routes/posts.js
const router = express.Router();
router.use('/:postId/comments', require('./comments'));
module.exports = router;

// app.js
app.use('/posts', require('./routes/posts'));
// GET /posts/5/comments -> reaches comments.js, with req.params.postId === '5'
```

[↑ Back to top](#table-of-contents)

---

### 11. How do you pass data from a parent router/middleware to a nested router? 🔴

- For **route parameters**, pass `{ mergeParams: true }` when creating the child router (Q10) — without it, a nested router can't see the parent's `:params`.
- For arbitrary data, attach it to `req` (e.g. `req.user = ...` in an auth middleware) — any downstream middleware/router sees the same `req` object.

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement versioned APIs in Express routing? 🔴

- Mount different router modules under version-prefixed paths — lets you run multiple API versions side by side during a migration.

```js
app.use('/api/v1', require('./routes/v1'));
app.use('/api/v2', require('./routes/v2'));
```

[↑ Back to top](#table-of-contents)

---

### 13. How do you define routes with optional parameters or wildcards? 🔴

- Optional parameter: append `?` to the parameter segment. Wildcard: `*` matches any remaining path.

```js
app.get('/users/:id?', (req, res) => { /* id may be undefined */ });
app.get('/files/*', (req, res) => { /* req.params[0] = the matched wildcard portion */ });
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How did wildcard/optional parameter syntax change between Express 4 and Express 5's path-to-regexp version?

[↑ Back to top](#table-of-contents)

---
