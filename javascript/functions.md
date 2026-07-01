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

- **Function declaration**: `function foo() {}`, written as a standalone statement with a name. Before your code runs, JS scans the file and moves the *entire* function (name and body) to the top of its scope — this behavior is called **hoisting**. Because the whole function is hoisted, you can call `foo()` earlier in the file than where it's written, and it still works.
- **Function expression**: `const foo = function () {}`, where a function is created as a value and assigned to a variable, like any other value (`const x = 5`). Here, hoisting rules follow whatever you declared the variable with (`var`, `let`, or `const`) — but only the *variable* is set up early, not the function value itself. Trying to call it before the assignment line runs will fail, because at that point the variable doesn't hold a function yet.

```js
greet(); // works — function declarations are fully hoisted
function greet() { console.log('hi'); }

sayHi(); // TypeError: sayHi is not a function (at this point)
var sayHi = function () { console.log('hi'); }; // assignment hasn't run yet
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between a regular function and an arrow function? 🟢

- The biggest difference is how each figures out what `this` refers to:
  - **Regular functions** get their own `this`, determined dynamically by *how the function is called* (e.g. calling it as `obj.method()` makes `this` be `obj`; calling it plain makes `this` be `undefined` in strict mode).
  - **Arrow functions** don't have their own `this` at all — they simply reuse whatever `this` was in the surrounding ("enclosing") code where the arrow function was written. This is called **lexical `this`** ("lexical" meaning "based on where it's written in the code," not "based on how it's called").
- Arrow functions also don't have their own `arguments` object or `super` — same idea, they inherit those from the enclosing scope too, if present.
- Arrow functions **can't be used as constructors** — writing `new arrowFn()` throws an error, because arrow functions were never designed to build new objects with `new`.
- Arrow functions have no `prototype` property (since they can't be constructors, there's no prototype to set up), and they can't be written as generator functions (no `function*` equivalent for arrows).

```js
const obj = {
  value: 42,
  regular() { return this.value; },   // `this` = obj, because called as obj.regular()
  arrow: () => this?.value,           // `this` = whatever `this` was outside `obj`, not `obj`
};
obj.regular(); // 42
obj.arrow();   // undefined — arrow function ignores obj entirely
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between a parameter and an argument? 🟢

- These two terms describe the same "slot," just at different points in time — definition versus usage — and people often use them interchangeably in casual conversation, but interviewers may still ask you to distinguish them.
- **Parameter**: the placeholder name you write when *defining* the function — it's a variable that will receive a value later.
- **Argument**: the actual, concrete value you pass in when *calling* the function.

```js
function add(a, b) { // a, b are parameters — placeholders, no values yet
  return a + b;
}
add(2, 3); // 2, 3 are arguments — the real values plugged into a and b
```

