# Data Types & Variables

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are the primitive data types in JavaScript?](#1-what-are-the-primitive-data-types-in-javascript)
- [2. What's the difference between `var`, `let`, and `const`?](#2-whats-the-difference-between-var-let-and-const)
- [3. What does `typeof` return, and what are its quirks?](#3-what-does-typeof-return-and-what-are-its-quirks)
- [4. What is the difference between `null` and `undefined`?](#4-what-is-the-difference-between-null-and-undefined)
- [5. What are "truthy" and "falsy" values?](#5-what-are-truthy-and-falsy-values)

**🟡 Medium**
- [6. Why is `NaN === NaN` false, and how do you actually check for `NaN`?](#6-why-is-nan--nan-false-and-how-do-you-actually-check-for-nan)
- [7. What is type coercion, and what are common gotchas?](#7-what-is-type-coercion-and-what-are-common-gotchas)
- [8. What's the difference between primitive types and reference types?](#8-whats-the-difference-between-primitive-types-and-reference-types)
- [9. What is destructuring assignment?](#9-what-is-destructuring-assignment)
- [10. What are template literals, and what is a tagged template?](#10-what-are-template-literals-and-what-is-a-tagged-template)
- [11. What's the difference between the spread operator and rest parameters?](#11-whats-the-difference-between-the-spread-operator-and-rest-parameters)

**🔴 Hard**
- [12. What is a `Symbol`, and what's it used for?](#12-what-is-a-symbol-and-whats-it-used-for)
- [13. What is `BigInt`, and when would you need it?](#13-what-is-bigint-and-when-would-you-need-it)
- [14. How does JavaScript let you call methods like `.toUpperCase()` on a string primitive?](#14-how-does-javascript-let-you-call-methods-like-touppercase-on-a-string-primitive)

---

### 1. What are the primitive data types in JavaScript? 🟢

- Seven primitives: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
- Everything else (objects, arrays, functions, dates, etc.) is a **reference type** (`object`).
- Primitives are immutable and compared **by value**; objects are compared **by reference**.

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between `var`, `let`, and `const`? 🟢

| | Scope | Hoisting behavior | Re-assignable | Re-declarable |
|---|---|---|---|---|
| `var` | function | hoisted, initialized as `undefined` | yes | yes |
| `let` | block | hoisted, stays in TDZ | yes | no |
| `const` | block | hoisted, stays in TDZ | no (binding fixed) | no |

- `const` doesn't make a value immutable — it just prevents **re-assigning the variable**. Objects/arrays declared with `const` can still have their contents mutated.

```js
const arr = [1, 2];
arr.push(3); // fine — array contents changed, not the binding
arr = [];    // TypeError — can't reassign
```

[↑ Back to top](#table-of-contents)

---

### 3. What does `typeof` return, and what are its quirks? 🟢

- Returns a string naming the type: `"string"`, `"number"`, `"boolean"`, `"undefined"`, `"function"`, `"object"`, `"symbol"`, `"bigint"`.
- Famous quirk: `typeof null` returns `"object"` — a long-standing bug kept for backward compatibility.
- Arrays, dates, and `null` all report as `"object"` — use `Array.isArray()`, `instanceof`, or `Object.prototype.toString.call()` to tell them apart.

```js
typeof null;        // "object"  (quirk)
typeof undefined;   // "undefined"
typeof function(){}; // "function"
typeof [];           // "object"
```

[↑ Back to top](#table-of-contents)

---

### 4. What is the difference between `null` and `undefined`? 🟢

- `undefined`: a variable has been declared but not yet assigned a value — JS sets this automatically.
- `null`: an **intentional** absence of value, explicitly assigned by the developer.
- `null == undefined` → `true` (loose equality treats them as equal); `null === undefined` → `false` (different types).

[↑ Back to top](#table-of-contents)

---

### 5. What are "truthy" and "falsy" values? 🟢

- Every value coerces to `true` or `false` in a boolean context (like an `if` condition).
- **Falsy values (only 8):** `false`, `0`, `-0`, `0n` (BigInt zero), `""`, `null`, `undefined`, `NaN`.
- Everything else — including `"0"`, `"false"`, `[]`, and `{}` — is **truthy**.

```js
if ([]) console.log('runs'); // runs — empty array is truthy
if ('0') console.log('runs'); // runs — non-empty string is truthy
```

[↑ Back to top](#table-of-contents)

---

### 6. Why is `NaN === NaN` false, and how do you actually check for `NaN`? 🟡

- `NaN` ("Not a Number") is the only value in JS that is **not equal to itself**, by IEEE-754 floating point spec design.
- Don't use `==`/`===` to check for it. Use `Number.isNaN(value)` (strict, no coercion) rather than the global `isNaN(value)` (which coerces its argument first and can give false positives, e.g. `isNaN('hello')` is `true`).

```js
NaN === NaN;          // false
Number.isNaN(NaN);    // true
Number.isNaN('hello'); // false (no coercion)
isNaN('hello');        // true  (coerces 'hello' -> NaN first)
```

[↑ Back to top](#table-of-contents)

---

### 7. What is type coercion, and what are common gotchas? 🟡

- Automatic conversion of a value from one type to another, usually triggered by operators like `+`, `==`, or template literals.
- `+` prefers string concatenation if either operand is a string; other arithmetic operators (`-`, `*`, `/`) coerce both sides to numbers.

```js
1 + '2';   // '12'  (number -> string)
'5' - 2;   // 3     (string -> number)
1 + true;  // 2     (boolean -> number)
[] + [];   // ''    (both arrays coerced to '')
[] + {};   // '[object Object]'
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does `[] + []` produce an empty string instead of an error?
> - What's the difference between implicit and explicit coercion?

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between primitive types and reference types? 🟡

- **Primitives** are stored and copied **by value** — assigning or passing one creates an independent copy.
- **Objects/arrays/functions** are stored and copied **by reference** — assigning or passing one copies the reference (memory address), so both variables point to the same underlying data.

```js
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 — unaffected

const obj1 = { val: 10 };
const obj2 = obj1;
obj2.val = 20;
console.log(obj1.val); // 20 — same object, both see the change
```

> [!TIP]
> **Real-life example:** passing a settings object into a function and having it accidentally mutate the caller's original object — a common source of bugs caused by reference semantics.

[↑ Back to top](#table-of-contents)

---

### 9. What is destructuring assignment? 🟡

- Syntax for unpacking values from arrays or properties from objects into individual variables in one line.
- Supports default values, renaming, and nesting.

```js
const { name, age = 18 } = { name: 'Vaibhav' };
// name = 'Vaibhav', age = 18 (default used)

const [first, , third] = [1, 2, 3]; // skip index 1
// first = 1, third = 3

const { user: { id } } = { user: { id: 42 } }; // nested
```

[↑ Back to top](#table-of-contents)

---

### 10. What are template literals, and what is a tagged template? 🟡

- Template literals (backtick strings) support embedded expressions (`${...}`) and multi-line strings without `\n` or concatenation.
- A **tagged template** passes the literal's strings and interpolated values to a function for custom processing instead of directly building a string.

```js
const name = 'Vaibhav';
console.log(`Hello, ${name}!`); // "Hello, Vaibhav!"

function tag(strings, ...values) {
  return strings.reduce((acc, str, i) => acc + str + (values[i] ?? ''), '');
}
tag`Hi ${name}, welcome!`; // custom-processed string
```

> [!TIP]
> **Real-life example:** the `styled-components` library uses tagged templates to parse CSS written inside JS template strings.

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between the spread operator and rest parameters? 🟡

- Same `...` syntax, opposite direction:
  - **Spread**: expands an iterable into individual elements (used in array/object literals, function calls).
  - **Rest**: collects multiple remaining arguments/elements **into** a single array.

```js
// Spread
const arr1 = [1, 2];
const arr2 = [...arr1, 3]; // [1, 2, 3]

// Rest
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3); // 6
```

[↑ Back to top](#table-of-contents)

---

### 12. What is a `Symbol`, and what's it used for? 🔴

- A primitive type that creates a **guaranteed-unique** value, even if two symbols have the same description.
- Used to create object keys that won't collide with other code's property names, and to define special "well-known" behaviors (e.g. `Symbol.iterator` to make an object iterable).

```js
const id1 = Symbol('id');
const id2 = Symbol('id');
id1 === id2; // false — always unique

const obj = { [id1]: 123 };
```

> [!TIP]
> **Real-life example:** `Symbol.iterator` is what makes arrays, strings, and `Map`/`Set` work with `for...of` loops.

[↑ Back to top](#table-of-contents)

---

### 13. What is `BigInt`, and when would you need it? 🔴

- A primitive for representing integers larger than `Number.MAX_SAFE_INTEGER` (2⁵³ − 1), beyond which regular numbers lose precision.
- Created by appending `n` to an integer literal, or via `BigInt(value)`.
- Cannot be mixed with regular numbers in arithmetic without explicit conversion (`TypeError` if you try).

```js
const big = 9007199254740993n;
const normal = Number.MAX_SAFE_INTEGER + 2;
console.log(normal); // 9007199254740992 — wrong, lost precision
console.log(big);    // 9007199254740993n — correct

big + 1; // TypeError: Cannot mix BigInt and other types
```

[↑ Back to top](#table-of-contents)

---

### 14. How does JavaScript let you call methods like `.toUpperCase()` on a string primitive? 🔴

- Primitives have no methods of their own. When you access a property/method on one, JS **temporarily wraps it** in the corresponding wrapper object (`String`, `Number`, `Boolean`), calls the method, then discards the wrapper.
- This is called **auto-boxing**. It's why `typeof 'abc'` is still `"string"` even though `'abc'.toUpperCase()` works.

```js
const str = 'hello';
str.toUpperCase(); // 'HELLO' — wrapped in a String object behind the scenes, then unwrapped
typeof str;         // still "string", not "object"
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you create a string with `new String('hello')` instead of a literal?
> - Why is `new String('a') === 'a'` false?

[↑ Back to top](#table-of-contents)

---
