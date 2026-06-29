# API Routes

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are API Routes in Next.js?](#1-what-are-api-routes-in-nextjs)
- [2. How do you create an API endpoint?](#2-how-do-you-create-an-api-endpoint)
- [3. What are Route Handlers, and how do they differ from Pages Router API Routes?](#3-what-are-route-handlers-and-how-do-they-differ-from-pages-router-api-routes)

**🟡 Medium**
- [4. How do you handle CORS in Next.js API routes?](#4-how-do-you-handle-cors-in-nextjs-api-routes)
- [5. How do you manage cookies in Next.js?](#5-how-do-you-manage-cookies-in-nextjs)
- [6. How do you use the proper HTTP methods in an API route?](#6-how-do-you-use-the-proper-http-methods-in-an-api-route)
- [7. How do you handle errors in API routes?](#7-how-do-you-handle-errors-in-api-routes)
- [8. How do you validate and sanitize input data in API routes?](#8-how-do-you-validate-and-sanitize-input-data-in-api-routes)

**🔴 Hard**
- [9. How do you prevent an API route from being misused by the client?](#9-how-do-you-prevent-an-api-route-from-being-misused-by-the-client)
- [10. How do you handle JWT tokens in Next.js?](#10-how-do-you-handle-jwt-tokens-in-nextjs)
- [11. How can you implement simple rate limiting for an API route?](#11-how-can-you-implement-simple-rate-limiting-for-an-api-route)
- [12. How do you use the Edge Runtime for an API route?](#12-how-do-you-use-the-edge-runtime-for-an-api-route)
- [13. How do you keep API routes modular and organized at scale?](#13-how-do-you-keep-api-routes-modular-and-organized-at-scale)
- [14. How do you use middleware specifically within an API route?](#14-how-do-you-use-middleware-specifically-within-an-api-route)

---

### 1. What are API Routes in Next.js? 🟢

- A way to build backend HTTP endpoints **inside** a Next.js app — any file under `pages/api/` (or a Route Handler under `app/.../route.js`) becomes a server-side endpoint, instead of needing a separate backend project.

[↑ Back to top](#table-of-contents)

---

### 2. How do you create an API endpoint? 🟢

```js
// pages/api/hello.js (Pages Router)
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello' });
}
```

```js
// app/api/hello/route.js (App Router)
export async function GET() {
  return Response.json({ message: 'Hello' });
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What are Route Handlers, and how do they differ from Pages Router API Routes? 🟢

- Route Handlers (`app/.../route.js`) are the App Router's equivalent of API Routes, but defined with **one exported function per HTTP method** (`GET`, `POST`, etc.) instead of a single handler switching on `req.method` — and they use the standard Web `Request`/`Response` objects instead of Next's custom `req`/`res`.

[↑ Back to top](#table-of-contents)

---

### 4. How do you handle CORS in Next.js API routes? 🟡

- Manually set the relevant headers (`Access-Control-Allow-Origin`, etc.) in the response, or centralize it in middleware so every API route gets consistent CORS handling.

```js
export async function GET(request) {
  return new Response(JSON.stringify({ ok: true }), {
    headers: { 'Access-Control-Allow-Origin': 'https://example.com' },
  });
}
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you manage cookies in Next.js? 🟡

- Pages Router: read/write via `req.cookies`/`res.setHeader('Set-Cookie', ...)`, or the `cookie` package.
- App Router: use the `cookies()` function from `next/headers` inside Server Components, Route Handlers, or Server Actions.

```js
import { cookies } from 'next/headers';

export async function GET() {
  const cookieStore = cookies();
  const theme = cookieStore.get('theme');
  return Response.json({ theme: theme?.value });
}
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you use the proper HTTP methods in an API route? 🟡

- Match the semantics of REST: `GET` for reading, `POST` for creating, `PUT`/`PATCH` for updating, `DELETE` for removing — and reject unsupported methods with a `405`.

```js
// app/api/posts/route.js
export async function GET() { /* list posts */ }
export async function POST(request) { /* create a post */ }
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you handle errors in API routes? 🟡

- Wrap logic in `try`/`catch`, return an appropriate status code and a clear error message — never let unhandled exceptions silently crash the function with an unhelpful response.

```js
export async function GET() {
  try {
    const data = await fetchFromDb();
    return Response.json(data);
  } catch (err) {
    return Response.json({ error: 'Failed to fetch data' }, { status: 500 });
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 8. How do you validate and sanitize input data in API routes? 🟡

- Use a schema validation library (Zod, Yup, Joi) to check shape/types **before** acting on request data — never trust a client-supplied payload directly.

```js
import { z } from 'zod';
const schema = z.object({ title: z.string().min(1) });

export async function POST(request) {
  const body = await request.json();
  const result = schema.safeParse(body);
  if (!result.success) return Response.json({ error: 'Invalid input' }, { status: 400 });
  // proceed with result.data
}
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you prevent an API route from being misused by the client? 🔴

- Always verify **authentication and authorization** server-side on every request (never trust a hidden form field or client-side check alone) — check the user's session/token and confirm they're allowed to perform that specific action, not just that they're logged in.

[↑ Back to top](#table-of-contents)

---

### 10. How do you handle JWT tokens in Next.js? 🔴

- Issue a JWT on login (e.g. signed server-side), store it in an **httpOnly** cookie (safer than `localStorage`, since JS can't read it, mitigating XSS token theft), and verify/decode it server-side in middleware or each protected Route Handler.

```js
import jwt from 'jsonwebtoken';

export async function POST(request) {
  const { username, password } = await request.json();
  // ...verify credentials...
  const token = jwt.sign({ username }, process.env.JWT_SECRET, { expiresIn: '1h' });
  const response = Response.json({ ok: true });
  response.headers.set('Set-Cookie', `token=${token}; HttpOnly; Path=/; Max-Age=3600`);
  return response;
}
```

[↑ Back to top](#table-of-contents)

---

### 11. How can you implement simple rate limiting for an API route? 🔴

- Track request counts per IP/user (in-memory for a single instance, or Redis for multi-instance deployments) within a time window, and reject requests exceeding the limit.

```js
const requests = new Map();

export async function POST(request) {
  const ip = request.headers.get('x-forwarded-for');
  const count = requests.get(ip) || 0;
  if (count >= 10) return Response.json({ error: 'Too many requests' }, { status: 429 });
  requests.set(ip, count + 1);
  // ...handle request...
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why doesn't an in-memory `Map` work for rate limiting across multiple serverless instances?

[↑ Back to top](#table-of-contents)

---

### 12. How do you use the Edge Runtime for an API route? 🔴

- Export a `runtime` config to run the route on Vercel's/Next's lightweight **Edge** runtime (V8 isolates, deployed closer to users globally) instead of a full Node.js server — faster cold starts, but a restricted API surface (no full Node.js APIs like `fs`).

```js
export const runtime = 'edge';

export async function GET() {
  return Response.json({ message: 'Running on the edge' });
}
```

[↑ Back to top](#table-of-contents)

---

### 13. How do you keep API routes modular and organized at scale? 🔴

- Extract shared logic (DB access, validation schemas, auth checks) into separate utility modules imported by multiple routes, rather than duplicating logic per route file.
- Group related routes under shared folders (`app/api/users/...`, `app/api/posts/...`), and centralize cross-cutting concerns (auth, logging, error formatting) in middleware or shared handler wrappers instead of repeating them in every route.

[↑ Back to top](#table-of-contents)

---

### 14. How do you use middleware specifically within an API route? 🔴

- Next.js's top-level `middleware.js` (see [Middleware](middleware.md)) runs **before** the request reaches any route, including API routes/Route Handlers — useful for centralizing auth checks, logging, or header injection across multiple API routes without repeating the logic in each one.

[↑ Back to top](#table-of-contents)

---
