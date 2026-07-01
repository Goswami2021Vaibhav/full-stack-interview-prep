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

- A `Set` is a built-in collection that automatically refuses to store duplicate values — if you try to add a value that's already there, it's silently ignored. That makes it the easiest way to dedupe: build a `Set` from the array, then spread (`...`) it back into a new array.
- Alternative approach with `filter()`: for each element, check whether its **first occurrence** index (`indexOf`) equals its **current** index. If they match, it's the first time we've seen that value, so keep it; if a later duplicate comes along, its current index won't match its first-occurrence index, so it gets filtered out.
- The `Set` approach is shorter and generally faster; the `filter()` approach is useful to know because it generalizes to "dedupe by some custom rule."

```js
const arr = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(arr)];
// [1, 2, 3, 4]

// Equivalent using filter() + indexOf()
const unique2 = arr.filter((value, index) => arr.indexOf(value) === index);
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

- `slice(start, end)`: **reads** a portion of the array without touching it. It returns a brand-new array containing the elements from `start` up to (but not including) `end`. The original array is left completely unchanged — this is why it's called a "non-mutating" method.
- `splice(start, deleteCount, ...items)`: **edits** the array in place. It removes `deleteCount` elements starting at `start`, can insert new `items` in their place, and returns an array of whatever was removed. Because it changes the original array directly, it's called a "mutating" method.
- A simple way to remember the difference: **slice** = copy a piece out, leave the original alone. **splice** = surgically cut into the original and change it.

```js
const arr = [1, 2, 3, 4, 5];
arr.slice(1, 3);   // [2, 3] — arr is still [1, 2, 3, 4, 5], unchanged
arr.splice(1, 2);  // returns [2, 3] (the removed items); arr is now [1, 4, 5]
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between `indexOf()` and `includes()`? 🟢

- `indexOf(value)`: searches the array and returns the **position** (index) of the first matching element, or `-1` if nothing matches. Useful when you need to know *where* something is, not just *whether* it's there.
- `includes(value)`: searches the array and returns a simple `true`/`false` — cleaner to read when you only care about presence, not position.
- The one tricky difference: `indexOf` compares values using `===` (strict equality), and `NaN === NaN` is always `false` in JavaScript (a quirky rule of the language). So `indexOf` can never find `NaN`. `includes` uses a slightly different comparison internally (called "SameValueZero") that treats `NaN` as equal to itself, so it can find it correctly.

```js
[1, NaN, 3].indexOf(NaN);  // -1 — indexOf uses === , which fails for NaN
[1, NaN, 3].includes(NaN); // true — includes uses a NaN-aware comparison

[1, 2, 3].indexOf(2);   // 1 — found at index 1
[1, 2, 3].includes(2);  // true
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you check if a variable is actually an array? 🟢

- It's tempting to reach for `typeof value`, but that returns the string `"object"` for both plain objects (`{}`) and arrays (`[]`) — JavaScript doesn't give arrays their own `typeof` category, so this check can't distinguish them.
- `Array.isArray(value)` is the correct, reliable way — it's a dedicated built-in function written specifically to answer this question, and it works correctly even across different execution contexts (e.g. an array created in an `<iframe>`), where other tricks like checking the constructor can fail.

```js
Array.isArray([1, 2]); // true
Array.isArray({});      // false
typeof [];               // "object" — not useful here, same as typeof {}
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `push`/`pop` and `shift`/`unshift`? 🟢

- `push(item)` adds an element to the **end** of the array; `pop()` removes and returns the last element. Both are fast — technically "O(1)" (constant time), meaning the array's size doesn't matter, because nothing else in the array needs to move or be renumbered.
- `unshift(item)` adds an element to the **beginning**; `shift()` removes and returns the first element. These are slower — "O(n)" (time grows with array size) — because every remaining element's index has to shift up or down by one to make room.
- Rule of thumb: prefer `push`/`pop` when you can, especially for large arrays, since they avoid the re-indexing cost.

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

- `map()` runs a function on every element and collects the return values into a **brand-new array** of the same length — it's for when you want a transformed copy of the data.
- `forEach()` also runs a function on every element, but it always returns `undefined` — it doesn't build anything for you. It's meant purely for **side effects**: logging, pushing into an outside array, updating a variable, etc.
- A good way to decide which to use: if you need the result of the loop as a new array, use `map()`. If you're just "doing something" for each item and don't need a new array back, use `forEach()`.
- Neither one can be stopped early with `break` or `continue` (those keywords only work in real `for`/`while` loops). If you need to exit early, use a plain `for` loop, or `some()`/`every()` which stop as soon as their callback returns `true`/`false` respectively.
- Both silently skip "holes" in sparse arrays (see Q13) — they don't call the callback for missing indices at all.

