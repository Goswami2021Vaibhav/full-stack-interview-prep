# Password Security

_Part of [Authentication](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Why should you never store passwords in plaintext?](#1-why-should-you-never-store-passwords-in-plaintext)
- [2. What is password hashing?](#2-what-is-password-hashing)
- [3. What is a salt, and why is it needed?](#3-what-is-a-salt-and-why-is-it-needed)

**🟡 Medium**
- [4. Why is a fast hash like SHA-256 a bad choice for password hashing?](#4-why-is-a-fast-hash-like-sha-256-a-bad-choice-for-password-hashing)
- [5. What is `bcrypt`, and how does it work?](#5-what-is-bcrypt-and-how-does-it-work)
- [6. What is a rainbow table attack, and how does salting prevent it?](#6-what-is-a-rainbow-table-attack-and-how-does-salting-prevent-it)
- [7. How do you securely compare a submitted password against a stored hash?](#7-how-do-you-securely-compare-a-submitted-password-against-a-stored-hash)
- [8. What makes a good password policy?](#8-what-makes-a-good-password-policy)

**🔴 Hard**
- [9. What's the difference between `bcrypt`, `scrypt`, and `argon2`?](#9-whats-the-difference-between-bcrypt-scrypt-and-argon2)
- [10. What is a timing attack, and how does it relate to password comparison?](#10-what-is-a-timing-attack-and-how-does-it-relate-to-password-comparison)
- [11. How would you implement a secure "forgot password" flow?](#11-how-would-you-implement-a-secure-forgot-password-flow)
- [12. How do you handle a confirmed password database breach?](#12-how-do-you-handle-a-confirmed-password-database-breach)

---

### 1. Why should you never store passwords in plaintext? 🟢

- If the database is ever breached (or even just viewed by an insider), plaintext passwords are immediately and fully compromised for **every** user — and since people reuse passwords across sites, that exposure cascades to their accounts elsewhere too.

[↑ Back to top](#table-of-contents)

---

### 2. What is password hashing? 🟢

- Running the password through a **one-way** function before storing it — you can verify a future login attempt by hashing the input and comparing, but you can't reverse the stored hash back into the original password.

[↑ Back to top](#table-of-contents)

---

### 3. What is a salt, and why is it needed? 🟢

- A random value generated per-password and combined with it **before** hashing, then stored alongside the hash. Without a salt, two users with the same password would produce the **identical** hash — a salt ensures every hash is unique even for identical passwords, defeating precomputed lookup attacks (Q6).

[↑ Back to top](#table-of-contents)

---

### 4. Why is a fast hash like SHA-256 a bad choice for password hashing? 🟡

- General-purpose hashes are designed to be **fast** (useful for checksums, Q in [Node.js › Modules](../nodejs/modules.md#12-how-do-you-generate-and-verify-a-checksum-of-a-string-in-nodejs)) — but that same speed lets an attacker who steals the hash database try **billions** of password guesses per second on cheap hardware (especially GPUs). Password hashing needs to be **deliberately slow** to make brute-forcing impractical.

[↑ Back to top](#table-of-contents)

---

### 5. What is `bcrypt`, and how does it work? 🟡

- A password-hashing algorithm built to be **slow and tunable** — it takes a **cost factor** (work factor) controlling how many rounds of internal hashing it performs, so you can deliberately increase the computational cost as hardware gets faster over time. It also automatically handles salting internally.

```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12); // cost factor of 12
const isMatch = await bcrypt.compare(inputPassword, hash);
```

[↑ Back to top](#table-of-contents)

---

### 6. What is a rainbow table attack, and how does salting prevent it? 🟡

- A rainbow table is a **precomputed** lookup of hash → plaintext pairs for common passwords, letting an attacker reverse a stolen hash instantly without brute-forcing. A unique salt per password means the attacker would need a **separate** precomputed table per salt — making precomputation practically useless against a properly salted password database.

[↑ Back to top](#table-of-contents)

---

### 7. How do you securely compare a submitted password against a stored hash? 🟡

- Use the hashing library's own **compare** function (`bcrypt.compare()`), which re-hashes the input with the same salt/parameters stored in the existing hash and compares results in a way that resists timing attacks (Q10) — never manually extract and re-implement the comparison yourself.

[↑ Back to top](#table-of-contents)

---

### 8. What makes a good password policy? 🟡

- Favor **length** over complex character-class rules (a long passphrase is harder to crack than `P@ssw0rd1!`), check new passwords against known-breached password lists, and avoid mandatory periodic password rotation (modern guidance — e.g. NIST — considers it often counterproductive, encouraging weaker, more predictable passwords). Pair with MFA rather than relying purely on password strength.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `bcrypt`, `scrypt`, and `argon2`? 🔴

- **`bcrypt`**: CPU-hard, but not memory-hard — vulnerable to acceleration via specialized hardware (ASICs/GPUs) more easily than the alternatives below.
- **`scrypt`**: deliberately **memory-hard** as well as CPU-hard, making hardware-accelerated cracking significantly more expensive (memory is costlier to parallelize than raw compute).
- **`argon2`**: the winner of the Password Hashing Competition (2015) and current best-practice recommendation — configurable memory, time, and parallelism costs, designed specifically to resist both GPU and ASIC-based cracking better than the older two.

[↑ Back to top](#table-of-contents)

---

### 10. What is a timing attack, and how does it relate to password comparison? 🔴

- If a naive string comparison (`stored === input`) returns as soon as it finds the first mismatched character, the time taken to fail subtly reveals **how many characters matched** — an attacker can exploit these tiny timing differences to guess a secret byte-by-byte. Password hashing libraries' `compare` functions use **constant-time comparison**, taking the same amount of time regardless of where (or whether) a mismatch occurs.

[↑ Back to top](#table-of-contents)

---

### 11. How would you implement a secure "forgot password" flow? 🔴

- Generate a **cryptographically random**, single-use, short-expiry reset token; store only its **hash** server-side (so a database leak doesn't directly expose usable tokens); email a link containing the raw token; on submission, hash the provided token and compare; invalidate the token immediately after use (or after expiry) so it can't be replayed.

```js
const token = crypto.randomBytes(32).toString('hex');
const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
// store tokenHash + expiry in DB, email the raw `token` to the user
```

[↑ Back to top](#table-of-contents)

---

### 12. How do you handle a confirmed password database breach? 🔴

- Force a password reset for **all** affected users immediately, invalidate all existing sessions/tokens, notify users transparently about what was exposed, audit how the breach happened and close the gap, and check whether the hashing algorithm used was actually strong enough that the leaked hashes resist practical cracking in the time it takes users to reset.

> [!IMPORTANT]
> **Follow-up questions:**
> - If passwords were hashed with a slow algorithm like `argon2` at breach time, does that meaningfully reduce the urgency of forcing a reset — or should you force one regardless?

[↑ Back to top](#table-of-contents)

---
