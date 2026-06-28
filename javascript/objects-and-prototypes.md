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

- **Object literal**: `{ key: 'value' }` — most common.
- **Constructor function**: `new Person()`.
- **`Object.create(proto)`**: creates an object with a specific prototype.
- **ES6 class**: `new MyClass()`.
- **`Object.assign({}, ...)`** or spread `{ ...obj }`: creates a new object from existing ones.

```js
const a = { x: 1 };               // literal
const b = new Object();           // constructor
const c = Object.create(null);    // no prototype at all
class Point { constructor(x) { this.x = x; } }
const d = new Point(5);           // class instance
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between dot notation and bracket notation? 🟢

- **Dot notation** (`obj.key`): requires a valid identifier name known at write-time.
- **Bracket notation** (`obj['key']`): takes a string/expression, so it works with dynamic keys, keys with spaces, or keys that are reserved words.

```js
const obj = { 'first-name': 'Vaibhav' };
obj.first-name;      // syntax error / wrong (interpreted as subtraction)
obj['first-name'];   // 'Vaibhav' — works

const key = 'first-name';
obj[key]; // dynamic access — only possible with brackets
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between `Object.freeze()` and `Object.seal()`? 🟢

- `Object.freeze()`: no adding, removing, or **modifying** existing properties — fully locked (shallow only).
- `Object.seal()`: no adding or removing properties, but **existing** property values can still be changed.

```js
const frozen = Object.freeze({ a: 1 });
frozen.a = 2; // silently fails (throws in strict mode)

const sealed = Object.seal({ a: 1 });
sealed.a = 2; // works — value can change
sealed.b = 3; // fails — can't add new properties
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Is `Object.freeze()` deep or shallow? How would you make a deep freeze?

[↑ Back to top](#table-of-contents)

---

### 4. How do you check if an object has a property? 🟢

- `'key' in obj` — checks the object **and its prototype chain**.
- `obj.hasOwnProperty('key')` — checks only the object's **own** properties, ignoring inherited ones.
- `obj.key !== undefined` — works but gives a false negative if the property exists and is explicitly set to `undefined`.

```js
const obj = { a: 1 };
'toString' in obj;             // true — inherited from Object.prototype
obj.hasOwnProperty('toString'); // false — not its own property
obj.hasOwnProperty('a');        // true
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `Object.keys()`, `Object.values()`, and `Object.entries()`? 🟢

- `Object.keys(obj)` → array of the object's own enumerable property **names**.
- `Object.values(obj)` → array of the corresponding **values**.
- `Object.entries(obj)` → array of `[key, value]` pairs — handy for `for...of` or converting to a `Map`.

```js
const user = { name: 'Vaibhav', age: 25 };
Object.keys(user);   // ['name', 'age']
Object.values(user); // ['Vaibhav', 25]
Object.entries(user); // [['name', 'Vaibhav'], ['age', 25]]
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the prototype chain? 🟡

- Every object has an internal link (`[[Prototype]]`) to another object. When you access a property, JS looks on the object itself first, then walks up this chain until it finds the property or hits `null`.
- This is how built-in methods like `.toString()` or array `.map()` are available without being defined on every individual object/array.

```js
const arr = [1, 2, 3];
arr.map; // found on Array.prototype, not on `arr` itself
arr.__proto__ === Array.prototype; // true
Array.prototype.__proto__ === Object.prototype; // true
Object.prototype.__proto__; // null — end of the chain
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `__proto__` and `prototype`? 🟡

- `prototype`: a property that exists **on constructor functions/classes** — defines what gets attached to the `[[Prototype]]` of instances created with `new`.
- `__proto__`: the actual (legacy, informal) accessor on an **instance** pointing to its `[[Prototype]]` — i.e., a reference to its constructor's `prototype` object.
- Prefer `Object.getPrototypeOf(obj)` over `__proto__` directly (which is deprecated-ish/non-standard in spirit, kept only for compatibility).

```js
function Person() {}
const p = new Person();
p.__proto__ === Person.prototype; // true
```

