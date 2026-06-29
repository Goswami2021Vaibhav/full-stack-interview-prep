# Core Concepts

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between authentication and authorization?](#1-whats-the-difference-between-authentication-and-authorization)
- [2. What are the common factors used in authentication?](#2-what-are-the-common-factors-used-in-authentication)
- [3. What is identity in the context of authentication?](#3-what-is-identity-in-the-context-of-authentication)

**🟡 Medium**
- [4. What's the difference between authentication and identification?](#4-whats-the-difference-between-authentication-and-identification)
- [5. What is single-factor vs. multi-factor authentication?](#5-what-is-single-factor-vs-multi-factor-authentication)
- [6. What is a credential, and what forms can it take?](#6-what-is-a-credential-and-what-forms-can-it-take)
- [7. What's the difference between client-side and server-side authentication state?](#7-whats-the-difference-between-client-side-and-server-side-authentication-state)
- [8. What is delegated authentication?](#8-what-is-delegated-authentication)

**🔴 Hard**
- [9. What's the difference between authentication and federation?](#9-whats-the-difference-between-authentication-and-federation)
- [10. What does "trust boundary" mean in an authentication system?](#10-what-does-trust-boundary-mean-in-an-authentication-system)
- [11. How does authentication interact with REST's stateless constraint?](#11-how-does-authentication-interact-with-rests-stateless-constraint)
- [12. What's the difference between authenticating a user vs. authenticating a service/machine?](#12-whats-the-difference-between-authenticating-a-user-vs-authenticating-a-servicemachine)
- [13. What is the principle of "defense in depth" as applied to authentication systems?](#13-what-is-the-principle-of-defense-in-depth-as-applied-to-authentication-systems)

---

### 1. What's the difference between authentication and authorization? 🟢

- **Authentication (AuthN)**: confirming **who you are** — verifying an identity claim (e.g. via a password, a token).
- **Authorization (AuthZ)**: confirming **what you're allowed to do** once your identity is known — checking permissions/roles/scopes.

[↑ Back to top](#table-of-contents)

---

### 2. What are the common factors used in authentication? 🟢

- **Something you know**: a password, a PIN.
- **Something you have**: a phone (OTP/authenticator app), a hardware security key.
- **Something you are**: biometrics (fingerprint, face). Combining factors from **different** categories is what makes multi-factor authentication meaningfully stronger (Q5).

[↑ Back to top](#table-of-contents)

---

### 3. What is identity in the context of authentication? 🟢

- The set of attributes (a username, an email, a unique ID) that represent **who** a user/service claims to be. Authentication is the process of verifying that an identity claim is genuinely backed by valid credentials.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between authentication and identification? 🟡

- **Identification**: simply claiming an identity (typing a username/email) — anyone can claim to be anyone.
- **Authentication**: actually **proving** that claim is true, via a credential only the real identity holder should possess.

[↑ Back to top](#table-of-contents)

---

### 5. What is single-factor vs. multi-factor authentication? 🟡

- **Single-factor**: only one category of proof (e.g. just a password).
- **Multi-factor (MFA)**: two or more **different categories** of proof (e.g. a password **plus** a one-time code from a phone) — using two passwords would still only be single-factor, since both come from the "something you know" category. See [SSO & MFA](sso-and-mfa.md) for implementation details.

[↑ Back to top](#table-of-contents)

---

### 6. What is a credential, and what forms can it take? 🟡

- Any artifact used to prove an identity claim — a password, a cryptographic key pair, a signed token (JWT), a certificate, or a biometric template. Different credential types carry different tradeoffs in security, revocability, and user experience.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between client-side and server-side authentication state? 🟡

- **Server-side state**: the server stores session data (a session ID maps to user info in a server-side store) — the client just holds a reference (a session cookie).
- **Client-side state**: the credential itself (e.g. a JWT) carries all necessary claims — the server verifies it cryptographically without needing to look anything up. See [Sessions vs. Tokens](sessions-vs-tokens.md) for the full comparison.

[↑ Back to top](#table-of-contents)

---

### 8. What is delegated authentication? 🟡

- Letting a **trusted third party** (Google, an identity provider) handle the actual authentication, rather than your own app managing passwords directly — your app trusts that third party's assertion of identity. OAuth/OIDC (see [OAuth & OIDC](oauth-and-oidc.md)) are the standard mechanisms for this.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between authentication and federation? 🔴

- **Authentication**: verifying identity within a **single** system/domain.
- **Federation**: extending trust **across** systems/organizations — a user authenticates once with their home identity provider, and multiple separate, independently-operated services trust that single authentication (the basis of SSO across organizational boundaries — see [SSO & MFA](sso-and-mfa.md)).

[↑ Back to top](#table-of-contents)

---

### 10. What does "trust boundary" mean in an authentication system? 🔴

- The line separating components that trust each other implicitly from those that must verify each other's claims explicitly. E.g. inside your own backend, services might trust an already-validated `req.user` object; but at the boundary where an external client's request first enters, nothing is trusted until the credential is verified — every request crossing that boundary needs independent verification.

[↑ Back to top](#table-of-contents)

---

### 11. How does authentication interact with REST's stateless constraint? 🔴

- Token-based auth (a self-contained credential like a JWT, verified independently per request) aligns naturally with statelessness — see [REST API › Authentication & Authorization](../rest-api/authentication-and-authorization.md#5-why-does-storing-session-state-on-the-server-conflict-with-rests-stateless-constraint) for the deeper tradeoff between session-based and token-based approaches in a stateless API context.

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between authenticating a user vs. authenticating a service/machine? 🔴

- **User authentication**: typically interactive — a login form, MFA, often issuing a relatively short-lived credential tied to a human session.
- **Machine-to-machine authentication**: no human in the loop — uses long-lived client credentials (API keys, client certificates, OAuth client-credentials grant) exchanged programmatically, since there's no one available to type a password or approve an MFA prompt.

[↑ Back to top](#table-of-contents)

---

### 13. What is the principle of "defense in depth" as applied to authentication systems? 🔴

- No single control should be the **only** thing standing between an attacker and account compromise — combine multiple independent layers (strong password hashing, MFA, rate limiting on login attempts, anomaly detection, session expiry) so that a failure or bypass of any **one** layer doesn't immediately lead to a full compromise.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is relying on password strength alone, with no MFA and no rate limiting, considered an insufficient authentication design today?

[↑ Back to top](#table-of-contents)

---
