# Error Handling

_Part of [REST API](README.md) interview notes._

> This file covers error response **design** for REST APIs. Express-specific error-handling middleware mechanics are covered in [Express.js › Error Handling](../expressjs/error-handling.md).

## Table of Contents

**🟢 Easy**
- [1. What makes a good REST API error response?](#1-what-makes-a-good-rest-api-error-response)
- [2. Should error details be in the response body or just the status code?](#2-should-error-details-be-in-the-response-body-or-just-the-status-code)

**🟡 Medium**
- [3. How do you design a consistent error response format across an API?](#3-how-do-you-design-a-consistent-error-response-format-across-an-api)
- [4. How do you handle validation errors with multiple field-level issues?](#4-how-do-you-handle-validation-errors-with-multiple-field-level-issues)
- [5. What information should NOT be included in an error response?](#5-what-information-should-not-be-included-in-an-error-response)
- [6. How do you communicate rate-limit errors to a client?](#6-how-do-you-communicate-rate-limit-errors-to-a-client)

**🔴 Hard**
- [7. How do you design stable error codes clients can branch on programmatically?](#7-how-do-you-design-stable-error-codes-clients-can-branch-on-programmatically)
- [8. How should an API handle partial failures in a batch/bulk request?](#8-how-should-an-api-handle-partial-failures-in-a-batchbulk-request)
- [9. How do you version error response formats without breaking existing clients?](#9-how-do-you-version-error-response-formats-without-breaking-existing-clients)
- [10. How would you design error responses to support internationalization?](#10-how-would-you-design-error-responses-to-support-internationalization)
- [11. What's the difference between client and server errors in terms of expected retry behavior?](#11-whats-the-difference-between-client-and-server-errors-in-terms-of-expected-retry-behavior)
- [12. How do you avoid leaking internal implementation details in error responses?](#12-how-do-you-avoid-leaking-internal-implementation-details-in-error-responses)

---

### 1. What makes a good REST API error response? 🟢

- A correct HTTP **status code**, a clear human-readable message, and (ideally) a machine-readable error **code** the client can branch on — consistent in structure across every endpoint in the API.

[↑ Back to top](#table-of-contents)

---

### 2. Should error details be in the response body or just the status code? 🟢

- Both — the status code tells generic HTTP tooling (and a quick glance) whether something failed, while the body gives the **specific** reason, which the status code alone can't fully express (e.g. *which* field failed validation).

[↑ Back to top](#table-of-contents)

---

### 3. How do you design a consistent error response format across an API? 🟡

- Use one fixed shape for every error response, regardless of which endpoint produced it.

```json
{ "error": { "code": "VALIDATION_ERROR", "message": "Invalid request", "details": [] } }
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you handle validation errors with multiple field-level issues? 🟡

- Return all failing fields in **one** response (not just the first one found), each with its own message — lets the client display/fix everything at once instead of round-tripping per field.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "details": [
      { "field": "email", "message": "Must be a valid email" },
      { "field": "age", "message": "Must be a positive number" }
    ]
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What information should NOT be included in an error response? 🟡

- Stack traces, raw database error messages, internal file paths, or any detail revealing implementation internals — these aid attackers and leak unnecessary information; log them server-side, but keep the client-facing message generic and safe (see Q12).

[↑ Back to top](#table-of-contents)

---

### 6. How do you communicate rate-limit errors to a client? 🟡

- **429 Too Many Requests**, ideally with a `Retry-After` header and/or `X-RateLimit-Remaining`/`X-RateLimit-Reset` headers so the client knows exactly when it's safe to retry.

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you design stable error codes clients can branch on programmatically? 🔴

- Use a fixed **string code** (`"USER_NOT_FOUND"`, `"INSUFFICIENT_FUNDS"`) separate from the human-readable `message` — clients should branch on the **code**, never on parsing the message text, since messages may change wording (or be localized, Q10) without that counting as a breaking change.

[↑ Back to top](#table-of-contents)

---

### 8. How should an API handle partial failures in a batch/bulk request? 🔴

- Return a per-item result so the client can see exactly which items succeeded and which failed and why, rather than failing the entire batch for one bad item (or silently dropping failures) — see [HTTP Methods & Status Codes](http-methods-and-status-codes.md#13-how-do-you-choose-a-status-code-for-a-bulk-operation-where-some-items-succeed-and-some-fail) for the status-code angle.

[↑ Back to top](#table-of-contents)

---

### 9. How do you version error response formats without breaking existing clients? 🔴

- Treat the error response shape itself as part of the API contract — changing it is a breaking change, subject to the same versioning rules as any other response (see [Versioning](versioning.md)). Adding **new optional** fields to an error object is safe; renaming/removing existing fields (like `code` or `message`) is not.

[↑ Back to top](#table-of-contents)

---

### 10. How would you design error responses to support internationalization? 🔴

- Keep the stable error **code** as the primary machine-readable signal (Q7), and let the **client** (or a translation layer) map that code to a localized message based on the request's `Accept-Language` — don't bake a single hardcoded English message into the contract as the only signal.

```json
{ "error": { "code": "INSUFFICIENT_FUNDS", "message": "अपर्याप्त शेष राशि" } }
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between client and server errors in terms of expected retry behavior? 🔴

- **4xx (client error)**: the request itself was the problem — retrying the **exact same** request will fail again; the client must fix something first (different input, valid credentials).
- **5xx (server error)**: the request was probably fine — the failure was on the server's end, so retrying (often with backoff) may succeed once the underlying issue clears.

[↑ Back to top](#table-of-contents)

---

### 12. How do you avoid leaking internal implementation details in error responses? 🔴

- Catch errors centrally (one place, not scattered per-route), log full details server-side, and map internal exceptions to a small, deliberate set of safe, generic client-facing error codes/messages — never let a raw caught exception's message flow directly into the response body unfiltered.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is it dangerous to return a raw database constraint-violation error message directly to an API client?

[↑ Back to top](#table-of-contents)

---
