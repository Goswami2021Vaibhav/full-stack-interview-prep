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

- A **primitive** is a basic, single value that isn't an object and has no methods of its own attached to it (see Q14 for how methods still seem to work on them). JavaScript has seven: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, and `bigint`.
- Everything that isn't one of those seven is a **reference type**, reported by `typeof` as `"object"` — this includes plain objects, arrays, functions, and dates.
- The key practical difference:
  - Primitives are **immutable** (a string like `"hi"` can never be changed in place — any "modification" actually creates a brand-new string) and are compared **by value**, meaning two primitives are equal if their contents match.
  - Objects are compared **by reference** — two separate objects with identical contents are *not* equal, because equality checks whether they're literally the same object in memory (see Q8 in this file for a full example).

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between `var`, `let`, and `const`? 🟢

| | Scope | Hoisting behavior | Re-assignable | Re-declarable |
|---|---|---|---|---|
| `var` | function | hoisted, initialized as `undefined` | yes | yes |
| `let` | block | hoisted, stays in TDZ | yes | no |
| `const` | block | hoisted, stays in TDZ | no (binding fixed) | no |

- **Scope** answers "where in the code is this variable visible?" `var` is **function-scoped** — it's visible anywhere inside the function it was declared in, even inside nested `{ }` blocks like `if` or `for`. `let`/`const` are **block-scoped** — they only exist inside the nearest pair of curly braces `{ }` they were declared in.
- **Hoisting behavior** (see the Core Concepts file for a full explanation of hoisting): all three are hoisted, but `var` becomes usable immediately as `undefined`, while `let`/`const` stay in the "Temporal Dead Zone" and throw if accessed before their declaration line runs.
- A common point of confusion: `const` does **not** make the *value* immutable — it only prevents **re-assigning the variable name to a new value**. If the value is an object or array, its contents can still be changed (mutated), because that doesn't change what the variable name points to.

```js
const arr = [1, 2];
arr.push(3); // fine — array contents changed, not the binding
arr = [];    // TypeError — can't reassign
```

