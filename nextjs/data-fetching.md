# Data Fetching

_Part of [Next.js](README.md) interview notes._

> Build-time/request-time data fetching for pages (`getStaticProps`, `getServerSideProps`, ISR) is covered in [Rendering Strategies](rendering-strategies.md). This file covers fetching mechanics, Server Actions, and client/global state in the App Router.

## Table of Contents

**🟢 Easy**
- [1. How do you perform client-side data fetching in Next.js?](#1-how-do-you-perform-client-side-data-fetching-in-nextjs)
- [2. How does the `fetch` API behave differently in the App Router?](#2-how-does-the-fetch-api-behave-differently-in-the-app-router)
- [3. What is the `use client` directive?](#3-what-is-the-use-client-directive)

**🟡 Medium**
- [4. What is `use server`, and when do you use it?](#4-what-is-use-server-and-when-do-you-use-it)
- [5. What's the difference between `use server` and `use client`?](#5-whats-the-difference-between-use-server-and-use-client)
- [6. What are Server Actions, and what are their benefits?](#6-what-are-server-actions-and-what-are-their-benefits)
- [7. What problems can arise from using Server Actions?](#7-what-problems-can-arise-from-using-server-actions)
- [8. What is the `form action` pattern in Next.js?](#8-what-is-the-form-action-pattern-in-nextjs)
- [9. How do you set up GraphQL in a Next.js app?](#9-how-do-you-set-up-graphql-in-a-nextjs-app)

**🔴 Hard**
- [10. What are alternatives to Server Actions, and when would you choose them?](#10-what-are-alternatives-to-server-actions-and-when-would-you-choose-them)
- [11. How do you handle global state management in the App Router?](#11-how-do-you-handle-global-state-management-in-the-app-router)
- [12. How do you differentiate between Server and Client Components, and what are the rules for each?](#12-how-do-you-differentiate-between-server-and-client-components-and-what-are-the-rules-for-each)
- [13. How does data-fetching caching differ between the Pages Router and the App Router?](#13-how-does-data-fetching-caching-differ-between-the-pages-router-and-the-app-router)
- [14. Can a Server Component import a Client Component, and vice versa?](#14-can-a-server-component-import-a-client-component-and-vice-versa)

---

### 1. How do you perform client-side data fetching in Next.js? 🟢

- Same as in any React app — `fetch`/`axios` inside `useEffect`, or (recommended) a data-fetching library like `SWR` or `React Query` for caching, revalidation, and loading/error states out of the box.

```jsx
'use client';
import useSWR from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', (url) => fetch(url).then((r) => r.json()));
  if (error) return <p>Failed to load</p>;
  return data ? <p>{data.name}</p> : <p>Loading...</p>;
}
```

[↑ Back to top](#table-of-contents)

---

### 2. How does the `fetch` API behave differently in the App Router? 🟢

- Next.js **extends** the native `fetch` with automatic caching/deduping when called inside Server Components — by default, identical fetch calls during a single render are deduped, and results can be cached/revalidated via the `next: { revalidate }` option (see [Rendering Strategies](rendering-strategies.md#11-how-do-you-enable-isr-revalidation-in-the-app-router)).

[↑ Back to top](#table-of-contents)

---

### 3. What is the `use client` directive? 🟢

- A file-level directive marking a component (and everything it imports) as a **Client Component** — it gets bundled and hydrated in the browser, and can use state/effects/browser APIs/event handlers, unlike Server Components.

```jsx
'use client';
export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

[↑ Back to top](#table-of-contents)

---

### 4. What is `use server`, and when do you use it? 🟡

- Marks a function as a **Server Action** — code that's guaranteed to run only on the server, callable directly from Client Components (e.g. as a form's `action`), without manually creating an API route for it.

```jsx
// actions.js
'use server';
export async function createPost(formData) {
  await db.posts.insert({ title: formData.get('title') });
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `use server` and `use client`? 🟡

- `'use client'`: marks a **component module** to be bundled and run in the browser.
- `'use server'`: marks a **function** (often in its own file, or at the top of an async function) to run exclusively on the server, callable from client code as if it were a normal async function call — Next.js handles the network request under the hood.

[↑ Back to top](#table-of-contents)

---

### 6. What are Server Actions, and what are their benefits? 🟡

- Async functions marked `'use server'` that can be called directly from forms or event handlers in Client Components, **without** manually building a REST API route for every mutation.
- Benefits: less boilerplate (no separate API route + fetch call), progressive enhancement (works even before JS loads, via native form submission), and automatic integration with revalidation (`revalidatePath`/`revalidateTag`).

[↑ Back to top](#table-of-contents)

---

### 7. What problems can arise from using Server Actions? 🟡

- Easy to accidentally expose sensitive logic if not careful about what data a client can pass in (still need the same input validation/auth checks as any API endpoint).
- Less visible/discoverable than a dedicated REST API — harder for external clients (mobile apps, third parties) to consume, since they're tightly coupled to the Next.js app's forms rather than being a standalone documented endpoint.
- Testing and reasoning about caching/revalidation behavior can be less obvious than with a traditional explicit API call.

[↑ Back to top](#table-of-contents)

---

### 8. What is the `form action` pattern in Next.js? 🟡

- Pass a Server Action directly as a `<form>`'s `action` prop — the browser submits the form (even without JS), and Next.js routes it to the server function.

```jsx
import { createPost } from './actions';

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you set up GraphQL in a Next.js app? 🟡

- Client-side: use Apollo Client or `urql`, typically initialized in a Client Component/provider.
- Server-side: a Route Handler (`app/api/graphql/route.js`) can host a GraphQL server (e.g. via `graphql-yoga` or Apollo Server), or Server Components can query a GraphQL API directly with `fetch` during render.

[↑ Back to top](#table-of-contents)

---

### 10. What are alternatives to Server Actions, and when would you choose them? 🔴

- **Traditional API Routes/Route Handlers** + client-side `fetch`: better when you need a stable, documented, externally-consumable API (e.g. a mobile app also calls it), or finer control over HTTP semantics (status codes, headers).
- Server Actions are better suited for **internal, form-driven mutations** tightly coupled to your own UI, where you don't need an independently versioned API contract.

[↑ Back to top](#table-of-contents)

---

### 11. How do you handle global state management in the App Router? 🔴

- Client-side global state (Redux, Zustand, Context) still works the same as in any React app, but must live inside a **Client Component** boundary — typically a top-level provider marked `'use client'` that wraps the parts of the tree needing it.
- Server state (data from a database/API) generally doesn't need a client store at all — fetch it directly in Server Components and pass down as props/serialized data instead of duplicating it into client state.

```jsx
// app/providers.js
'use client';
export function Providers({ children }) {
  return <ReduxProvider store={store}>{children}</ReduxProvider>;
}
```

[↑ Back to top](#table-of-contents)

---

### 12. How do you differentiate between Server and Client Components, and what are the rules for each? 🔴

- **Server Components** (default): can directly access backend resources (DB, filesystem), `async`/`await` data fetching in the component body, but **cannot** use state, effects, refs, or browser-only APIs/event handlers.
- **Client Components** (`'use client'`): can use Hooks/state/effects/event handlers, but **cannot** directly access server-only resources, and add to the client JS bundle.
- Rule of thumb: keep components as Server Components by default; only mark the smallest necessary leaf components `'use client'` where interactivity is actually needed.

[↑ Back to top](#table-of-contents)

---

### 13. How does data-fetching caching differ between the Pages Router and the App Router? 🔴

- **Pages Router**: caching behavior is mostly determined by *which* data function you use (`getStaticProps` = cached/static, `getServerSideProps` = never cached) — a page-level, all-or-nothing choice.
- **App Router**: caching is controlled **per fetch call** (via `cache`/`next.revalidate` options on `fetch`), so a single route can mix cached and uncached data fetches, with much more granular control than the Pages Router allowed.

[↑ Back to top](#table-of-contents)

---

### 14. Can a Server Component import a Client Component, and vice versa? 🔴

- A **Server Component can import and render a Client Component** — common and expected (e.g. a server-rendered page including an interactive client-side button).
- A **Client Component cannot directly import a Server Component** as a regular import (since Server Components may use server-only APIs that can't run in the browser) — but a Server Component can be passed **as `children`/props** into a Client Component, letting you still compose them, just not via a direct top-down import from the client side.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does passing a Server Component as `children` work, when a direct import doesn't?

[↑ Back to top](#table-of-contents)

---
