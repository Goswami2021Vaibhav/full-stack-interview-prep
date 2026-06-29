# Rendering Strategies (SSR/SSG/ISR)

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is pre-rendering in Next.js?](#1-what-is-pre-rendering-in-nextjs)
- [2. What is Server-Side Rendering (SSR)?](#2-what-is-server-side-rendering-ssr)
- [3. What is Static Site Generation (SSG)?](#3-what-is-static-site-generation-ssg)
- [4. What's the difference between client-side and server-side rendering?](#4-whats-the-difference-between-client-side-and-server-side-rendering)

**🟡 Medium**
- [5. What's the difference between SSG and SSR?](#5-whats-the-difference-between-ssg-and-ssr)
- [6. What is `getStaticProps`?](#6-what-is-getstaticprops)
- [7. What is `getServerSideProps`?](#7-what-is-getserversideprops)
- [8. What's the difference between `getStaticProps` and `getServerSideProps`?](#8-whats-the-difference-between-getstaticprops-and-getserversideprops)
- [9. What is `getStaticPaths`, and what does `fallback` control?](#9-what-is-getstaticpaths-and-what-does-fallback-control)
- [10. What is Incremental Static Regeneration (ISR)?](#10-what-is-incremental-static-regeneration-isr)

**🔴 Hard**
- [11. How do you enable ISR (revalidation) in the App Router?](#11-how-do-you-enable-isr-revalidation-in-the-app-router)
- [12. What are React Server Components, and how do they change rendering in the App Router?](#12-what-are-react-server-components-and-how-do-they-change-rendering-in-the-app-router)
- [13. How do you handle streaming and Suspense in the App Router?](#13-how-do-you-handle-streaming-and-suspense-in-the-app-router)
- [14. What's the difference between static and dynamic rendering in the App Router?](#14-whats-the-difference-between-static-and-dynamic-rendering-in-the-app-router)

---

### 1. What is pre-rendering in Next.js? 🟢

- Generating a page's HTML **in advance** (either at build time or on the server per-request), instead of shipping an empty shell and rendering everything in the browser — improves perceived load time and SEO, since crawlers/users see real content immediately.

[↑ Back to top](#table-of-contents)

---

### 2. What is Server-Side Rendering (SSR)? 🟢

- The page's HTML is generated **on the server, for every request** — always reflects the latest data, at the cost of a server round-trip on each page load.

[↑ Back to top](#table-of-contents)

---

### 3. What is Static Site Generation (SSG)? 🟢

- The page's HTML is generated **once, at build time** — served instantly from a CDN on every request afterward, but content is "frozen" until the next build (or ISR revalidation).

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between client-side and server-side rendering? 🟢

- **Client-side rendering (CSR)**: the browser downloads a minimal HTML shell + JS bundle, then JS renders the actual content in the browser — slower first paint, weaker default SEO.
- **Server-side rendering (SSR)**: the server sends fully-rendered HTML — faster first paint, better SEO, but the server does more work per request.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between SSG and SSR? 🟡

| | SSG | SSR |
|---|---|---|
| When HTML is built | At build time | Per request |
| Speed | Instant (served from CDN/cache) | Depends on server response time |
| Data freshness | Stale until next build/revalidation | Always current |
| Best for | Blogs, marketing pages, docs | Dashboards, user-specific/frequently-changing data |

[↑ Back to top](#table-of-contents)

---

### 6. What is `getStaticProps`? 🟡

- (Pages Router) An `async` function exported from a page that runs **at build time**, fetching data and passing it as props — used for SSG.

```jsx
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data } };
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What is `getServerSideProps`? 🟡

- (Pages Router) An `async` function exported from a page that runs **on every request**, on the server — used for SSR.

```jsx
export async function getServerSideProps(context) {
  const data = await fetchData();
  return { props: { data } };
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `getStaticProps` and `getServerSideProps`? 🟡

- `getStaticProps`: runs at **build time** (or on a revalidation interval with ISR) — produces static HTML.
- `getServerSideProps`: runs on **every incoming request** — always fresh, but adds server latency to each page load.

[↑ Back to top](#table-of-contents)

---

### 9. What is `getStaticPaths`, and what does `fallback` control? 🟡

- Used alongside `getStaticProps` on **dynamic** SSG routes (`[id].js`) to specify which param values should be pre-built at build time.
- `fallback` controls what happens for params **not** included in that list: `false` → 404; `true` → serves a loading state then renders on first request, caching it after; `'blocking'` → renders on the server for the first request (like SSR) and caches the result for next time.

```jsx
export async function getStaticPaths() {
  return { paths: [{ params: { id: '1' } }], fallback: 'blocking' };
}
```

[↑ Back to top](#table-of-contents)

---

### 10. What is Incremental Static Regeneration (ISR)? 🟡

- Lets statically-generated pages be **re-generated in the background** after a specified time interval, without requiring a full site rebuild — combines SSG's speed with periodically-refreshed data.

```jsx
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data }, revalidate: 60 }; // regenerate at most once every 60 seconds
}
```

[↑ Back to top](#table-of-contents)

---

### 11. How do you enable ISR (revalidation) in the App Router? 🔴

- Set a `revalidate` value on the route segment, or pass `{ next: { revalidate } }` to a `fetch` call within it.

```jsx
// app/posts/page.js
export const revalidate = 60; // whole route revalidates at most every 60s

// or per-fetch:
fetch('https://api.example.com/posts', { next: { revalidate: 60 } });
```

[↑ Back to top](#table-of-contents)

---

### 12. What are React Server Components, and how do they change rendering in the App Router? 🔴

- In the App Router, components are **Server Components by default** — they render on the server, can directly `await` data, and ship **zero JS** to the client for that component, unless explicitly marked `'use client'`.
- This shifts the default rendering model from "ship JS, hydrate everything" to "render on server, only hydrate what's interactive" — generally smaller bundles and faster initial loads than the Pages Router's all-client-hydrated model.

[↑ Back to top](#table-of-contents)

---

### 13. How do you handle streaming and Suspense in the App Router? 🔴

- Wrap a slow-loading part of a page in `<Suspense>` with a fallback — Next.js can send the rest of the page's HTML to the browser **immediately**, then stream in the suspended part once its data is ready, instead of blocking the entire page on the slowest piece of data.

```jsx
export default function Page() {
  return (
    <div>
      <Header />
      <Suspense fallback={<Skeleton />}>
        <SlowComments /> {/* streamed in once ready, doesn't block Header */}
      </Suspense>
    </div>
  );
}
```

[↑ Back to top](#table-of-contents)

---

### 14. What's the difference between static and dynamic rendering in the App Router? 🔴

- **Static rendering** (default): rendered at build time (or revalidated via ISR) — same output for every user/request, cached.
- **Dynamic rendering**: rendered **per-request** — triggered automatically by using things like cookies, headers, search params, or an uncached `fetch` call inside a route, since those inherently differ per request.

> [!IMPORTANT]
> **Follow-up questions:**
> - What specific APIs/usages force a route into dynamic rendering even if you didn't explicitly ask for SSR?

[↑ Back to top](#table-of-contents)

---
