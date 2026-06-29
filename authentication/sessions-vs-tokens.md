# Sessions vs Tokens

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is session-based authentication?](#1-what-is-session-based-authentication)
- [2. What is token-based authentication?](#2-what-is-token-based-authentication)
- [3. What's the role of a cookie in session-based authentication?](#3-whats-the-role-of-a-cookie-in-session-based-authentication)

**🟡 Medium**
- [4. What are the main tradeoffs between sessions and tokens?](#4-what-are-the-main-tradeoffs-between-sessions-and-tokens)
- [5. How do you scale session-based authentication across multiple servers?](#5-how-do-you-scale-session-based-authentication-across-multiple-servers)
- [6. How do you revoke a session vs. revoking a token?](#6-how-do-you-revoke-a-session-vs-revoking-a-token)
- [7. What's the difference between a stateful and a stateless authentication model?](#7-whats-the-difference-between-a-stateful-and-a-stateless-authentication-model)

**🔴 Hard**
- [8. Why is token revocation harder than session revocation, and how do systems work around it?](#8-why-is-token-revocation-harder-than-session-revocation-and-how-do-systems-work-around-it)
- [9. How would you implement a hybrid approach combining sessions and tokens?](#9-how-would-you-implement-a-hybrid-approach-combining-sessions-and-tokens)
- [10. What storage tradeoffs exist for tokens on the client (cookie vs. localStorage)?](#10-what-storage-tradeoffs-exist-for-tokens-on-the-client-cookie-vs-localstorage)
- [11. How do you decide between sessions and tokens for a new project?](#11-how-do-you-decide-between-sessions-and-tokens-for-a-new-project)

---

### 1. What is session-based authentication? 🟢

- After login, the server creates a **session** (stored server-side, e.g. in memory or Redis) and gives the client a **session ID** (typically via a cookie). On each request, the client sends the session ID, and the server looks up the corresponding session data to know who's making the request.

[↑ Back to top](#table-of-contents)

---

### 2. What is token-based authentication? 🟢

- After login, the server issues a **self-contained token** (typically a JWT) containing the user's identity/claims, signed so it can't be tampered with. The client sends this token with each request, and the server **verifies** it cryptographically — no server-side lookup needed.

[↑ Back to top](#table-of-contents)

---

### 3. What's the role of a cookie in session-based authentication? 🟢

- The cookie carries the **session ID** (an opaque reference, not the actual data) — the browser automatically attaches it to every request to the same domain, so the client doesn't need to manually manage it.

[↑ Back to top](#table-of-contents)

---

### 4. What are the main tradeoffs between sessions and tokens? 🟡

- **Sessions**: easy to revoke instantly (just delete server-side record), but require server-side storage shared across instances when scaling horizontally.
- **Tokens**: scale easily (any server verifies independently, no shared store needed), but are hard to revoke before expiry since the token itself remains valid until it expires (Q8).

[↑ Back to top](#table-of-contents)

---

### 5. How do you scale session-based authentication across multiple servers? 🟡

- Store sessions in a **shared** store (Redis, a database) accessible by every server instance, instead of in each server's own memory — otherwise a user's session would only be recognized by whichever specific instance created it, breaking as soon as a load balancer routes them to a different instance.

[↑ Back to top](#table-of-contents)

---

### 6. How do you revoke a session vs. revoking a token? 🟡

- **Session**: trivial — delete the session record server-side; the session ID immediately becomes invalid on the next lookup.
- **Token**: not directly possible without extra infrastructure — a signed JWT remains cryptographically valid until it expires, **regardless** of server-side state, unless you maintain a denylist/check a revocation list on every request (which partially reintroduces the server-side lookup tokens were meant to avoid).

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between a stateful and a stateless authentication model? 🟡

- **Stateful** (sessions): the server must remember something between requests to recognize the user.
- **Stateless** (tokens): each request carries everything needed to verify the user, with no server-side memory required — see [REST API › Authentication & Authorization](../rest-api/authentication-and-authorization.md#4-whats-the-difference-between-session-based-and-token-based-authentication-for-rest-apis) for how this maps onto REST API design specifically.

[↑ Back to top](#table-of-contents)

---

### 8. Why is token revocation harder than session revocation, and how do systems work around it? 🔴

- A signed token is self-validating — the server doesn't consult any external state to trust it, which is precisely what makes it stateless and scalable, but also means there's no natural place to "turn it off" early.
- Common workarounds: keep tokens **short-lived** (minutes, not days) so a compromised token has a small exploitable window, paired with a **refresh token** (often stored server-side and revocable) to issue new short-lived access tokens — revoking the refresh token effectively ends the session without needing a denylist of every issued access token.

[↑ Back to top](#table-of-contents)

---

### 9. How would you implement a hybrid approach combining sessions and tokens? 🔴

- Use short-lived, stateless **access tokens** for fast, scalable per-request verification, backed by a **server-side-tracked refresh token** (essentially a session) used only occasionally to mint new access tokens — gets most of the scalability benefit of tokens while retaining a real revocation point via the refresh token.

[↑ Back to top](#table-of-contents)

---

### 10. What storage tradeoffs exist for tokens on the client (cookie vs. localStorage)? 🔴

- **`httpOnly` cookie**: not accessible to JS, so immune to token theft via XSS — but vulnerable to CSRF unless mitigated (`SameSite`, CSRF tokens), and auto-attached by the browser to every matching-domain request, including ones you might not want it on.
- **`localStorage`**: not auto-sent (must be attached manually to each request, avoiding CSRF concerns by default) — but fully readable by any JS running on the page, so a single XSS vulnerability anywhere on the site can exfiltrate the token entirely.

[↑ Back to top](#table-of-contents)

---

### 11. How do you decide between sessions and tokens for a new project? 🔴

- **Sessions**: simpler to reason about, easy instant revocation, fine for a monolith/traditional web app where you control the server and don't need the scaling benefits tokens offer.
- **Tokens**: better fit for distributed systems, multiple services/APIs needing to verify identity independently, mobile/SPA clients, or when you specifically need stateless horizontal scaling — at the cost of more complex revocation handling (Q8/Q9).

> [!IMPORTANT]
> **Follow-up questions:**
> - For an API consumed by both a web frontend and a mobile app, would you reach for sessions, tokens, or a mix — and why might the answer differ per client type?

[↑ Back to top](#table-of-contents)

---
