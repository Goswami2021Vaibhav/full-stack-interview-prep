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

- You create a class with the `class` keyword, give it a `constructor` (runs once when you create an instance with `new`) and any number of methods.
- Under the hood, JS doesn't actually have a separate "class" mechanism — `class` is just nicer-looking **syntax sugar** on top of the prototype-based system that already existed (see [objects-and-prototypes.md](objects-and-prototypes.md#6-what-is-the-prototype-chain)). There's no real difference in what happens at runtime.
- Specifically: every method you write inside a class body gets attached to the class's `.prototype` object — the same object every instance falls back on when looking up a method. This is exactly what you'd get by manually attaching methods to a constructor function's `.prototype` (see Q2).

```js
class Person {
  constructor(name) { this.name = name; }
  greet() { console.log(`Hi, ${this.name}`); }
}
typeof Person;                          // "function" — classes are functions under the hood
typeof Person.prototype.greet;          // "function" — greet() lives on the prototype, not on each instance
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a constructor function? 🟢

- A constructor function is just a normal JavaScript function that's designed to be called with the `new` keyword, so it can create and set up multiple similar objects — this was the standard way to do "OOP" in JS before the `class` keyword existed (Q1 shows `class` is really just a nicer wrapper around this same pattern).
- By convention, constructor function names start with a capital letter (`Person`, not `person`) so it's obvious at a glance they're meant to be used with `new`.
- Any methods you want every instance to share should go on the function's `.prototype`, not be redefined inside the constructor itself — putting them on the prototype means every instance shares one copy of the method instead of getting its own separate copy (saving memory).

```js
function Person(name) {
  this.name = name; // instance-specific data
}
Person.prototype.greet = function () {
  console.log(`Hi, ${this.name}`); // shared by every instance via the prototype chain
};
const p = new Person('Vaibhav');
p.greet(); // "Hi, Vaibhav"
```

[↑ Back to top](#table-of-contents)

---

### 3. What does the `new` keyword do internally? 🟢

`new` looks like a single keyword, but it actually triggers a small sequence of steps behind the scenes. When you call `new Fn(...)`, JavaScript:
1. Creates a brand-new, empty object (`{}`).
2. Links that new object's prototype to `Fn.prototype`, so it can inherit any shared methods (see [objects-and-prototypes.md](objects-and-prototypes.md#6-what-is-the-prototype-chain)).
3. Runs `Fn`'s code with `this` pointing at that new object — so any `this.x = ...` inside `Fn` sets properties on the new object.
4. Returns the new object automatically — *unless* `Fn` explicitly returns its own object (e.g. `return { custom: true }`), in which case that returned object is used instead and the freshly created one is discarded.

```js
function Person(name) {
  this.name = name; // `this` is the new object new created for us
}
const p = new Person('Vaibhav');
p.name; // "Vaibhav" — set via `this` inside the constructor
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you call a constructor function **without** `new`?
> - How would you write a polyfill that mimics what `new` does?

