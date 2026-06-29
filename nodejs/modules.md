# Modules

_Part of [Node.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between CommonJS and ES Modules in Node.js?](#1-whats-the-difference-between-commonjs-and-es-modules-in-nodejs)
- [2. How do you export and import code with CommonJS?](#2-how-do-you-export-and-import-code-with-commonjs)
- [3. What is the `path` module used for?](#3-what-is-the-path-module-used-for)

**🟡 Medium**
- [4. How does module caching work in Node.js?](#4-how-does-module-caching-work-in-nodejs)
- [5. What is the `os` module used for?](#5-what-is-the-os-module-used-for)
- [6. What is the `util` module used for?](#6-what-is-the-util-module-used-for)
- [7. What is the `crypto` module used for?](#7-what-is-the-crypto-module-used-for)
- [8. What is the `dns` module used for?](#8-what-is-the-dns-module-used-for)

**🔴 Hard**
- [9. How does Node.js resolve a `require()` call internally?](#9-how-does-nodejs-resolve-a-require-call-internally)
- [10. What are the use cases for the `vm` core module?](#10-what-are-the-use-cases-for-the-vm-core-module)
- [11. How does `assert` work in Node.js?](#11-how-does-assert-work-in-nodejs)
- [12. How do you generate and verify a checksum of a string in Node.js?](#12-how-do-you-generate-and-verify-a-checksum-of-a-string-in-nodejs)

---

### 1. What's the difference between CommonJS and ES Modules in Node.js? 🟢

- **CommonJS** (`require`/`module.exports`): Node's original module system — synchronous loading, dynamic (can `require()` conditionally).
- **ES Modules** (`import`/`export`, `.mjs` or `"type": "module"` in `package.json`): the standard JS module system — statically analyzable, supports top-level `await`, asynchronous loading under the hood.

```js
// CommonJS
const fs = require('fs');
module.exports = myFunction;

// ES Modules
import fs from 'fs';
export default myFunction;
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you export and import code with CommonJS? 🟢

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add };
// or: exports.add = add;

// app.js
const { add } = require('./math');
add(2, 3); // 5
```

[↑ Back to top](#table-of-contents)

---

### 3. What is the `path` module used for? 🟢

- Cross-platform-safe utilities for working with file/directory paths — avoids manually concatenating strings with the wrong separator (`/` vs `\`) across OSes.

```js
const path = require('path');
path.join('users', 'vaibhav', 'file.txt'); // 'users/vaibhav/file.txt' (or '\' on Windows)
path.extname('photo.png'); // '.png'
path.basename('/a/b/file.txt'); // 'file.txt'
```

[↑ Back to top](#table-of-contents)

---

### 4. How does module caching work in Node.js? 🟡

- The **first** time a module is `require()`d, Node executes it and caches the resulting `module.exports` object by resolved file path. Every subsequent `require()` of the **same** file returns the **cached** object instead of re-executing the module — meaning module-level state is shared across every place that imports it.

```js
// counter.js
let count = 0;
module.exports = { increment: () => ++count };

// a.js and b.js both require('./counter') — they share the SAME `count`, not separate copies
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the `os` module used for? 🟡

- Provides OS-level information: CPU details, total/free memory, platform, home directory, network interfaces — useful for system diagnostics or scaling decisions (e.g. `os.cpus().length` to decide how many cluster workers to spawn).

```js
const os = require('os');
os.cpus().length;  // number of logical CPU cores
os.freemem();        // free memory in bytes
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the `util` module used for? 🟡

- A grab-bag of utility functions: `util.promisify()` (converts a callback-style function to return a Promise), `util.inspect()` (detailed object string representation, what `console.log` uses internally), `util.debuglog()` (conditional debug logging via the `NODE_DEBUG` env var).

```js
const util = require('util');
const fs = require('fs');
const readFile = util.promisify(fs.readFile);
const data = await readFile('file.txt', 'utf8');
```

[↑ Back to top](#table-of-contents)

---

### 7. What is the `crypto` module used for? 🟡

- Built-in cryptographic functionality: hashing (`createHash`), HMAC, symmetric/asymmetric encryption, secure random byte generation, and key derivation — no external dependency needed for common cryptographic tasks.

```js
const crypto = require('crypto');
crypto.createHash('sha256').update('hello').digest('hex');
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the `dns` module used for? 🟡

- Performs DNS lookups and resolution programmatically — resolving a hostname to an IP address, or querying specific DNS record types (MX, TXT, etc.), useful for diagnostics or custom routing logic.

```js
const dns = require('dns');
dns.lookup('example.com', (err, address) => console.log(address));
```

[↑ Back to top](#table-of-contents)

---

### 9. How does Node.js resolve a `require()` call internally? 🔴

For `require('./foo')` or `require('foo')`, Node roughly:
1. If it's a **core module** name (`fs`, `path`), use the built-in directly.
2. If it starts with `./`/`../`/`/`, resolve it as a **relative/absolute file path** — try the exact file, then with `.js`/`.json`/`.node` extensions, then as a directory (looking for its `package.json` `main` field or `index.js`).
3. Otherwise, treat it as a **package name** — search up the directory tree through each `node_modules` folder until found (or throw `MODULE_NOT_FOUND`).

[↑ Back to top](#table-of-contents)

---

### 10. What are the use cases for the `vm` core module? 🔴

- Runs JavaScript code in a separate **V8 context** with its own global object — used for safely executing untrusted/dynamic code with some isolation (e.g. plugin systems, online code-execution sandboxes), though it's **not** a full security sandbox on its own (it can still access the host process's resources unless carefully restricted).

```js
const vm = require('vm');
const script = new vm.Script('1 + 2');
script.runInNewContext(); // 3, isolated from the surrounding scope's variables
```

[↑ Back to top](#table-of-contents)

---

### 11. How does `assert` work in Node.js? 🔴

- A built-in module for writing runtime sanity checks — throws an `AssertionError` if a condition isn't met. Predates widespread use of dedicated testing libraries, and is still used internally by Node's own test suite and for simple invariant checks.

```js
const assert = require('assert');
assert.strictEqual(1 + 1, 2); // passes silently
assert.strictEqual(1 + 1, 3); // throws AssertionError
```

[↑ Back to top](#table-of-contents)

---

### 12. How do you generate and verify a checksum of a string in Node.js? 🔴

- Hash the string with `crypto.createHash()`, then compare the resulting digest against an expected value to verify integrity (e.g. confirming a downloaded file wasn't corrupted/tampered with).

```js
const crypto = require('crypto');

function checksum(data, algorithm = 'sha256') {
  return crypto.createHash(algorithm).update(data).digest('hex');
}

const sum = checksum('hello world');
const isValid = sum === checksum('hello world'); // true — matches
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is a simple hash comparison not sufficient for verifying a message hasn't been tampered with by an attacker who controls the data (vs. accidental corruption)? What would you use instead (HMAC)?

[↑ Back to top](#table-of-contents)

---
