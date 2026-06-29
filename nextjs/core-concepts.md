# Core Concepts

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is Next.js?](#1-what-is-nextjs)
- [2. What are the key features of Next.js?](#2-what-are-the-key-features-of-nextjs)
- [3. What's the difference between Next.js and React?](#3-whats-the-difference-between-nextjs-and-react)
- [4. What is the default port for a Next.js app, and how do you change it?](#4-what-is-the-default-port-for-a-nextjs-app-and-how-do-you-change-it)
- [5. What is the `public` folder used for?](#5-what-is-the-public-folder-used-for)

**🟡 Medium**
- [6. How do you enable TypeScript in a Next.js project?](#6-how-do-you-enable-typescript-in-a-nextjs-project)
- [7. How do you handle environment variables in Next.js?](#7-how-do-you-handle-environment-variables-in-nextjs)
- [8. What is Fast Refresh?](#8-what-is-fast-refresh)
- [9. What is `next.config.js` used for?](#9-what-is-nextconfigjs-used-for)
- [10. What is React Strict Mode, and is it on by default in Next.js?](#10-what-is-react-strict-mode-and-is-it-on-by-default-in-nextjs)

**🔴 Hard**
- [11. What is a singleton router in Next.js?](#11-what-is-a-singleton-router-in-nextjs)
- [12. How is Next.js considered a full-stack framework?](#12-how-is-nextjs-considered-a-full-stack-framework)
- [13. What are the limitations of Next.js?](#13-what-are-the-limitations-of-nextjs)
- [14. Is Next.js suitable for large-scale applications?](#14-is-nextjs-suitable-for-large-scale-applications)
- [15. What is static optimization in Next.js?](#15-what-is-static-optimization-in-nextjs)
- [16. How do you add polyfills in Next.js?](#16-how-do-you-add-polyfills-in-nextjs)

---

### 1. What is Next.js? 🟢

- A React **framework** (not just a library) that adds production-oriented features on top of React: file-based routing, server-side rendering/static generation, API routes, image/font optimization, and bundling — out of the box, with minimal config.

[↑ Back to top](#table-of-contents)

---

### 2. What are the key features of Next.js? 🟢

- File-based routing, multiple rendering strategies (SSR/SSG/ISR/CSR), built-in API routes, automatic code-splitting, Image/Font optimization components, Fast Refresh during development, and (App Router) React Server Components + Server Actions.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between Next.js and React? 🟢

- **React**: a UI library — handles components/rendering, but you must bring your own routing, data-fetching strategy, and build tooling.
- **Next.js**: a framework **built on React** — provides routing, rendering strategy, and tooling decisions for you, so you don't have to wire them up from scratch.

[↑ Back to top](#table-of-contents)

---

### 4. What is the default port for a Next.js app, and how do you change it? 🟢

- Default: `3000`. Change it via the dev script or CLI flag.

```bash
next dev -p 4000
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the `public` folder used for? 🟢

- Static assets (images, fonts, `favicon.ico`, `robots.txt`) placed here are served directly from the site's root URL, unprocessed by the bundler.

```
public/logo.png  →  accessible at  /logo.png
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you enable TypeScript in a Next.js project? 🟡

- Add a `tsconfig.json` file (even an empty one) and rename files to `.tsx`/`.ts` — running `next dev` afterward auto-installs the needed TypeScript dependencies and fills in the config.

```bash
touch tsconfig.json
npm run dev # Next.js detects it and finishes TS setup automatically
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you handle environment variables in Next.js? 🟡

- Define them in `.env.local` (or `.env.production`, etc.). Variables prefixed with `NEXT_PUBLIC_` are exposed to the **browser**; everything else stays **server-only**.

```
DATABASE_URL=postgres://...        # server-only
NEXT_PUBLIC_API_URL=https://api.example.com  # exposed to the client bundle
```

[↑ Back to top](#table-of-contents)

---

### 8. What is Fast Refresh? 🟡

- Next.js's hot-reloading system — edits to a component are reflected in the browser almost instantly **without losing component state**, unlike a full page reload.

[↑ Back to top](#table-of-contents)

---

### 9. What is `next.config.js` used for? 🟡

- The central config file for customizing build/runtime behavior: redirects, headers, image domains, Webpack/Babel overrides, environment exposure, experimental flags, and more.

```js
module.exports = {
  images: { domains: ['example.com'] },
  reactStrictMode: true,
};
```

[↑ Back to top](#table-of-contents)

---

### 10. What is React Strict Mode, and is it on by default in Next.js? 🟡

- A development-only tool that highlights potential problems (unsafe lifecycle usage, side effects in render) by intentionally double-invoking renders/effects. Next.js has had `reactStrictMode: true` as the default in `create-next-app` projects since Next 13+.

[↑ Back to top](#table-of-contents)

---

### 11. What is a singleton router in Next.js? 🔴

- Next.js maintains **one single router instance** shared across the whole app (accessible via `useRouter()`/`next/router`), rather than each component managing its own routing state — ensures consistent navigation state everywhere, and is why `useRouter()` always reflects the current, app-wide route.

[↑ Back to top](#table-of-contents)

---

### 12. How is Next.js considered a full-stack framework? 🔴

- Beyond rendering UI, it lets you write actual **backend logic** in the same project: API Routes/Route Handlers for REST endpoints, Server Actions for mutations, middleware for request interception — meaning a Next.js app can be both the frontend and the backend, without a separate server project.

[↑ Back to top](#table-of-contents)

---

### 13. What are the limitations of Next.js? 🔴

- Opinionated structure/conventions can feel restrictive for non-standard architectures; the framework evolves quickly (App Router vs Pages Router churn) which can mean re-learning patterns; SSR/ISR add server infrastructure considerations a plain static SPA wouldn't have; can be overkill for very simple sites that don't need SSR/routing complexity.

[↑ Back to top](#table-of-contents)

---

### 14. Is Next.js suitable for large-scale applications? 🔴

- Generally yes — used in production by many large companies (TikTok, Twitch, Hulu, Notion). It scales well due to automatic code-splitting, flexible per-route rendering strategy (mix SSR/SSG/ISR as needed), and built-in performance optimizations, though large apps still need careful architecture (route groups, modular API routes, caching strategy) regardless of framework.

[↑ Back to top](#table-of-contents)

---

### 15. What is static optimization in Next.js? 🔴

- Next.js **automatically** prerenders pages to static HTML at build time if they have no blocking data requirements (no `getServerSideProps`, or in the App Router, no dynamic/uncached data access) — you get the performance benefits of static generation without manually opting in, as long as nothing in the page forces dynamic rendering.

[↑ Back to top](#table-of-contents)

---

### 16. How do you add polyfills in Next.js? 🔴

- Next.js auto-polyfills broadly supported core features. For anything beyond that, import the polyfill directly in `_app.js` (Pages Router) or a root layout/entry (App Router) so it loads before any page code runs.

```js
// pages/_app.js
import 'core-js/stable';
import 'whatwg-fetch';
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might adding unnecessary polyfills hurt your bundle size/performance?

[↑ Back to top](#table-of-contents)

---
