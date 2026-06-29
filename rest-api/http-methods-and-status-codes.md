# HTTP Methods & Status Codes

_Part of [REST API](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What HTTP methods are commonly used in REST APIs?](#1-what-http-methods-are-commonly-used-in-rest-apis)
- [2. What's the difference between PUT and PATCH?](#2-whats-the-difference-between-put-and-patch)
- [3. What do the HTTP status code ranges mean?](#3-what-do-the-http-status-code-ranges-mean)

**🟡 Medium**
- [4. What's the difference between 401 and 403?](#4-whats-the-difference-between-401-and-403)
- [5. What's the difference between 200, 201, and 204?](#5-whats-the-difference-between-200-201-and-204)
- [6. What's the difference between 404 and 410?](#6-whats-the-difference-between-404-and-410)
- [7. When should you use 400 vs. 422?](#7-when-should-you-use-400-vs-422)
- [8. What's the difference between using PUT and POST for an update?](#8-whats-the-difference-between-using-put-and-post-for-an-update)

**🔴 Hard**
- [9. Why is returning 200 for every response, even errors, considered bad practice?](#9-why-is-returning-200-for-every-response-even-errors-considered-bad-practice)
- [10. What's the difference between 301, 302, 307, and 308 redirects?](#10-whats-the-difference-between-301-302-307-and-308-redirects)
- [11. How should a DELETE request behave when the resource doesn't exist — 404 or 204?](#11-how-should-a-delete-request-behave-when-the-resource-doesnt-exist--404-or-204)
- [12. What's the difference between 500 and 503?](#12-whats-the-difference-between-500-and-503)
- [13. How do you choose a status code for a bulk operation where some items succeed and some fail?](#13-how-do-you-choose-a-status-code-for-a-bulk-operation-where-some-items-succeed-and-some-fail)
- [14. What's the purpose of the `Retry-After` header?](#14-whats-the-purpose-of-the-retry-after-header)

---

### 1. What HTTP methods are commonly used in REST APIs? 🟢

- `GET` (read), `POST` (create), `PUT` (replace/update), `PATCH` (partial update), `DELETE` (remove). Less commonly: `HEAD` (like GET, no body — just headers) and `OPTIONS` (discover allowed methods, used in CORS preflight).

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between PUT and PATCH? 🟢

- `PUT`: replaces the **entire** resource with the provided representation — fields not included are typically treated as removed/reset.
- `PATCH`: applies a **partial** update — only the included fields are changed, the rest stay as-is.

```http
PUT /users/42    { "name": "Vaibhav", "email": "v@x.com" }   // full replacement
PATCH /users/42  { "email": "new@x.com" }                     // only updates email
```

[↑ Back to top](#table-of-contents)

---

### 3. What do the HTTP status code ranges mean? 🟢

- **1xx**: informational (rare in REST APIs).
- **2xx**: success.
- **3xx**: redirection.
- **4xx**: client error (the request itself was wrong).
- **5xx**: server error (the request was fine, but the server failed).

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between 401 and 403? 🟡

- **401 Unauthorized**: you're **not authenticated** — the server doesn't know who you are (missing/invalid credentials).
- **403 Forbidden**: you **are** authenticated, but you're **not allowed** to perform this action (insufficient permissions).

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between 200, 201, and 204? 🟡

- **200 OK**: generic success, response has a body.
- **201 Created**: a new resource was created (typically returned by `POST`), often with a `Location` header pointing to the new resource.
- **204 No Content**: success, but there's intentionally no response body (common for `DELETE` or some `PUT` operations).

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between 404 and 410? 🟡

- **404 Not Found**: the resource doesn't exist (or never did) at this URL — ambiguous about whether it's permanent.
- **410 Gone**: the resource **used to exist** but has been permanently removed — a stronger, more specific signal than 404 that clients shouldn't expect it to come back.

[↑ Back to top](#table-of-contents)

---

### 7. When should you use 400 vs. 422? 🟡

- **400 Bad Request**: the request is malformed at a structural level (invalid JSON, missing required field, wrong data type).
- **422 Unprocessable Entity**: the request is well-formed and parses fine, but fails **semantic/business-rule** validation (e.g. an end date earlier than the start date).

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between using PUT and POST for an update? 🟡

- `POST` to a collection (`POST /users`) typically **creates** a new resource and is not idempotent — calling it twice creates two resources.
- `PUT` to a specific resource (`PUT /users/42`) **replaces** that resource and is idempotent — calling it twice with the same body leaves the resource in the same final state.

[↑ Back to top](#table-of-contents)

---

### 9. Why is returning 200 for every response, even errors, considered bad practice? 🔴

- It defeats the purpose of HTTP status codes as a **machine-readable** signal — clients/proxies/monitoring tools rely on the status code (not the body) to distinguish success from failure at a glance (e.g. for retries, alerting, caching decisions). Always returning 200 forces every caller to parse the body just to know if something went wrong, and breaks generic HTTP tooling that assumes status codes are meaningful.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between 301, 302, 307, and 308 redirects? 🔴

- **301 Moved Permanently** / **308 Permanent Redirect**: the resource has permanently moved — clients should update bookmarks/links. 308 explicitly preserves the original HTTP method; legacy 301 implementations sometimes changed `POST` to `GET` on redirect.
- **302 Found** / **307 Temporary Redirect**: temporary — don't update bookmarks. 307 explicitly preserves the method; 302 historically had the same method-changing ambiguity as 301.
- In modern API design, 307/308 are preferred over 301/302 when method preservation matters.

[↑ Back to top](#table-of-contents)

---

### 11. How should a DELETE request behave when the resource doesn't exist — 404 or 204? 🔴

- Common convention: return **404** if it never existed (or already gone, and you want to be explicit), or **204** to keep `DELETE` cleanly idempotent (deleting something already deleted is still "successfully in the deleted state"). Both are defensible — the key is to be **consistent** across your API, and document the choice clearly.

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between 500 and 503? 🔴

- **500 Internal Server Error**: a generic, unexpected failure on the server (a bug, an unhandled exception).
- **503 Service Unavailable**: the server is **temporarily** unable to handle the request (overloaded, in maintenance, a downstream dependency is down) — often paired with a `Retry-After` header (Q14) to tell the client when to try again.

[↑ Back to top](#table-of-contents)

---

### 13. How do you choose a status code for a bulk operation where some items succeed and some fail? 🔴

- There's no single perfect HTTP status for "partial success" — common approaches: return **207 Multi-Status** (borrowed from WebDAV) with a per-item breakdown in the body, or return **200**/**201** with a body listing which items succeeded/failed and why, letting the client inspect per-item results rather than relying on one top-level status code for a mixed outcome.

```json
{
  "results": [
    { "id": 1, "status": "success" },
    { "id": 2, "status": "failed", "error": "Invalid email" }
  ]
}
```

[↑ Back to top](#table-of-contents)

---

### 14. What's the purpose of the `Retry-After` header? 🔴

- Tells the client how long to wait before retrying — used alongside **503** (temporary unavailability) or **429 Too Many Requests** (rate limiting), as either a number of seconds or an HTTP date.

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might a 429 response with no `Retry-After` header lead to poorly-behaved client retry logic?

[↑ Back to top](#table-of-contents)

---
