# Request & Response

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are the most commonly used properties on the `req` object?](#1-what-are-the-most-commonly-used-properties-on-the-req-object)
- [2. What are the most commonly used methods on the `res` object?](#2-what-are-the-most-commonly-used-methods-on-the-res-object)
- [3. How do you send a JSON response?](#3-how-do-you-send-a-json-response)

**🟡 Medium**
- [4. What's the difference between `res.send()`, `res.json()`, and `res.end()`?](#4-whats-the-difference-between-ressend-resjson-and-resend)
- [5. How do you set custom response headers?](#5-how-do-you-set-custom-response-headers)
- [6. How do you set the HTTP status code of a response?](#6-how-do-you-set-the-http-status-code-of-a-response)
- [7. How do you access request body data?](#7-how-do-you-access-request-body-data)
- [8. How do you handle redirects in Express?](#8-how-do-you-handle-redirects-in-express)

**🔴 Hard**
- [9. What's the difference between `req.params`, `req.query`, and `req.body`?](#9-whats-the-difference-between-reqparams-reqquery-and-reqbody)
- [10. How do you stream a response instead of buffering it fully?](#10-how-do-you-stream-a-response-instead-of-buffering-it-fully)
- [11. How does Express handle content negotiation?](#11-how-does-express-handle-content-negotiation)
- [12. How would you implement response-time logging using `res` events?](#12-how-would-you-implement-response-time-logging-using-res-events)

---

### 1. What are the most commonly used properties on the `req` object? 🟢

- `req.params` (route parameters), `req.query` (query string), `req.body` (parsed body, needs body-parsing middleware), `req.headers`, `req.method`, `req.path`/`req.originalUrl`, `req.cookies` (needs `cookie-parser`).

[↑ Back to top](#table-of-contents)

---

### 2. What are the most commonly used methods on the `res` object? 🟢

- `res.send()`, `res.json()`, `res.status()`, `res.redirect()`, `res.render()` (templates), `res.set()`/`res.header()` (headers), `res.cookie()`, `res.end()`.

[↑ Back to top](#table-of-contents)

---

### 3. How do you send a JSON response? 🟢

```js
app.get('/users', (req, res) => {
  res.json({ id: 1, name: 'Vaibhav' }); // sets Content-Type: application/json automatically
});
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between `res.send()`, `res.json()`, and `res.end()`? 🟡

- `res.json(obj)`: serializes to JSON and sets the `Content-Type` header accordingly — always.
- `res.send(data)`: more general — accepts a string, Buffer, object (auto-converts to JSON), or array; infers content type based on the input.
- `res.end([data])`: the lowest-level — ends the response with optional raw data, no content-type inference or formatting at all.

[↑ Back to top](#table-of-contents)

---

### 5. How do you set custom response headers? 🟡

```js
res.set('X-Custom-Header', 'value');
// or
res.header('X-Custom-Header', 'value');
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you set the HTTP status code of a response? 🟡

```js
res.status(201).json({ created: true });
res.sendStatus(204); // sets status AND sends the status text as the body
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you access request body data? 🟡

- Requires body-parsing middleware first (`express.json()` for JSON, `express.urlencoded()` for form data) — then it's available on `req.body`.

```js
app.use(express.json());
app.post('/users', (req, res) => {
  console.log(req.body.name);
});
```

[↑ Back to top](#table-of-contents)

---

### 8. How do you handle redirects in Express? 🟡

```js
res.redirect('/login');           // defaults to 302
res.redirect(301, '/new-location'); // explicit permanent redirect
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `req.params`, `req.query`, and `req.body`? 🔴

- `req.params`: named segments from the **route path** (`/users/:id` → `req.params.id`).
- `req.query`: the **query string** after `?` (`/users?sort=asc` → `req.query.sort`).
- `req.body`: the **request body**, populated only after body-parsing middleware runs — typically used with `POST`/`PUT`/`PATCH`.

[↑ Back to top](#table-of-contents)

---

### 10. How do you stream a response instead of buffering it fully? 🔴

- Pipe a readable stream (a file, a database cursor, a transform stream) directly into `res`, which is itself a writable stream — avoids loading the entire payload into memory before sending.

```js
app.get('/download', (req, res) => {
  const fileStream = fs.createReadStream('large-file.csv');
  fileStream.pipe(res); // streams directly to the client, chunk by chunk
});
```

[↑ Back to top](#table-of-contents)

---

### 11. How does Express handle content negotiation? 🔴

- `res.format()` picks a response representation based on the client's `Accept` header, letting one route serve multiple formats.

```js
app.get('/users/:id', (req, res) => {
  res.format({
    json: () => res.json({ id: req.params.id }),
    html: () => res.send(`<p>User ${req.params.id}</p>`),
    default: () => res.status(406).send('Not Acceptable'),
  });
});
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement response-time logging using `res` events? 🔴

- `res` emits a `'finish'` event once the response has been fully sent — record a timestamp at the start of the middleware chain, then compute the elapsed time when `'finish'` fires.

```js
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.originalUrl} - ${res.statusCode} - ${Date.now() - start}ms`);
  });
  next();
});
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why use the `'finish'` event instead of just logging right before calling `res.send()` in every route?

[↑ Back to top](#table-of-contents)

---
