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

- ES6 (also called ES2015) was the biggest single update JavaScript has ever had — it's the release that turned JS from a somewhat clunky scripting language into what people today think of as "modern JavaScript."
- Headline additions: `let`/`const` (block-scoped variables, replacing many uses of `var`), arrow functions (shorter function syntax with lexical `this`), `class` syntax (a cleaner way to write constructor functions and prototypes), template literals (backtick strings with `${}` interpolation), destructuring (unpacking values from arrays/objects into variables), default/rest/spread parameters, native Promises (a cleaner way to handle async code than callbacks), a real module system (`import`/`export`), the `Map`/`Set` collection types, and generator functions.
- Why it matters for interviews: almost every "modern JS" feature you use daily traces back to ES6. Later yearly releases (ES2016, ES2017, ... ES2023+) add smaller, incremental features on top (like `async`/`await` in ES2017, or optional chaining in ES2020) rather than anything as sweeping as ES6 itself.

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between CommonJS and ES Modules? 🟢

- A "module system" is just the mechanism a language uses to split code across files and share code between them. JavaScript has ended up with two competing systems.
- **CommonJS** (`require`/`module.exports`): Node.js's original module system, created before JS had an official one built in.
  - **Synchronous**: `require('fs')` loads and executes the module immediately, blocking until it's done — fine for local files on a server.
  - **Dynamic**: you can call `require()` conditionally, inside an `if` block or a function, because it's just a regular function call evaluated at runtime.