[↑ Back to top](#table-of-contents)

---

### 4. What is the `super` keyword used for? 🟢

`super` is how a subclass (a class that `extends` another) reaches back to its parent class. It's used in two places, with two different meanings:

- Inside a subclass's `constructor`, calling `super(...)` runs the **parent class's constructor**. This is required before you can use `this` in the subclass constructor — JS won't let you touch `this` until the parent has had a chance to set itself up first.
- Inside a regular method, `super.methodName()` calls the **parent's version** of that method. This is handy when you're overriding a method (Q7) but still want to run the original logic as part of the new one, instead of fully replacing it.

```js
class Animal {
  speak() { console.log('Some sound'); }
}
class Dog extends Animal {
  speak() {
    super.speak();        // runs Animal's speak() first
    console.log('Woof');  // then adds Dog's own behavior
  }
}
new Dog().speak(); // logs "Some sound", then "Woof"
```

[↑ Back to top](#table-of-contents)

---

### 5. What are static members in a class? 🟢

- Normal methods/properties belong to each **instance** (or are shared via the prototype, but conceptually "belong" to instances). **Static** members are different — they live on the **class itself**, and you access them as `ClassName.member`, never `instance.member`.
- They're useful for things that relate to the class as a whole rather than to any one instance — utility/helper functions that don't need instance data, or shared state like a running count of how many instances have been created.

```js
class Counter {
  static instanceCount = 0;               // lives on Counter itself, not on instances
  constructor() { Counter.instanceCount++; }
  static getCount() { return Counter.instanceCount; }
}
new Counter(); new Counter();
Counter.getCount();       // 2 — tracked on the class, shared across all instances
new Counter().getCount;   // undefined — static methods aren't available on instances
```

[↑ Back to top](#table-of-contents)

---

### 6. What are private class fields? 🟡

- Private fields are properties or methods declared with a `#` prefix (e.g. `#balance`) that can genuinely **not** be accessed or even seen from outside the class — not just "hidden by convention," but enforced by the language itself.
- Before this existed, developers used an `_underscore` prefix (like `_balance`) as a visual signal meaning "please don't touch this from outside" — but it was just a naming convention; nothing actually stopped external code from reading or changing `_balance` directly.
- Private fields were introduced in ES2022. Trying to access `obj.#balance` from outside the class isn't just `undefined` — it's a hard `SyntaxError`, because the `#name` syntax is only even valid to write inside the class body that declared it.

```js
class BankAccount {
  #balance = 0; // only accessible inside this class
  deposit(amount) { this.#balance += amount; }
  getBalance() { return this.#balance; }
}
const acc = new BankAccount();
acc.deposit(100);
acc.getBalance(); // 100 — accessed through a public method
acc.#balance;      // SyntaxError — #balance isn't even valid syntax outside the class
```

[↑ Back to top](#table-of-contents)

---

### 7. What is method overriding in JavaScript? 🟡

- Method overriding happens when a subclass defines a method with the **exact same name** as one already defined in its parent class. When you call that method on an instance of the subclass, JS finds the subclass's version first (since the prototype chain is searched starting from the instance outward — see [objects-and-prototypes.md](objects-and-prototypes.md#6-what-is-the-prototype-chain)), so it "shadows" (hides) the parent's version.
- This lets a subclass customize or completely replace inherited behavior for its own instances, without touching the parent class.
- If you still want to run the parent's original logic as part of the override, rather than fully replacing it, call `super.methodName()` inside the override (see Q4).

```js
class Shape {
  area() { return 0; } // generic default
}
class Circle extends Shape {
  constructor(r) { super(); this.r = r; }
  area() { return Math.PI * this.r ** 2; } // overrides Shape's area() entirely
}
new Circle(2).area(); // 12.56... — uses Circle's version, not Shape's
```

[↑ Back to top](#table-of-contents)

---

### 8. How does JavaScript implement encapsulation, given it had no truly private fields before ES2022? 🟡

Encapsulation just means "hiding an object's internal details so outside code can only interact with it through a controlled, public interface." JS reached that goal in different ways over time:

- **Pre-ES2022 workaround #1 — closures / the module pattern**: a function creates local variables that only its inner functions can see (a "closure" — see [Closures & Scope](closures-and-scope.md)). Returning just the inner functions (and never the variables themselves) gives you real, unbreakable privacy, since there's no syntax at all for reaching a variable outside its closure.
- **Pre-ES2022 workaround #2 — `_underscore` naming convention**: prefixing a property with `_` (like `_balance`) to *signal* "treat this as private, please don't touch it directly." This is purely a social contract between developers — nothing in the language actually blocks access, so it's easy to accidentally (or deliberately) break.
- **Pre-ES2022 workaround #3 — `WeakMap`-based storage**: keep the "private" data in a `WeakMap` defined outside the class, keyed by `this`, so only code with access to that `WeakMap` variable (i.e., code inside the same module/closure) can read it.
- **Modern (ES2022+)**: the `#privateField` syntax (Q6) gives real, language-enforced privacy without needing any of the above tricks.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between composition and inheritance? 🟡

Both are strategies for reusing code and behavior across objects, but they connect things in fundamentally different ways:

- **Inheritance**: one class extends another and automatically gets all of its shape and behavior — describing an **"is-a"** relationship (`Dog extends Animal` means "a Dog *is an* Animal"). It creates a rigid, fixed hierarchy decided upfront.
- **Composition**: instead of inheriting from one fixed parent, you build behavior by combining several small, independent, reusable pieces — describing a **"has-a"** relationship (a duck *has* the ability to fly, and *has* the ability to swim, without needing an "AnimalThatFliesAndSwims" class). You can mix and match pieces freely.
- Modern JS (and especially React, where it's the idiomatic pattern) generally favors composition, because deep inheritance chains become rigid over time — changing a behavior high up in the hierarchy can unexpectedly affect every subclass beneath it, while composed pieces can be swapped independently.

```js
// Each function adds one independent capability, without a shared parent class
const canFly = (obj) => ({ ...obj, fly: () => console.log('flying') });
const canSwim = (obj) => ({ ...obj, swim: () => console.log('swimming') });

const duck = canSwim(canFly({ name: 'Duck' })); // composed from two independent behaviors
duck.fly();  // "flying"
duck.swim(); // "swimming"
```

[↑ Back to top](#table-of-contents)

---

### 10. What is a mixin, and how would you implement one? 🟡

- A mixin is a way to add extra, reusable methods to a class without making it formally inherit from another class — it's a practical form of composition (Q9), useful because JS classes can only `extend` one parent, so mixins are how you add "extra abilities" from multiple sources.
- The common pattern in JS: write a function that takes a class as input and returns a **new class** that extends it with additional methods bolted on. You then extend from the result of calling that function instead of the original class directly.

```js
const Serializable = (Base) => class extends Base {
  serialize() { return JSON.stringify(this); } // extra ability being "mixed in"
};

class User { constructor(name) { this.name = name; } }
class SerializableUser extends Serializable(User) {} // User + the Serializable mixin

new SerializableUser('Vaibhav').serialize(); // '{"name":"Vaibhav"}' — gained serialize() without a new parent hierarchy
```

[↑ Back to top](#table-of-contents)

---

### 11. How would you implement the Singleton pattern in JavaScript? 🔴

- The Singleton pattern guarantees a class can only ever produce **one single instance**, no matter how many times `new` is called on it — every call gets back the same object.
- The trick: store the first created instance in a place the class can check later (here, a private static field, see [oops.md Q6](#6-what-are-private-class-fields)). On every subsequent `new`, check if an instance already exists — if so, return that one instead of creating a fresh object (recall from Q3 that a constructor can override what `new` returns by explicitly returning an object).

```js
class Database {
  static #instance; // shared across all instances, remembers the one true instance

  constructor() {
    if (Database.#instance) return Database.#instance; // already exists — reuse it
    Database.#instance = this; // first time — remember this instance
  }
}
const db1 = new Database();
const db2 = new Database();
db1 === db2; // true — db2 is actually just a reference to db1
```

> [!TIP]
> **Real-life example:** a single shared database connection pool or a global app configuration object that should never be duplicated.

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement the Factory pattern in JavaScript? 🔴

- A Factory is simply a function whose job is to create and return objects, hiding the decision-making logic for *which kind* of object to build behind a single, simple call — the caller just says what they want (e.g. `'circle'`) without knowing or caring how it's constructed internally.
- This is especially useful when creating an object involves conditional logic — choosing between several possible "shapes" of object based on input — since it keeps that branching logic in one centralized place instead of scattered across the codebase.

```js
function createShape(type) {
  switch (type) {
    case 'circle': return { type, area: (r) => Math.PI * r ** 2 };
    case 'square': return { type, area: (s) => s * s };
    default: throw new Error('Unknown shape');
  }
}
const circle = createShape('circle'); // caller doesn't need to know the internal object shape
circle.area(2); // 12.56...
```

[↑ Back to top](#table-of-contents)

---

### 13. How would you implement the Observer pattern in JavaScript? 🔴

- The Observer pattern has one "subject" (the thing being watched) keep a list of interested "observer" functions. Whenever something noteworthy happens on the subject, it loops through that list and calls each one — the observers don't need to know about each other, and the subject doesn't need to know what each observer actually does with the data.
- This **decouples** the code that produces a change from the code that reacts to it — they only communicate through the subscribe/notify mechanism, not by calling each other directly.

```js
class Subject {
  #observers = [];
  subscribe(fn) { this.#observers.push(fn); }                                  // register interest
  unsubscribe(fn) { this.#observers = this.#observers.filter((o) => o !== fn); } // stop listening
  notify(data) { this.#observers.forEach((fn) => fn(data)); }                    // broadcast to everyone
}

const stockPrice = new Subject();
stockPrice.subscribe((price) => console.log(`Investor A sees: ${price}`));
stockPrice.notify(150); // "Investor A sees: 150" — every subscriber gets called with the new data
```

> [!TIP]
> **Real-life example:** this is the same idea behind the [pub-sub event system](dom-and-events.md#14-how-would-you-implement-a-custom-pub-sub-event-system-without-using-the-dom) built earlier, Node's `EventEmitter`, and RxJS Observables — all are variations on the Observer pattern.

[↑ Back to top](#table-of-contents)

---

### 14. How would you implement the Strategy pattern in JavaScript? 🔴

- The Strategy pattern defines a set of interchangeable behaviors (or "algorithms") for doing the same kind of task, and lets you pick which one to use at runtime — instead of hardcoding the choice with a long chain of `if`/`else` or `switch` statements.
- In JS, the easiest way to implement it is with a plain object that maps a "strategy name" to a function — picking a strategy then becomes a simple lookup (`strategies[name]`) rather than a branching chain, and adding a new strategy just means adding a new key, with no existing code to touch.

```js
const paymentStrategies = {
  creditCard: (amount) => console.log(`Paying ${amount} via Credit Card`),
  paypal: (amount) => console.log(`Paying ${amount} via PayPal`),
  crypto: (amount) => console.log(`Paying ${amount} via Crypto`),
};

function checkout(method, amount) {
  paymentStrategies[method](amount); // look up and run the chosen strategy — no if/switch needed
}
checkout('paypal', 50); // "Paying 50 via PayPal"
```

> [!TIP]
> **Real-life example:** a checkout flow that swaps payment-processing logic based on the method the user picks, without an ever-growing `if`/`else` chain.

[↑ Back to top](#table-of-contents)

---

### 15. How does ES6 `class` syntax differ from an ES5 constructor function under the hood? 🔴

Even though `class` compiles down to the same prototype-based mechanism as a constructor function (Q1), there are real behavioral differences the engine enforces:

- **Enumerability of methods**: methods defined inside a `class` body are **non-enumerable** by default — they won't show up when you loop with `for...in` or list with `Object.keys()`/`Object.entries()`. Methods manually assigned to a constructor function's `.prototype` (the old-school way) are enumerable by default, so they *would* show up in those loops. This matters if you're iterating over an object's properties and don't want to accidentally pick up its methods.
- **Must be called with `new`**: calling a class as a plain function (`Person()` instead of `new Person()`) throws a `TypeError` immediately. A constructor function has no such protection — calling it without `new` still "works" (doesn't error), but silently misbehaves, since `this` inside it won't be a new object; it'll fall back to the global object (or `undefined` in strict mode/modules).
- **Automatic strict mode**: all code inside a `class` body runs in strict mode automatically, regardless of whether the surrounding file uses `'use strict'`. A constructor function only runs in strict mode if the surrounding code opts into it.
- **No hoisting the same way**: function declarations are "hoisted" (usable before the line they're defined on). Classes are not — referencing a class before its declaration line throws a `ReferenceError` because it sits in the "Temporal Dead Zone," the same restriction `let`/`const` variables have.

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you try `class Foo {}` then reference `Foo` before its declaration line?
> - Why might non-enumerable class methods matter when serializing or iterating an instance?

[↑ Back to top](#table-of-contents)

---
