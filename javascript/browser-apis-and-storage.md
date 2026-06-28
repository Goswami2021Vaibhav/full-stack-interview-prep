# Browser APIs & Storage

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between `localStorage`, `sessionStorage`, and cookies?](#1-whats-the-difference-between-localstorage-sessionstorage-and-cookies)
- [2. What is the same-origin policy?](#2-what-is-the-same-origin-policy)
- [3. What is CORS, and why does it exist?](#3-what-is-cors-and-why-does-it-exist)
- [4. What's the difference between `fetch()` and `XMLHttpRequest`?](#4-whats-the-difference-between-fetch-and-xmlhttprequest)
- [5. What is AJAX?](#5-what-is-ajax)

**🟡 Medium**
- [6. Can you fix a CORS error from the client side?](#6-can-you-fix-a-cors-error-from-the-client-side)
- [7. What is JSONP, and why was it used before CORS was widely supported?](#7-what-is-jsonp-and-why-was-it-used-before-cors-was-widely-supported)
- [8. What is a WebSocket, and how is it different from a regular HTTP request?](#8-what-is-a-websocket-and-how-is-it-different-from-a-regular-http-request)
- [9. What are Server-Sent Events (SSE), and how do they differ from WebSockets?](#9-what-are-server-sent-events-sse-and-how-do-they-differ-from-websockets)
- [10. What is XSS, and how do you prevent it?](#10-what-is-xss-and-how-do-you-prevent-it)
- [11. What is CSRF, and how do you prevent it?](#11-what-is-csrf-and-how-do-you-prevent-it)

**🔴 Hard**
- [12. What is Content Security Policy (CSP)?](#12-what-is-content-security-policy-csp)
- [13. How would you implement a request cache/dedupe layer on top of `fetch`?](#13-how-would-you-implement-a-request-cachededupe-layer-on-top-of-fetch)
- [14. When would you reach for IndexedDB instead of `localStorage`?](#14-when-would-you-reach-for-indexeddb-instead-of-localstorage)
- [15. How does the browser decide whether to send a CORS preflight (`OPTIONS`) request?](#15-how-does-the-browser-decide-whether-to-send-a-cors-preflight-options-request)

---

### 1. What's the difference between `localStorage`, `sessionStorage`, and cookies? 🟢

| | Capacity | Lifetime | Sent to server? |
|---|---|---|---|
| `localStorage` | ~5-10MB | Persists until explicitly cleared | No |
| `sessionStorage` | ~5-10MB | Cleared when the tab closes | No |
| Cookies | ~4KB | Configurable expiry | Yes, with **every** matching HTTP request |

- Cookies are the only one of the three accessible from the **server** (via response headers), and the only one automatically sent on every request — which is also why oversized cookies hurt performance.

```js
localStorage.setItem('theme', 'dark');
sessionStorage.setItem('draft', 'hello');
document.cookie = 'sessionId=abc123; max-age=3600';
```

[↑ Back to top](#table-of-contents)

---

### 2. What is the same-origin policy? 🟢

- A browser security rule: a script running on one **origin** (protocol + domain + port) can't read data from a different origin's page/response, unless that other origin explicitly allows it.
- Exists to stop a malicious site from reading your logged-in data on another site (e.g. your bank) just because both happen to be open in your browser.

```
https://example.com:443
└── protocol: https, domain: example.com, port: 443 — all three must match for "same-origin"
```

[↑ Back to top](#table-of-contents)

---

### 3. What is CORS, and why does it exist? 🟢

- **C**ross-**O**rigin **R**esource **S**haring — a set of HTTP headers that let a **server** explicitly tell the browser "it's okay for this other origin to read my response."
- It's a relaxation mechanism for the same-origin policy — without it, no legitimate cross-origin API calls (e.g. your frontend on `app.com` calling an API on `api.com`) would work in the browser.

```
Access-Control-Allow-Origin: https://app.com
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Is CORS enforced by the browser or the server?
> - What happens if the server doesn't send the right CORS headers?

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between `fetch()` and `XMLHttpRequest`? 🟢

- `fetch()`: modern, Promise-based, cleaner syntax, supports streaming response bodies.
- `XMLHttpRequest` (XHR): older, callback/event-based (`onreadystatechange`), more verbose — but still needed for things `fetch` doesn't natively support well, like upload progress events.

```js
// fetch
const res = await fetch('/api/data');
const data = await res.json();

// XHR
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data');
xhr.onload = () => console.log(JSON.parse(xhr.responseText));
xhr.send();
```

[↑ Back to top](#table-of-contents)

---

### 5. What is AJAX? 🟢

- **A**synchronous **J**avaScript **A**nd **X**ML — the general technique of making background HTTP requests from JS without reloading the page (despite the name, JSON is far more common than XML today).
- `fetch()` and `XMLHttpRequest` are both ways of doing AJAX, not separate concepts from it.

[↑ Back to top](#table-of-contents)

---

### 6. Can you fix a CORS error from the client side? 🟡

- Almost always **no** — CORS is enforced by the browser based on headers the **server** sends back. If the server doesn't allow your origin, no client-side trick (changing fetch options, adding headers) makes the browser ignore that.
- The only real client-side "workarounds" are routing the request through a proxy you control (which then talks server-to-server, where CORS doesn't apply) or during development, a dev-server proxy config.

[↑ Back to top](#table-of-contents)

---

### 7. What is JSONP, and why was it used before CORS was widely supported? 🟡

- A hack that exploited the fact that `<script src="...">` tags **aren't** restricted by the same-origin policy — the "API" returns JS that calls a global callback function with the data, instead of returning plain JSON.
- Obsolete now that CORS is universally supported — also a security risk (executes arbitrary returned script), so avoid it in new code.

```js
function handleData(data) { console.log(data); }
// the script tag's src points to an endpoint that returns:
// handleData({ ...someJsonData });
```

[↑ Back to top](#table-of-contents)

---

### 8. What is a WebSocket, and how is it different from a regular HTTP request? 🟡

- HTTP is request-response: the client asks, the server answers, connection closes (or is reused, but each exchange is still request-driven).
- A WebSocket opens a **persistent, full-duplex** connection — either side can push messages at any time, without the overhead of a new HTTP request each time.

```js
const socket = new WebSocket('wss://example.com/chat');
socket.onmessage = (event) => console.log('Received:', event.data);
socket.send('Hello server!');
```

> [!TIP]
> **Real-life example:** live chat apps, real-time multiplayer games, and live stock tickers all rely on WebSockets for instant bidirectional updates.

[↑ Back to top](#table-of-contents)

---

### 9. What are Server-Sent Events (SSE), and how do they differ from WebSockets? 🟡

- SSE: a one-way stream from **server to client only**, built on plain HTTP via the `EventSource` API — simpler than WebSockets when the client never needs to push data back.
- WebSockets: full **two-way** communication, but requires a different protocol/handshake and more setup.

```js
const events = new EventSource('/api/notifications');
events.onmessage = (e) => console.log('New notification:', e.data);
```

[↑ Back to top](#table-of-contents)

---

### 10. What is XSS, and how do you prevent it? 🟡

- **Cross-Site Scripting**: an attacker injects malicious JS into a page (e.g. via unsanitized user input rendered as HTML), which then runs in other users' browsers with full access to that page's data/cookies.
- Prevent by: never injecting raw user input via `innerHTML` (use `textContent` or a framework's auto-escaping templating), sanitizing any HTML you must render, and setting a Content Security Policy (Q12).

```js
// Vulnerable
el.innerHTML = userComment; // if userComment = '<img src=x onerror="stealCookies()">'

// Safer
el.textContent = userComment; // rendered as literal text, script doesn't execute
```

[↑ Back to top](#table-of-contents)

---

### 11. What is CSRF, and how do you prevent it? 🟡

- **Cross-Site Request Forgery**: a malicious site tricks a logged-in user's browser into making an unwanted request to another site where they're authenticated (the browser automatically attaches cookies, so the request looks legitimate).
- Prevent by: CSRF tokens (a random token the server expects back on state-changing requests, which an attacker's site can't read/forge), `SameSite` cookie attributes, and checking the `Origin`/`Referer` header for sensitive actions.

[↑ Back to top](#table-of-contents)

---

### 12. What is Content Security Policy (CSP)? 🔴

- A response header that tells the browser which sources of scripts, styles, images, etc. are allowed to load/execute on the page — anything not on the allowlist is blocked, even if injected via an XSS attack.

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com
```

- This is a strong **defense-in-depth** layer against XSS: even if an attacker manages to inject a `<script>` tag, the browser will refuse to execute it if it violates the policy.

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement a request cache/dedupe layer on top of `fetch`? 🔴

- Cache in-flight and completed request promises keyed by URL, so duplicate concurrent calls reuse the same promise instead of firing multiple network requests.

```js
const cache = new Map();

function cachedFetch(url) {
  if (cache.has(url)) return cache.get(url);
  const promise = fetch(url).then((res) => res.json());
  cache.set(url, promise);
  return promise;
}

// Both calls share the same in-flight request instead of firing twice
cachedFetch('/api/user');
cachedFetch('/api/user');
```

[↑ Back to top](#table-of-contents)

---

### 14. When would you reach for IndexedDB instead of `localStorage`? 🔴

- `localStorage` is synchronous, string-only, and capped around 5-10MB — fine for small settings/flags, but it can **block the main thread** and isn't suited for structured or large data.
- `IndexedDB` is asynchronous, supports much larger storage, structured data (objects, indexes, transactions), and querying — the right choice for offline-first apps, caching large datasets, or storing files/blobs client-side.

[↑ Back to top](#table-of-contents)

---

### 15. How does the browser decide whether to send a CORS preflight (`OPTIONS`) request? 🔴

- A **simple request** (GET/HEAD/POST with only standard headers and a few allowed `Content-Type`s like `application/x-www-form-urlencoded`) skips the preflight and goes straight through.
- Anything else — custom headers (`Authorization`, custom `X-` headers), methods like `PUT`/`DELETE`/`PATCH`, or `Content-Type: application/json` — triggers the browser to first send an `OPTIONS` preflight request, asking the server "are you okay with this exact request?" before sending the real one.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does sending JSON (`Content-Type: application/json`) trigger a preflight, when it seems like a "simple" request?
> - What headers does the server need to respond with for the preflight to succeed?

[↑ Back to top](#table-of-contents)

---