[↑ Back to top](#table-of-contents)

---

### 4. What are default parameters? 🟢

- Default parameters let you specify a fallback value for a parameter, which is used only when the caller doesn't provide that argument — either by leaving it out entirely, or by explicitly passing `undefined`. Passing any other "falsy" value like `0`, `''`, or `null` does **not** trigger the default (only `undefined` does).
- The default expression isn't calculated once ahead of time — it's evaluated fresh **every time the function is called** without that argument, and it's allowed to reference parameters that were declared before it in the parameter list.

```js
function greet(name = 'Guest', greeting = `Hello, ${name}`) {
  console.log(greeting);
}
greet();          // "Hello, Guest" — name defaults, then greeting uses that default name
greet('Vaibhav'); // "Hello, Vaibhav" — name is provided, greeting builds off it
```

[↑ Back to top](#table-of-contents)

---

### 5. What is a callback function? 🟢

- In JavaScript, functions are "first-class" values — they can be passed around just like a number or a string. A callback is simply a function that you pass as an argument into another function, so that the other function can call ("invoke") it at the right moment, rather than you calling it yourself directly.
- The callback might run **synchronously** (immediately, as part of the same operation — e.g. inside `forEach`) or **asynchronously** (later, after something else finishes — e.g. after a network request completes or a timer expires).
- This is the core pattern behind array methods like `map`/`filter`/`forEach` (they call your callback once per element), event handlers (the callback runs whenever the event fires), and older async APIs from before Promises existed (the callback runs once the async operation finishes).

```js
[1, 2, 3].forEach(function callback(num) {
  console.log(num); // forEach calls this callback once for each array element
});
```

> [!TIP]
> **Real-life example:** `button.addEventListener('click', callback)` — the callback runs only when the user actually clicks.

[↑ Back to top](#table-of-contents)

---

### 6. What is a higher-order function? 🟡

- "Higher-order" just means the function operates at a level above ordinary values — it treats *other functions* as its input or output, rather than just numbers/strings/objects. A function qualifies if it does at least one of these:
  - **Takes another function as an argument** (like `array.map(callback)` — it takes your callback and calls it internally).
  - **Returns a function** (like the example below, which builds and returns a new, enhanced version of a function).
  - Some higher-order functions do both.
- JS's own built-in array methods — `map`, `filter`, `reduce`, `forEach`, `sort` — are all higher-order functions, since each one takes a callback function as an argument.

```js
function withLogging(fn) {
  return function (...args) {           // returns a new function...
    console.log('Calling with', args);
    return fn(...args);                 // ...that wraps the original one
  };
}
const loggedAdd = withLogging((a, b) => a + b); // takes a function in
loggedAdd(2, 3); // logs "Calling with [2, 3]", returns 5
```

[↑ Back to top](#table-of-contents)

---

### 7. What is a pure function, and why does it matter? 🟡

- A function is "pure" if it satisfies two rules:
  1. **Same input always gives the same output** — call it with `add(2, 3)` a hundred times, and it always returns `5`; it never depends on anything outside its own arguments (like a global variable, the current time, or random numbers).
  2. **No side effects** — it doesn't change anything outside of itself: no modifying a variable declared outside the function, no touching the DOM, no mutating the objects/arrays it was given as arguments, no logging, no network calls.
- Why this matters: pure functions are much easier to reason about, because you can look at just the function itself (and its inputs) to understand exactly what it will do — no need to track down what else in the program it might affect or depend on. This also makes them easier to **test** (no setup/mocking needed), **memoize** (cache results safely, since the same input always gives the same output), and run **in parallel** (since they can't interfere with each other by touching shared state).
- This idea — building programs mostly out of pure functions — is the foundation of functional programming.

```js
// Pure — output depends only on a and b, changes nothing else
function add(a, b) { return a + b; }

// Impure — depends on and mutates `total`, which lives outside the function
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

- All three methods exist to explicitly control what `this` refers to inside a function — useful when a function needs to run with a `this` other than whatever it would normally get based on how it's called (see Q2).
- `call(thisArg, a, b, ...)` — runs the function **immediately**, and you pass arguments individually, one at a time, separated by commas.
- `apply(thisArg, [a, b, ...])` — also runs the function **immediately**, but you pass all arguments bundled together as a single array. Handy when you already have your arguments as an array rather than separate variables.
- `bind(thisArg, a, b, ...)` — the odd one out: it does **not** run the function right away. Instead, it returns a **new function** that has `this` (and optionally some arguments) permanently locked in, ready to be called later, as many times as you want.

```js
function greet(greeting) { console.log(`${greeting}, ${this.name}`); }
const person = { name: 'Vaibhav' };

greet.call(person, 'Hi');     // "Hi, Vaibhav" — runs immediately
greet.apply(person, ['Hey']); // "Hey, Vaibhav" — runs immediately, args as array

const bound = greet.bind(person, 'Yo'); // returns a new function, doesn't run yet
bound(); // "Yo, Vaibhav" — runs later, whenever you call it
```

[↑ Back to top](#table-of-contents)

---

### 9. What is function currying? 🟡

- Currying is a transformation: instead of one function that takes several arguments all at once (`fn(a, b, c)`), you rewrite it as a chain of functions that each take **exactly one** argument and return the next function in the chain (`fn(a)(b)(c)`).
- Why bother: it lets you "lock in" some arguments early and get back a smaller, specialized, reusable function — this is called **partial application**. You call the curried function with just the first argument, save the function it returns, and reuse that as many times as you like with different remaining arguments.

```js
const multiply = (a) => (b) => (c) => a * b * c; // curried: one argument at a time
multiply(2)(3)(4); // 24 — calling all three at once

const double = multiply(2);  // `a` is locked in as 2; returns a function expecting (b)(c)
double(3)(4); // 24 — same result, reusing `double` as a specialized function
```

> [!TIP]
> **Real-life example:** building a reusable `discountBy(percentage)` function that returns a price calculator for that specific discount.

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `function.length` and `arguments.length`? 🟡

- These two look similar but measure completely different things:
- `function.length` (a property on the function itself, checked from outside): counts how many parameters were **declared** in the function's definition — but only up to (and not including) the first parameter that has a default value or is a rest parameter (`...rest`). It's fixed the moment the function is defined and never changes.
- `arguments.length` (accessed from **inside** the function body, via the special `arguments` object every regular function has): counts how many arguments were **actually passed in** at the moment this particular call happened — this can be more or fewer than the number of declared parameters, and is different on every call.

```js
function foo(a, b, c = 1, ...rest) {}
foo.length; // 2 — stops counting at `c`, since it has a default value

function bar(a, b) {
  console.log(arguments.length);
}
bar(1, 2, 3); // 3 — all 3 passed-in arguments count, even though only a, b are declared
```

[↑ Back to top](#table-of-contents)

---

### 11. What is recursion, and what's a practical example? 🟡

- Recursion is when a function calls **itself** to solve a smaller version of the same problem, repeating until it reaches a simple case it can answer directly.
- Every recursive function needs a **base case** — a condition that stops the recursion and returns an answer directly, without calling itself again. Without one, the function keeps calling itself forever, which eventually crashes with a "stack overflow" error (the call stack, mentioned in the error-handling notes, runs out of space to track all the nested calls).
- Recursion is a natural fit for problems that are made up of smaller versions of themselves — a "self-similar" structure — like traversing a tree (each subtree is itself a smaller tree), computing a factorial (`n!` is `n` times `(n-1)!`), or walking through nested data (an object containing objects containing objects).

```js
function factorial(n) {
  if (n <= 1) return 1;        // base case — stops the recursion here
  return n * factorial(n - 1); // recursive case — calls itself with a smaller n
}
factorial(5); // 120 — computed as 5 * 4 * 3 * 2 * 1
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you forget the base case?
> - What is tail-call optimization, and does JS support it?

[↑ Back to top](#table-of-contents)

---

### 12. What is a generator function, and how does `yield` work? 🔴

- A normal function, once called, always runs start to finish in one uninterrupted go. A generator function — declared with the special `function*` syntax — is different: it can **pause itself in the middle** and later pick up exactly where it left off, instead of running straight through.
- Calling a generator function does **not** run its body right away, unlike a normal function. Instead, it immediately returns a special **iterator** object (an object with a `.next()` method used to step through a sequence of values one at a time).
- Each time you call `.next()` on that iterator, the generator's code runs **up to the next `yield` keyword**, pauses there, and hands back whatever value was `yield`ed, wrapped in an object like `{ value, done }`. Calling `.next()` again resumes execution right after that `yield`, continuing until the next `yield` (or until the function ends, at which point `done` becomes `true`).

```js
function* counter() {
  yield 1; // pauses here on first .next()
  yield 2; // pauses here on second .next()
  yield 3; // pauses here on third .next()
}
const gen = counter(); // body hasn't run yet — just creates the iterator
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }
gen.next(); // { value: 3, done: false }
gen.next(); // { value: undefined, done: true } — function has finished
```

> [!TIP]
> **Real-life example:** generators are the foundation `async`/`await` was originally built on top of, and are useful for lazily producing infinite sequences (e.g. an ID generator) without computing everything upfront.

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement a custom `bind()` polyfill? 🔴

- A "polyfill" is a hand-written implementation of a built-in feature, used to understand (or replicate) how it works internally. To build our own `bind()`, we need to reproduce exactly what it does: return a **new function** that, whenever it's eventually called, invokes the original function with `this` forced to be the bound object, and with any "preset" arguments (given at bind time) combined with whatever arguments are passed at the actual call time.
- Step by step:
  1. Inside `myBind`, `this` refers to the original function being bound (since `myBind` is called as `someFunction.myBind(...)`) — save it as `originalFn` so it isn't lost.
  2. Collect the arguments passed to `myBind` itself (besides `thisArg`) as `presetArgs`.
  3. Return a new function that, when later called with its own arguments (`callArgs`), uses `apply()` (see Q8) to invoke `originalFn` with `thisArg` and both sets of arguments merged together.

```js
Function.prototype.myBind = function (thisArg, ...presetArgs) {
  const originalFn = this; // the function myBind was called on
  return function (...callArgs) {
    // merge preset args (from bind time) with call args (from call time)
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

- Function composition means combining several small, single-purpose functions into one bigger function, where the output of each step becomes the input to the next — like an assembly line, passing a value from one worker to the next.
- `compose` and `pipe` are two helper functions that both build this assembly line automatically, given a list of functions — they only differ in which direction they read:
  - `compose(f, g)` applies functions **right-to-left**: it runs `g` first, then feeds its result into `f`. So `compose(f, g)(x)` means `f(g(x))`.
  - `pipe(f, g)` applies functions **left-to-right**: it runs `f` first, then feeds its result into `g`. So `pipe(f, g)(x)` means `g(f(x))`.
- Both are implemented with `reduce`/`reduceRight` (see Array methods notes), which repeatedly apply each function to the running result, starting from the initial input `x`.

```js
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x); // right-to-left
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);         // left-to-right

const double = (n) => n * 2;
const addOne = (n) => n + 1;

compose(double, addOne)(5); // double(addOne(5)) = double(6) = 12
pipe(double, addOne)(5);    // addOne(double(5)) = addOne(10) = 11
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would `compose`/`pipe` need to change to support async functions?
> - How does this relate to chaining in libraries like lodash/Ramda?

[↑ Back to top](#table-of-contents)

---
