# Security & Best Practices

_Part of [REST API](README.md) interview notes._

> General Express-level security middleware (Helmet, CORS setup, rate-limiting code) is covered in [Express.js › Security](../expressjs/security.md). This file focuses on API-design-level security concerns.

## Table of Contents

**🟢 Easy**
- [1. Why should REST APIs always use HTTPS?](#1-why-should-rest-apis-always-use-https)
- [2. What is CORS, and why does a REST API need to configure it?](#2-what-is-cors-and-why-does-a-rest-api-need-to-configure-it)

**🟡 Medium**
- [3. How do you prevent mass assignment vulnerabilities in a REST API?](#3-how-do-you-prevent-mass-assignment-vulnerabilities-in-a-rest-api)
- [4. How do you validate and sanitize input consistently across all endpoints?](#4-how-do-you-validate-and-sanitize-input-consistently-across-all-endpoints)
- [5. How do you protect a REST API from injection attacks?](#5-how-do-you-protect-a-rest-api-from-injection-attacks)
- [6. What HTTP security headers should a REST API set?](#6-what-http-security-headers-should-a-rest-api-set)

**🔴 Hard**
- [7. How do you design a REST API to avoid over-fetching of sensitive fields?](#7-how-do-you-design-a-rest-api-to-avoid-over-fetching-of-sensitive-fields)
- [8. How would you implement field-level authorization?](#8-how-would-you-implement-field-level-authorization)
- [9. How do you protect against IDOR vulnerabilities in REST endpoints?](#9-how-do-you-protect-against-idor-vulnerabilities-in-rest-endpoints)
- [10. How would you design audit logging for sensitive API operations?](#10-how-would-you-design-audit-logging-for-sensitive-api-operations)
- [11. How does the principle of least privilege apply to REST API design?](#11-how-does-the-principle-of-least-privilege-apply-to-rest-api-design)
- [12. How do you rotate API keys/secrets without downtime?](#12-how-do-you-rotate-api-keyssecrets-without-downtime)
- [13. How would you design rate limiting and quotas for a public API with multiple client tiers?](#13-how-would-you-design-rate-limiting-and-quotas-for-a-public-api-with-multiple-client-tiers)

---

### 1. Why should REST APIs always use HTTPS? 🟢

- Without it, credentials, tokens, and data travel in **plaintext** over the network — trivially interceptable (e.g. on shared/public Wi-Fi). HTTPS encrypts the transport, and is a baseline, non-negotiable requirement for any API handling auth or sensitive data.

[↑ Back to top](#table-of-contents)

---

### 2. What is CORS, and why does a REST API need to configure it? 🟢

- A browser security mechanism restricting which **origins** (domains) can call your API from client-side JS. An API serving browser-based clients on a different origin needs to explicitly opt those origins in via CORS headers, or browser requests from those origins will be blocked.

[↑ Back to top](#table-of-contents)

---

### 3. How do you prevent mass assignment vulnerabilities in a REST API? 🟡

- Don't blindly save an entire incoming request body onto a model — explicitly **whitelist** which fields a given endpoint is allowed to set, so a client can't sneak in fields like `isAdmin: true` that were never meant to be client-settable.

```js
// Vulnerable: client could include { "isAdmin": true } in the body
const user = new User(req.body);

// Safe: only allow specific fields
const { name, email } = req.body;
const user = new User({ name, email });
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you validate and sanitize input consistently across all endpoints? 🟡

- Define a schema per endpoint (Zod/Joi) and run it as middleware before the handler, so validation logic is declared once and applied uniformly — rather than ad-hoc manual checks scattered (or missing) across individual routes.

[↑ Back to top](#table-of-contents)

---

### 5. How do you protect a REST API from injection attacks? 🟡

- Never interpolate raw user input into a query string — use parameterized queries/prepared statements (SQL) or a query builder/ORM that escapes automatically, and avoid passing raw request bodies directly into NoSQL query operators (see [Express.js › Security](../expressjs/security.md#3-how-do-you-prevent-sqlnosql-injection-in-an-express-app)).

[↑ Back to top](#table-of-contents)

---

### 6. What HTTP security headers should a REST API set? 🟡

- `Strict-Transport-Security` (force HTTPS), `X-Content-Type-Options: nosniff`, `Content-Security-Policy` (if serving any HTML), and a sensible `Access-Control-Allow-Origin` (not a blanket `*` for any authenticated endpoint). Most of these are set in one line via Helmet (see [Express.js › Security](../expressjs/security.md#1-what-is-helmetjs-and-what-does-it-do)).

[↑ Back to top](#table-of-contents)

---

### 7. How do you design a REST API to avoid over-fetching of sensitive fields? 🔴

- Don't return your full internal data model by default — explicitly serialize only the fields a given response is meant to expose (a dedicated response DTO/serializer per endpoint), so an internal field like `passwordHash` or `internalNotes` can never accidentally leak just because a new column was added to the underlying table.

```js
// Explicit allowlist, not just returning the raw DB row
function toUserResponse(user) {
  return { id: user.id, name: user.name, email: user.email }; // no passwordHash, no internalFlags
}
```

[↑ Back to top](#table-of-contents)

---

### 8. How would you implement field-level authorization? 🔴

- Apply different serialization logic depending on the **requester's** identity/role — e.g. an admin viewing a user sees their email and phone, while a regular peer viewing the same resource sees only their public name and avatar.

```js
function toUserResponse(user, requester) {
  const base = { id: user.id, name: user.name };
  if (requester.role === 'admin' || requester.id === user.id) {
    base.email = user.email;
  }
  return base;
}
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you protect against IDOR vulnerabilities in REST endpoints? 🔴

- **I**nsecure **D**irect **O**bject **R**eference: when an endpoint trusts a client-supplied ID without checking the **requester actually owns/can access** that specific resource (e.g. `GET /invoices/123` returning someone else's invoice just because the ID was guessable). Always check ownership/permission against the **authenticated user**, not just that the resource with that ID exists.

```js
app.get('/invoices/:id', async (req, res) => {
  const invoice = await Invoice.findById(req.params.id);
  if (!invoice || invoice.userId !== req.user.id) return res.sendStatus(404); // not 403 — don't confirm existence
  res.json(invoice);
});
```

[↑ Back to top](#table-of-contents)

---

### 10. How would you design audit logging for sensitive API operations? 🔴

- Log **who** did **what**, to **which** resource, **when**, and from **where** (IP/user agent), for any state-changing or sensitive-read operation — stored separately from regular application logs (often append-only/immutable), since audit logs need to survive even if the rest of the system is compromised, for post-incident investigation and compliance.

[↑ Back to top](#table-of-contents)

---

### 11. How does the principle of least privilege apply to REST API design? 🔴

- Every client/token/role should have access to **only** what it specifically needs — narrow OAuth scopes rather than one all-access token, role checks per endpoint rather than a single binary "is logged in" check, and default-deny on new endpoints until explicitly granted, rather than default-allow.

[↑ Back to top](#table-of-contents)

---

### 12. How do you rotate API keys/secrets without downtime? 🔴

- Support **two valid keys/secrets simultaneously** during a transition window — issue the new one, let clients migrate to it, and only revoke the old one after confirming usage has fully shifted over, rather than invalidating the old key instantly (which would break every client still using it).

[↑ Back to top](#table-of-contents)

---

### 13. How would you design rate limiting and quotas for a public API with multiple client tiers? 🔴

- Tie limits to the authenticated client's plan/tier (Q11 of [Authentication & Authorization](authentication-and-authorization.md#11-how-does-rate-limiting-interact-with-authentication)) rather than a single global limit — e.g. free tier: 100 req/hour, paid tier: 10,000 req/hour — enforced via a shared store (Redis) so limits are consistent across multiple server instances, and communicated to the client via `X-RateLimit-*` headers so they can self-throttle before hitting 429s.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does rate limiting need a shared store like Redis instead of in-memory counters once an API runs on more than one server instance?

[↑ Back to top](#table-of-contents)

---