```js
const nums = [1, 2, 3];

const doubled = nums.map(n => n * 2); // [2, 4, 6] — new array returned
const sideEffectOnly = nums.forEach(n => console.log(n)); // undefined — nothing useful returned
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you stop iteration early — which array methods support that?
> - What does `map()` do internally with sparse/empty array slots?

[↑ Back to top](#table-of-contents)

---

### 7. How does `reduce()` work, and what's a practical use case? 🟡

- `reduce()` walks through the array one element at a time, carrying along a running total (called the **accumulator**) that gets updated at each step. You give it a callback `(accumulator, currentValue) => newAccumulator` and a starting value; it calls that callback once per element, feeding in the result from the previous call each time.
- Think of it like a snowball rolling downhill, picking up a bit more snow (data) at every step until you're left with one final value.
- It's the most flexible array method — you can implement `map`, `filter`, summing, flattening, or grouping items, all using `reduce()`, because it lets you fully control what gets carried forward.

```js
const cart = [{ price: 10 }, { price: 20 }, { price: 5 }];
// sum starts at 0, then becomes 10, then 30, then 35
const total = cart.reduce((sum, item) => sum + item.price, 0); // 35

// Step by step: sum=0 -> sum+10=10 -> sum+20=30 -> sum+5=35
```

> [!TIP]
> **Real-life example:** calculating a shopping cart total, or grouping a list of orders by customer ID.

[↑ Back to top](#table-of-contents)

---

### 8. Why do you often need to pass a compare function to `sort()`? 🟡

- By default, `sort()` doesn't know your elements are numbers — it converts everything to a **string** first and compares those strings character-by-character (lexicographic/alphabetical order). That's why `10` ends up before `2`: as strings, `"1"` comes before `"2"`.
- Passing a **compare function** `(a, b) => ...` overrides this default. The rule is: return a negative number if `a` should come first, a positive number if `b` should come first, and `0` if they're equal. For numbers, `(a, b) => a - b` gives ascending order because it naturally produces negative/zero/positive in the right cases.
- `sort()` also **mutates** the original array in place (it doesn't return a new one) — so if you need to keep the original order elsewhere, copy the array first with `[...arr].sort(...)`.

```js
[10, 2, 1].sort();                  // [1, 10, 2] — wrong, compared as strings "1", "10", "2"
[10, 2, 1].sort((a, b) => a - b);   // [1, 2, 10] — correct, numeric comparison

const original = [3, 1, 2];
const sortedCopy = [...original].sort((a, b) => a - b); // original stays [3, 1, 2]
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `flat()` and `flatMap()`? 🟡

- `flat(depth)` takes an array that contains nested arrays inside it and "flattens" it — pulls the inner elements out to the top level — up to `depth` levels deep (default `1`, meaning only one level of nesting is unwrapped).
- `flatMap(fn)` does two things in one pass: first it runs `map(fn)` on every element (transforming each one, often into a small array), then it flattens the result by exactly one level. It's equivalent to calling `.map(fn).flat()`, but done in a single, slightly more efficient pass.
- `flatMap` is especially handy when a mapping function sometimes needs to produce zero, one, or multiple output items per input item (e.g. splitting a sentence into words).

