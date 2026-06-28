# ES6+ Features

_Part of [JavaScript](README.md) interview notes._

> Destructuring, template literals, spread/rest, `Symbol`, and `BigInt` are covered in [Data Types & Variables](data-types-and-variables.md). Arrow functions, default params, and generators are in [Functions](functions.md). Classes and private fields are in [OOP in JavaScript](oops.md). Promises/async-await are in [Async, Promises & Event Loop](async-and-promises.md). This file covers the remaining ES6+ additions.

## Table of Contents

**🟢 Easy**
- [1. What's new in ES6 (ES2015), at a high level?](#1-whats-new-in-es6-es2015-at-a-high-level)
- [2. What's the difference between CommonJS and ES Modules?](#2-whats-the-difference-between-commonjs-and-es-modules)
- [3. What is optional chaining (`?.`)?](#3-what-is-optional-chaining-)
- [4. What is the nullish coalescing operator (`??`), and how is it different from `||`?](#4-what-is-the-nullish-coalescing-operator--and-how-is-it-different-from-)
- [5. What is `Map`, and how is it different from a plain object?](#5-what-is-map-and-how-is-it-different-from-a-plain-object)

**🟡 Medium**
- [6. What is `Set`, and what's a practical use case?](#6-what-is-set-and-whats-a-practical-use-case)
- [7. What are `WeakMap` and `WeakSet`, and why use them over `Map`/`Set`?](#7-what-are-weakmap-and-weakset-and-why-use-them-over-mapset)
- [8. What is dynamic `import()`, and when would you use it?](#8-what-is-dynamic-import-and-when-would-you-use-it)
- [9. What is `Object.fromEntries()` used for?](#9-what-is-objectfromentries-used-for)
- [10. What is a `Proxy`, and what can it be used for?](#10-what-is-a-proxy-and-what-can-it-be-used-for)

**🔴 Hard**
- [11. What is `Reflect`, and how does it relate to `Proxy`?](#11-what-is-reflect-and-how-does-it-relate-to-proxy)
- [12. What is top-level `await`, and what constraints does it have?](#12-what-is-top-level-await-and-what-constraints-does-it-have)
- [13. How would you implement a simple reactive object using `Proxy`?](#13-how-would-you-implement-a-simple-reactive-object-using-proxy)

---

### 1. What's new in ES6 (ES2015), at a high level? 🟢

- `let`/`const`, arrow functions, classes, template literals, destructuring, default/rest/spread params, Promises, modules (`import`/`export`), `Map`/`Set`, and generators.
- ES6 was a major turning point — almost every "modern JS" feature people associate with the language today traces back to it; later versions (ES2016+) add smaller, incremental features on top.

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between CommonJS and ES Modules? 🟢

- **CommonJS** (`require`/`module.exports`): Node's original module system — synchronous, dynamic (can `require()` conditionally at runtime).
- **ES Modules** (`import`/`export`): the official JS standard — statically analyzable (imports resolved at parse time, enabling tree-shaking), asynchronous-friendly, supports top-level `await`.

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

### 3. What is optional chaining (`?.`)? 🟢

- Safely access a deeply nested property without manually checking every level — short-circuits to `undefined` if anything in the chain is `null`/`undefined`, instead of throwing.

```js
const user = { profile: null };
user.profile.bio;   // TypeError: Cannot read properties of null
user.profile?.bio;  // undefined — no error
user.greet?.();     // also works on function calls — skips if `greet` doesn't exist
```

[↑ Back to top](#table-of-contents)

---

### 4. What is the nullish coalescing operator (`??`), and how is it different from `||`? 🟢

- `??` returns the right-hand value only if the left is `null` or `undefined`.
- `||` returns the right-hand value for **any** falsy left value (`0`, `''`, `false`, `NaN` included) — which can be wrong when `0` or `''` are valid values.

```js
const count = 0;
count || 10; // 10 — wrong! 0 is a valid count, but treated as falsy
count ?? 10; // 0  — correct, 0 is not null/undefined
```

[↑ Back to top](#table-of-contents)

---

### 5. What is `Map`, and how is it different from a plain object? 🟢

- A key-value collection, like an object, but: keys can be **any type** (not just strings/symbols), preserves insertion order, has a `.size` property, and doesn't inherit prototype properties that could collide with real keys.

```js
const map = new Map();
map.set('a', 1);
map.set(42, 'number key');
map.set({}, 'object key');
map.size; // 3
```

[↑ Back to top](#table-of-contents)

---

### 6. What is `Set`, and what's a practical use case? 🟡

- A collection of **unique** values of any type, with insertion order preserved.
- Most common use: deduping an array (`[...new Set(arr)]`), or fast membership checks (`set.has(x)` is O(1), versus `array.includes(x)` which is O(n)).

```js
const tags = new Set(['js', 'react', 'js']);
tags.size;       // 2
tags.has('js');  // true
```

[↑ Back to top](#table-of-contents)

---

### 7. What are `WeakMap` and `WeakSet`, and why use them over `Map`/`Set`? 🟡

- Same idea as `Map`/`Set`, but keys (in `WeakMap`) or values (in `WeakSet`) must be **objects**, and are held only **weakly** — if nothing else references that object, it can be garbage collected even while still "in" the WeakMap/WeakSet.
- Neither is iterable and has no `.size`, by design — you can't enumerate them, which avoids accidentally relying on GC-dependent state.
- Useful for attaching private/extra data to an object without preventing it from being cleaned up.

```js
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = /* expensive computation */ obj;
  cache.set(obj, result);
  return result;
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is dynamic `import()`, and when would you use it? 🟡

- `import()` (as a function call, not the static `import` statement) loads a module **asynchronously at runtime**, returning a Promise — enabling code-splitting.
- Useful for loading large dependencies or routes only when actually needed (e.g. lazy-loading a modal's code only when the user opens it).

```js
button.addEventListener('click', async () => {
  const { default: Chart } = await import('./chart.js');
  new Chart();
});
```

[↑ Back to top](#table-of-contents)

---

### 9. What is `Object.fromEntries()` used for? 🟡

- Converts an iterable of `[key, value]` pairs (like a `Map`, or the output of `Object.entries()`) back into a plain object — the inverse of `Object.entries()`.

```js
const map = new Map([['a', 1], ['b', 2]]);
Object.fromEntries(map); // { a: 1, b: 2 }

const params = new URLSearchParams('x=1&y=2');
Object.fromEntries(params); // { x: '1', y: '2' }
```

[↑ Back to top](#table-of-contents)

---

### 10. What is a `Proxy`, and what can it be used for? 🟡

- Wraps an object with custom behavior for fundamental operations (get, set, delete, has, etc.) via "trap" handlers — lets you intercept and customize how the object is interacted with.

```js
const target = { name: 'Vaibhav' };
const proxy = new Proxy(target, {
  get(obj, key) {
    console.log(`Reading ${key}`);
    return obj[key];
  },
  set(obj, key, value) {
    console.log(`Setting ${key} = ${value}`);
    obj[key] = value;
    return true;
  },
});
proxy.name;        // logs "Reading name", returns 'Vaibhav'
proxy.name = 'New'; // logs "Setting name = New"
```

> [!TIP]
> **Real-life example:** Vue 3's reactivity system uses `Proxy` to automatically detect when reactive state is read or mutated.

[↑ Back to top](#table-of-contents)

---

### 11. What is `Reflect`, and how does it relate to `Proxy`? 🔴

- A built-in object providing methods that mirror the internal operations `Proxy` traps intercept (`Reflect.get`, `Reflect.set`, `Reflect.has`, etc.).
- Inside a `Proxy` handler, using `Reflect.method(...)` instead of manually re-implementing the default behavior ensures correct semantics (proper `this` binding, prototype chain handling) and is the idiomatic way to "fall through" to default behavior.

```js
const proxy = new Proxy(target, {
  get(obj, key, receiver) {
    console.log(`Reading ${key}`);
    return Reflect.get(obj, key, receiver); // correct default behavior
  },
});
```

[↑ Back to top](#table-of-contents)

---

### 12. What is top-level `await`, and what constraints does it have? 🔴

- Lets you use `await` directly at the top level of an **ES module**, without wrapping it in an `async` function.
- Only works in actual ES modules (`type="module"` in browsers, or `.mjs`/`"type": "module"` in Node) — not in regular scripts or CommonJS files.
- A module using top-level `await` will delay anything that imports it until the awaited promise settles — can affect load order/performance if overused.

```js
// data.mjs
export const data = await fetch('/api/data').then((r) => r.json());
```

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement a simple reactive object using `Proxy`? 🔴

- Intercept `set` to detect mutations and trigger registered callbacks — the core idea behind frameworks like Vue's reactivity system.

```js
function reactive(obj, onChange) {
  return new Proxy(obj, {
    set(target, key, value) {
      target[key] = value;
      onChange(key, value);
      return true;
    },
  });
}

const state = reactive({ count: 0 }, (key, value) => {
  console.log(`${key} changed to ${value}`);
});
state.count = 1; // logs "count changed to 1"
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you also track **reads** (for dependency tracking, like Vue's `effect`)?
> - How would this handle nested objects, which aren't proxied automatically?

[↑ Back to top](#table-of-contents)

---
