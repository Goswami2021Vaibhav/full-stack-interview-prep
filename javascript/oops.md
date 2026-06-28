# OOP in JavaScript

_Part of [JavaScript](README.md) interview notes._

> For the general theory (encapsulation, polymorphism, SOLID, design patterns), see the language-agnostic [OOPs](../oops/README.md) topic. This file covers how JavaScript specifically implements OOP.

## Table of Contents

**🟢 Easy**
- [1. How do you create a class in JavaScript, and how does it relate to prototypes?](#1-how-do-you-create-a-class-in-javascript-and-how-does-it-relate-to-prototypes)
- [2. What is a constructor function?](#2-what-is-a-constructor-function)
- [3. What does the `new` keyword do internally?](#3-what-does-the-new-keyword-do-internally)
- [4. What is the `super` keyword used for?](#4-what-is-the-super-keyword-used-for)
- [5. What are static members in a class?](#5-what-are-static-members-in-a-class)

**🟡 Medium**
- [6. What are private class fields?](#6-what-are-private-class-fields)
- [7. What is method overriding in JavaScript?](#7-what-is-method-overriding-in-javascript)
- [8. How does JavaScript implement encapsulation, given it had no truly private fields before ES2022?](#8-how-does-javascript-implement-encapsulation-given-it-had-no-truly-private-fields-before-es2022)
- [9. What's the difference between composition and inheritance?](#9-whats-the-difference-between-composition-and-inheritance)
- [10. What is a mixin, and how would you implement one?](#10-what-is-a-mixin-and-how-would-you-implement-one)

**🔴 Hard**
- [11. How would you implement the Singleton pattern in JavaScript?](#11-how-would-you-implement-the-singleton-pattern-in-javascript)
- [12. How would you implement the Factory pattern in JavaScript?](#12-how-would-you-implement-the-factory-pattern-in-javascript)
- [13. How would you implement the Observer pattern in JavaScript?](#13-how-would-you-implement-the-observer-pattern-in-javascript)
- [14. How would you implement the Strategy pattern in JavaScript?](#14-how-would-you-implement-the-strategy-pattern-in-javascript)
- [15. How does ES6 `class` syntax differ from an ES5 constructor function under the hood?](#15-how-does-es6-class-syntax-differ-from-an-es5-constructor-function-under-the-hood)

---

### 1. How do you create a class in JavaScript, and how does it relate to prototypes? 🟢

- `class` is syntax sugar over JavaScript's existing prototype-based inheritance — there's no separate "class" mechanism at runtime.
- Methods defined in a class body are added to the constructor's `.prototype`, exactly like manually assigning methods to a constructor function's prototype.

```js
class Person {
  constructor(name) { this.name = name; }
  greet() { console.log(`Hi, ${this.name}`); }
}
typeof Person; // "function" — classes are functions under the hood
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a constructor function? 🟢

- A regular function intended to be called with `new`, used to create and initialize multiple similar objects (the pre-ES6 way of doing what `class` does today).

```js
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {
  console.log(`Hi, ${this.name}`);
};
const p = new Person('Vaibhav');
p.greet(); // "Hi, Vaibhav"
```

[↑ Back to top](#table-of-contents)

---

### 3. What does the `new` keyword do internally? 🟢

When you call `new Fn(...)`, JavaScript:
1. Creates a brand-new empty object.
2. Sets that object's `[[Prototype]]` to `Fn.prototype`.
3. Calls `Fn` with `this` bound to the new object.
4. Returns the new object — unless `Fn` explicitly returns its own object, in which case that's returned instead.

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you call a constructor function **without** `new`?
> - How would you write a polyfill that mimics what `new` does?

[↑ Back to top](#table-of-contents)

---

### 4. What is the `super` keyword used for? 🟢

- Inside a subclass constructor, `super(...)` calls the parent class's constructor (must be called before using `this`).
- Inside a method, `super.methodName()` calls the parent class's version of that method — useful when overriding but still wanting the original behavior.

```js
class Animal {
  speak() { console.log('Some sound'); }
}
class Dog extends Animal {
  speak() {
    super.speak();        // call parent's version too
    console.log('Woof');
  }
}
new Dog().speak(); // "Some sound" then "Woof"
```

[↑ Back to top](#table-of-contents)

---

### 5. What are static members in a class? 🟢

- Methods/properties defined on the **class itself**, not on instances — called via `ClassName.member`, not `instance.member`.
- Useful for utility functions or shared state related to the class as a whole (e.g. a counter of how many instances exist).

```js
class Counter {
  static instanceCount = 0;
  constructor() { Counter.instanceCount++; }
  static getCount() { return Counter.instanceCount; }
}
new Counter(); new Counter();
Counter.getCount(); // 2
```

[↑ Back to top](#table-of-contents)

---

### 6. What are private class fields? 🟡

- Fields/methods prefixed with `#`, truly inaccessible from outside the class (unlike the old `_field` naming convention, which was just a visual hint, not real privacy).
- Introduced in ES2022 — accessing a `#field` from outside the class throws a `SyntaxError`, not just `undefined`.

```js
class BankAccount {
  #balance = 0;
  deposit(amount) { this.#balance += amount; }
  getBalance() { return this.#balance; }
}
const acc = new BankAccount();
acc.deposit(100);
acc.getBalance(); // 100
acc.#balance;      // SyntaxError — truly inaccessible
```

[↑ Back to top](#table-of-contents)

---

### 7. What is method overriding in JavaScript? 🟡

- A subclass defines a method with the **same name** as one in its parent class — the subclass's version replaces (shadows) the parent's when called on a subclass instance.
- Use `super.methodName()` inside the override if you still want to run the parent's logic too (see Q4).

```js
class Shape {
  area() { return 0; }
}
class Circle extends Shape {
  constructor(r) { super(); this.r = r; }
  area() { return Math.PI * this.r ** 2; } // overridden
}
```

[↑ Back to top](#table-of-contents)

---

### 8. How does JavaScript implement encapsulation, given it had no truly private fields before ES2022? 🟡

- **Pre-ES2022 workarounds**: closures (module pattern, see [Closures & Scope](closures-and-scope.md)), or a `_underscore` naming convention (which is purely a *convention*, not enforced — still accessible).
- **Modern**: `#privateField` syntax gives real enforced privacy (Q6), and `WeakMap`-based private storage was a common trick before that.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between composition and inheritance? 🟡

- **Inheritance**: a class extends another, inheriting its shape and behavior ("is-a" relationship) — `Dog extends Animal`.
- **Composition**: an object is built by combining smaller, independent pieces of behavior ("has-a" relationship) — e.g. passing in dependencies or composing functions/objects together instead of subclassing.
- Composition is generally favored in modern JS (and React, where it's idiomatic) because deep inheritance chains get rigid and hard to change.

```js
// Composition
const canFly = (obj) => ({ ...obj, fly: () => console.log('flying') });
const canSwim = (obj) => ({ ...obj, swim: () => console.log('swimming') });
const duck = canSwim(canFly({ name: 'Duck' }));
duck.fly(); duck.swim();
```

[↑ Back to top](#table-of-contents)

---

### 10. What is a mixin, and how would you implement one? 🟡

- A way to "borrow" methods from one object/class into another without using full class inheritance — a form of composition.
- Common pattern: a function that takes a class and returns a new class extending it with extra methods.

```js
const Serializable = (Base) => class extends Base {
  serialize() { return JSON.stringify(this); }
};

class User { constructor(name) { this.name = name; } }
class SerializableUser extends Serializable(User) {}

new SerializableUser('Vaibhav').serialize(); // '{"name":"Vaibhav"}'
```

[↑ Back to top](#table-of-contents)

---

### 11. How would you implement the Singleton pattern in JavaScript? 🔴

- Ensure a class only ever has **one instance**, by caching it after the first creation and returning that cached instance on subsequent calls.

```js
class Database {
  static #instance;
  constructor() {
    if (Database.#instance) return Database.#instance;
    Database.#instance = this;
  }
}
const db1 = new Database();
const db2 = new Database();
db1 === db2; // true — same instance
```

> [!TIP]
> **Real-life example:** a single shared database connection pool or a global app configuration object that should never be duplicated.

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement the Factory pattern in JavaScript? 🔴

- A function (or method) that creates and returns objects without exposing the exact class/constructor logic to the caller — useful when object creation needs conditional logic.

```js
function createShape(type) {
  switch (type) {
    case 'circle': return { type, area: (r) => Math.PI * r ** 2 };
    case 'square': return { type, area: (s) => s * s };
    default: throw new Error('Unknown shape');
  }
}
const circle = createShape('circle');
circle.area(2);
```

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement the Observer pattern in JavaScript? 🔴

- A "subject" maintains a list of subscriber functions and notifies all of them whenever something happens — decouples the thing that changes from the things that react to it.

```js
class Subject {
  #observers = [];
  subscribe(fn) { this.#observers.push(fn); }
  unsubscribe(fn) { this.#observers = this.#observers.filter((o) => o !== fn); }
  notify(data) { this.#observers.forEach((fn) => fn(data)); }
}

const stockPrice = new Subject();
stockPrice.subscribe((price) => console.log(`Investor A sees: ${price}`));
stockPrice.notify(150); // "Investor A sees: 150"
```

> [!TIP]
> **Real-life example:** this is the same idea behind the [pub-sub event system](dom-and-events.md#14-how-would-you-implement-a-custom-pub-sub-event-system-without-using-the-dom) built earlier, Node's `EventEmitter`, and RxJS Observables — all are variations on the Observer pattern.

[↑ Back to top](#table-of-contents)

---

### 14. How would you implement the Strategy pattern in JavaScript? 🔴

- Define a family of interchangeable behaviors/algorithms, and select one at runtime instead of hardcoding a long `if`/`switch` chain.

```js
const paymentStrategies = {
  creditCard: (amount) => console.log(`Paying ${amount} via Credit Card`),
  paypal: (amount) => console.log(`Paying ${amount} via PayPal`),
  crypto: (amount) => console.log(`Paying ${amount} via Crypto`),
};

function checkout(method, amount) {
  paymentStrategies[method](amount);
}
checkout('paypal', 50); // "Paying 50 via PayPal"
```

> [!TIP]
> **Real-life example:** a checkout flow that swaps payment-processing logic based on the method the user picks, without an ever-growing `if`/`else` chain.

[↑ Back to top](#table-of-contents)

---

### 15. How does ES6 `class` syntax differ from an ES5 constructor function under the hood? 🔴

- Class methods are **non-enumerable** by default (won't show up in `for...in`/`Object.keys()`); manually-assigned prototype methods on a constructor function are enumerable by default.
- A class **must** be called with `new` — calling it as a plain function throws a `TypeError`. A constructor function can be called without `new` (though it'll misbehave, since `this` won't be the new instance).
- Class bodies execute in **strict mode** automatically, regardless of the surrounding code.
- Class declarations are **not hoisted** the way function declarations are — they stay in the Temporal Dead Zone, just like `let`/`const`.

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you try `class Foo {}` then reference `Foo` before its declaration line?
> - Why might non-enumerable class methods matter when serializing or iterating an instance?

[↑ Back to top](#table-of-contents)

---
