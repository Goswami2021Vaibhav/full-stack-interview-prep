# SSO & MFA

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is Single Sign-On (SSO)?](#1-what-is-single-sign-on-sso)
- [2. What is Multi-Factor Authentication (MFA)?](#2-what-is-multi-factor-authentication-mfa)
- [3. What is a one-time password (OTP)?](#3-what-is-a-one-time-password-otp)

**🟡 Medium**
- [4. What's the difference between TOTP and SMS-based OTP?](#4-whats-the-difference-between-totp-and-sms-based-otp)
- [5. What is an Identity Provider (IdP) vs. a Service Provider (SP) in SSO?](#5-what-is-an-identity-provider-idp-vs-a-service-provider-sp-in-sso)
- [6. What is SAML, and how does it relate to SSO?](#6-what-is-saml-and-how-does-it-relate-to-sso)
- [7. Why is SMS-based MFA considered weaker than an authenticator app?](#7-why-is-sms-based-mfa-considered-weaker-than-an-authenticator-app)
- [8. What are backup/recovery codes, and why are they necessary with MFA?](#8-what-are-backuprecovery-codes-and-why-are-they-necessary-with-mfa)

**🔴 Hard**
- [9. How does TOTP actually generate a time-based code?](#9-how-does-totp-actually-generate-a-time-based-code)
- [10. What is a hardware security key (e.g. FIDO2/WebAuthn), and why is it considered phishing-resistant?](#10-what-is-a-hardware-security-key-eg-fido2webauthn-and-why-is-it-considered-phishing-resistant)
- [11. How does SSO across multiple independent organizations (federation) actually establish trust?](#11-how-does-sso-across-multiple-independent-organizations-federation-actually-establish-trust)

---

### 1. What is Single Sign-On (SSO)? 🟢

- Authenticate **once** with a central identity provider, and gain access to **multiple** separate applications/services without logging in again to each one — common within an organization (one corporate login for email, Slack, internal tools, etc.).

[↑ Back to top](#table-of-contents)

---

### 2. What is Multi-Factor Authentication (MFA)? 🟢

- Requiring **two or more** different categories of credential to authenticate (see [Core Concepts](core-concepts.md#2-what-are-the-common-factors-used-in-authentication)) — typically a password plus a time-limited code from a phone/app, significantly raising the bar for an attacker who's only obtained the password.

[↑ Back to top](#table-of-contents)

---

### 3. What is a one-time password (OTP)? 🟢

- A short numeric code valid for only a single use (or a short time window) — delivered via SMS, email, or generated locally by an authenticator app, used as a second factor in MFA.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between TOTP and SMS-based OTP? 🟡

- **TOTP** (Time-based One-Time Password): generated **locally** on the user's device (an authenticator app) from a shared secret and the current time — no network transmission involved at all.
- **SMS-based OTP**: the server generates a code and sends it over the cellular network — introduces dependency on (and vulnerability via) the telecom network (Q7).

[↑ Back to top](#table-of-contents)

---

### 5. What is an Identity Provider (IdP) vs. a Service Provider (SP) in SSO? 🟡

- **Identity Provider (IdP)**: the system that actually authenticates the user and asserts their identity (e.g. Okta, Azure AD, Google Workspace).
- **Service Provider (SP)**: the application the user wants to access, which **trusts** the IdP's assertion instead of authenticating the user itself (e.g. your company's internal CRM tool).

[↑ Back to top](#table-of-contents)

---

### 6. What is SAML, and how does it relate to SSO? 🟡

- **S**ecurity **A**ssertion **M**arkup **L**anguage — an older, XML-based standard for exchanging authentication/authorization assertions between an IdP and SP, widely used for enterprise SSO. OIDC (see [OAuth & OIDC](oauth-and-oidc.md)) is the more modern, JSON/REST-friendly alternative serving a similar role, increasingly preferred for new integrations.

[↑ Back to top](#table-of-contents)

---

### 7. Why is SMS-based MFA considered weaker than an authenticator app? 🟡

- Vulnerable to **SIM-swapping** (an attacker socially engineers the carrier into porting the victim's number to a device they control) and **SS7 protocol interception** — both let an attacker receive the OTP without ever touching the victim's actual phone. A locally-generated TOTP code has no such network-level interception point.

[↑ Back to top](#table-of-contents)

---

### 8. What are backup/recovery codes, and why are they necessary with MFA? 🟡

- A set of single-use codes generated at MFA setup time, meant to be stored somewhere safe — used to regain account access if the user loses their primary MFA device (phone lost/broken), preventing a legitimate user from being permanently locked out.

[↑ Back to top](#table-of-contents)

---

### 9. How does TOTP actually generate a time-based code? 🔴

- At setup, the server and the user's app share a secret (often via a QR code). Both independently compute `HMAC(secret, current_time_step)` — where the time step is the current Unix time divided into fixed intervals (typically 30 seconds) — and derive a short numeric code from the result. Since both sides compute the same function from the same secret and (roughly) the same time, the codes match without any network communication, and naturally rotate every interval.

[↑ Back to top](#table-of-contents)

---

### 10. What is a hardware security key (e.g. FIDO2/WebAuthn), and why is it considered phishing-resistant? 🔴

- A physical device (or platform authenticator, like a laptop's fingerprint sensor) that performs **public-key cryptographic** authentication tied to the **specific origin/domain** that registered it. Unlike a password or OTP — which a user can be tricked into typing into a fake/phishing site — a WebAuthn credential simply **won't work** against the wrong domain at all, since the cryptographic challenge-response is bound to the legitimate origin from the start.

[↑ Back to top](#table-of-contents)

---

### 11. How does SSO across multiple independent organizations (federation) actually establish trust? 🔴

- Each Service Provider is pre-configured to **trust a specific Identity Provider's signing key** (exchanged out-of-band during setup, often via metadata files) — when the IdP issues a signed assertion (SAML assertion or OIDC ID token), the SP verifies the signature against that pre-trusted key. Trust isn't automatic or discovered dynamically; it's explicitly established once between the two parties' administrators before any user ever logs in.

> [!IMPORTANT]
> **Follow-up questions:**
> - What would happen to existing SSO sessions/trust if an IdP's signing key were rotated without coordinating with every registered Service Provider?

[↑ Back to top](#table-of-contents)

---
