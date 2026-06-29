# Security Best Practices

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Why should authentication endpoints always be served over HTTPS?](#1-why-should-authentication-endpoints-always-be-served-over-https)
- [2. What is brute-force login protection, and why is it necessary?](#2-what-is-brute-force-login-protection-and-why-is-it-necessary)

**🟡 Medium**
- [3. How do you implement rate limiting on a login endpoint?](#3-how-do-you-implement-rate-limiting-on-a-login-endpoint)
- [4. What is account enumeration, and how do you prevent it?](#4-what-is-account-enumeration-and-how-do-you-prevent-it)
- [5. What is credential stuffing, and how do you defend against it?](#5-what-is-credential-stuffing-and-how-do-you-defend-against-it)
- [6. How should you handle session expiry and "remember me" functionality securely?](#6-how-should-you-handle-session-expiry-and-remember-me-functionality-securely)

**🔴 Hard**
- [7. What is session fixation, and how do you prevent it?](#7-what-is-session-fixation-and-how-do-you-prevent-it)
- [8. How do you securely implement "logout" when using JWTs?](#8-how-do-you-securely-implement-logout-when-using-jwts)
- [9. What is account lockout, and what tradeoff does it introduce?](#9-what-is-account-lockout-and-what-tradeoff-does-it-introduce)
- [10. How would you detect and respond to anomalous login behavior (impossible travel, new device)?](#10-how-would-you-detect-and-respond-to-anomalous-login-behavior-impossible-travel-new-device)
- [11. How do you securely handle authentication for a microservices architecture?](#11-how-do-you-securely-handle-authentication-for-a-microservices-architecture)
- [12. What's the security risk of long-lived access tokens, and how do short-lived tokens with refresh mitigate it?](#12-whats-the-security-risk-of-long-lived-access-tokens-and-how-do-short-lived-tokens-with-refresh-mitigate-it)

---

### 1. Why should authentication endpoints always be served over HTTPS? 🟢

- Login requests carry credentials in plaintext over the connection — without TLS, anyone on the network path (public Wi-Fi, a compromised router) can intercept them trivially. This is non-negotiable for any endpoint handling passwords or tokens.

[↑ Back to top](#table-of-contents)

---

### 2. What is brute-force login protection, and why is it necessary? 🟢

- Limiting how many login attempts can be made (per account, per IP, or both) in a given time window — without it, an attacker can simply try millions of password guesses against a login form until one succeeds, especially against weak/common passwords.

[↑ Back to top](#table-of-contents)

---

### 3. How do you implement rate limiting on a login endpoint? 🟡

- Throttle attempts both **per account** (prevents targeted brute-forcing of one user) and **per IP** (prevents broad automated abuse from one source), with escalating delays or temporary blocks after repeated failures.

```js
const rateLimit = require('express-rate-limit');
app.post('/login', rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }), loginHandler);
```

[↑ Back to top](#table-of-contents)

---

### 4. What is account enumeration, and how do you prevent it? 🟡

- When an attacker can determine **which emails/usernames have an account** by observing different responses (e.g. "user not found" vs. "wrong password"). Prevent it by returning the **same generic message** ("Invalid email or password") regardless of whether the email exists or the password was wrong, and keeping response **timing** consistent between both cases.

[↑ Back to top](#table-of-contents)

---

### 5. What is credential stuffing, and how do you defend against it? 🟡

- Attackers use **leaked username/password pairs from other breaches** (since many people reuse passwords) to automatically try logging into your service at scale. Defend with rate limiting, CAPTCHA after repeated failures, MFA, and checking new passwords against known-breached password databases at signup/change time.

[↑ Back to top](#table-of-contents)

---

### 6. How should you handle session expiry and "remember me" functionality securely? 🟡

- Use **short** default session/token lifetimes, with "remember me" issuing a separate, longer-lived (but still revocable, and ideally rotated) token rather than just extending the regular session indefinitely — so a forgotten-to-logout session on a shared/public computer doesn't stay valid indefinitely by default.

[↑ Back to top](#table-of-contents)

---

### 7. What is session fixation, and how do you prevent it? 🔴

- An attacker tricks a victim into using a **session ID the attacker already knows** (e.g. via a crafted link setting a session cookie before login) — once the victim logs in under that session, the attacker (who already has that same session ID) is now also logged in as the victim. Prevent it by **always issuing a brand-new session ID upon successful login**, never reusing whatever pre-login session ID existed.

[↑ Back to top](#table-of-contents)

---

### 8. How do you securely implement "logout" when using JWTs? 🔴

- A plain JWT can't be invalidated server-side (see [JWT](jwt.md#9-why-cant-a-jwt-be-easily-revoked-before-it-expires)) — practical approaches: keep access tokens **short-lived** so logout mostly just needs to revoke the associated **refresh token** (which is server-tracked) and clear the client-side token, accepting that the already-issued short-lived access token remains technically valid for its brief remaining lifetime.

[↑ Back to top](#table-of-contents)

---

### 9. What is account lockout, and what tradeoff does it introduce? 🔴

- Temporarily (or permanently, until manual unlock) disabling login after a threshold of failed attempts — strong protection against brute-forcing, but introduces a **denial-of-service** angle: an attacker who simply knows a victim's email can deliberately fail logins to lock the **legitimate** user out. Mitigated by combining lockout with rate limiting/CAPTCHA and progressive delays rather than an outright hard lock from a low threshold.

[↑ Back to top](#table-of-contents)

---

### 10. How would you detect and respond to anomalous login behavior (impossible travel, new device)? 🔴

- Track metadata per login (IP/geolocation, device fingerprint, user agent) and flag/challenge logins that deviate sharply from a user's established pattern — e.g. "impossible travel" (a login from a new country minutes after one from a country thousands of miles away) or a brand-new, never-seen device. Respond with an extra verification step (email confirmation, forced MFA) rather than outright blocking, to avoid false positives locking out legitimate users on a new phone/location.

[↑ Back to top](#table-of-contents)

---

### 11. How do you securely handle authentication for a microservices architecture? 🔴

- Authenticate once at the edge (an API gateway or dedicated auth service), and propagate a verified identity token internally to downstream services — each internal service should still **independently verify** the token's signature (not blindly trust an internal header claiming "this user is already authenticated"), since a compromised internal service shouldn't be able to forge identities for the rest of the system.

[↑ Back to top](#table-of-contents)

---

### 12. What's the security risk of long-lived access tokens, and how do short-lived tokens with refresh mitigate it? 🔴

- A long-lived access token, if leaked (XSS, a logged request, a compromised device), remains usable by an attacker for its **entire** remaining lifetime, with no practical way to revoke it (Q8 of [JWT](jwt.md)). Short-lived access tokens (minutes) paired with a separately-revocable refresh token shrink that exploitable window dramatically — a leaked access token expires almost immediately, and the refresh token (the only long-lived credential) can actually be revoked server-side if compromise is detected.

> [!IMPORTANT]
> **Follow-up questions:**
> - If an attacker steals both the access token AND the refresh token, does shortening the access token's lifetime still help at all?

[↑ Back to top](#table-of-contents)

---
