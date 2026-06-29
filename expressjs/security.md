# Security

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is Helmet.js, and what does it do?](#1-what-is-helmetjs-and-what-does-it-do)
- [2. How do you enable CORS in an Express app?](#2-how-do-you-enable-cors-in-an-express-app)

**🟡 Medium**
- [3. How do you prevent SQL/NoSQL injection in an Express app?](#3-how-do-you-prevent-sqlnosql-injection-in-an-express-app)
- [4. How do you implement rate limiting in Express?](#4-how-do-you-implement-rate-limiting-in-express)
- [5. How do you sanitize/validate request input?](#5-how-do-you-sanitizevalidate-request-input)
- [6. How do you securely store and compare passwords?](#6-how-do-you-securely-store-and-compare-passwords)

**🔴 Hard**
- [7. How do you prevent XSS attacks in an Express app?](#7-how-do-you-prevent-xss-attacks-in-an-express-app)
- [8. How do you prevent CSRF attacks in an Express app?](#8-how-do-you-prevent-csrf-attacks-in-an-express-app)
- [9. How do you secure HTTP headers and cookies in production?](#9-how-do-you-secure-http-headers-and-cookies-in-production)
- [10. How would you protect an Express API from abusive/DDoS-style traffic?](#10-how-would-you-protect-an-express-api-from-abusivedos-style-traffic)
- [11. How do you handle secrets/environment variables securely?](#11-how-do-you-handle-secretsenvironment-variables-securely)
- [12. What security considerations apply specifically to file upload endpoints?](#12-what-security-considerations-apply-specifically-to-file-upload-endpoints)

---

### 1. What is Helmet.js, and what does it do? 🟢

- Middleware that sets a collection of security-related HTTP response headers (e.g. `X-Content-Type-Options`, `Strict-Transport-Security`, a basic `Content-Security-Policy`) with sensible defaults, in one line — reduces a class of common header-misconfiguration vulnerabilities.

```js
const helmet = require('helmet');
app.use(helmet());
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you enable CORS in an Express app? 🟢

```js
const cors = require('cors');
app.use(cors({ origin: 'https://example.com' })); // only allow this origin
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you prevent SQL/NoSQL injection in an Express app? 🟡

- Never build queries via raw string concatenation of user input — use **parameterized queries**/prepared statements (SQL) or an ORM/query builder that escapes values automatically, and for NoSQL (MongoDB), avoid passing raw `req.body` objects directly into query operators without sanitizing (e.g. stripping `$`-prefixed keys).

```js
// Vulnerable
db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);

// Safe — parameterized
db.query('SELECT * FROM users WHERE email = ?', [req.body.email]);
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you implement rate limiting in Express? 🟡

```js
const rateLimit = require('express-rate-limit');
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 })); // 100 requests per 15 min per IP
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you sanitize/validate request input? 🟡

- Use a schema validation library (`Joi`, `Zod`, `express-validator`) as middleware to reject malformed/unexpected input **before** it reaches business logic.

```js
const { body, validationResult } = require('express-validator');

app.post('/users', body('email').isEmail(), (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
  // proceed
});
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you securely store and compare passwords? 🟡

- Hash passwords with a slow, salted algorithm (`bcrypt`/`argon2`) — **never** store plaintext or use a fast general-purpose hash like plain SHA-256 (too fast to brute-force).

```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 10);
const isMatch = await bcrypt.compare(inputPassword, hash);
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you prevent XSS attacks in an Express app? 🔴

- If serving HTML, never inject unsanitized user input directly (use a templating engine's auto-escaping, or sanitize explicitly). Set a `Content-Security-Policy` header (via Helmet) restricting which script sources can execute. For a pure JSON API, the risk is mostly on the **frontend's** rendering side, not Express itself — but still escape any user content that ends up reflected into HTML anywhere.

[↑ Back to top](#table-of-contents)

---

### 8. How do you prevent CSRF attacks in an Express app? 🔴

- For cookie-based session auth: use CSRF tokens (e.g. via the `csurf`/community-maintained alternatives, since cookie-session-based apps are the primary target) and set cookies with `SameSite=Strict`/`Lax`.
- For token-based auth (JWT in an `Authorization` header, not a cookie), CSRF is largely a non-issue, since a malicious site can't read/attach another origin's `Authorization` header automatically the way browsers auto-attach cookies.

[↑ Back to top](#table-of-contents)

---

### 9. How do you secure HTTP headers and cookies in production? 🔴

- Use Helmet for headers (Q1). For cookies: set `httpOnly` (JS can't read it, mitigates XSS token theft), `secure` (only sent over HTTPS), and `sameSite` (mitigates CSRF) flags.

```js
res.cookie('token', jwt, { httpOnly: true, secure: true, sameSite: 'strict' });
```

[↑ Back to top](#table-of-contents)

---

### 10. How would you protect an Express API from abusive/DDoS-style traffic? 🔴

- Layer defenses: rate limiting (Q4) at the app level, a reverse proxy/CDN (Cloudflare, Nginx) with its own request throttling in front of the app, connection limits, and request size limits (`express.json({ limit: '1mb' })`) to prevent oversized payload abuse. True large-scale DDoS mitigation generally needs infrastructure-level protection beyond what application code alone can handle.

[↑ Back to top](#table-of-contents)

---

### 11. How do you handle secrets/environment variables securely? 🔴

- Never commit secrets to source control — load them from `.env` (via `dotenv`, gitignored) locally, and from the hosting platform's secret manager (environment config, Vault, AWS Secrets Manager) in production. Validate at startup that required secrets are actually present, failing fast instead of behaving unpredictably with `undefined` values.

[↑ Back to top](#table-of-contents)

---

### 12. What security considerations apply specifically to file upload endpoints? 🔴

- Restrict allowed file types/size **server-side** (never trust a client-side check), scan/verify actual file content (not just the extension or claimed `mimetype`), store uploads outside the publicly-served directory (or behind access control) to prevent uploaded scripts from being executed if mistakenly placed in an executable path, and generate randomized filenames to avoid path traversal/overwrite attacks (see [File Uploads](file-uploads.md)).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is checking only the file extension (e.g. `.jpg`) insufficient to confirm a file is actually a safe image?

[↑ Back to top](#table-of-contents)

---
