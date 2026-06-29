# Middleware

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is middleware in Next.js?](#1-what-is-middleware-in-nextjs)
- [2. How does middleware work in Next.js?](#2-how-does-middleware-work-in-nextjs)

**🟡 Medium**
- [3. How do you handle middleware in the Pages Router?](#3-how-do-you-handle-middleware-in-the-pages-router)
- [4. How do you handle middleware in the App Router?](#4-how-do-you-handle-middleware-in-the-app-router)
- [5. What is a custom server in Next.js, and when would you need one?](#5-what-is-a-custom-server-in-nextjs-and-when-would-you-need-one)

**🔴 Hard**
- [6. How do you implement authentication/authorization logic in middleware?](#6-how-do-you-implement-authenticationauthorization-logic-in-middleware)
- [7. What's the relationship between middleware and the Edge Runtime?](#7-whats-the-relationship-between-middleware-and-the-edge-runtime)
- [8. What are the limitations of what middleware can do?](#8-what-are-the-limitations-of-what-middleware-can-do)
- [9. How do you redirect users from middleware based on auth state?](#9-how-do-you-redirect-users-from-middleware-based-on-auth-state)
- [10. How do you scope middleware to only run on specific paths?](#10-how-do-you-scope-middleware-to-only-run-on-specific-paths)

---

### 1. What is middleware in Next.js? 🟢

- Code that runs **before** a request completes, intercepting it to read/modify the request or response — used for things like auth checks, redirects, logging, or A/B testing, before the request ever reaches a page or API route.

[↑ Back to top](#table-of-contents)

---

### 2. How does middleware work in Next.js? 🟢

- A single `middleware.js`/`middleware.ts` file at the project root, exporting a `middleware()` function, runs on the **Edge Runtime** for every matching request (configurable via a `matcher`/path check) — it can let the request continue (`NextResponse.next()`), redirect, rewrite, or modify headers.

```js
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  return NextResponse.next();
}
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you handle middleware in the Pages Router? 🟡

- The same single top-level `middleware.js` file applies regardless of router — middleware isn't tied to Pages vs. App Router specifically, it operates at the request level, before routing even decides which router/page handles it.

[↑ Back to top](#table-of-contents)

---

### 4. How do you handle middleware in the App Router? 🟡

- Identical setup to the Pages Router (same single `middleware.js` file) — but commonly paired with App Router features like reading `cookies()`/`headers()` to gate access to specific route segments before rendering Server Components.

```js
export function middleware(request) {
  const token = request.cookies.get('token');
  if (!token) return NextResponse.redirect(new URL('/login', request.url));
  return NextResponse.next();
}

export const config = { matcher: '/dashboard/:path*' };
```

[↑ Back to top](#table-of-contents)

---

### 5. What is a custom server in Next.js, and when would you need one? 🟡

- Running Next.js as middleware **within** your own Node.js server (e.g. Express) instead of using the built-in `next start`/Vercel deployment model — needed for advanced custom routing logic or integrating with existing Node infrastructure.
- Generally discouraged unless truly necessary: it opts you out of some platform optimizations and adds operational complexity. Most apps can fully meet their needs with file-based routing + middleware + API routes instead.

[↑ Back to top](#table-of-contents)

---

### 6. How do you implement authentication/authorization logic in middleware? 🔴

- Read the auth token/cookie, verify it (e.g. check a JWT's signature/expiry), and redirect unauthenticated/unauthorized requests **before** they reach the protected page or API route — centralizing the check in one place instead of repeating it per route.

```js
import { NextResponse } from 'next/server';
import { verifyToken } from './lib/auth';

export function middleware(request) {
  const token = request.cookies.get('token')?.value;
  const isValid = token && verifyToken(token);
  if (!isValid) return NextResponse.redirect(new URL('/login', request.url));
  return NextResponse.next();
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the relationship between middleware and the Edge Runtime? 🔴

- Middleware **always** runs on the lightweight Edge Runtime (V8 isolates, not full Node.js) — fast startup, runs geographically close to the user, but with a restricted API surface (no direct filesystem/Node-specific APIs, no direct database driver connections that require Node sockets).

[↑ Back to top](#table-of-contents)

---

### 8. What are the limitations of what middleware can do? 🔴

- Can't use most Node.js-only APIs or libraries that depend on them (many DB clients, `fs`).
- Can't directly query a relational database — calls out to an HTTP API instead, or defers the actual data work to the page/route itself.
- Runs on **every** matched request, so any slow logic in it adds latency to every page load it covers — keep it fast and lightweight.

[↑ Back to top](#table-of-contents)

---

### 9. How do you redirect users from middleware based on auth state? 🔴

```js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const isLoggedIn = Boolean(request.cookies.get('token'));
  const isAuthPage = request.nextUrl.pathname.startsWith('/login');

  if (!isLoggedIn && !isAuthPage) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  if (isLoggedIn && isAuthPage) {
    return NextResponse.redirect(new URL('/dashboard', request.url)); // already logged in
  }
  return NextResponse.next();
}
```

[↑ Back to top](#table-of-contents)

---

### 10. How do you scope middleware to only run on specific paths? 🔴

- Export a `config.matcher` — an array of path patterns — so middleware only executes for matching requests, avoiding unnecessary overhead on unrelated routes (like static assets).

```js
export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*'],
};
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is scoping the matcher important for performance, given middleware runs on every matched request?

[↑ Back to top](#table-of-contents)

---
