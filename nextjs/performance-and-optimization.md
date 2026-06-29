# Performance & Optimization

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is the `Image` component, and why use it over a plain `<img>`?](#1-what-is-the-image-component-and-why-use-it-over-a-plain-img)
- [2. What is dynamic import in Next.js?](#2-what-is-dynamic-import-in-nextjs)
- [3. What is `next/script` used for?](#3-what-is-nextscript-used-for)

**🟡 Medium**
- [4. What does `ssr: false` do in a dynamic import?](#4-what-does-ssr-false-do-in-a-dynamic-import)
- [5. How do you optimize fonts in Next.js?](#5-how-do-you-optimize-fonts-in-nextjs)
- [6. What are common performance optimization techniques in Next.js?](#6-what-are-common-performance-optimization-techniques-in-nextjs)
- [7. How does caching work in the Pages Router?](#7-how-does-caching-work-in-the-pages-router)
- [8. How do you add meta tags / handle SEO in Next.js?](#8-how-do-you-add-meta-tags--handle-seo-in-nextjs)

**🔴 Hard**
- [9. How do you use placeholders with the `next/image` component?](#9-how-do-you-use-placeholders-with-the-nextimage-component)
- [10. What is AMP, and how do you enable it in Next.js?](#10-what-is-amp-and-how-do-you-enable-it-in-nextjs)
- [11. How do you add a sitemap to a Next.js app?](#11-how-do-you-add-a-sitemap-to-a-nextjs-app)
- [12. What is `next-seo`, and what does it simplify?](#12-what-is-next-seo-and-what-does-it-simplify)
- [13. How does cache revalidation work in the Pages Router?](#13-how-does-cache-revalidation-work-in-the-pages-router)
- [14. What performance differences exist between the Pages Router and App Router caching models?](#14-what-performance-differences-exist-between-the-pages-router-and-app-router-caching-models)

---

### 1. What is the `Image` component, and why use it over a plain `<img>`? 🟢

- `next/image` automatically optimizes images: lazy loading by default, automatic resizing/format conversion (e.g. to WebP) per device, and preventing layout shift by requiring width/height up front.

```jsx
import Image from 'next/image';
<Image src="/photo.jpg" width={400} height={300} alt="A photo" />
```

[↑ Back to top](#table-of-contents)

---

### 2. What is dynamic import in Next.js? 🟢

- `next/dynamic` lazily loads a component **only when needed**, splitting it into a separate chunk — reduces initial bundle size for components not needed on first render (modals, heavy widgets).

```jsx
import dynamic from 'next/dynamic';
const HeavyChart = dynamic(() => import('../components/HeavyChart'));
```

[↑ Back to top](#table-of-contents)

---

### 3. What is `next/script` used for? 🟢

- Loading third-party scripts (analytics, ads, chat widgets) with control over **loading strategy** (`beforeInteractive`, `afterInteractive`, `lazyOnload`) — avoids them blocking the main thread/page load the way a plain `<script>` tag might.

```jsx
import Script from 'next/script';
<Script src="https://example.com/widget.js" strategy="lazyOnload" />
```

[↑ Back to top](#table-of-contents)

---

### 4. What does `ssr: false` do in a dynamic import? 🟡

- Disables server-side rendering for that specific component — it renders **only on the client**. Necessary for components depending on browser-only APIs (`window`, `document`) that would crash during server rendering.

```jsx
const MapWidget = dynamic(() => import('../components/MapWidget'), { ssr: false });
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you optimize fonts in Next.js? 🟡

- `next/font` self-hosts fonts (including Google Fonts) at build time — no runtime request to an external font CDN, automatic `font-display` handling, and no layout shift from late-loading fonts.

```jsx
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'] });
<html className={inter.className}>
```

[↑ Back to top](#table-of-contents)

---

### 6. What are common performance optimization techniques in Next.js? 🟡

- Use `next/image` and `next/font` for automatic media/font optimization, code-split heavy/rarely-used components with `next/dynamic`, choose the right rendering strategy per page (don't SSR what could be static), minimize Client Component boundaries in the App Router, and analyze bundle size with `@next/bundle-analyzer`.

[↑ Back to top](#table-of-contents)

---

### 7. How does caching work in the Pages Router? 🟡

- Pages generated via `getStaticProps` are cached as static HTML (served from CDN); `getServerSideProps` pages are **never** cached by default (fresh on every request) unless you manually set `Cache-Control` headers in the response.

[↑ Back to top](#table-of-contents)

---

### 8. How do you add meta tags / handle SEO in Next.js? 🟡

- Pages Router: use `next/head` per-page.
- App Router: export a `metadata` object (or `generateMetadata` function for dynamic values) from a page/layout — Next.js injects the appropriate `<head>` tags automatically.

```jsx
// App Router
export const metadata = { title: 'My Page', description: '...' };
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you use placeholders with the `next/image` component? 🔴

- `placeholder="blur"` shows a blurred low-res version while the full image loads — pass `blurDataURL` for a remote image, or it's generated automatically for static imports.

```jsx
import Image from 'next/image';
import photo from '../public/photo.jpg';
<Image src={photo} placeholder="blur" alt="A photo" /> {/* blurDataURL auto-generated for static imports */}
```

[↑ Back to top](#table-of-contents)

---

### 10. What is AMP, and how do you enable it in Next.js? 🔴

- AMP (Accelerated Mobile Pages) is a stripped-down HTML standard for extremely fast-loading mobile pages, mainly historically pushed for SEO/search ranking benefits. Next.js had built-in support — export `const config = { amp: true }` (or `'hybrid'` for both AMP and regular versions) from a page.
- Largely a **legacy/niche concern today** — Google de-emphasized AMP as a ranking factor, so few new projects adopt it.

[↑ Back to top](#table-of-contents)

---

### 11. How do you add a sitemap to a Next.js app? 🔴

- App Router: export a `sitemap.js` file from the `app/` directory returning an array of URL entries — Next.js serves it at `/sitemap.xml` automatically.

```js
// app/sitemap.js
export default function sitemap() {
  return [
    { url: 'https://example.com', lastModified: new Date() },
    { url: 'https://example.com/about', lastModified: new Date() },
  ];
}
```

[↑ Back to top](#table-of-contents)

---

### 12. What is `next-seo`, and what does it simplify? 🔴

- A community library that simplifies adding common SEO tags (Open Graph, Twitter cards, canonical URLs, JSON-LD structured data) — particularly useful in the Pages Router, where you'd otherwise hand-write a lot of repetitive `next/head` boilerplate per page.

[↑ Back to top](#table-of-contents)

---

### 13. How does cache revalidation work in the Pages Router? 🔴

- Controlled via the `revalidate` value returned from `getStaticProps` (ISR, see [Rendering Strategies](rendering-strategies.md#10-what-is-incremental-static-regeneration-isr)) — Next.js serves the existing cached page instantly while regenerating it **in the background** once the revalidate window has passed, rather than blocking the request on a rebuild.

[↑ Back to top](#table-of-contents)

---

### 14. What performance differences exist between the Pages Router and App Router caching models? 🔴

- Pages Router caching is **page-level** and tied to which data function you used — coarse-grained.
- App Router caching is **per-fetch-call** and layered (request memoization, data cache, full route cache, router cache on the client) — finer-grained control, but a steeper learning curve to reason about exactly what's cached and when it's invalidated.

> [!IMPORTANT]
> **Follow-up questions:**
> - What are the four distinct caching layers Next.js's App Router documentation describes?

[↑ Back to top](#table-of-contents)

---
