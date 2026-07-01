# Objects & Prototypes

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are the different ways to create an object in JavaScript?](#1-what-are-the-different-ways-to-create-an-object-in-javascript)
- [2. What's the difference between dot notation and bracket notation?](#2-whats-the-difference-between-dot-notation-and-bracket-notation)
- [3. What's the difference between `Object.freeze()` and `Object.seal()`?](#3-whats-the-difference-between-objectfreeze-and-objectseal)
- [4. How do you check if an object has a property?](#4-how-do-you-check-if-an-object-has-a-property)
- [5. What's the difference between `Object.keys()`, `Object.values()`, and `Object.entries()`?](#5-whats-the-difference-between-objectkeys-objectvalues-and-objectentries)

**🟡 Medium**
- [6. What is the prototype chain?](#6-what-is-the-prototype-chain)
- [7. What's the difference between `__proto__` and `prototype`?](#7-whats-the-difference-between-proto-and-prototype)
- [8. What is `Object.create()`, and how is it used for inheritance?](#8-what-is-objectcreate-and-how-is-it-used-for-inheritance)
- [9. What's the difference between a shallow copy and a deep copy?](#9-whats-the-difference-between-a-shallow-copy-and-a-deep-copy)
- [10. What are getters and setters on an object?](#10-what-are-getters-and-setters-on-an-object)
- [11. What's the difference between `Object.assign()` and the spread operator for copying objects?](#11-whats-the-difference-between-objectassign-and-the-spread-operator-for-copying-objects)

**🔴 Hard**
- [12. How would you implement your own deep clone function?](#12-how-would-you-implement-your-own-deep-clone-function)
- [13. What are property descriptors?](#13-what-are-property-descriptors)
- [14. How does prototypal inheritance differ from classical inheritance?](#14-how-does-prototypal-inheritance-differ-from-classical-inheritance)

---

### 1. What are the different ways to create an object in JavaScript? 🟢

An "object" in JS is just a collection of key-value pairs (plus, optionally, a link to another object it can borrow behavior from — see Q6 on the prototype chain). There are several ways to build one:

- **Object literal** — `{ key: 'value' }`. The simplest and most common way; just write the object directly.
- **Constructor function** — `new Person()`. A regular function that's meant to be called with `new` so it can stamp out many similar objects (see [oops.md](oops.md#2-what-is-a-constructor-function)).
- **`Object.create(proto)`** — builds an object and explicitly wires up what it should inherit from (its "prototype"). Passing `null` gives you an object with no inherited methods at all, not even `toString`.
- **ES6 `class`** — `new MyClass()`. Modern syntax that does the same thing constructor functions do, just with cleaner syntax (it's "sugar" on top of prototypes).
- **`Object.assign({}, ...)` or spread `{ ...obj }`** — builds a new object by copying properties out of one or more existing objects.

```js
const a = { x: 1 };               // literal
const b = new Object();           // constructor
const c = Object.create(null);    // no prototype at all — no inherited methods
class Point { constructor(x) { this.x = x; } }
const d = new Point(5);           // class instance
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between dot notation and bracket notation? 🟢

Both are just two ways of reading or writing a property on an object — they end up doing the same thing, but they accept different kinds of keys:

- **Dot notation** (`obj.key`) — you type the property name directly in the code. Because of this, the name has to be a valid identifier (letters, digits, `_`, `$`, no spaces, can't start with a digit) and it has to be known when you write the code — you can't swap it out at runtime.
- **Bracket notation** (`obj['key']`) — the key is a string (or any expression that evaluates to a string), so it can hold spaces, hyphens, reserved words, or even a variable computed while the program is running.

```js
const obj = { 'first-name': 'Vaibhav' };
obj.first-name;      // wrong — JS reads this as (obj.first) - (name), not a property lookup
obj['first-name'];   // 'Vaibhav' — works, because the key is passed as a string

const key = 'first-name';
obj[key]; // 'Vaibhav' — dynamic access; only possible with brackets, since `key` is a variable
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between `Object.freeze()` and `Object.seal()`? 🟢

Both "lock down" an object so it can't be changed as freely, but they differ in how strict they are:

- `Object.freeze()` — the strictest option. You can't add new properties, can't remove existing ones, and can't change the value of existing ones either. The object becomes fully read-only.
- `Object.seal()` — a bit more relaxed. You still can't add or remove properties, but any property that already exists can have its **value** changed.
- Both are **shallow**: they only lock the object's own top-level properties. If a property's value is itself an object, that nested object is still fully mutable.

```js
const frozen = Object.freeze({ a: 1 });
frozen.a = 2; // silently fails (throws a TypeError in strict mode) — value stays 1

const sealed = Object.seal({ a: 1 });
sealed.a = 2; // works — existing values can still change
sealed.b = 3; // fails silently — can't add new properties
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Is `Object.freeze()` deep or shallow? How would you make a deep freeze?

[↑ Back to top](#table-of-contents)

---

### 4. How do you check if an object has a property? 🟢

There are three common ways, and they don't always agree with each other — the difference matters when a property is *inherited* rather than defined directly on the object:

- `'key' in obj` — returns `true` if the property exists anywhere on the object **or anywhere up its prototype chain** (the chain of objects it inherits methods/properties from — see Q6).
- `obj.hasOwnProperty('key')` — returns `true` only if the property is defined directly **on the object itself**, ignoring anything it inherited.
- `obj.key !== undefined` — the simplest-looking check, but it's misleading: if the property exists but was deliberately set to `undefined`, this incorrectly reports "doesn't have it."

```js
const obj = { a: 1 };
'toString' in obj;              // true — inherited from Object.prototype, further up the chain
obj.hasOwnProperty('toString'); // false — not defined directly on `obj`
obj.hasOwnProperty('a');        // true — defined directly on `obj`

const obj2 = { a: undefined };
obj2.a !== undefined;   // false — wrongly suggests the property is missing
'a' in obj2;            // true — correctly reports it exists
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `Object.keys()`, `Object.values()`, and `Object.entries()`? 🟢

All three turn a plain object into an array so you can loop over it, filter it, or map it like any other array — they just extract different parts:

- `Object.keys(obj)` → an array of the object's own property **names** (as strings). "Own" means it ignores anything inherited from the prototype chain.
- `Object.values(obj)` → an array of just the **values**, in the same order as `Object.keys()` would give the names.
- `Object.entries(obj)` → an array of `[key, value]` pairs — useful for `for...of` loops, or for converting a plain object into a `Map`.

```js
const user = { name: 'Vaibhav', age: 25 };
Object.keys(user);    // ['name', 'age']
Object.values(user);  // ['Vaibhav', 25]
Object.entries(user); // [['name', 'Vaibhav'], ['age', 25]]

// entries() pairs nicely with for...of:
for (const [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`); // "name: Vaibhav", then "age: 25"
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the prototype chain? 🟡

- Every object in JS has a hidden internal link (called `[[Prototype]]`) pointing to another object — think of it as "the object I fall back on if I don't have what's being asked for."
- When you try to read a property, JS first looks on the object itself. If it's not there, it follows the link to the next object, then the next, and so on — this chain of "fallback" objects is the **prototype chain**. The search stops once the property is found, or once it reaches an object whose link is `null` (nothing left to check).
- This is exactly how built-in methods work without being copied onto every single value: `.map()` isn't stored on every array you create — it lives once on `Array.prototype`, and every array's prototype chain leads there.

```js
const arr = [1, 2, 3];
arr.map; // found by walking up: not on `arr` itself, found on Array.prototype

// The chain, step by step:
arr.__proto__ === Array.prototype;              // true — arr's fallback is Array.prototype
Array.prototype.__proto__ === Object.prototype; // true — Array.prototype's fallback is Object.prototype
Object.prototype.__proto__;                     // null — end of the chain, nothing further to check
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `__proto__` and `prototype`? 🟡

These two names look similar and are related, but they live on different things:

- `prototype` — a regular property that exists **only on functions/classes used as constructors** (things you plan to call with `new`). It holds the object that will become the `[[Prototype]]` of every instance created from that constructor. Think of it as "the blueprint of shared methods every instance will fall back on."
- `__proto__` — a property that exists **on instances/objects**, and is basically a window into that object's actual `[[Prototype]]` link. For an object created with `new Person()`, `p.__proto__` and `Person.prototype` point to the exact same object.
- `__proto__` is an old, informal way to access this link, kept around only for backward compatibility. The modern, recommended way to read it is `Object.getPrototypeOf(obj)`, and to set it, `Object.setPrototypeOf(obj, proto)`.

```js
function Person() {}
const p = new Person();
p.__proto__ === Person.prototype;              // true — same object, two ways to reach it
Object.getPrototypeOf(p) === Person.prototype; // true — the preferred, modern way to check
```

[↑ Back to top](#table-of-contents)

---

### 8. What is `Object.create()`, and how is it used for inheritance? 🟡

- `Object.create(proto)` builds a brand-new, empty object and directly wires its `[[Prototype]]` (its "fallback" link, see Q6) to whatever object you pass in.
- It's the most explicit, no-frills way to set up prototypal inheritance — you're not using a constructor function or a class at all, just saying "this new object should inherit from that object."

```js
const animal = {
  speak() { console.log(`${this.name} makes a sound.`); },
};
const dog = Object.create(animal); // dog's prototype is now `animal`
dog.name = 'Rex';
dog.speak(); // "Rex makes a sound." — dog doesn't have speak() itself,
             // so JS walks the prototype chain and finds it on `animal`
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between a shallow copy and a deep copy? 🟡

- **Shallow copy**: only the outer, top-level properties get copied into a new object. But if a property's value is itself an object or array, the copy just stores the same **reference** to it — both the original and the copy end up pointing at the same nested object. Change the nested object through one, and you'll see the change through the other too.
- **Deep copy**: goes further down — it recursively copies every nested object/array as well, so the result has no shared references with the original at all. Changing the copy never affects the original, no matter how deeply nested the change is.

```js
const original = { a: 1, nested: { b: 2 } };
const shallow = { ...original }; // spread only copies the top level
shallow.nested.b = 99;           // this mutates the SAME nested object original points to
console.log(original.nested.b);  // 99 — nested object was shared, not copied!

const deep = structuredClone(original); // built-in true deep copy (modern browsers/Node)
deep.nested.b = 100;
console.log(original.nested.b); // still 99 — completely independent now
```

[↑ Back to top](#table-of-contents)

---

### 10. What are getters and setters on an object? 🟡

- A **getter** (`get`) is a function that runs automatically whenever a property is **read**, and a **setter** (`set`) is a function that runs whenever a property is **written to**. From the outside, they still look and behave like plain properties — the caller just writes `user.fullName`, with no idea a function ran behind the scenes.
- They're useful for adding logic — validation, computed values, logging — without changing how the property is used.

```js
const user = {
  firstName: 'Vaibhav',
  lastName: 'Goswami',
  get fullName() {
    // runs every time someone reads user.fullName
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    // runs every time someone writes user.fullName = ...
    [this.firstName, this.lastName] = value.split(' ');
  },
};
user.fullName;         // "Vaibhav Goswami" — getter ran, computed from firstName + lastName
user.fullName = 'A B'; // setter ran — updates firstName to "A", lastName to "B"
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between `Object.assign()` and the spread operator for copying objects? 🟡

- Both copy properties from one or more source objects into a target — and both only go one level deep (a **shallow** copy, see Q9), so for plain objects they behave almost identically.
- The real difference is *what* they act on: `Object.assign(target, ...sources)` copies properties **onto an existing object** you pass as the first argument — it mutates that `target` and also returns it.
- Spread (`{ ...obj }`) always builds a **brand-new object literal** — it never mutates anything that already exists, which makes it easier to reason about and is generally the preferred style in modern code.

```js
const a = { x: 1 };
const merged1 = Object.assign({}, a, { y: 2 }); // { x: 1, y: 2 } — {} is the target being mutated
const merged2 = { ...a, y: 2 };                  // { x: 1, y: 2 } — same result, new object literal

// Where the mutation shows up:
const target = { x: 1 };
Object.assign(target, { y: 2 });
target; // { x: 1, y: 2 } — target itself was changed in place
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement your own deep clone function? 🔴

- The core idea is recursion: to deep-clone a value, copy each of its properties one by one; if a property's value is itself an object or array, don't just copy the reference — call the clone function on it too, so it gets copied all the way down.
- Primitives (numbers, strings, booleans, `null`, `undefined`) are returned as-is, since they're copied by value automatically in JS — there's nothing to "clone."
- Real-world edge cases to think about: arrays vs. plain objects (need different empty containers to start from), special built-in types like `Date` or `Map`, and **circular references** — an object that (directly or indirectly) contains a reference to itself. Without special handling, cloning a circular structure would recurse forever and crash. The fix is to remember ("seen") every object we've already started cloning, using a `WeakMap` (a Map-like structure that doesn't prevent its keys from being garbage collected).

```js
function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== 'object') return value; // primitives: nothing to clone

  if (seen.has(value)) return seen.get(value); // already cloning this object — reuse the clone, avoid infinite loop

  const clone = Array.isArray(value) ? [] : {};
  seen.set(value, clone); // remember this clone BEFORE recursing, so circular refs resolve correctly

  for (const key in value) {
    clone[key] = deepClone(value[key], seen); // recurse into nested values
  }
  return clone;
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How does `structuredClone()` compare to a hand-rolled version like this?
> - Why doesn't `JSON.parse(JSON.stringify(obj))` work as a general-purpose deep clone (functions, `undefined`, `Date`, circular refs)?

[↑ Back to top](#table-of-contents)

---

### 13. What are property descriptors? 🔴

- Every property on an object has hidden metadata describing how it's allowed to behave — this metadata is the "property descriptor." You normally never see it, since object literals set sensible defaults, but you can inspect it with `Object.getOwnPropertyDescriptor()` and control it precisely with `Object.defineProperty()`.
- There are four flags in a descriptor:
  - `value` — the actual value stored.
  - `writable` — can the value be reassigned later?
  - `enumerable` — does it show up when looping with `for...in` or listing with `Object.keys()`?
  - `configurable` — can the property be deleted, or can its descriptor be changed again later?
- Properties created the normal way (`obj.x = 1` or in a literal) get all four flags set to `true` automatically. But `Object.defineProperty()` defaults every flag to `false` unless you explicitly set it — so it's easy to accidentally create a property that's invisible to `Object.keys()` if you forget to set `enumerable: true`.

```js
const obj = {};
Object.defineProperty(obj, 'id', {
  value: 1,
  writable: false,     // can't reassign obj.id
  enumerable: false,    // hidden from Object.keys()/for...in
  configurable: false,  // can't delete or redefine it later
});
obj.id = 2;       // silently fails (or throws in strict mode) — not writable
obj.id;           // 1 — the value never changed
Object.keys(obj); // [] — not enumerable, so it's invisible to normal iteration
```

[↑ Back to top](#table-of-contents)

---

### 14. How does prototypal inheritance differ from classical inheritance? 🔴

- **Classical inheritance** (used in languages like Java or C++): a "class" is a fixed blueprint defined ahead of time. Every instance is stamped out from that blueprint, and the hierarchy of classes is locked in when the code is compiled — you can't casually reshape it while the program runs.
- **Prototypal inheritance** (JavaScript): there's no real separate "class" concept at the engine level — objects inherit directly from **other live objects**, by following the prototype chain (Q6) at lookup time. Even the `class` keyword is just syntax sugar; underneath, it still sets up the same prototype links.
- Because inheritance is just "an object pointing to another object," you can change what an object inherits **at runtime**, by modifying its prototype directly — something classical languages don't allow in the same dynamic way.

```js
class Animal {
  speak() { console.log('...'); }
}
class Dog extends Animal {}
// Under the hood, `extends` just wires up the prototype chain:
Dog.prototype.__proto__ === Animal.prototype; // true

// And because it's just object references, you can change it at runtime:
const dog = new Dog();
Object.getPrototypeOf(Dog.prototype).speak = function () {
  console.log('Woof, dynamically added!');
};
dog.speak(); // "Woof, dynamically added!" — behavior changed without redefining the class
```

[↑ Back to top](#table-of-contents)

---
