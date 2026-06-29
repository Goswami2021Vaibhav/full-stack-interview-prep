# Best Practices

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Why should you separate the Express `app` from the `server`?](#1-why-should-you-separate-the-express-app-from-the-server)
- [2. What's the recommended way to organize routes/controllers in a larger app?](#2-whats-the-recommended-way-to-organize-routescontrollers-in-a-larger-app)

**🟡 Medium**
- [3. How do you manage environment-specific configuration in an Express app?](#3-how-do-you-manage-environment-specific-configuration-in-an-express-app)
- [4. What's a good practice for validating request bodies consistently across routes?](#4-whats-a-good-practice-for-validating-request-bodies-consistently-across-routes)
- [5. How do you avoid repeating try/catch in every async route handler?](#5-how-do-you-avoid-repeating-trycatch-in-every-async-route-handler)
- [6. How do you structure middleware for cross-cutting concerns?](#6-how-do-you-structure-middleware-for-cross-cutting-concerns)

**🔴 Hard**
- [7. How would you implement graceful shutdown in an Express server?](#7-how-would-you-implement-graceful-shutdown-in-an-express-server)
- [8. What are best practices for API versioning in Express?](#8-what-are-best-practices-for-api-versioning-in-express)
- [9. How do you write testable Express route handlers?](#9-how-do-you-write-testable-express-route-handlers)
- [10. How would you implement health-check and readiness endpoints?](#10-how-would-you-implement-health-check-and-readiness-endpoints)
- [11. What's a sensible default folder structure for a medium-sized Express + database project?](#11-whats-a-sensible-default-folder-structure-for-a-medium-sized-express--database-project)
- [12. How do you avoid tight coupling between Express and your business logic?](#12-how-do-you-avoid-tight-coupling-between-express-and-your-business-logic)

---

### 1. Why should you separate the Express `app` from the `server`? 🟢

- Lets you `require()` and test the app (e.g. with Supertest) without binding to a real port — see [Core Concepts](core-concepts.md#10-why-should-you-separate-the-express-app-and-server) for the full explanation.

[↑ Back to top](#table-of-contents)

---

### 2. What's the recommended way to organize routes/controllers in a larger app? 🟢

- Group route definitions by **resource** (`routes/users.js`, `routes/orders.js`) using `express.Router()`, and keep the actual request-handling logic in separate **controller** functions imported into the route file, rather than inlining large handler bodies directly in the route definitions.

[↑ Back to top](#table-of-contents)

---

### 3. How do you manage environment-specific configuration in an Express app? 🟡

- Load config from environment variables (via `dotenv` locally), with a single config module that reads and validates them once at startup — rather than scattering `process.env.X` reads throughout the codebase.

```js
// config.js
module.exports = {
  port: process.env.PORT || 3000,
  dbUrl: process.env.DATABASE_URL,
};
```

[↑ Back to top](#table-of-contents)

---

### 4. What's a good practice for validating request bodies consistently across routes? 🟡

- Define a validation schema per route (Zod/Joi/`express-validator`) and apply it as middleware **before** the route handler — keeps handlers focused on business logic instead of manual field-checking, and ensures every route validates input the same consistent way.

[↑ Back to top](#table-of-contents)

---

### 5. How do you avoid repeating try/catch in every async route handler? 🟡

- Wrap async handlers in a reusable helper that forwards rejections to `next()` automatically — see [Middleware](middleware.md#10-how-would-you-implement-an-async-middleware-wrapper-to-catch-promise-rejections-automatically).

[↑ Back to top](#table-of-contents)

---

### 6. How do you structure middleware for cross-cutting concerns? 🟡

- Keep each concern (auth, logging, validation, error handling) in its **own** middleware module, composed in `app.js` — instead of one giant middleware function doing several unrelated things, which gets hard to test/reuse/reorder independently.

[↑ Back to top](#table-of-contents)

---

### 7. How would you implement graceful shutdown in an Express server? 🔴

- On `SIGTERM`/`SIGINT`, stop accepting new connections, let in-flight requests finish, close DB connections, then exit — avoids abruptly dropping active requests during a deploy/restart.

```js
const server = app.listen(3000);

process.on('SIGTERM', () => {
  server.close(async () => {
    await db.disconnect();
    process.exit(0);
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 8. What are best practices for API versioning in Express? 🔴

- Version explicitly in the **URL path** (`/api/v1/...`) for clarity and easy routing (see [Routing](routing.md#12-how-would-you-implement-versioned-apis-in-express-routing)), keep older versions running unchanged while a new version is developed alongside them, and deprecate with advance notice (response headers like `Deprecation`/`Sunset`) rather than breaking clients abruptly.

[↑ Back to top](#table-of-contents)

---

### 9. How do you write testable Express route handlers? 🔴

- Keep handlers **thin** — parse/validate input, delegate to a service function for actual business logic, then shape the response. The service function itself (plain JS, no `req`/`res`) can be unit tested directly without spinning up Express at all; the route handler itself is tested at a higher level (e.g. Supertest) for integration coverage.

```js
// controller — thin, just wiring
async function createUser(req, res, next) {
  try {
    const user = await userService.create(req.body); // pure business logic, testable alone
    res.status(201).json(user);
  } catch (err) { next(err); }
}
```

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement health-check and readiness endpoints? 🔴

- `/health` (liveness): a lightweight endpoint confirming the process is up and responding — used by orchestrators (Kubernetes) to know whether to restart the container.
- `/ready` (readiness): checks that the app's actual dependencies (DB connection, cache) are reachable — used to know whether to **route traffic** to this instance yet (e.g. right after startup, before a DB connection is established).

```js
app.get('/health', (req, res) => res.sendStatus(200));
app.get('/ready', async (req, res) => {
  const dbOk = await db.ping().then(() => true).catch(() => false);
  res.sendStatus(dbOk ? 200 : 503);
});
```

[↑ Back to top](#table-of-contents)

---

### 11. What's a sensible default folder structure for a medium-sized Express + database project? 🔴

```
src/
  routes/
  controllers/
  services/
  models/
  middleware/
  config/
  utils/
  app.js
server.js
tests/
```

- See [Core Concepts](core-concepts.md#13-how-would-you-structure-a-larger-express-application) for the rationale behind each layer's separation.

[↑ Back to top](#table-of-contents)

---

### 12. How do you avoid tight coupling between Express and your business logic? 🔴

- Keep business/domain logic in plain functions/classes that know nothing about `req`/`res`/Express at all — route handlers become a thin adapter layer translating HTTP concerns into calls against that logic. This makes the core logic reusable (CLI scripts, queue workers, a future migration off Express) and trivially unit-testable without any HTTP machinery involved.

> [!IMPORTANT]
> **Follow-up questions:**
> - What would migrating from Express to another framework (e.g. Fastify) look like if business logic were tightly coupled to `req`/`res`, versus cleanly separated?

[↑ Back to top](#table-of-contents)

---
