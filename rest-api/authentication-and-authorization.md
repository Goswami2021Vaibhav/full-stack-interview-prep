# Authentication & Authorization

_Part of [REST API](README.md) interview notes._

> This file covers how auth fits into REST API design specifically. Deep mechanics of JWT structure, OAuth flows, and session implementation are covered in the dedicated [Authentication](../authentication/README.md) topic.

## Table of Contents

**🟢 Easy**
- [1. How does authentication typically work in a stateless REST API?](#1-how-does-authentication-typically-work-in-a-stateless-rest-api)
- [2. What's the difference between authentication and authorization?](#2-whats-the-difference-between-authentication-and-authorization)
- [3. Where should an API token be sent in a request?](#3-where-should-an-api-token-be-sent-in-a-request)

**🟡 Medium**
- [4. What's the difference between session-based and token-based authentication for REST APIs?](#4-whats-the-difference-between-session-based-and-token-based-authentication-for-rest-apis)
- [5. Why does storing session state on the server conflict with REST's stateless constraint?](#5-why-does-storing-session-state-on-the-server-conflict-with-rests-stateless-constraint)
- [6. How do you implement role-based access control for REST endpoints?](#6-how-do-you-implement-role-based-access-control-for-rest-endpoints)
- [7. How should a REST API respond when a request is unauthenticated vs. unauthorized?](#7-how-should-a-rest-api-respond-when-a-request-is-unauthenticated-vs-unauthorized)

**🔴 Hard**
- [8. How do you secure a REST API for machine-to-machine communication?](#8-how-do-you-secure-a-rest-api-for-machine-to-machine-communication)
- [9. What is API key authentication, and what are its limitations vs. OAuth?](#9-what-is-api-key-authentication-and-what-are-its-limitations-vs-oauth)
- [10. How would you implement scoped, fine-grained permissions for a REST API?](#10-how-would-you-implement-scoped-fine-grained-permissions-for-a-rest-api)
- [11. How does rate limiting interact with authentication?](#11-how-does-rate-limiting-interact-with-authentication)
- [12. How would you design an API to support both first-party and third-party client authentication?](#12-how-would-you-design-an-api-to-support-both-first-party-and-third-party-client-authentication)

---

### 1. How does authentication typically work in a stateless REST API? 🟢

- The client includes credentials (typically a **token**) with every request — most commonly a Bearer token in the `Authorization` header — and the server verifies it independently on each request, without needing to remember anything from previous requests.

```http
GET /profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between authentication and authorization? 🟢

- **Authentication**: confirming **who** you are (login, verifying credentials).
- **Authorization**: confirming **what you're allowed to do** once identified (permissions, roles, access control).

[↑ Back to top](#table-of-contents)

---

### 3. Where should an API token be sent in a request? 🟢

- Conventionally in the `Authorization` header as a Bearer token — not in the URL/query string (it can leak via browser history, server logs, and Referer headers).

```http
Authorization: Bearer <token>
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between session-based and token-based authentication for REST APIs? 🟡

- **Session-based**: the server stores session state (typically server-side, keyed by a cookie) — simple, but requires the server to remember each client, working against REST's stateless ideal and complicating horizontal scaling (sessions need to be shared, e.g. via Redis).
- **Token-based** (e.g. JWT): the server embeds the necessary claims **in** the token itself — any server instance can verify it independently with no shared session store, aligning better with REST's statelessness.

[↑ Back to top](#table-of-contents)

---

### 5. Why does storing session state on the server conflict with REST's stateless constraint? 🟡

- REST's stateless constraint says each request must carry everything needed to process it, with the server holding **no** client context between requests. A server-side session means a request's meaning depends on state stored from a **previous** request — which also means any server instance handling a request must have access to that same session data, complicating load balancing and scaling.

[↑ Back to top](#table-of-contents)

---

### 6. How do you implement role-based access control for REST endpoints? 🟡

- Attach a role (or roles) to the authenticated user (from the token/session), and check it in middleware before the route handler runs, rejecting with 403 if the role lacks permission for that action.

```js
function requireRole(role) {
  return (req, res, next) => {
    if (req.user.role !== role) return res.sendStatus(403);
    next();
  };
}
app.delete('/users/:id', requireRole('admin'), deleteUser);
```

[↑ Back to top](#table-of-contents)

---

### 7. How should a REST API respond when a request is unauthenticated vs. unauthorized? 🟡

- Unauthenticated (no/invalid credentials): **401 Unauthorized**.
- Authenticated but lacking permission: **403 Forbidden**.
- (See [HTTP Methods & Status Codes](http-methods-and-status-codes.md#4-whats-the-difference-between-401-and-403) for the fuller distinction.)

[↑ Back to top](#table-of-contents)

---

### 8. How do you secure a REST API for machine-to-machine communication? 🔴

- No end user/browser involved, so flows centered on user login (like an interactive OAuth authorization code flow) don't apply — use **client credentials**: each service has its own client ID/secret (or a signed certificate), exchanged for an access token used for subsequent requests, with no human interaction needed.

[↑ Back to top](#table-of-contents)

---

### 9. What is API key authentication, and what are its limitations vs. OAuth? 🔴

- A long-lived static string identifying the calling application (not a specific user), sent as a header or query param. Simple to implement, but: doesn't represent a specific end user's identity/permissions, can't easily express fine-grained or time-limited scopes, and if leaked, must be manually rotated (no built-in expiry/refresh mechanism like OAuth tokens have).

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement scoped, fine-grained permissions for a REST API? 🔴

- Encode specific **scopes** (e.g. `orders:read`, `orders:write`) into the token/credentials rather than just a coarse role, and check the required scope per-endpoint — lets you grant a client exactly the access it needs (principle of least privilege) instead of all-or-nothing role-based access.

```js
function requireScope(scope) {
  return (req, res, next) => {
    if (!req.user.scopes.includes(scope)) return res.sendStatus(403);
    next();
  };
}
```

[↑ Back to top](#table-of-contents)

---

### 11. How does rate limiting interact with authentication? 🔴

- Once a request is authenticated, rate limits can be applied **per authenticated identity** (per user/API key/client) rather than just per IP — more accurate (multiple legitimate users behind the same NAT'd IP aren't unfairly limited together) and lets you offer different limits per tier/plan.

[↑ Back to top](#table-of-contents)

---

### 12. How would you design an API to support both first-party and third-party client authentication? 🔴

- **First-party** (your own frontend): can use simpler, tightly-coupled auth (session cookies, or short-lived tokens issued directly after login).
- **Third-party** (external developers/partners): needs a more formal, standardized flow — OAuth 2.0 with explicit scopes and consent, since you can't trust a third-party client the way you trust your own frontend, and users need to explicitly grant permission for what the third party can access.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is it risky to let a third-party web app store a long-lived access token directly, compared to using a short-lived token with a refresh flow?

[↑ Back to top](#table-of-contents)

---
