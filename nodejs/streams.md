# Streams

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How many types of streams does Node.js have?](#1-how-many-types-of-streams-does-nodejs-have)
- [2. What problem do streams solve when handling large data?](#2-what-problem-do-streams-solve-when-handling-large-data)

**🟡 Medium**
- [3. What is backpressure in Node.js streams?](#3-what-is-backpressure-in-nodejs-streams)
- [4. What's the difference between `pipe()` and `pipeline()`?](#4-whats-the-difference-between-pipe-and-pipeline)
- [5. What's the difference between flowing mode and paused mode?](#5-whats-the-difference-between-flowing-mode-and-paused-mode)
- [6. What is `highWaterMark`, and how does it affect performance?](#6-what-is-highwatermark-and-how-does-it-affect-performance)

**🔴 Hard**
- [7. How do you implement a custom Duplex or Transform stream?](#7-how-do-you-implement-a-custom-duplex-or-transform-stream)
- [8. How do you handle stream errors properly?](#8-how-do-you-handle-stream-errors-properly)

---

### 1. How many types of streams does Node.js have? 🟢

- Four: **Readable** (data you read from, e.g. `fs.createReadStream`), **Writable** (data you write to, e.g. `fs.createWriteStream`), **Duplex** (both readable and writable, e.g. a TCP socket), and **Transform** (a Duplex stream that modifies data as it passes through, e.g. `zlib.createGzip()`).

[↑ Back to top](#table-of-contents)

---

### 2. What problem do streams solve when handling large data? 🟢

- Reading an entire large file/response into memory at once (`readFile`) can exhaust memory and delays processing until the whole thing is loaded. Streams process data **incrementally**, in small chunks, as it arrives — constant, predictable memory usage regardless of total size, and you can start processing before the whole transfer finishes.

[↑ Back to top](#table-of-contents)

---

### 3. What is backpressure in Node.js streams? 🟡

- When a writable destination can't consume data as fast as a readable source is producing it, the excess builds up in an internal buffer. Backpressure is the stream's mechanism to **signal the source to pause** sending more data until the destination catches up — preventing unbounded memory growth.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between `pipe()` and `pipeline()`? 🟡

- `.pipe()`: connects a readable to a writable, automatically handling backpressure — but **doesn't** automatically clean up/destroy streams if an error occurs partway, which can leak resources.
- `pipeline()` (from `stream` module): does the same connecting, **plus** properly closes/destroys every stream in the chain on completion or error, and gives you a single callback/Promise for the whole chain's outcome — the recommended modern approach over chaining raw `.pipe()` calls.

```js
const { pipeline } = require('stream');
pipeline(readStream, transformStream, writeStream, (err) => {
  if (err) console.error('Pipeline failed', err);
});
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between flowing mode and paused mode? 🟡

- **Paused mode** (default): you must explicitly call `.read()` to pull a chunk.
- **Flowing mode**: data is pushed to you automatically and continuously, via a `'data'` event listener (or after calling `.pipe()`/`.resume()`) — you consume it as it arrives rather than pulling it manually.

[↑ Back to top](#table-of-contents)

---

### 6. What is `highWaterMark`, and how does it affect performance? 🟡

- The size (in bytes, or objects for object-mode streams) of the internal buffer at which a stream starts applying backpressure — a higher value buffers more before pausing the source (fewer pause/resume cycles, more memory use); a lower value uses less memory but may pause/resume more frequently.

```js
fs.createReadStream('file.txt', { highWaterMark: 64 * 1024 }); // 64KB chunks
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you implement a custom Duplex or Transform stream? 🔴

- Extend `stream.Transform` and implement a `_transform(chunk, encoding, callback)` method that processes each chunk and pushes the result.

```js
const { Transform } = require('stream');

class UppercaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

process.stdin.pipe(new UppercaseTransform()).pipe(process.stdout);
```

[↑ Back to top](#table-of-contents)

---

### 8. How do you handle stream errors properly? 🔴

- Always attach an `'error'` listener on **every** stream in a manual chain — an unhandled `'error'` event on a stream throws and can crash the process. `pipeline()` (Q4) handles this for you automatically across the whole chain, which is why it's preferred over raw `.pipe()` for production code.

```js
readStream.on('error', (err) => console.error('Read failed:', err));
writeStream.on('error', (err) => console.error('Write failed:', err));
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens to a Node.js process if a stream emits an `'error'` event with no listener attached?

[↑ Back to top](#table-of-contents)

---
