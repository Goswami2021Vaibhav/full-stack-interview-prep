# OAuth & OIDC

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is OAuth 2.0, and what problem does it solve?](#1-what-is-oauth-20-and-what-problem-does-it-solve)
- [2. What's the difference between OAuth and OpenID Connect (OIDC)?](#2-whats-the-difference-between-oauth-and-openid-connect-oidc)
- [3. What are the main roles/parties in an OAuth flow?](#3-what-are-the-main-roles-parties-in-an-oauth-flow)

**🟡 Medium**
- [4. What is the Authorization Code grant flow?](#4-what-is-the-authorization-code-grant-flow)
- [5. What is a scope in OAuth?](#5-what-is-a-scope-in-oauth)
- [6. What is the Client Credentials grant, and when is it used?](#6-what-is-the-client-credentials-grant-and-when-is-it-used)
- [7. What is an ID token in OIDC, and how does it differ from an access token?](#7-what-is-an-id-token-in-oidc-and-how-does-it-differ-from-an-access-token)
- [8. What is "Login with Google/GitHub" actually doing under the hood?](#8-what-is-login-with-googlegithub-actually-doing-under-the-hood)

**🔴 Hard**
- [9. Why was the Implicit grant deprecated in favor of Authorization Code + PKCE?](#9-why-was-the-implicit-grant-deprecated-in-favor-of-authorization-code--pkce)
- [10. What is PKCE, and what attack does it prevent?](#10-what-is-pkce-and-what-attack-does-it-prevent)
- [11. What is the `state` parameter in OAuth used for?](#11-what-is-the-state-parameter-in-oauth-used-for)
- [12. How does token introspection/validation work for opaque (non-JWT) access tokens?](#12-how-does-token-introspectionvalidation-work-for-opaque-non-jwt-access-tokens)
- [13. What's the difference between authentication delegation and authorization delegation in OAuth's original design intent?](#13-whats-the-difference-between-authentication-delegation-and-authorization-delegation-in-oauths-original-design-intent)

---

### 1. What is OAuth 2.0, and what problem does it solve? 🟢

- An **authorization** framework that lets a user grant a third-party application **limited access** to their resources on another service, without ever sharing their actual password with that third party (e.g. letting a photo-printing app access your Google Photos without giving it your Google password).

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between OAuth and OpenID Connect (OIDC)? 🟢

- **OAuth 2.0**: solves **authorization** — "can this app access this resource on my behalf?"
- **OIDC**: built **on top of** OAuth 2.0, adds **authentication** — "who is this user?" — by introducing a standardized **ID token** (Q7) that proves identity, which plain OAuth alone was never designed to provide reliably.

[↑ Back to top](#table-of-contents)

---

### 3. What are the main roles/parties in an OAuth flow? 🟢

- **Resource Owner**: the user.
- **Client**: the application requesting access.
- **Authorization Server**: issues tokens after the user authorizes (e.g. Google's auth server).
- **Resource Server**: the API holding the actual protected data (e.g. Google Photos' API).

[↑ Back to top](#table-of-contents)

---

### 4. What is the Authorization Code grant flow? 🟡

- The standard, most secure flow for apps with a backend: the user is redirected to the authorization server, logs in and consents, gets redirected back with a short-lived **authorization code**, which the app's **backend** then exchanges (with its client secret) for an access token — the actual token exchange never happens in the browser, keeping the client secret safe.

```
1. App redirects user to: /authorize?client_id=...&redirect_uri=...&scope=...
2. User logs in & consents
3. Auth server redirects back: /callback?code=AUTH_CODE
4. App's backend exchanges code + client secret for an access token (server-to-server call)
```

[↑ Back to top](#table-of-contents)

---

### 5. What is a scope in OAuth? 🟡

- A string representing a specific **permission** being requested (`read:photos`, `email`) — the user sees exactly what they're granting access to, and the issued token is limited to only those scopes, following the principle of least privilege.

[↑ Back to top](#table-of-contents)

---

### 6. What is the Client Credentials grant, and when is it used? 🟡

- A flow with **no end user involved at all** — a service authenticates directly with its own client ID/secret to get a token representing **itself**, used for machine-to-machine/backend-to-backend communication (see [Core Concepts](core-concepts.md#12-whats-the-difference-between-authenticating-a-user-vs-authenticating-a-servicemachine)).

[↑ Back to top](#table-of-contents)

---

### 7. What is an ID token in OIDC, and how does it differ from an access token? 🟡

- **ID token**: a JWT specifically asserting the user's **identity** (who they are) — meant to be consumed by the **client app itself**, not sent to APIs.
- **Access token**: used to actually **call** a resource server's API on the user's behalf — doesn't necessarily even need to be a JWT (could be an opaque string, Q12), and isn't meant to be parsed/trusted by the client app for identity purposes.

[↑ Back to top](#table-of-contents)

---

### 8. What is "Login with Google/GitHub" actually doing under the hood? 🟡

- It's an OIDC flow: your app redirects to Google's authorization server, the user authenticates with Google (not your app), and Google redirects back with an **ID token** asserting "this is user X, verified by Google" — your app trusts Google's assertion instead of managing a password for that user itself.

[↑ Back to top](#table-of-contents)

---

### 9. Why was the Implicit grant deprecated in favor of Authorization Code + PKCE? 🔴

- The Implicit grant returned the access token **directly in the browser's URL fragment**, with no backend exchange step — meaning the token could leak via browser history, referrer headers, or malicious scripts, and there was no way to use a client secret (since it would be exposed in a public client like a SPA). Authorization Code + PKCE (Q10) avoids exposing the token in the URL and adds protection even without a traditional client secret.

[↑ Back to top](#table-of-contents)

---

### 10. What is PKCE, and what attack does it prevent? 🔴

- **P**roof **K**ey for **C**ode **E**xchange: the client generates a random `code_verifier`, sends its hashed form (`code_challenge`) with the initial authorization request, then must present the **original** `code_verifier` when exchanging the authorization code for a token. This prevents an **authorization code interception attack** — even if an attacker intercepts the authorization code (e.g. via a malicious app on the same mobile device intercepting the redirect), they can't exchange it for a token without the original `code_verifier`, which never left the legitimate client.

[↑ Back to top](#table-of-contents)

---

### 11. What is the `state` parameter in OAuth used for? 🔴

- An opaque, random value the client generates before redirecting to the authorization server, and verifies matches when the user is redirected back — protects against **CSRF attacks on the OAuth callback** (an attacker tricking a victim into completing an OAuth flow initiated by the attacker, potentially linking the attacker's account to the victim's session).

[↑ Back to top](#table-of-contents)

---

### 12. How does token introspection/validation work for opaque (non-JWT) access tokens? 🔴

- Unlike a JWT, an opaque token carries no self-contained claims the resource server can verify on its own — the resource server must call the authorization server's **introspection endpoint**, passing the token and getting back whether it's valid and what it's scoped to. This reintroduces a network round-trip per validation (unlike stateless JWT verification), trading some performance for the ability to revoke instantly server-side.

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between authentication delegation and authorization delegation in OAuth's original design intent? 🔴

- OAuth was originally designed **purely** for authorization delegation ("let this app access my data") — using a plain OAuth access token to infer "who the user is" was a common but technically unsound misuse, since access tokens were never guaranteed to identify a specific user reliably or in a standardized way. OIDC exists specifically to formalize **authentication** delegation properly (via the ID token), rather than developers hacking identity assertions on top of raw OAuth access tokens.

> [!IMPORTANT]
> **Follow-up questions:**
> - What could go wrong if an application used a plain OAuth access token (instead of an OIDC ID token) to determine which user is logged in?

[↑ Back to top](#table-of-contents)

---
