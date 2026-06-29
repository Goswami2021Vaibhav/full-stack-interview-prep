# JWT

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a JWT?](#1-what-is-a-jwt)
- [2. What are the three parts of a JWT?](#2-what-are-the-three-parts-of-a-jwt)
- [3. What does it mean that a JWT is "signed, not encrypted"?](#3-what-does-it-mean-that-a-jwt-is-signed-not-encrypted)

**🟡 Medium**
- [4. What are common JWT claims?](#4-what-are-common-jwt-claims)
- [5. What's the difference between `HS256` and `RS256` signing algorithms?](#5-whats-the-difference-between-hs256-and-rs256-signing-algorithms)
- [6. How do you verify a JWT on the server?](#6-how-do-you-verify-a-jwt-on-the-server)
- [7. What is a refresh token, and why is it separate from the access token?](#7-what-is-a-refresh-token-and-why-is-it-separate-from-the-access-token)
- [8. What happens if a JWT's payload is tampered with?](#8-what-happens-if-a-jwts-payload-is-tampered-with)

**🔴 Hard**
- [9. Why can't a JWT be easily revoked before it expires?](#9-why-cant-a-jwt-be-easily-revoked-before-it-expires)
- [10. What is the "alg: none" vulnerability in JWT libraries?](#10-what-is-the-alg-none-vulnerability-in-jwt-libraries)
- [11. How should you securely store a JWT on the client?](#11-how-should-you-securely-store-a-jwt-on-the-client)
- [12. How does refresh token rotation work, and what does it protect against?](#12-how-does-refresh-token-rotation-work-and-what-does-it-protect-against)
- [13. Why is it risky to put sensitive data directly in a JWT payload?](#13-why-is-it-risky-to-put-sensitive-data-directly-in-a-jwt-payload)

---

### 1. What is a JWT? 🟢

- **J**SON **W**eb **T**oken — a compact, URL-safe, self-contained token format for representing claims (e.g. user identity) between two parties, digitally signed so the receiver can verify it hasn't been tampered with.

[↑ Back to top](#table-of-contents)

---

### 2. What are the three parts of a JWT? 🟢

- `header.payload.signature`, each Base64URL-encoded and separated by dots.
  - **Header**: metadata — the signing algorithm and token type.
  - **Payload**: the actual claims (user ID, expiry, etc.).
  - **Signature**: cryptographic proof the header+payload haven't been altered.

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0MiJ9.4Adcj3UFmtMKLp1l...
└── header ──────────┘└── payload ─────┘└── signature ──┘
```

[↑ Back to top](#table-of-contents)

---

### 3. What does it mean that a JWT is "signed, not encrypted"? 🟢

- The payload is **plainly readable** by anyone (just Base64-decode it) — the signature only proves it **wasn't tampered with**, it doesn't hide the contents. Never put secrets/sensitive data directly in a standard JWT payload (Q13).

[↑ Back to top](#table-of-contents)

---

### 4. What are common JWT claims? 🟡

- `sub` (subject — typically the user ID), `iat` (issued at), `exp` (expiration time), `iss` (issuer), `aud` (audience — intended recipient), plus any custom application-specific claims (`role`, `email`).

```json
{ "sub": "42", "role": "admin", "iat": 1719600000, "exp": 1719603600 }
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `HS256` and `RS256` signing algorithms? 🟡

- **`HS256`** (symmetric): the **same** secret key both signs and verifies — simple, but anything that can verify a token can also forge one, since it knows the secret.
- **`RS256`** (asymmetric): a **private** key signs, a corresponding **public** key verifies — lets you safely distribute the public key to multiple services for verification without giving them the ability to issue new tokens themselves.

[↑ Back to top](#table-of-contents)

---

### 6. How do you verify a JWT on the server? 🟡

```js
const jwt = require('jsonwebtoken');

try {
  const payload = jwt.verify(token, SECRET_KEY); // throws if invalid/expired/tampered
  req.user = payload;
} catch (err) {
  res.status(401).json({ error: 'Invalid token' });
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What is a refresh token, and why is it separate from the access token? 🟡

- A **long-lived**, typically server-tracked (and revocable) credential used only to obtain a **new short-lived access token** — kept separate so the frequently-used access token can stay short-lived (limiting damage if leaked) without forcing the user to log in again every few minutes.

[↑ Back to top](#table-of-contents)

---

### 8. What happens if a JWT's payload is tampered with? 🟡

- The signature, computed over the **original** header+payload, no longer matches what's recomputed from the **altered** content — verification fails, and the server rejects the token, **provided** the server actually checks the signature (the failure mode in Q10 is when it doesn't).

[↑ Back to top](#table-of-contents)

---

### 9. Why can't a JWT be easily revoked before it expires? 🔴

- Verification is purely cryptographic and stateless — the server doesn't check any external "is this still valid" record by default, that's the whole point of a self-contained token. Working around this requires reintroducing some server-side state (a denylist, a refresh-token-based revocation point — see [Sessions vs. Tokens](sessions-vs-tokens.md#8-why-is-token-revocation-harder-than-session-revocation-and-how-do-systems-work-around-it)), which partially undoes the statelessness benefit.

[↑ Back to top](#table-of-contents)

---

### 10. What is the "alg: none" vulnerability in JWT libraries? 🔴

- Older/misconfigured JWT libraries would honor an `alg` value of `"none"` in the header, meaning **no signature verification happens at all** — an attacker could craft a token with arbitrary claims (e.g. `role: admin`) and a `none` algorithm, and a vulnerable server would accept it as valid. Fixed by **always** explicitly specifying which algorithms are acceptable when verifying, never trusting the `alg` field from the token itself.

```js
// Vulnerable: trusts whatever alg the token claims
jwt.verify(token, secret);

// Safe: explicitly restrict accepted algorithms
jwt.verify(token, secret, { algorithms: ['HS256'] });
```

[↑ Back to top](#table-of-contents)

---

### 11. How should you securely store a JWT on the client? 🔴

- Prefer an `httpOnly`, `secure`, `sameSite` cookie over `localStorage` — see [Sessions vs. Tokens](sessions-vs-tokens.md#10-what-storage-tradeoffs-exist-for-tokens-on-the-client-cookie-vs-localstorage) for the full XSS-vs-CSRF tradeoff between the two storage options.

[↑ Back to top](#table-of-contents)

---

### 12. How does refresh token rotation work, and what does it protect against? 🔴

- Each time a refresh token is used to get a new access token, the **refresh token itself is replaced** with a brand-new one, and the old one is invalidated. If a refresh token is ever reused after being rotated (a sign it was stolen and the attacker is using a copy), the server detects the reuse and can revoke the **entire** token family — protecting against a stolen refresh token being silently reused alongside the legitimate user's.

[↑ Back to top](#table-of-contents)

---

### 13. Why is it risky to put sensitive data directly in a JWT payload? 🔴

- The payload is **not encrypted** (Q3) — anyone who intercepts or even just has a copy of the token (e.g. visible in browser dev tools, logs) can read every claim inside it. Only include data that's safe to be publicly readable by the token holder; never put passwords, full PII, or secrets directly in the claims.

> [!IMPORTANT]
> **Follow-up questions:**
> - If you genuinely needed to keep claims confidential from the token holder itself, what would you reach for instead of a standard signed JWT (hint: JWE)?

[↑ Back to top](#table-of-contents)

---