[↑ Back to top](#table-of-contents)

---

### 3. What does `typeof` return, and what are its quirks? 🟢

- `typeof` is an operator that returns a string telling you the general category of a value: `"string"`, `"number"`, `"boolean"`, `"undefined"`, `"function"`, `"object"`, `"symbol"`, or `"bigint"`.
- Its most famous quirk: `typeof null` returns `"object"`, even though `null` is its own primitive type, not an object. This is a bug from JavaScript's very first version (1995) that has never been fixed because doing so would break too much existing code on the web.
- Another gotcha: arrays and dates are objects under the hood, so `typeof` reports both of them (and `null`) as `"object"` — it can't tell them apart. To distinguish them, use more specific tools: `Array.isArray(value)` for arrays, `instanceof` for checking against a specific constructor (e.g. `value instanceof Date`), or `Object.prototype.toString.call(value)` for a more precise, universal type tag (e.g. `"[object Array]"`).

```js
typeof null;        // "object"  (quirk)
typeof undefined;   // "undefined"
typeof function(){}; // "function"
typeof [];           // "object"
```

[↑ Back to top](#table-of-contents)

---

### 4. What is the difference between `null` and `undefined`? 🟢

- `undefined` means "no value has been set yet" — and it's usually **set automatically by JavaScript itself**, e.g. a declared-but-unassigned variable, a missing function argument, or accessing an object property that doesn't exist.
- `null` means "explicitly nothing" — it's a value a **developer deliberately assigns** to signal "this is intentionally empty" (e.g. `let user = null;` before a user logs in).
- So the difference is really about intent: `undefined` usually happens by default/accident, while `null` is a conscious choice in your code.
- `null == undefined` → `true`, because loose equality (`==`) has a special rule treating them as equal to each other (but not equal to anything else, like `0` or `""`). `null === undefined` → `false`, because strict equality also checks type, and they are different primitive types.

[↑ Back to top](#table-of-contents)

---

### 5. What are "truthy" and "falsy" values? 🟢

- Whenever JavaScript needs a value to behave like a boolean — for example, inside an `if (value)` condition, or with the `&&`/`||` operators — it automatically converts (coerces) that value to either `true` or `false`. A value that becomes `true` is called **truthy**; one that becomes `false` is called **falsy**.
- There are only **8 falsy values** in all of JavaScript, so it's easier to just memorize this short list than to guess: `false`, `0`, `-0`, `0n` (BigInt zero), `""` (empty string), `null`, `undefined`, and `NaN`.
- **Every other value is truthy** — including ones that feel like they "should" be false, which is a classic interview trap: the string `"0"` (non-empty string), the string `"false"` (also non-empty), an empty array `[]`, and an empty object `{}` are all truthy.

```js
if ([]) console.log('runs'); // runs — empty array is truthy
if ('0') console.log('runs'); // runs — non-empty string is truthy
```

[↑ Back to top](#table-of-contents)

---

### 6. Why is `NaN === NaN` false, and how do you actually check for `NaN`? 🟡

- `NaN` stands for "Not a Number" — it's the special value JavaScript returns when a math operation doesn't produce a legitimate number (e.g. `0 / 0` or `"abc" * 2`).
- Bizarrely, `NaN` is the **only value in JavaScript that is never equal to itself**, not even with `===`. This isn't a JS-specific bug — it comes from the IEEE-754 floating-point standard that most programming languages use for numbers, which defines `NaN` this way so that any calculation involving an invalid number reliably stays "flagged" as invalid instead of comparing equal to some other invalid result by coincidence.
- Because of this, you can never reliably detect `NaN` using `==` or `===`. Instead:
  - Use `Number.isNaN(value)` — this only returns `true` if the value is *literally* `NaN`, with no type conversion.
  - Avoid the older global `isNaN(value)` — it first tries to convert its argument to a number, which causes false positives. For example, `isNaN('hello')` is `true`, because `'hello'` can't be converted to a number (becomes `NaN`), even though `'hello'` itself was never `NaN`.

```js
NaN === NaN;          // false
Number.isNaN(NaN);    // true
Number.isNaN('hello'); // false (no coercion)
isNaN('hello');        // true  (coerces 'hello' -> NaN first)
```

[↑ Back to top](#table-of-contents)

---

### 7. What is type coercion, and what are common gotchas? 🟡

- **Type coercion** is JavaScript automatically converting a value from one type to another so an operation can proceed, without you explicitly asking for the conversion. It happens with operators like `+` and `==`, or when a value is used somewhere a different type is expected (like inside a template literal).
- The `+` operator is the trickiest one because it does double duty: if **either** operand is a string, `+` treats the whole operation as **string concatenation** (joining text together) rather than addition. Every other arithmetic operator (`-`, `*`, `/`) always coerces both sides to numbers first, since there's no "subtract two strings" concept.
- Knowing this one rule about `+` explains most of the surprising results below.

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

- **Primitives** (string, number, boolean, etc.) are stored and copied **by value** — when you assign one variable's value to another, or pass it into a function, JavaScript makes a completely independent copy. Changing the copy never affects the original.
- **Objects, arrays, and functions** are stored and copied **by reference** — the variable doesn't hold the actual object data itself, it holds a reference (like an address, pointing to where the real data lives in memory). When you assign or pass an object, you're copying that address, not the data — so both variables end up pointing at the very same underlying object. Changing the object through one variable is visible through the other, because there's really only one object.
- This is one of the most common sources of real bugs in JavaScript, so it's worth internalizing with the example below: notice how the primitive copy (`a`/`b`) stays independent, but the object copy (`obj1`/`obj2`) does not.

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

- **Destructuring** is a shorthand syntax for "unpacking" values out of an array, or properties out of an object, and assigning them straight to individual variables — instead of accessing each one manually with `array[0]` or `object.property` on separate lines.
- It also supports a few extra conveniences:
  - **Default values**: if a property/index doesn't exist (is `undefined`), you can supply a fallback value inline.
  - **Renaming**: when destructuring an object, you can pull out a property but store it under a different variable name.
  - **Nesting**: you can reach into nested objects/arrays directly in one destructuring pattern, instead of chaining multiple lines.

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

- **Template literals** are strings written with backticks (`` ` ``) instead of quotes. Their two big advantages over regular strings:
  - You can embed any JavaScript expression directly inside the string using `${...}`, and it gets evaluated and inserted automatically — no more manually gluing pieces together with `+`.
  - You can write strings that span **multiple lines** just by pressing enter inside the backticks, without needing `\n` escape codes or concatenation.
- A **tagged template** is a more advanced form: you put a function name right before the backtick string. Instead of JavaScript building the final string for you automatically, it instead calls your function with the literal's raw text pieces and the interpolated values as separate arguments, letting you process or transform them however you like before (optionally) producing a string yourself.

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

- Both use the exact same `...` syntax, but they do opposite jobs — the easiest way to tell them apart is by context: are you *building* something (spread) or *receiving* something (rest)?
  - **Spread**: takes an existing iterable (array, string, etc.) and **expands** it into its individual elements — used when constructing a new array/object literal, or passing many arguments into a function call.
  - **Rest**: does the reverse — it **collects** multiple individual values (like leftover function arguments, or remaining array elements after destructuring) and gathers them **into** a single new array.

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

- `Symbol` is a primitive type whose entire purpose is to produce a value that is **guaranteed unique** — calling `Symbol()` always returns a brand-new, one-of-a-kind value, even if you pass the exact same description string to two different calls (the description is just a debugging label, it doesn't affect uniqueness).
- Practical uses:
  - As **object property keys** that are guaranteed never to clash with any other property — useful when writing a library, so you can attach hidden metadata to an object without risking a collision with the user's own property names.
  - As **"well-known symbols"** — special built-in symbols the language itself looks for to enable specific behaviors. The most common one is `Symbol.iterator`, which an object can implement to declare "here's how to loop over my values," making it compatible with `for...of` and the spread operator.

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

- JavaScript's regular `number` type stores all numbers (including integers) as 64-bit floating-point values, which means it can only represent whole numbers **exactly** up to `Number.MAX_SAFE_INTEGER` (2⁵³ − 1, about 9 quadrillion). Beyond that, integers start silently losing precision — you can get wrong results without any error being thrown.
- `BigInt` is a separate primitive type designed specifically to represent integers of **arbitrary size**, exactly, with no upper limit (other than available memory).
- You create one either by appending an `n` suffix to an integer literal (e.g. `10n`), or by calling `BigInt(value)`.
- `BigInt` and regular `number` values are considered different types, so you **cannot mix them in arithmetic directly** — trying to add a `BigInt` and a regular number throws a `TypeError`. You must explicitly convert one to match the other first (e.g. `Number(big)` or `BigInt(num)`).

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

- Recall from Q1 that primitives are simple values with no methods attached — so how does `'hello'.toUpperCase()` work at all?
- The answer: whenever you try to access a property or call a method on a primitive, JavaScript automatically and **temporarily** wraps that primitive value in its corresponding "wrapper object" (a `String`, `Number`, or `Boolean` object, which *do* have methods). It calls the method on that wrapper, gets the result, and then immediately throws the wrapper away.
- This behind-the-scenes wrap-use-discard process is called **auto-boxing**, and it happens so fast and transparently that you never notice it — it's why `typeof 'abc'` still correctly reports `"string"` (the primitive itself was never permanently changed into an object) even though calling `.toUpperCase()` on it works fine.

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