```js
[1, [2, [3, [4]]]].flat();    // [1, 2, [3, [4]]] — only 1 level unwrapped
[1, [2, [3, [4]]]].flat(2);   // [1, 2, 3, [4]] — 2 levels unwrapped

// flatMap: double each number, keep both original and doubled, flattened into one array
[1, 2, 3].flatMap(n => [n, n * 2]); // [1, 2, 2, 4, 3, 6]
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `for...of` and `for...in`? 🟡

- `for...of` loops over the **values** of anything "iterable" (arrays, strings, Maps, Sets — see Q12 for what makes something iterable). Each time through the loop, you get the next actual value. This is the natural choice when working with arrays.
- `for...in` loops over an object's **enumerable property keys** (the names/indices of its properties), including any it inherits through the prototype chain (a lookup chain objects use to find properties not directly on themselves). For an array, this means you get back the indices as **strings** (`'0'`, `'1'`, ...) rather than the values themselves, and it can accidentally pick up extra inherited properties that aren't really "array elements" — this is why `for...in` is generally avoided for arrays.
- Rule of thumb: use `for...of` for arrays (and other iterables) when you want the values; use `for...in` only when you specifically want to enumerate an object's keys.

```js
const arr = ['a', 'b', 'c'];
for (const val of arr) console.log(val);   // 'a', 'b', 'c' — actual values
for (const key in arr) console.log(key);   // '0', '1', '2' — string indices, not numbers
```

[↑ Back to top](#table-of-contents)

---

### 11. What is `Array.from()` used for? 🟡

- Some things "look like" arrays but aren't actually arrays — they don't have methods like `.map()` or `.filter()` on them. `Array.from()` converts these into a real array so you can use all the normal array methods on them. It works on two kinds of inputs:
  - **Iterables** — things you can loop over with `for...of`, like a `Set`, `Map`, or string.
  - **Array-like objects** — objects that have a numeric `.length` property and indexed items (like `0`, `1`, `2`, ...), such as the `arguments` object inside a function, or a DOM `NodeList` returned by `querySelectorAll`.
- It also accepts an optional second argument: a mapping function, applied to each item as it's converted — similar to chaining `.map()` right after, but done in one step.

```js
Array.from('abc');               // ['a', 'b', 'c'] — string is iterable
Array.from({ length: 3 }, (_, i) => i * 2); // [0, 2, 4] — array-like + mapping function
Array.from(document.querySelectorAll('div')); // real array of DOM nodes, now supports .map(), .filter(), etc.
```

[↑ Back to top](#table-of-contents)

---

### 12. What makes an object "iterable" in JavaScript? 🔴

- "Iterable" means an object knows how to hand out its values one at a time, in a standard way that `for...of`, the spread operator (`...`), and destructuring all understand how to consume.
- Technically, an object becomes iterable by implementing a method named `Symbol.iterator` (a special built-in "well-known symbol" that JS uses as a reserved method name for this purpose). That method must return an **iterator** — an object with a `.next()` method. Each call to `.next()` returns a plain object `{ value, done }`: `value` is the next item, and `done` is `true` once there's nothing left to give out.
- Arrays, strings, `Map`s, and `Set`s all come with this built in already, which is why you can already use `for...of` and spread on them. You can add the same capability to your own custom objects by implementing `Symbol.iterator` yourself, as shown below.

```js
const range = {
  from: 1, to: 3,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        // Each call returns the next number, until we pass `last`
        return current <= last
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      },
    };
  },
};
[...range]; // [1, 2, 3] — works because it implements the iterable protocol
for (const n of range) console.log(n); // 1, 2, 3
```

[↑ Back to top](#table-of-contents)

---

### 13. What is a sparse array, and how does it affect methods like `map()`/`forEach()`? 🔴

- A normal array has a value (even if that value is `undefined`) sitting at every index. A **sparse array** has "holes" instead — indices that were never assigned anything at all. This can happen if you create an array with a given length but no values (`new Array(3)`), or delete an element without shifting the rest (`delete arr[1]`).
- A hole is different from storing `undefined` explicitly: `[undefined, undefined]` has two real elements, both holding the value `undefined`. `new Array(2)` has two holes — nothing is stored there yet.
- Iteration methods like `map()`, `forEach()`, and `filter()` **skip holes entirely** — the callback is never even called for those indices, unlike a real `undefined` value which would trigger the callback normally. This means a sparse array can silently produce fewer results than expected if you're not aware of holes.

```js
const sparse = [1, , 3]; // hole at index 1 (note the empty slot between the commas)
sparse.forEach(x => console.log(x)); // logs 1, then 3 — skips the hole entirely, no log for index 1
sparse.map(x => x * 2);              // [2, <1 empty item>, 6] — hole is preserved, not computed
sparse.length;                        // 3 — holes still count toward length
```

[↑ Back to top](#table-of-contents)

---

### 14. How would you implement your own `flatten(array, depth)` function? 🔴

- The core idea is **recursion** — a function that calls itself to solve a smaller version of the same problem. Go through each element of the array:
  - If it's **not** an array, just keep it as-is and add it to the result.
  - If it **is** an array and we still have depth remaining, call `flatten` again on that nested array with `depth - 1`, and merge its result in.
  - Once `depth` reaches `0`, stop recursing and return the array as-is (don't unwrap any further).
- `reduce()` is a convenient way to build up the flattened result: at each step, `concat` merges either a single value or a recursively-flattened sub-array into the accumulator.

```js
function flatten(arr, depth = 1) {
  if (depth < 1) return arr.slice(); // no more flattening allowed, return a shallow copy
  return arr.reduce(
    (acc, val) => acc.concat(Array.isArray(val) ? flatten(val, depth - 1) : val),
    []
  );
}
flatten([1, [2, [3, [4]]]], 2); // [1, 2, 3, [4]] — 2 levels unwrapped, [4] stays nested
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you flatten to **infinite** depth?
> - Can you implement this without recursion (using a stack)?

[↑ Back to top](#table-of-contents)

---
