# Arrays & Iterables

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you remove duplicate values from an array?](#1-how-do-you-remove-duplicate-values-from-an-array)

**🟡 Medium**
- [2. What is the difference between `map()` and `forEach()`?](#2-what-is-the-difference-between-map-and-foreach)

**🔴 Hard**
- _Coming soon_

---

### 1. How do you remove duplicate values from an array? 🟢

- Easiest way: spread a `Set` (which only stores unique values) back into an array.
- Alternative: `filter()` + `indexOf()` — keep an element only if its first occurrence index matches the current index.

```js
const arr = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(arr)];
// [1, 2, 3, 4]
```

> [!TIP]
> **Real-life example:** Deduping a list of user IDs collected from multiple event logs before processing them.

> [!IMPORTANT]
> **Follow-up questions:**
> - Does the `Set` approach work for an array of objects? Why or why not?
> - How would you dedupe based on a specific object property (e.g. `id`)?

[↑ Back to top](#table-of-contents)

---

### 2. What is the difference between `map()` and `forEach()`? 🟡

- `map()` returns a **new array** with transformed values; `forEach()` returns `undefined`.
- `map()` is meant for transforming data; `forEach()` is meant for side effects (logging, mutating external state, etc.).
- Both skip empty slots in sparse arrays and cannot be stopped early with `break` (use a `for` loop or `some()`/`every()` if you need early exit).

```js
const nums = [1, 2, 3];

const doubled = nums.map(n => n * 2); // [2, 4, 6]
const sideEffectOnly = nums.forEach(n => console.log(n)); // undefined
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you stop iteration early — which array methods support that?
> - What does `map()` do internally with sparse/empty array slots?

[↑ Back to top](#table-of-contents)

---
