# Routing

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is file-based routing in Next.js?](#1-what-is-file-based-routing-in-nextjs)
- [2. How do you create a route in the Pages Router?](#2-how-do-you-create-a-route-in-the-pages-router)
- [3. How do you create a route in the App Router?](#3-how-do-you-create-a-route-in-the-app-router)
- [4. What is the `Link` component used for?](#4-what-is-the-link-component-used-for)
- [5. What is the `useRouter` hook?](#5-what-is-the-userouter-hook)

**🟡 Medium**
- [6. What's the difference between `push` and `replace` in `useRouter`?](#6-whats-the-difference-between-push-and-replace-in-userouter)
- [7. How do you create a dynamic route?](#7-how-do-you-create-a-dynamic-route)
- [8. What is a catch-all segment?](#8-what-is-a-catch-all-segment)
- [9. What is `_app.js`, and what is it used for?](#9-what-is-_appjs-and-what-is-it-used-for)
- [10. What is `_document.js`, and how does it differ from `_app.js`?](#10-what-is-_documentjs-and-how-does-it-differ-from-_appjs)
- [11. How do you create a custom 404 page?](#11-how-do-you-create-a-custom-404-page)
- [12. What is the `loading.js` file in the App Router?](#12-what-is-the-loadingjs-file-in-the-app-router)
- [13. How do you handle a not-found page in the App Router?](#13-how-do-you-handle-a-not-found-page-in-the-app-router)

**🔴 Hard**
- [14. What are route groups in the App Router?](#14-what-are-route-groups-in-the-app-router)
- [15. What are parallel routes in the App Router?](#15-what-are-parallel-routes-in-the-app-router)
- [16. How do you implement intercepting routes?](#16-how-do-you-implement-intercepting-routes)
- [17. How do you implement nested layouts in the App Router?](#17-how-do-you-implement-nested-layouts-in-the-app-router)
- [18. What is `template.js`, and how does it differ from `layout.js`?](#18-what-is-templatejs-and-how-does-it-differ-from-layoutjs)
- [19. When should you choose the Pages Router vs. the App Router?](#19-when-should-you-choose-the-pages-router-vs-the-app-router)

---

### 1. What is file-based routing in Next.js? 🟢

- The folder/file structure inside `pages/` (or `app/`) directly **defines** your app's routes — no manual route-config file needed, unlike most other React routing setups.

```
pages/about.js        →  /about
app/about/page.js     →  /about
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you create a route in the Pages Router? 🟢

- Add a file under `pages/` — the file path becomes the URL path, and its default export is the page component.

```jsx
// pages/about.js
export default function About() {
  return <h1>About us</h1>;
}
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you create a route in the App Router? 🟢

- Create a folder under `app/` matching the URL segment, containing a `page.js` (or `.tsx`) file.

```jsx
// app/about/page.js
export default function About() {
  return <h1>About us</h1>;
}
```

[↑ Back to top](#table-of-contents)

---

### 4. What is the `Link` component used for? 🟢

- Client-side navigation between routes without a full page reload — also automatically prefetches the linked page in the background for faster perceived navigation.

```jsx
import Link from 'next/link';
<Link href="/about">About</Link>
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the `useRouter` hook? 🟢

- Gives access to the router object inside a component — current path, query params, and navigation methods (`push`, `replace`, `back`).

```jsx
import { useRouter } from 'next/router'; // Pages Router
const router = useRouter();
router.pathname; // current path
```

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between `push` and `replace` in `useRouter`? 🟡

- `push(url)`: navigates and adds a **new entry** to browser history (back button returns to the previous page).
- `replace(url)`: navigates but **replaces** the current history entry — back button skips over it entirely.

```jsx
router.push('/dashboard');   // back button returns here
router.replace('/dashboard'); // back button skips this page
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you create a dynamic route? 🟡

- Wrap a folder/file name in square brackets — the bracketed segment becomes a URL parameter.

```
pages/posts/[id].js     →  /posts/1, /posts/2, ...
app/posts/[id]/page.js  →  /posts/1, /posts/2, ...
```

```jsx
// App Router
export default function Post({ params }) {
  return <h1>Post {params.id}</h1>;
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is a catch-all segment? 🟡

- `[...slug]` matches **any number** of remaining path segments as an array; `[[...slug]]` (double brackets) makes it **optional**, so the base route without any extra segments also matches.

```
pages/docs/[...slug].js  →  /docs/a, /docs/a/b, /docs/a/b/c (slug = ['a','b','c'])
pages/docs/[[...slug]].js → also matches plain /docs (slug = undefined)
```

[↑ Back to top](#table-of-contents)

---

### 9. What is `_app.js`, and what is it used for? 🟡

- (Pages Router) The top-level component wrapping **every** page — used for persistent layout, global CSS imports, and wrapping the app in providers (theme, Redux, etc.) that should survive across page navigations.

```jsx
// pages/_app.js
export default function MyApp({ Component, pageProps }) {
  return <Layout><Component {...pageProps} /></Layout>;
}
```

[↑ Back to top](#table-of-contents)

---

### 10. What is `_document.js`, and how does it differ from `_app.js`? 🟡

- `_document.js` customizes the **static HTML shell** (the `<html>`/`<head>`/`<body>` tags) — runs only on the server, no access to React state/props beyond what's passed in.
- `_app.js` customizes the **React tree** rendered inside that shell, runs on both server and client, and re-renders on navigation; `_document.js` does not.

[↑ Back to top](#table-of-contents)

---

### 11. How do you create a custom 404 page? 🟢

- Pages Router: add `pages/404.js`. App Router: add `app/not-found.js`.

```jsx
// pages/404.js
export default function Custom404() {
  return <h1>404 - Page Not Found</h1>;
}
```

[↑ Back to top](#table-of-contents)

---

### 12. What is the `loading.js` file in the App Router? 🟡

- Automatically wraps a route segment (and its children) in a `<Suspense>` boundary — its export is shown as an **instant loading UI** while the actual page's data/render is still in progress.

```jsx
// app/dashboard/loading.js
export default function Loading() {
  return <Spinner />;
}
```

[↑ Back to top](#table-of-contents)

---

### 13. How do you handle a not-found page in the App Router? 🟡

- Add a `not-found.js` file in a route segment for a segment-specific 404, or call the `notFound()` function from inside a page/layout to trigger it programmatically (e.g. when fetched data doesn't exist).

```jsx
// app/posts/[id]/page.js
import { notFound } from 'next/navigation';

export default async function Post({ params }) {
  const post = await getPost(params.id);
  if (!post) notFound(); // renders the nearest not-found.js
  return <article>{post.title}</article>;
}
```

[↑ Back to top](#table-of-contents)

---

### 14. What are route groups in the App Router? 🔴

- A folder wrapped in parentheses `(groupName)` that organizes routes **without** affecting the URL path — useful for applying a shared layout to a set of routes, or just organizing files logically.

```
app/(marketing)/about/page.js   →  /about  (parens don't appear in the URL)
app/(shop)/products/page.js     →  /products
```

[↑ Back to top](#table-of-contents)

---

### 15. What are parallel routes in the App Router? 🔴

- Render **multiple independent pages** in the same layout simultaneously, each in its own named "slot" (`@slotName` folder) — useful for dashboards with independently-loading sections (e.g. a feed and a sidebar that load separately).

```
app/@feed/page.js
app/@sidebar/page.js
app/layout.js  → receives both `feed` and `sidebar` as props to render side by side
```

[↑ Back to top](#table-of-contents)

---

### 16. How do you implement intercepting routes? 🔴

- `(.)folder` (and `(..)`, `(...)` variants for different nesting levels) lets a route **intercept** navigation to another route and show it in a different context (e.g. a modal) while keeping the original URL — but a direct page load/refresh still shows the full, non-intercepted page.

```
app/feed/page.js
app/feed/(.)photo/[id]/page.js   →  intercepts navigation to /photo/[id] from /feed, shown as a modal
app/photo/[id]/page.js            →  the full page, shown on direct visit/refresh
```

> [!TIP]
> **Real-life example:** Instagram-style feeds where clicking a photo opens it as a modal overlay, but visiting that photo's URL directly loads a full standalone page.

[↑ Back to top](#table-of-contents)

---

### 17. How do you implement nested layouts in the App Router? 🔴

- Each folder can have its own `layout.js`, which wraps its `page.js` **and** any nested child routes' layouts — layouts compose automatically based on folder nesting, and persist (don't re-render) across navigations within them.

```
app/layout.js              → root layout, wraps everything
app/dashboard/layout.js    → wraps all /dashboard/* routes
app/dashboard/page.js
app/dashboard/settings/page.js  → wrapped by BOTH layouts above
```

[↑ Back to top](#table-of-contents)

---

### 18. What is `template.js`, and how does it differ from `layout.js`? 🔴

- Looks similar to `layout.js`, but **re-mounts** (creates a fresh instance, resetting state) on every navigation, whereas `layout.js` persists across navigations within it. Useful when you specifically want enter/exit animations or per-navigation state reset.

[↑ Back to top](#table-of-contents)

---

### 19. When should you choose the Pages Router vs. the App Router? 🔴

- **App Router** (recommended for new projects): React Server Components, Server Actions, nested layouts, streaming/Suspense, more granular caching — the actively-developed direction of Next.js.
- **Pages Router**: still fully supported, simpler mental model (no server/client component split to reason about), and may still make sense for existing large codebases not yet worth migrating, or teams not ready for the App Router's RSC paradigm shift.

> [!IMPORTANT]
> **Follow-up questions:**
> - Can a single Next.js project use both routers simultaneously during a migration?

[↑ Back to top](#table-of-contents)

---
