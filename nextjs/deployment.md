# Deployment

_Part of [Next.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you deploy a Next.js app to Vercel?](#1-how-do-you-deploy-a-nextjs-app-to-vercel)
- [2. What is the `next export` command used for?](#2-what-is-the-next-export-command-used-for)
- [3. How do you handle redirects in Next.js?](#3-how-do-you-handle-redirects-in-nextjs)

**🟡 Medium**
- [4. How do you add custom headers in Next.js?](#4-how-do-you-add-custom-headers-in-nextjs)
- [5. How do you configure Webpack in Next.js?](#5-how-do-you-configure-webpack-in-nextjs)
- [6. How do you handle internationalization (i18n) in Next.js?](#6-how-do-you-handle-internationalization-i18n-in-nextjs)
- [7. How do you add Google Analytics to a Next.js project?](#7-how-do-you-add-google-analytics-to-a-nextjs-project)

**🔴 Hard**
- [8. How should you safely expose environment variables to the client?](#8-how-should-you-safely-expose-environment-variables-to-the-client)
- [9. What common security practices should you follow in a Next.js app?](#9-what-common-security-practices-should-you-follow-in-a-nextjs-app)
- [10. How do you configure a custom Babel setup in Next.js?](#10-how-do-you-configure-a-custom-babel-setup-in-nextjs)
- [11. What deployment considerations differ between SSR/ISR apps and a fully static export?](#11-what-deployment-considerations-differ-between-ssrisr-apps-and-a-fully-static-export)
- [12. What security considerations apply to a Next.js app deployed on the Edge?](#12-what-security-considerations-apply-to-a-nextjs-app-deployed-on-the-edge)

---

### 1. How do you deploy a Next.js app to Vercel? 🟢

- Push the repo to GitHub/GitLab/Bitbucket, then import it in Vercel (the company behind Next.js) — it auto-detects the framework, builds, and deploys on every push, with zero config needed for most apps.

[↑ Back to top](#table-of-contents)

---

### 2. What is the `next export` command used for? 🟢

- Generates a fully **static** HTML/CSS/JS output with no Node.js server required — deployable to any static host (S3, GitHub Pages). Only works for pages that don't rely on server-only features (`getServerSideProps`, API routes, ISR, middleware aren't supported in a static export).

[↑ Back to top](#table-of-contents)

---

### 3. How do you handle redirects in Next.js? 🟢

- Configure them in `next.config.js` for static, build-time-known redirects, or return a redirect from `getServerSideProps`/middleware for dynamic, runtime-determined ones.

```js
// next.config.js
module.exports = {
  async redirects() {
    return [{ source: '/old-page', destination: '/new-page', permanent: true }];
  },
};
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you add custom headers in Next.js? 🟡

```js
// next.config.js
module.exports = {
  async headers() {
    return [
      { source: '/api/:path*', headers: [{ key: 'X-Custom-Header', value: 'value' }] },
    ];
  },
};
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you configure Webpack in Next.js? 🟡

- Export a `webpack` function from `next.config.js` that receives and returns a modified config object — used for custom loaders, aliases, or plugins not covered by Next's defaults.

```js
module.exports = {
  webpack(config) {
    config.resolve.alias['@components'] = path.resolve(__dirname, 'components');
    return config;
  },
};
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you handle internationalization (i18n) in Next.js? 🟡

- Pages Router: built-in i18n routing config (`next.config.js`'s `i18n` field) handles locale-based sub-paths/domains automatically.
- App Router: no built-in routing i18n yet — typically implemented manually via a `[locale]` dynamic route segment plus a library like `next-intl` for translations.

```js
// next.config.js (Pages Router)
module.exports = {
  i18n: { locales: ['en', 'fr'], defaultLocale: 'en' },
};
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you add Google Analytics to a Next.js project? 🟡

- Load the GA script via `next/script` (with an appropriate loading strategy), and track route changes manually since GA's default page-view tracking assumes full page reloads, which don't happen in a Next.js SPA-style navigation.

```jsx
<Script src={`https://www.googletagmanager.com/gtag/js?id=${GA_ID}`} strategy="afterInteractive" />
```

[↑ Back to top](#table-of-contents)

---

### 8. How should you safely expose environment variables to the client? 🔴

- Only prefix variables with `NEXT_PUBLIC_` if they're genuinely safe to be public (they get **inlined into the JS bundle** at build time, visible to anyone inspecting the site) — never put secrets (API keys, DB credentials) behind that prefix.
- Anything without the prefix stays server-only, accessible only in server-side code (API routes, `getServerSideProps`, Server Components).

[↑ Back to top](#table-of-contents)

---

### 9. What common security practices should you follow in a Next.js app? 🔴

- Validate/sanitize all input on API routes and Server Actions (never trust the client).
- Store secrets only in non-`NEXT_PUBLIC_` env vars.
- Use httpOnly cookies for auth tokens, not `localStorage`.
- Set security headers (CSP, `X-Frame-Options`) via `next.config.js` or middleware.
- Keep dependencies updated, and avoid `dangerouslySetInnerHTML` with unsanitized content.

[↑ Back to top](#table-of-contents)

---

### 10. How do you configure a custom Babel setup in Next.js? 🔴

- Add a `.babelrc` (or `babel.config.js`) at the project root — Next.js automatically merges it with its own default preset, **but** opting into a custom Babel config disables the faster Rust-based compiler (SWC) for the affected scope, so only do this if you specifically need a Babel-only plugin.

```json
{ "presets": ["next/babel"], "plugins": ["my-custom-plugin"] }
```

[↑ Back to top](#table-of-contents)

---

### 11. What deployment considerations differ between SSR/ISR apps and a fully static export? 🔴

- **SSR/ISR**: needs a Node.js (or Edge) runtime environment to handle per-request rendering/revalidation — can't be hosted on a plain static file host; needs Vercel, a Node server, or similar.
- **Static export** (`next export`): produces plain files deployable anywhere (even a basic CDN/S3 bucket), but loses access to any feature requiring a server at request time (SSR, ISR, API routes, middleware, image optimization's on-demand resizing).

[↑ Back to top](#table-of-contents)

---

### 12. What security considerations apply to a Next.js app deployed on the Edge? 🔴

- Edge functions run in a more restricted, sandboxed environment (no full Node.js APIs) — be mindful that secrets/env vars used there still need the same care as anywhere server-side (don't accidentally leak them in error messages/logs that might surface to a client).
- Since Edge middleware runs **before** routing on every matched request, a bug there (e.g. an overly broad matcher, or slow logic) impacts the security/performance of every request that matches it — test matchers carefully.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might caching at the Edge accidentally serve one user's personalized response to another user, if not configured carefully?

[↑ Back to top](#table-of-contents)

---
