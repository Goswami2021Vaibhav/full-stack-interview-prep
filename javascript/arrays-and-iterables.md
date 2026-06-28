# Arrays & Iterables

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you remove duplicate values from an array?](#1-how-do-you-remove-duplicate-values-from-an-array)
- [2. What's the difference between `slice()` and `splice()`?](#2-whats-the-difference-between-slice-and-splice)
- [3. What's the difference between `indexOf()` and `includes()`?](#3-whats-the-difference-between-indexof-and-includes)
- [4. How do you check if a variable is actually an array?](#4-how-do-you-check-if-a-variable-is-actually-an-array)
- [5. What's the difference between `push`/`pop` and `shift`/`unshift`?](#5-whats-the-difference-between-pushpop-and-shiftunshift)

**🟡 Medium**
- [6. What is the difference between `map()` and `forEach()`?](#6-what-is-the-difference-between-map-and-foreach)
- [7. How does `reduce()` work, and what's a practical use case?](#7-how-does-reduce-work-and-whats-a-practical-use-case)
- [8. Why do you often need to pass a compare function to `sort()`?](#8-why-do-you-often-need-to-pass-a-compare-function-to-sort)
- [9. What's the difference between `flat()` and `flatMap()`?](#9-whats-the-difference-between-flat-and-flatmap)
- [10. What's the difference between `for...of` and `for...in`?](#10-whats-the-difference-between-forof-and-forin)
- [11. What is `Array.from()` used for?](#11-what-is-arrayfrom-used-for)

**🔴 Hard**
- [12. What makes an object "iterable" in JavaScript?](#12-what-makes-an-object-iterable-in-javascript)
- [13. What is a sparse array, and how does it affect methods like `map()`/`forEach()`?](#13-what-is-a-sparse-array-and-how-does-it-affect-methods-like-mapforeach)
- [14. How would you implement your own `flatten(array, depth)` function?](#14-how-would-you-implement-your-own-flattenarray-depth-function)

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

### 2. What's the difference between `slice()` and `splice()`? 🟢

- `slice(start, end)`: returns a **new array** with the selected portion — doesn't touch the original.
- `splice(start, deleteCount, ...items)`: **mutates** the original array — removes/replaces/inserts elements in place and returns the removed items.

```js
const arr = [1, 2, 3, 4, 5];
arr.slice(1, 3);   // [2, 3] — arr unchanged
arr.splice(1, 2);  // [2, 3] removed; arr is now [1, 4, 5]
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between `indexOf()` and `includes()`? 🟢

- `indexOf(value)`: returns the index of the first match, or `-1` if not found.
- `includes(value)`: returns a plain `true`/`false`, and unlike `indexOf`, can correctly detect `NaN`.

```js
[1, NaN, 3].indexOf(NaN);  // -1 — indexOf uses === , which fails for NaN
[1, NaN, 3].includes(NaN); // true — includes uses a NaN-aware comparison
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you check if a variable is actually an array? 🟢

- `Array.isArray(value)` is the reliable way — `typeof` returns `"object"` for arrays too, so it can't tell them apart from plain objects.

```js
Array.isArray([1, 2]); // true
Array.isArray({});      // false
typeof [];               // "object" — not useful here
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `push`/`pop` and `shift`/`unshift`? 🟢

- `push`/`pop`: add/remove from the **end** of the array — fast (O(1)).
- `shift`/`unshift`: add/remove from the **beginning** — slower (O(n)), since every other element has to be re-indexed.

```js
const arr = [1, 2, 3];
arr.push(4);    // [1, 2, 3, 4]
arr.pop();      // removes 4 -> [1, 2, 3]
arr.unshift(0); // [0, 1, 2, 3]
arr.shift();    // removes 0 -> [1, 2, 3]
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the difference between `map()` and `forEach()`? 🟡

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

### 7. How does `reduce()` work, and what's a practical use case? 🟡

- Walks through the array, building up a single accumulated value by applying a callback `(accumulator, currentValue) => newAccumulator` at each step.
- Can implement `map`, `filter`, `sum`, `flatten`, grouping, and more — it's the most general-purpose array method.

```js
const cart = [{ price: 10 }, { price: 20 }, { price: 5 }];
const total = cart.reduce((sum, item) => sum + item.price, 0); // 35
```

> [!TIP]
> **Real-life example:** calculating a shopping cart total, or grouping a list of orders by customer ID.

[↑ Back to top](#table-of-contents)

---

### 8. Why do you often need to pass a compare function to `sort()`? 🟡

- Without one, `sort()` converts elements to **strings** and compares them lexicographically — which breaks for numbers (`10` sorts before `2`).
- `sort()` also **mutates** the original array in place.

```js
[10, 2, 1].sort();                  // [1, 10, 2] — wrong, string comparison
[10, 2, 1].sort((a, b) => a - b);   // [1, 2, 10] — correct, numeric comparison
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `flat()` and `flatMap()`? 🟡

- `flat(depth)`: flattens nested arrays up to the given depth (default `1`).
- `flatMap(fn)`: maps each element with `fn`, then flattens the result by one level — equivalent to `.map(fn).flat()` but slightly more efficient.

```js
[1, [2, [3, [4]]]].flat();    // [1, 2, [3, [4]]] — depth 1
[1, [2, [3, [4]]]].flat(2);   // [1, 2, 3, [4]]

[1, 2, 3].flatMap(n => [n, n * 2]); // [1, 2, 2, 4, 3, 6]
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `for...of` and `for...in`? 🟡

- `for...of`: iterates over the **values** of an iterable (arrays, strings, Maps, Sets) — the right choice for arrays.
- `for...in`: iterates over **enumerable property keys** of an object, including inherited ones — for arrays, it gives string indices (and can pick up unwanted inherited keys), which is why it's discouraged for arrays.

```js
const arr = ['a', 'b', 'c'];
for (const val of arr) console.log(val);   // 'a', 'b', 'c'
for (const key in arr) console.log(key);   // '0', '1', '2' (strings, not numbers)
```

[↑ Back to top](#table-of-contents)

---

### 11. What is `Array.from()` used for? 🟡

- Creates a real array from an **iterable** (Set, Map, string) or an **array-like** object (something with a `.length`, like `arguments` or a DOM `NodeList`) — both of which lack native array methods until converted.
- Also accepts an optional mapping function, similar to `map()`.

```js
Array.from('abc');               // ['a', 'b', 'c']
Array.from({ length: 3 }, (_, i) => i * 2); // [0, 2, 4]
Array.from(document.querySelectorAll('div')); // real array of DOM nodes
```

[↑ Back to top](#table-of-contents)

---

### 12. What makes an object "iterable" in JavaScript? 🔴

- An object is iterable if it implements `Symbol.iterator` — a method that returns an iterator object with a `.next()` method returning `{ value, done }`.
- This protocol is what powers `for...of`, spread (`...`), and destructuring for arrays, strings, Maps, and Sets.

```js
const range = {
  from: 1, to: 3,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      },
    };
  },
};
[...range]; // [1, 2, 3] — works because it implements the iterable protocol
```

[↑ Back to top](#table-of-contents)

---

### 13. What is a sparse array, and how does it affect methods like `map()`/`forEach()`? 🔴

- A sparse array has "holes" — missing indices rather than actual `undefined` values (e.g. created via `new Array(3)` or `delete arr[1]`).
- Iteration methods like `map`, `forEach`, and `filter` **skip holes entirely** rather than treating them as `undefined`.

```js
const sparse = [1, , 3]; // hole at index 1
sparse.forEach(x => console.log(x)); // logs 1, then 3 — skips the hole
sparse.map(x => x * 2);              // [2, <1 empty item>, 6]
```

[↑ Back to top](#table-of-contents)

---

### 14. How would you implement your own `flatten(array, depth)` function? 🔴

- Recursively check each element: if it's an array and depth remains, recurse into it with `depth - 1`; otherwise, keep the element as-is.

```js
function flatten(arr, depth = 1) {
  if (depth < 1) return arr.slice();
  return arr.reduce(
    (acc, val) => acc.concat(Array.isArray(val) ? flatten(val, depth - 1) : val),
    []
  );
}
flatten([1, [2, [3, [4]]]], 2); // [1, 2, 3, [4]]
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you flatten to **infinite** depth?
> - Can you implement this without recursion (using a stack)?

[↑ Back to top](#table-of-contents)

---
