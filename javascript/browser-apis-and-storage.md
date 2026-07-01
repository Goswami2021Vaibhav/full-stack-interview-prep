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

- All three are ways to store small pieces of data in the browser, but they differ in how long the data lasts, how much you can store, and who can see it.
- `localStorage`: data written here stays on the user's device indefinitely — it survives closing the tab, closing the browser, even restarting the computer — until code explicitly removes it (`removeItem`/`clear`) or the user clears their browser data.
- `sessionStorage`: works the same way as `localStorage`, but the data is wiped as soon as that specific browser tab is closed. Each tab also gets its own separate `sessionStorage`, even for the same site.
- Cookies: much smaller capacity (~4KB), but they're the only one of the three that the **server** can read and write too (via HTTP headers), and the only one automatically attached to every matching HTTP request the browser makes. That auto-sending is convenient for things like login sessions, but it also means large or numerous cookies add extra weight to every request — hurting performance.

```js
localStorage.setItem('theme', 'dark');       // persists across browser restarts
sessionStorage.setItem('draft', 'hello');    // gone once this tab closes
document.cookie = 'sessionId=abc123; max-age=3600'; // sent to the server on every request for 1 hour
```

[↑ Back to top](#table-of-contents)

---

### 2. What is the same-origin policy? 🟢

- The same-origin policy is a security rule built into every browser: a script running on one **origin** is not allowed to read data from a different origin's page or response, unless that other origin explicitly says it's okay (see CORS in Q3).
- An "origin" is defined by three parts together: the **protocol** (`http` vs `https`), the **domain** (`example.com`), and the **port** (`443`, `3000`, etc.). If even one of these three differs between two URLs, they're considered different origins.
- Without this rule, if you had your bank's website open in one tab and a malicious website open in another tab, JavaScript on the malicious site could silently make requests to your bank and read the responses (since your browser sends your login cookies automatically) — the same-origin policy is what prevents exactly that scenario.

```
https://example.com:443
└── protocol: https, domain: example.com, port: 443 — all three must match for "same-origin"

https://example.com   vs   http://example.com   → different origin (protocol differs)
https://example.com   vs   https://api.example.com → different origin (domain differs)
```

[↑ Back to top](#table-of-contents)

---

### 3. What is CORS, and why does it exist? 🟢

- CORS stands for **C**ross-**O**rigin **R**esource **S**haring. It's a set of special HTTP response headers that let a **server** explicitly grant permission: "it's okay for a script running on this other origin to read my response."
- CORS exists to safely relax the same-origin policy (Q2) for legitimate use cases. Without it, a frontend running on `app.com` could never successfully call an API on a different origin like `api.com` from the browser — the browser would just block reading the response, even if the request itself technically went through.
- Note that CORS is enforced entirely by the **browser**, based on headers the **server** chooses to send back. The server always receives the request either way — CORS only controls whether the browser lets your JavaScript code read the response.

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

- `fetch()` is the modern way to make HTTP requests from JavaScript. It's built around Promises, so it works naturally with `.then()`/`.catch()` or `async`/`await`, resulting in noticeably cleaner code. It also supports streaming a response body incrementally, instead of only getting it all at once.
- `XMLHttpRequest` (often shortened to "XHR") is the older API that predates Promises entirely. It's event-based — you attach listeners like `onreadystatechange` or `onload` and check status codes manually, which tends to be more verbose.
- `XHR` isn't fully obsolete: it still supports a few things `fetch()` doesn't handle natively, most notably **upload progress events** (tracking how much of a file has been uploaded so far), which is why some file-upload code still uses it.

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

- AJAX stands for **A**synchronous **J**avaScript **A**nd **X**ML. It's not a specific technology or API — it's the general **technique** of a web page making HTTP requests to a server in the background, and updating parts of the page with the response, all without a full page reload.
- The name is a bit of a historical artifact: AJAX was coined when XML was the common data format for these requests, but today almost everyone uses JSON instead — the "XML" in the name stuck around anyway.
- `fetch()` and `XMLHttpRequest` are both simply tools you can use **to do** AJAX — they're not competing concepts, AJAX is just the umbrella term for the pattern itself.

[↑ Back to top](#table-of-contents)

---

### 6. Can you fix a CORS error from the client side? 🟡

- Almost always **no**. CORS is a permission decision enforced by the **browser**, based entirely on headers that the **server** chooses to send back. If the server hasn't allowed your origin, there's no fetch option, custom header, or client-side trick that makes the browser ignore that rule — the restriction isn't a bug to work around, it's working as designed.
- The practical "workarounds" don't actually bypass CORS — they avoid triggering it in the first place:
  - Route the request through a **proxy server you control**. Your frontend calls your own backend (same origin, so no CORS issue), and your backend makes the actual cross-origin request server-to-server, where CORS rules don't apply at all (CORS is a browser-only concept).
  - During local development, configure a **dev-server proxy** (e.g. Vite's or webpack-dev-server's proxy config) so requests appear to come from the same origin during development.
- The real fix is always on the server side: it needs to add the correct `Access-Control-Allow-Origin` (and related) headers for your origin.

[↑ Back to top](#table-of-contents)

---

### 7. What is JSONP, and why was it used before CORS was widely supported? 🟡

- JSONP ("JSON with Padding") is an old workaround from before CORS existed, built on a loophole: the same-origin policy blocks reading `fetch`/XHR responses from other origins, but it does **not** block loading a `<script src="...">` from another origin — script tags have always been allowed to load cross-origin.
- The trick: instead of an endpoint returning plain JSON data, it returns actual JavaScript code — a call to a function name you specify (the "callback"), with the data passed as an argument. The browser loads this as a `<script>` tag, which executes immediately and calls your globally-defined function with the data inside it.
- It's considered obsolete today because CORS is now supported everywhere, and it's also risky: since the response is executed as **real JavaScript** (not parsed as inert data like JSON), a compromised or malicious endpoint could run arbitrary code on your page. Avoid it in new code.

```js
function handleData(data) { console.log(data); }
// A <script src="https://api.example.com/data?callback=handleData"> is added to the page.
// Instead of returning JSON, the endpoint returns actual JS code that executes on load:
// handleData({ ...someJsonData });
```

[↑ Back to top](#table-of-contents)

---

### 8. What is a WebSocket, and how is it different from a regular HTTP request? 🟡

- A regular HTTP request follows a strict request-response pattern: the client sends a request, the server sends back a response, and that exchange is done. Even if the underlying connection is reused for the next request (a performance optimization), each exchange is still initiated by the client — the server can never spontaneously push new data without the client asking first.
- A WebSocket instead opens one **persistent, full-duplex** connection between client and server — "full-duplex" meaning both sides can send messages independently, at any time, in either direction, over that same open connection. There's no need to start a brand-new request for each message, which removes a lot of overhead for frequent, real-time communication.
- Once you create a `WebSocket`, you get event handlers like `onmessage` (fires when the server sends data) and a `.send()` method (to send data to the server) — both sides can use these at will, whenever they have something new to communicate.

```js
const socket = new WebSocket('wss://example.com/chat'); // wss:// is the secure WebSocket protocol
socket.onmessage = (event) => console.log('Received:', event.data); // server pushed a message
socket.send('Hello server!'); // client sends a message, no new HTTP request needed
```

> [!TIP]
> **Real-life example:** live chat apps, real-time multiplayer games, and live stock tickers all rely on WebSockets for instant bidirectional updates.

[↑ Back to top](#table-of-contents)

---

### 9. What are Server-Sent Events (SSE), and how do they differ from WebSockets? 🟡

- SSE lets a server continuously push updates to the client over a single, long-lived connection — but it's strictly **one-way**: server to client only, with no way for the client to send messages back over that same channel (it would use a regular HTTP request for that instead).
- Crucially, SSE is built on **plain HTTP**, not a special protocol — it uses the `EventSource` browser API, which automatically handles reconnecting if the connection drops. This makes it noticeably simpler to set up than WebSockets for cases where the client only ever needs to *receive* live updates.
- WebSockets support full **two-way** communication (see Q8) but require their own protocol handshake (`ws://`/`wss://`) and more setup on both client and server. Use SSE for read-only live feeds (e.g. notifications, live scores); use WebSockets when the client also needs to send frequent messages back (e.g. chat, multiplayer games).

```js
const events = new EventSource('/api/notifications');
events.onmessage = (e) => console.log('New notification:', e.data); // server pushes; client never sends back on this connection
```

[↑ Back to top](#table-of-contents)

---

### 10. What is XSS, and how do you prevent it? 🟡

- **Cross-Site Scripting (XSS)** happens when an attacker manages to get their own malicious JavaScript to run inside your page, as if it were your own trusted code. A common way this happens: a page takes user input (e.g. a comment) and renders it directly as raw HTML without checking what's inside it — if that input actually contains a `<script>` tag or an HTML attribute with embedded JS, the browser will execute it.
- Once that malicious script runs, it has the same access as your legitimate code: it can read cookies, make requests as the logged-in user, modify the page, steal form input, etc. This is dangerous because it runs in **other users'** browsers, not just the attacker's.
- Main defenses:
  - Never insert raw, untrusted user input into the page using `innerHTML` (or `dangerouslySetInnerHTML` in React). Use `textContent` instead, which always renders the input as plain visible text, never as executable markup.
  - If you genuinely need to render user-supplied HTML (e.g. rich text), run it through a dedicated **sanitizing library** first, which strips out dangerous tags/attributes.
  - Add a Content Security Policy (see Q12) as an extra safety net, so even if malicious script slips through, the browser refuses to run it.

```js
// Vulnerable — if userComment = '<img src=x onerror="stealCookies()">', the onerror JS runs
el.innerHTML = userComment;

// Safer — always shown as literal text on the page, the script never executes
el.textContent = userComment;
```

[↑ Back to top](#table-of-contents)

---

### 11. What is CSRF, and how do you prevent it? 🟡

- **Cross-Site Request Forgery (CSRF)** exploits the fact that browsers automatically attach cookies (like your login session) to any request sent to a site, no matter which page triggered that request. An attacker's unrelated website can include something like a hidden auto-submitting form that sends a request to, say, your bank's "transfer money" endpoint. Your browser helpfully attaches your real login cookie, so the bank's server sees what looks like a legitimate, authenticated request from you — even though you never intended it.
- Main defenses:
  - **CSRF tokens**: the server includes a random, unpredictable token in the legitimate page (e.g. in a form field), and requires that exact token to be sent back on any state-changing request. An attacker's foreign site has no way to read or guess this token, so its forged request gets rejected.
  - **`SameSite` cookie attribute**: tells the browser not to send a cookie along with requests that originate from a different site, which blocks the attack at the browser level.
  - Checking the `Origin`/`Referer` header on sensitive requests, to confirm the request actually came from your own site.

[↑ Back to top](#table-of-contents)

---

### 12. What is Content Security Policy (CSP)? 🔴

- Content Security Policy (CSP) is a response header where a site declares an **allowlist**: exactly which sources are trusted to provide scripts, stylesheets, images, fonts, etc. for that page. The browser enforces this at the network/execution level — anything not on the allowlist simply won't be loaded or executed, no exceptions.
- The key benefit is that this check happens regardless of *how* the disallowed content got onto the page. Even if an XSS attack (Q10) successfully injects a `<script>` tag into your page's HTML, the browser will still refuse to execute that script if it isn't from an allowed source (or if inline scripts are disallowed) — CSP acts as a safety net that catches attacks even after they've technically slipped through.
- This is why CSP is called a **defense-in-depth** measure: it doesn't prevent the injection itself, but it limits the damage an attacker can do even if they succeed at injecting something.

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com
```
- `default-src 'self'` means "by default, only load resources from this same origin."
- `script-src 'self' https://trusted-cdn.com` overrides that specifically for scripts, additionally allowing a trusted CDN.

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement a request cache/dedupe layer on top of `fetch`? 🔴

- The key idea: store the **Promise itself** in a cache (keyed by the request URL), not just the eventual resolved data. This matters because if two calls for the same URL happen close together, the second call can reuse the still-pending Promise from the first call, instead of firing off a second, redundant network request — this is the "dedupe" part.
- Steps: check the cache for that URL first. If found, return the cached Promise (whether it's still pending or already resolved — callers can `await` it either way). If not found, start the `fetch`, immediately store the resulting Promise in the cache before it resolves, and return it.
- A more complete version would also add cache expiry (so stale data doesn't get served forever) and error handling (so a failed request doesn't get "stuck" cached forever as a rejected Promise).

```js
const cache = new Map();

function cachedFetch(url) {
  if (cache.has(url)) return cache.get(url); // reuse existing (maybe still pending) request
  const promise = fetch(url).then((res) => res.json());
  cache.set(url, promise); // store the Promise itself, before it has resolved
  return promise;
}

// Both calls share the same in-flight request instead of firing two network requests
cachedFetch('/api/user');
cachedFetch('/api/user');
```

[↑ Back to top](#table-of-contents)

---

### 14. When would you reach for IndexedDB instead of `localStorage`? 🔴

- `localStorage` is convenient for small, simple data, but it has real limitations: it's **synchronous** (every read/write briefly blocks the main thread — the same thread that handles rendering and user interaction — which can cause jank with larger data), it can only store **strings** (objects must be manually converted with `JSON.stringify`/`JSON.parse`), and it's capped at roughly 5-10MB.
- `IndexedDB` is a full, browser-built-in database designed for much bigger and more structured needs: it's **asynchronous** (doesn't block the main thread), can store structured data directly (objects, files, blobs) without manual serialization, supports **indexes** for efficient querying, and **transactions** for consistency, and can hold far more data (often hundreds of MB or more, browser-dependent).
- Use `localStorage` for small bits of state like a theme preference or a flag. Reach for `IndexedDB` when building offline-first apps, caching large API responses/datasets for offline use, or storing files/blobs on the client.

[↑ Back to top](#table-of-contents)

---

### 15. How does the browser decide whether to send a CORS preflight (`OPTIONS`) request? 🔴

- Before making certain cross-origin requests, the browser first sends a "test" `OPTIONS` request (called a **preflight**) to check with the server whether the real request is allowed, before sending the actual one. This adds an extra round-trip, so browsers only do it when necessary.
- A request qualifies as a **simple request** — and skips the preflight, going straight through — only if it meets all these conditions: it uses `GET`, `HEAD`, or `POST`; it only sets a small set of standard headers; and if it sets `Content-Type`, it's one of a few plain formats like `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`.
- Anything that doesn't meet those narrow conditions triggers a preflight — most commonly: custom headers (like an `Authorization` token, or custom `X-` headers), HTTP methods like `PUT`/`DELETE`/`PATCH`, or a `Content-Type` of `application/json` (since JSON isn't in that small allowed list). The browser sends the `OPTIONS` preflight first, and only proceeds with the real request if the server's response headers confirm it's allowed.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does sending JSON (`Content-Type: application/json`) trigger a preflight, when it seems like a "simple" request?
> - What headers does the server need to respond with for the preflight to succeed?

[↑ Back to top](#table-of-contents)

---