- **ES Modules (ESM)** (`import`/`export`): the official standard added in ES6, now the modern default for both browsers and Node.
  - **Statically analyzable**: the imports/exports are written in a fixed syntax that tools can figure out just by reading the file, without running it — this is what enables "tree-shaking" (bundlers can detect and strip out code that's never actually used).
  - **Asynchronous-friendly**: designed to work well with loading modules over a network (like in a browser), and supports top-level `await` (see Q12) since a module's own loading can now itself be asynchronous.

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

- Before optional chaining, safely reading a deeply nested property meant manually checking every level along the way (e.g. `user && user.profile && user.profile.bio`), because trying to read a property off `null`/`undefined` throws a `TypeError`.
- `?.` does that check automatically: at each `?.` step, if the value on the left is `null` or `undefined`, the **whole expression immediately stops and returns `undefined`** instead of throwing — it never even tries to read `.bio`.
- It works for property access (`obj?.prop`), array access (`arr?.[0]`), and function calls (`fn?.()`, which skips calling `fn` entirely if `fn` doesn't exist).

```js
const user = { profile: null };
user.profile.bio;   // TypeError: Cannot read properties of null
user.profile?.bio;  // undefined — short-circuits safely, no error
user.greet?.();     // undefined — skips the call since `greet` doesn't exist
```

[↑ Back to top](#table-of-contents)

---

### 4. What is the nullish coalescing operator (`??`), and how is it different from `||`? 🟢

- Both operators are used to provide a fallback value, but they disagree on what counts as "missing."
- `||` (logical OR) falls back to the right-hand side whenever the left side is **falsy** — and JS considers quite a few things falsy: `0`, `''` (empty string), `false`, `NaN`, `null`, and `undefined`. This is often too broad: if `0` or `''` are legitimate, intentional values in your app, `||` will incorrectly replace them with the fallback.
- `??` (nullish coalescing) only falls back when the left side is specifically `null` or `undefined` — meaning "this value was never actually set," as opposed to "this value happens to be falsy." Everything else, including `0`, `''`, and `false`, is treated as a valid value and kept as-is.

```js
const count = 0;
count || 10; // 10 — wrong! 0 is a valid count, but || treats it as "missing"
count ?? 10; // 0  — correct, ?? only replaces null/undefined
```

[↑ Back to top](#table-of-contents)

---

### 5. What is `Map`, and how is it different from a plain object? 🟢

- `Map` is a built-in collection for storing key-value pairs, similar in spirit to a plain object (`{}`), but designed specifically to be a general-purpose key-value store, which fixes a few rough edges plain objects have:
  - **Any type of key**: a plain object can only use strings or symbols as keys (numbers/objects get silently converted to strings), but a `Map` lets you use numbers, objects, functions — anything — as a key, without conversion.
  - **Guaranteed insertion order**: iterating a `Map` always visits entries in the order they were added, which isn't strictly guaranteed for all object key types.
  - **Easy size check**: `map.size` gives you the count directly; a plain object needs `Object.keys(obj).length`.
  - **No prototype pollution risk**: a plain object inherits properties from `Object.prototype` (like `toString`), which could accidentally collide with a real key you're trying to store (e.g. a key literally named `"toString"`). A `Map` has no such inherited properties to worry about.

```js
const map = new Map();
map.set('a', 1);
map.set(42, 'number key');   // numeric key works as-is
map.set({}, 'object key');   // object key works too
map.size; // 3
```

[↑ Back to top](#table-of-contents)

---

### 6. What is `Set`, and what's a practical use case? 🟡

- `Set` is a built-in collection that stores only **unique** values — if you try to add a value that's already in the `Set`, it's simply ignored, and no duplicates are kept. Insertion order is preserved when you iterate it.
- Two common practical uses:
  - **Removing duplicates from an array**: spreading a `Set` back into an array (`[...new Set(arr)]`) is the shortest way to dedupe a list, because the `Set` automatically drops repeats as it's built.
  - **Fast membership checks**: `set.has(x)` runs in constant time — O(1), meaning it takes roughly the same time no matter how large the `Set` is — versus `array.includes(x)`, which is O(n) and gets slower the bigger the array is, because it has to check every element one by one.

```js
const tags = new Set(['js', 'react', 'js']); // duplicate 'js' is dropped
tags.size;       // 2
tags.has('js');  // true — fast lookup, regardless of Set size
```

[↑ Back to top](#table-of-contents)

---

### 7. What are `WeakMap` and `WeakSet`, and why use them over `Map`/`Set`? 🟡

- First, a quick refresher on garbage collection (GC): JS automatically frees memory used by objects once nothing in your code can reach them anymore. Normally, if you store an object as a value somewhere (like in a regular `Map`), that counts as a reference — it keeps the object alive, even if nothing else in your program still needs it, which can cause memory leaks.
- `WeakMap`/`WeakSet` behave like `Map`/`Set`, with two key differences:
  - Keys (in `WeakMap`) or values (in `WeakSet`) must be **objects**, not primitives like strings or numbers.
  - Those references are **"weak"** — they don't count when GC decides whether an object is still in use. So if the only remaining reference to an object is as a key in a `WeakMap`, GC is still free to clean it up, and the entry silently disappears too.
- Neither is iterable and neither has a `.size` property — this is intentional. Since entries can vanish at any unpredictable moment (whenever GC runs), letting you enumerate them would produce inconsistent, hard-to-reason-about behavior.
- Typical use case: attaching extra data to an object (like a cached computation result) without accidentally preventing that object from ever being garbage collected once the rest of the app is done with it.

```js
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) return cache.get(obj); // reuse cached result
  const result = /* expensive computation */ obj;
  cache.set(obj, result);
  return result;
  // if `obj` is later discarded elsewhere, this cache entry can be GC'd too
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is dynamic `import()`, and when would you use it? 🟡

- The regular `import ... from '...'` statement is **static**: it must appear at the top of a file, and the module it points to is loaded immediately, before any of the file's code runs.
- Dynamic `import(path)`, written as a function call instead, loads a module **asynchronously, at runtime, whenever you decide to call it** — it returns a Promise that resolves to the module once it's finished loading. Because it's just a normal function call, you can use it conditionally, inside an event handler, in a loop, etc.
- This enables **code-splitting**: instead of bundling every module into one giant file that loads all at once, bundlers (like Webpack/Vite) can split code at each dynamic `import()` into a separate chunk that only loads when actually needed.
- Common use case: lazy-loading a heavy dependency or a whole page/route only at the moment the user needs it (e.g. only downloading a charting library's code when the user opens a page that shows a chart), which speeds up the initial page load.

```js
button.addEventListener('click', async () => {
  const { default: Chart } = await import('./chart.js'); // only fetched now, on click
  new Chart();
});
```

[↑ Back to top](#table-of-contents)

---

### 9. What is `Object.fromEntries()` used for? 🟡

- `Object.entries(obj)` turns an object into an array of `[key, value]` pairs. `Object.fromEntries()` does the **exact reverse**: it takes any iterable of `[key, value]` pairs and builds a plain object out of them.
- This is especially handy for converting other key-value-like structures (a `Map`, the entries of `URLSearchParams`, filtered/transformed entries) back into a plain object you can work with normally.

```js
const map = new Map([['a', 1], ['b', 2]]);
Object.fromEntries(map); // { a: 1, b: 2 } — Map converted to plain object

const params = new URLSearchParams('x=1&y=2');
Object.fromEntries(params); // { x: '1', y: '2' } — URL query params to object

// Round trip with Object.entries():
const obj = { a: 1, b: 2 };
Object.fromEntries(Object.entries(obj)); // { a: 1, b: 2 } — back to where we started
```

[↑ Back to top](#table-of-contents)

---

### 10. What is a `Proxy`, and what can it be used for? 🟡

- A `Proxy` wraps a "target" object and lets you intercept fundamental operations performed on it — reading a property, setting one, deleting one, checking if a key exists (`in`), and more — before they actually happen.
- You define this custom behavior using **"traps"**: handler functions with reserved names (`get`, `set`, `deleteProperty`, `has`, etc.) that fire automatically whenever that operation is attempted on the proxy.
- Think of it as a security guard standing in front of the real object: every interaction goes through the guard first, who can log it, block it, modify it, or just let it pass through to the real object underneath.

```js
const target = { name: 'Vaibhav' };
const proxy = new Proxy(target, {
  get(obj, key) {
    console.log(`Reading ${key}`);
    return obj[key]; // let the read actually happen
  },
  set(obj, key, value) {
    console.log(`Setting ${key} = ${value}`);
    obj[key] = value; // let the write actually happen
    return true; // must return true to signal success
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

- `Reflect` is a built-in object that bundles a set of methods (`Reflect.get`, `Reflect.set`, `Reflect.has`, etc.) which each do exactly what a plain JS operation would do by default (e.g. `Reflect.get(obj, key)` behaves like `obj[key]`) — but as explicit function calls instead of syntax.
- The connection to `Proxy`: each `Reflect` method corresponds one-to-one with a `Proxy` trap. When you write a `get` trap, for instance, you could manually re-implement "just return `obj[key]`" yourself — but this gets subtly wrong in edge cases (like correctly handling `this` inside getters, or objects that use inheritance). `Reflect.get(...)` guarantees you get the exact correct default behavior instead of an approximation.
- In practice: inside a `Proxy` trap, when you want the operation to just behave normally (after logging it, validating it, etc.), call the matching `Reflect` method rather than reimplementing the logic by hand. It's the idiomatic way to say "now let this fall through to default behavior."

```js
const proxy = new Proxy(target, {
  get(obj, key, receiver) {
    console.log(`Reading ${key}`);
    return Reflect.get(obj, key, receiver); // guaranteed-correct default behavior
  },
});
```

[↑ Back to top](#table-of-contents)

---

### 12. What is top-level `await`, and what constraints does it have? 🔴

- Normally, `await` can only be used inside a function marked `async` — using it anywhere else is a syntax error. Top-level `await` (ES2022) relaxes this rule specifically for **ES modules**: you can use `await` directly in the module's top-level code, with no wrapping `async function` needed.
- It only works inside actual ES modules — files loaded with `type="module"` in the browser, or `.mjs`/`"type": "module"` files in Node. It does not work in ordinary scripts or CommonJS files, because those weren't designed to pause execution while loading.
- The trade-off: if module A does a top-level `await`, any other module that `import`s A has to **wait** for that `await` to finish before it can finish loading A. Overusing this (e.g. awaiting a slow network request at the top of a widely-imported module) can silently slow down or reorder the loading of your whole app.

```js
// data.mjs
export const data = await fetch('/api/data').then((r) => r.json());
// Anything that imports data.mjs will wait for this fetch to finish first.
```

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement a simple reactive object using `Proxy`? 🔴

- "Reactive" means: when a piece of data changes, something else (like the UI) automatically responds to that change, without you manually writing code to trigger the update every time.
- The core idea: wrap the data object in a `Proxy` and intercept its `set` trap (see Q10). Every time a property on the object is mutated, the `set` trap fires *before* returning control, so you can run any callback you want — like re-rendering a UI, or logging the change — right at the moment the mutation happens.
- This is the foundational idea behind Vue 3's reactivity system: it wraps your component's state in a `Proxy` like this, so changing a variable automatically re-renders whatever depends on it.

```js
function reactive(obj, onChange) {
  return new Proxy(obj, {
    set(target, key, value) {
      target[key] = value;   // perform the actual mutation
      onChange(key, value);  // then notify listeners about it
      return true;
    },
  });
}

const state = reactive({ count: 0 }, (key, value) => {
  console.log(`${key} changed to ${value}`);
});
state.count = 1; // logs "count changed to 1" — the "set" happened automatically
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you also track **reads** (for dependency tracking, like Vue's `effect`)?
> - How would this handle nested objects, which aren't proxied automatically?

[↑ Back to top](#table-of-contents)

---
