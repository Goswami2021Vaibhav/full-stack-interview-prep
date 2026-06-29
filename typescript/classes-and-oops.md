# Classes & OOP

_Part of [TypeScript](README.md) interview notes._

> For general JS class mechanics (prototypes, `super`, static members), see [JavaScript › OOP in JavaScript](../javascript/oops.md). This file covers what TypeScript adds on top: access modifiers, abstract classes, and compile-time enforcement.

## Table of Contents

**🟢 Easy**
- [1. How do you type class properties in TypeScript?](#1-how-do-you-type-class-properties-in-typescript)
- [2. What are access modifiers (`public`, `private`, `protected`)?](#2-what-are-access-modifiers-public-private-protected)
- [3. What is parameter property shorthand in constructors?](#3-what-is-parameter-property-shorthand-in-constructors)

**🟡 Medium**
- [4. What's the difference between TypeScript's `private` keyword and JavaScript's real `#field`?](#4-whats-the-difference-between-typescripts-private-keyword-and-javascripts-real-field)
- [5. What are abstract classes, and when would you use one?](#5-what-are-abstract-classes-and-when-would-you-use-one)
- [6. How do you implement an interface in a class?](#6-how-do-you-implement-an-interface-in-a-class)
- [7. What does the `override` keyword do?](#7-what-does-the-override-keyword-do)

**🔴 Hard**
- [8. What's the difference between `implements` and `extends`?](#8-whats-the-difference-between-implements-and-extends)
- [9. How do generics work with classes?](#9-how-do-generics-work-with-classes)
- [10. Why can't static members use a class's generic type parameter?](#10-why-cant-static-members-use-a-classs-generic-type-parameter)
- [11. What does `strictPropertyInitialization` enforce?](#11-what-does-strictpropertyinitialization-enforce)

---

### 1. How do you type class properties in TypeScript? 🟢

- Declare properties with their type, optionally with a default value — TypeScript checks that the constructor (or default) actually initializes them.

```ts
class User {
  name: string;
  age: number = 0; // default value
  constructor(name: string) {
    this.name = name;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What are access modifiers (`public`, `private`, `protected`)? 🟢

- `public` (default): accessible from anywhere.
- `private`: accessible only **within the declaring class** itself.
- `protected`: accessible within the declaring class **and its subclasses**, but not from outside.
- All compile-time only — see Q4 for why this isn't real runtime privacy.

```ts
class Account {
  public id: number;
  private balance: number = 0;
  protected owner: string;

  constructor(id: number, owner: string) {
    this.id = id;
    this.owner = owner;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What is parameter property shorthand in constructors? 🟢

- Adding an access modifier directly to a constructor parameter automatically declares **and** assigns it as a class property — removes the need to separately declare the field and write `this.x = x`.

```ts
class Point {
  constructor(public x: number, public y: number) {}
}
// equivalent to manually declaring `x`/`y` and assigning them in the constructor body
const p = new Point(1, 2);
p.x; // 1
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between TypeScript's `private` keyword and JavaScript's real `#field`? 🟡

- TypeScript's `private` is **compile-time only** — it's erased when compiled to JS, so the property is still fully accessible at runtime if someone bypasses the type checker (e.g. via `(obj as any).secret`).
- JavaScript's native `#field` is **enforced at runtime** by the engine itself — truly inaccessible from outside the class, no matter what.

```ts
class A {
  private secret = 'ts-private';
  #realSecret = 'js-private';
}
const a = new A();
(a as any).secret;     // 'ts-private' — accessible at runtime, TS just hides the type error
(a as any).realSecret; // undefined — #realSecret isn't even named that at runtime
```

[↑ Back to top](#table-of-contents)

---

### 5. What are abstract classes, and when would you use one? 🟡

- A class that **cannot be instantiated directly** — meant to be extended. Can declare `abstract` methods that subclasses **must** implement, while still providing shared concrete logic for the rest.

```ts
abstract class Shape {
  abstract area(): number;      // must be implemented by subclasses
  describe(): string {           // shared, concrete implementation
    return `Area: ${this.area()}`;
  }
}
class Circle extends Shape {
  constructor(private radius: number) { super(); }
  area() { return Math.PI * this.radius ** 2; }
}
new Shape();  // Error — cannot instantiate an abstract class
new Circle(2).describe(); // OK
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you implement an interface in a class? 🟡

- `implements` makes a class commit to providing all the members an interface declares — TypeScript checks the class actually fulfills that contract.

```ts
interface Drivable {
  speed: number;
  drive(): void;
}
class Car implements Drivable {
  speed = 0;
  drive() { this.speed += 10; }
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What does the `override` keyword do? 🟡

- An explicit marker that a method is **intentionally overriding** a parent class method — TypeScript errors if the parent doesn't actually have a method by that name (catches typos or the parent method being renamed/removed later).

```ts
class Animal {
  speak() { console.log('...'); }
}
class Dog extends Animal {
  override speak() { console.log('Woof'); } // explicit, safer
  // override greet() {} // Error: no `greet` method exists on Animal
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `implements` and `extends`? 🔴

- `extends`: actual inheritance — the subclass gets the parent's **implementation** (real methods/properties) and can override them.
- `implements`: a **type-only contract** — the class must provide matching members, but gets no implementation for free; it's purely a compile-time check that the shape is satisfied.
- A class can `extend` only **one** class, but `implements` **multiple** interfaces.

```ts
class Base { greet() { console.log('hi'); } }
class Sub extends Base {} // inherits greet() implementation, works immediately

interface Flyable { fly(): void; }
class Bird implements Flyable {
  fly() { console.log('flying'); } // must write the implementation yourself
}
```

[↑ Back to top](#table-of-contents)

---

### 9. How do generics work with classes? 🔴

- A class can declare its own type parameter(s), used to type its properties/methods — the concrete type is fixed once an instance is created.

```ts
class Box<T> {
  constructor(private value: T) {}
  getValue(): T { return this.value; }
}
const stringBox = new Box<string>('hello');
const numberBox = new Box(42); // T inferred as number
```

[↑ Back to top](#table-of-contents)

---

### 10. Why can't static members use a class's generic type parameter? 🔴

- A class's generic type parameter is bound **per instance** (fixed when you call `new ClassName<T>()`) — but `static` members belong to the class itself, shared across **all** instances, before any specific `T` has even been chosen. There's no single `T` a static member could consistently refer to.

```ts
class Container<T> {
  static defaultValue: T; // Error: Static members cannot reference class type parameters
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you work around needing a generic-like static factory method?
> - Could a static **method** (not property) declare its own independent generic parameter instead?

[↑ Back to top](#table-of-contents)

---

### 11. What does `strictPropertyInitialization` enforce? 🔴

- Part of `strict` mode — requires every class property to be definitely assigned a value either via a default, or within the constructor, before the constructor finishes. Catches the bug of declaring a property but forgetting to actually set it.

```ts
class User {
  name: string; // Error: Property 'name' has no initializer and is not assigned in the constructor
  constructor() {}
}

class FixedUser {
  name: string;
  constructor(name: string) { this.name = name; } // OK — assigned in constructor
}
```

[↑ Back to top](#table-of-contents)

---
