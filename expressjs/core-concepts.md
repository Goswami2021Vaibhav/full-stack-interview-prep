# Core Concepts

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is Express.js, and why use it over plain Node.js `http`?](#1-what-is-expressjs-and-why-use-it-over-plain-nodejs-http)
- [2. How do you create a basic Express app?](#2-how-do-you-create-a-basic-express-app)
- [3. What is the `app` object?](#3-what-is-the-app-object)
- [4. What does `app.listen()` do?](#4-what-does-applisten-do)

**🟡 Medium**
- [5. What is the Express request-response cycle?](#5-what-is-the-express-request-response-cycle)
- [6. How do you serve static files in Express?](#6-how-do-you-serve-static-files-in-express)
- [7. What's the difference between `app.use()` and `app.get()`/`app.post()`?](#7-whats-the-difference-between-appuse-and-appgetapppost)
- [8. How do you read route parameters and query strings?](#8-how-do-you-read-route-parameters-and-query-strings)
- [9. What templating engines does Express support?](#9-what-templating-engines-does-express-support)

**🔴 Hard**
- [10. Why should you separate the Express `app` and `server`?](#10-why-should-you-separate-the-express-app-and-server)
- [11. How does Express's routing matching work internally?](#11-how-does-expresss-routing-matching-work-internally)
- [12. What changed between Express 4 and Express 5?](#12-what-changed-between-express-4-and-express-5)
- [13. How would you structure a larger Express application?](#13-how-would-you-structure-a-larger-express-application)

---

### 1. What is Express.js, and why use it over plain Node.js `http`? 🟢

- A minimal, unopinionated web framework built on top of Node's `http` module — adds routing, middleware, and request/response helpers so you're not manually parsing URLs, headers, and bodies for every endpoint.

```js
// Plain Node: manual method/path checking
http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/users') { /* ... */ }
});

// Express: declarative routing
app.get('/users', (req, res) => { /* ... */ });
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you create a basic Express app? 🟢

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => res.send('Hello World'));
app.listen(3000);
```

[↑ Back to top](#table-of-contents)

---

### 3. What is the `app` object? 🟢

- The main Express application instance — used to configure middleware, define routes, set app-wide settings (`app.set()`), and start the server. Created once via `express()`.

[↑ Back to top](#table-of-contents)

---

### 4. What does `app.listen()` do? 🟢

- Binds and starts the underlying HTTP server on a given port (and optional host), and begins accepting incoming connections. Internally, it's a thin wrapper around Node's `http.createServer(app).listen(...)`.

[↑ Back to top](#table-of-contents)

---

### 5. What is the Express request-response cycle? 🟡

- An incoming request enters Express's middleware stack and flows **top to bottom** through each registered middleware/route handler that matches, until one of them sends a response (`res.send`/`res.json`/etc.) — at which point the cycle ends, unless a handler calls `next()` to pass control further down the stack.

[↑ Back to top](#table-of-contents)

---

### 6. How do you serve static files in Express? 🟡

- `express.static()` serves files from a folder directly, without needing a route per file.

```js
app.use(express.static('public'));
// public/logo.png is now served at /logo.png
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `app.use()` and `app.get()`/`app.post()`? 🟡

- `app.use(path, fn)`: mounts middleware for **all** HTTP methods, and matches the path as a **prefix** (e.g. `/api` matches `/api/users` too).
- `app.get(path, fn)`/`app.post(path, fn)`: matches **only** that specific HTTP method, and the path must match more precisely (exact, unless using route patterns).

[↑ Back to top](#table-of-contents)

---

### 8. How do you read route parameters and query strings? 🟡

```js
app.get('/users/:id', (req, res) => {
  req.params.id;     // from /users/42 -> '42'
  req.query.sort;     // from /users/42?sort=asc -> 'asc'
});
```

[↑ Back to top](#table-of-contents)

---

### 9. What templating engines does Express support? 🟡

- Any engine implementing the `__express(filePath, options, callback)` interface — common choices: EJS, Pug, Handlebars. Configured via `app.set('view engine', 'ejs')` and `app.set('views', './views')`.

```js
app.set('view engine', 'ejs');
app.get('/', (req, res) => res.render('index', { name: 'Vaibhav' }));
```

[↑ Back to top](#table-of-contents)

---

### 10. Why should you separate the Express `app` and `server`? 🔴

- Keeping `app.js` (route/middleware definitions, exporting the `app`) separate from `server.js` (the file that actually calls `app.listen()`) lets you **import and test** the app (e.g. with Supertest) without binding to a real port — tests can hit the app in-memory, faster and without port conflicts.

```js
// app.js
const app = express();
/* routes/middleware */
module.exports = app;

// server.js
const app = require('./app');
app.listen(3000);
```

[↑ Back to top](#table-of-contents)

---

### 11. How does Express's routing matching work internally? 🔴

- Express maintains an ordered **stack of layers** (middleware + routes registered via `app.use`/`app.METHOD`). For each incoming request, it walks the stack **in registration order**, checking if each layer's path/method matches — the first matching layer runs; if it calls `next()`, Express continues to the next matching layer, and so on until a response is sent or the stack is exhausted (falling through to the default 404 handler).

[↑ Back to top](#table-of-contents)

---

### 12. What changed between Express 4 and Express 5? 🔴

- Express 5 (stabilized after a long beta) drops several long-deprecated APIs, changes some route-matching syntax (e.g. stricter handling of wildcard patterns, dropping support for certain regex-like string patterns in favor of standard path-to-regexp syntax), and requires Node.js promise rejection handling improvements (unhandled rejections in route handlers behave more predictably). Most day-to-day route/middleware code is unaffected, but some path-pattern edge cases need updating.

[↑ Back to top](#table-of-contents)

---

### 13. How would you structure a larger Express application? 🔴

```
src/
  routes/        # route definitions, grouped by resource
  controllers/    # request handlers — call services, shape the response
  services/       # business logic, independent of Express
  middleware/     # auth, validation, error handling
  models/         # data layer
  app.js          # wires everything together, exports `app`
server.js          # imports app, calls app.listen()
```

- Keeping **business logic out of route handlers** (in a separate `services/` layer) makes it testable without spinning up Express at all, and easier to reuse outside HTTP (e.g. a CLI script or a queue worker).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does separating controllers from services make unit testing significantly easier?

[↑ Back to top](#table-of-contents)

---
