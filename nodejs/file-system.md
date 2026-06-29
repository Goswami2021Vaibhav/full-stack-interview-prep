# File System

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How does Node.js read the contents of a file?](#1-how-does-nodejs-read-the-contents-of-a-file)
- [2. How do you write to a file in Node.js?](#2-how-do-you-write-to-a-file-in-nodejs)
- [3. How do you check if a file or directory exists?](#3-how-do-you-check-if-a-file-or-directory-exists)

**🟡 Medium**
- [4. How do you append content to a file?](#4-how-do-you-append-content-to-a-file)
- [5. How do you rename or move a file?](#5-how-do-you-rename-or-move-a-file)
- [6. How do you get file information (stats)?](#6-how-do-you-get-file-information-stats)
- [7. How do you work with directories (create/list/remove)?](#7-how-do-you-work-with-directories-createlistremove)

**🔴 Hard**
- [8. How do you watch a file for changes?](#8-how-do-you-watch-a-file-for-changes)
- [9. What's the difference between synchronous and asynchronous `fs` methods?](#9-whats-the-difference-between-synchronous-and-asynchronous-fs-methods)
- [10. How do you handle large files without reading them fully into memory?](#10-how-do-you-handle-large-files-without-reading-them-fully-into-memory)

---

### 1. How does Node.js read the contents of a file? 🟢

- The `fs` module's `readFile` (async, callback or Promise-based) or `readFileSync` (blocking).

```js
const fs = require('fs');

fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise-based
const data = await fs.promises.readFile('file.txt', 'utf8');
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you write to a file in Node.js? 🟢

- `fs.writeFile` **overwrites** the file entirely (creates it if it doesn't exist).

```js
fs.writeFile('output.txt', 'Hello, World!', (err) => {
  if (err) throw err;
});
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you check if a file or directory exists? 🟢

- `fs.existsSync()` for a quick synchronous check, or `fs.access()` (async) — checking existence asynchronously right before an operation is prone to race conditions, so it's often simpler to just attempt the operation and handle the `ENOENT` error if it doesn't exist.

```js
if (fs.existsSync('config.json')) {
  console.log('File exists');
}
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you append content to a file? 🟡

- `fs.appendFile` adds to the **end** of a file without overwriting existing content (creates the file if it doesn't exist).

```js
fs.appendFile('log.txt', 'New log entry\n', (err) => {
  if (err) throw err;
});
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you rename or move a file? 🟡

- `fs.rename(oldPath, newPath, callback)` — also doubles as "move" if `newPath` is in a different directory.

```js
fs.rename('old.txt', 'new.txt', (err) => {
  if (err) throw err;
});
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you get file information (stats)? 🟡

- `fs.stat()` returns a `Stats` object with size, timestamps, and type-checking helpers (`isFile()`, `isDirectory()`).

```js
fs.stat('file.txt', (err, stats) => {
  console.log(stats.size, stats.isFile(), stats.mtime);
});
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you work with directories (create/list/remove)? 🟡

```js
fs.mkdir('newFolder', (err) => {});
fs.readdir('.', (err, files) => console.log(files)); // list contents
fs.rmdir('newFolder', (err) => {}); // empty dir only
fs.rm('folder', { recursive: true, force: true }, (err) => {}); // recursive removal
```

[↑ Back to top](#table-of-contents)

---

### 8. How do you watch a file for changes? 🔴

- `fs.watch()` fires a callback whenever the watched file/directory changes — useful for auto-reload tooling, though it's somewhat platform-inconsistent (event types/firing behavior vary slightly across OSes), so libraries like `chokidar` are often used in production tooling for more reliable behavior.

```js
fs.watch('config.json', (eventType, filename) => {
  console.log(`${filename} changed: ${eventType}`);
});
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between synchronous and asynchronous `fs` methods? 🔴

- **Sync** (`readFileSync`, etc.): blocks the entire event loop until the operation completes — fine for one-off CLI scripts or startup-time config loading, but disastrous in a server's request-handling path (blocks every other concurrent request).
- **Async** (`readFile`, `fs.promises.readFile`): offloads the work to libuv's thread pool, returns control immediately, and resolves later — the correct choice for anything in a server's hot path.

[↑ Back to top](#table-of-contents)

---

### 10. How do you handle large files without reading them fully into memory? 🔴

- Use **streams** (`fs.createReadStream`/`createWriteStream`) instead of `readFile`/`writeFile`, which load the entire file into memory at once — streams process data in small chunks, keeping memory usage roughly constant regardless of file size (see [Streams](streams.md)).

```js
const readStream = fs.createReadStream('huge-file.csv');
const writeStream = fs.createWriteStream('output.csv');
readStream.pipe(writeStream); // processes in chunks, not all at once
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What would happen (memory-wise) if you used `readFileSync` on a 10GB file instead?

[↑ Back to top](#table-of-contents)

---