[↑ Back to top](#table-of-contents)

---

### 8. What is `Object.create()`, and how is it used for inheritance? 🟡

- Creates a new object with its `[[Prototype]]` explicitly set to the object passed in — the most direct way to set up prototypal inheritance without a constructor.

```js
const animal = {
  speak() { console.log(`${this.name} makes a sound.`); },
};
const dog = Object.create(animal);
dog.name = 'Rex';
dog.speak(); // "Rex makes a sound." — inherited via the prototype chain
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between a shallow copy and a deep copy? 🟡

- **Shallow copy**: copies only the top-level properties — nested objects/arrays are still shared by reference with the original.
- **Deep copy**: recursively copies every nested level, so the result is fully independent of the original.

```js
const original = { a: 1, nested: { b: 2 } };
const shallow = { ...original };
shallow.nested.b = 99;
console.log(original.nested.b); // 99 — nested object was shared!

const deep = structuredClone(original); // true deep copy (modern browsers/Node)
deep.nested.b = 100;
console.log(original.nested.b); // still 99 — unaffected
```

[↑ Back to top](#table-of-contents)

---

### 10. What are getters and setters on an object? 🟡

- Special methods (`get`/`set`) that run when a property is **read** or **written**, letting you add logic (validation, computed values) behind what looks like a plain property.

```js
const user = {
  firstName: 'Vaibhav',
  lastName: 'Goswami',
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(' ');
  },
};
user.fullName;            // "Vaibhav Goswami" (getter)
user.fullName = 'A B';    // (setter) updates firstName/lastName
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between `Object.assign()` and the spread operator for copying objects? 🟡

- Both perform a **shallow** merge/copy — functionally very similar for plain objects.
- `Object.assign(target, ...sources)` mutates and returns the `target` object.
- Spread (`{ ...obj }`) always creates a brand-new object — doesn't mutate anything, and is generally preferred for readability.

```js
const a = { x: 1 };
const merged1 = Object.assign({}, a, { y: 2 }); // { x: 1, y: 2 }
const merged2 = { ...a, y: 2 };                  // { x: 1, y: 2 } — same result
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement your own deep clone function? 🔴

- Recursively copy each property; if a value is an object/array, recurse into it instead of copying the reference.
- Need to handle edge cases: arrays, `null`, `Date`, circular references (a real-world implementation would track already-visited objects with a `WeakMap`).

```js
function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== 'object') return value;
  if (seen.has(value)) return seen.get(value); // handle circular refs

  const clone = Array.isArray(value) ? [] : {};
  seen.set(value, clone);

  for (const key in value) {
    clone[key] = deepClone(value[key], seen);
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

- Metadata that controls how a property behaves, accessible via `Object.getOwnPropertyDescriptor()` / set via `Object.defineProperty()`.
- Four flags: `value`, `writable` (can it be reassigned?), `enumerable` (does it show up in `for...in`/`Object.keys()`?), `configurable` (can it be deleted/redefined?).
- Properties created with normal assignment default all flags to `true`; `Object.defineProperty()` defaults them to `false` unless specified.

```js
const obj = {};
Object.defineProperty(obj, 'id', {
  value: 1,
  writable: false,
  enumerable: false,
  configurable: false,
});
obj.id = 2;            // silently fails — not writable
Object.keys(obj);       // [] — not enumerable, hidden from normal iteration
```

[↑ Back to top](#table-of-contents)

---

### 14. How does prototypal inheritance differ from classical inheritance? 🔴

- **Classical** (Java, C++): classes are blueprints; instances are created from a fixed class hierarchy defined at compile time.
- **Prototypal** (JavaScript): objects inherit directly from **other objects** via the prototype chain — there's no real "class," even `class` syntax is sugar over prototype-based delegation under the hood.
- This means you can change an object's behavior at runtime by modifying its prototype — not possible the same way in purely classical languages.

```js
class Animal {
  speak() { console.log('...'); }
}
class Dog extends Animal {}
// Under the hood: Dog.prototype.__proto__ === Animal.prototype
```

[↑ Back to top](#table-of-contents)

---
