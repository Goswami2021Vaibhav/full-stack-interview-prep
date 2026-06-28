# Functions

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between a function declaration and a function expression?](#1-whats-the-difference-between-a-function-declaration-and-a-function-expression)
- [2. What's the difference between a regular function and an arrow function?](#2-whats-the-difference-between-a-regular-function-and-an-arrow-function)
- [3. What's the difference between a parameter and an argument?](#3-whats-the-difference-between-a-parameter-and-an-argument)
- [4. What are default parameters?](#4-what-are-default-parameters)
- [5. What is a callback function?](#5-what-is-a-callback-function)

**🟡 Medium**
- [6. What is a higher-order function?](#6-what-is-a-higher-order-function)
- [7. What is a pure function, and why does it matter?](#7-what-is-a-pure-function-and-why-does-it-matter)
- [8. What's the difference between `call()`, `apply()`, and `bind()`?](#8-whats-the-difference-between-call-apply-and-bind)
- [9. What is function currying?](#9-what-is-function-currying)
- [10. What's the difference between `function.length` and `arguments.length`?](#10-whats-the-difference-between-functionlength-and-argumentslength)
- [11. What is recursion, and what's a practical example?](#11-what-is-recursion-and-whats-a-practical-example)

**🔴 Hard**
- [12. What is a generator function, and how does `yield` work?](#12-what-is-a-generator-function-and-how-does-yield-work)
- [13. How would you implement a custom `bind()` polyfill?](#13-how-would-you-implement-a-custom-bind-polyfill)
- [14. What is function composition, and how would you implement `compose`/`pipe`?](#14-what-is-function-composition-and-how-would-you-implement-composepipe)

---

### 1. What's the difference between a function declaration and a function expression? 🟢

- **Declaration**: `function foo() {}` — hoisted fully, callable before its line in the code.
- **Expression**: `const foo = function () {}` — only the variable is hoisted (per `var`/`let`/`const` rules), the function itself is not usable before assignment.

```js
greet(); // works
function greet() { console.log('hi'); }

sayHi(); // TypeError: sayHi is not a function (at this point)
var sayHi = function () { console.log('hi'); };
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between a regular function and an arrow function? 🟢

- Arrow functions don't have their own `this`, `arguments`, or `super` — they inherit `this` lexically from the enclosing scope.
- Arrow functions can't be used as constructors (`new arrowFn()` throws).
- Arrow functions have no `prototype` property and can't be generators.
- Regular functions get a fresh `this` based on how they're called.

```js
const obj = {
  value: 42,
  regular() { return this.value; },
  arrow: () => this?.value,
};
obj.regular(); // 42
obj.arrow();   // undefined — `this` is lexical, not `obj`
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between a parameter and an argument? 🟢

- **Parameter**: the named variable listed in a function's definition.
- **Argument**: the actual value passed in when the function is called.

```js
function add(a, b) { // a, b = parameters
  return a + b;
}
add(2, 3); // 2, 3 = arguments
```

[↑ Back to top](#table-of-contents)

---

### 4. What are default parameters? 🟢

- Let you specify a fallback value used when an argument is `undefined` (not passed, or explicitly passed as `undefined`).
- Default expressions are evaluated **at call time**, and can reference earlier parameters.

```js
function greet(name = 'Guest', greeting = `Hello, ${name}`) {
  console.log(greeting);
}
greet();          // "Hello, Guest"
greet('Vaibhav'); // "Hello, Vaibhav"
```

[↑ Back to top](#table-of-contents)

---

### 5. What is a callback function? 🟢

- A function passed as an argument to another function, to be invoked later — either synchronously or asynchronously.
- Core pattern behind array methods (`map`, `filter`), event handlers, and older async APIs (before promises).

```js
[1, 2, 3].forEach(function callback(num) {
  console.log(num);
});
```

> [!TIP]
> **Real-life example:** `button.addEventListener('click', callback)` — the callback runs only when the user actually clicks.

[↑ Back to top](#table-of-contents)

---

### 6. What is a higher-order function? 🟡

- A function that **takes another function as an argument**, **returns a function**, or both.
- Array methods like `map`, `filter`, `reduce` are higher-order functions built into JS.

```js
function withLogging(fn) {
  return function (...args) {
    console.log('Calling with', args);
    return fn(...args);
  };
}
const loggedAdd = withLogging((a, b) => a + b);
loggedAdd(2, 3); // logs "Calling with [2, 3]", returns 5
```

[↑ Back to top](#table-of-contents)

---

### 7. What is a pure function, and why does it matter? 🟡

- Given the same input, **always returns the same output**, and has **no side effects** (doesn't modify external state, the DOM, or its arguments).
- Easier to test, reason about, memoize, and run in parallel — a core idea in functional programming.

```js
// Pure
function add(a, b) { return a + b; }

// Impure — depends on/mutates external state
let total = 0;
function addToTotal(n) { total += n; return total; }
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Is `Math.random()` a pure function? Why not?
> - What is "referential transparency," and how does it relate to purity?

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `call()`, `apply()`, and `bind()`? 🟡

- All three let you explicitly set what `this` refers to inside a function.
- `call(thisArg, a, b, ...)` — invokes immediately, arguments passed individually.
- `apply(thisArg, [a, b, ...])` — invokes immediately, arguments passed as an array.
- `bind(thisArg, a, b, ...)` — does **not** invoke immediately; returns a new function with `this` (and optionally some arguments) permanently bound.

```js
function greet(greeting) { console.log(`${greeting}, ${this.name}`); }
const person = { name: 'Vaibhav' };

greet.call(person, 'Hi');     // "Hi, Vaibhav" — runs now
greet.apply(person, ['Hey']); // "Hey, Vaibhav" — runs now

const bound = greet.bind(person, 'Yo');
bound(); // "Yo, Vaibhav" — runs later, when called
```

[↑ Back to top](#table-of-contents)

---

### 9. What is function currying? 🟡

- Transforming a function that takes multiple arguments into a sequence of functions that each take **one** argument.
- Useful for creating specialized, reusable versions of a general function (partial application).

```js
const multiply = (a) => (b) => (c) => a * b * c;
multiply(2)(3)(4); // 24

const double = multiply(2);
double(3)(4); // 24 — `a` is "locked in" as 2
```

> [!TIP]
> **Real-life example:** building a reusable `discountBy(percentage)` function that returns a price calculator for that specific discount.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `function.length` and `arguments.length`? 🟡

- `function.length`: number of **declared parameters** (excluding default/rest params) — fixed at definition time.
- `arguments.length` (inside the function body): number of **arguments actually passed** at call time.

```js
function foo(a, b, c = 1, ...rest) {}
foo.length; // 2 — only counts params before the first default/rest param

function bar(a, b) {
  console.log(arguments.length);
}
bar(1, 2, 3); // 3 — all arguments passed, regardless of declared params
```

[↑ Back to top](#table-of-contents)

---

### 11. What is recursion, and what's a practical example? 🟡

- A function that calls itself, with a **base case** to stop the recursion and avoid infinite calls / stack overflow.
- Natural fit for problems with a repeating, self-similar structure (tree traversal, factorial, nested data).

```js
function factorial(n) {
  if (n <= 1) return 1;       // base case
  return n * factorial(n - 1); // recursive case
}
factorial(5); // 120
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you forget the base case?
> - What is tail-call optimization, and does JS support it?

[↑ Back to top](#table-of-contents)

---

### 12. What is a generator function, and how does `yield` work? 🔴

- Declared with `function*`, a generator can **pause** execution at each `yield` and resume later, instead of running to completion in one go.
- Calling a generator function doesn't run its body — it returns an **iterator** object; each `.next()` call runs until the next `yield`.

```js
function* counter() {
  yield 1;
  yield 2;
  yield 3;
}
const gen = counter();
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }
gen.next(); // { value: 3, done: false }
gen.next(); // { value: undefined, done: true }
```

> [!TIP]
> **Real-life example:** generators are the foundation `async`/`await` was originally built on top of, and are useful for lazily producing infinite sequences (e.g. an ID generator) without computing everything upfront.

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement a custom `bind()` polyfill? 🔴

- Return a new function that, when called, invokes the original function with `this` set to the bound object, merging any preset and call-time arguments.

```js
Function.prototype.myBind = function (thisArg, ...presetArgs) {
  const originalFn = this;
  return function (...callArgs) {
    return originalFn.apply(thisArg, [...presetArgs, ...callArgs]);
  };
};

function greet(greeting) { console.log(`${greeting}, ${this.name}`); }
const bound = greet.myBind({ name: 'Vaibhav' }, 'Hi');
bound(); // "Hi, Vaibhav"
```

[↑ Back to top](#table-of-contents)

---

### 14. What is function composition, and how would you implement `compose`/`pipe`? 🔴

- Combining multiple simple functions into one, where each function's output feeds into the next function's input.
- `compose` runs right-to-left, `pipe` runs left-to-right — otherwise identical.

```js
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

const double = (n) => n * 2;
const addOne = (n) => n + 1;

compose(double, addOne)(5); // double(addOne(5)) = 12
pipe(double, addOne)(5);    // addOne(double(5)) = 11
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would `compose`/`pipe` need to change to support async functions?
> - How does this relate to chaining in libraries like lodash/Ramda?

[↑ Back to top](#table-of-contents)

---
