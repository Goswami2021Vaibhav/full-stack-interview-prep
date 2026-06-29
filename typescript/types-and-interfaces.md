# Types & Interfaces

_Part of [TypeScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is an interface in TypeScript?](#1-what-is-an-interface-in-typescript)
- [2. What's the difference between `type` and `interface`?](#2-whats-the-difference-between-type-and-interface)
- [3. What are optional properties?](#3-what-are-optional-properties)
- [4. What is a `readonly` property?](#4-what-is-a-readonly-property)
- [5. What is the `enum` type, and when would you use it?](#5-what-is-the-enum-type-and-when-would-you-use-it)

**🟡 Medium**
- [6. How do you extend an interface?](#6-how-do-you-extend-an-interface)
- [7. What is declaration merging?](#7-what-is-declaration-merging)
- [8. What are index signatures?](#8-what-are-index-signatures)
- [9. How do you type a function using an interface vs a type alias?](#9-how-do-you-type-a-function-using-an-interface-vs-a-type-alias)
- [10. What's the difference between a `const enum` and a regular `enum`?](#10-whats-the-difference-between-a-const-enum-and-a-regular-enum)

**🔴 Hard**
- [11. How would you model a discriminated union using interfaces?](#11-how-would-you-model-a-discriminated-union-using-interfaces)
- [12. What's the difference between `interface` extension and `type` intersection (`&`)?](#12-whats-the-difference-between-interface-extension-and-type-intersection-)
- [13. How does TypeScript merge overloaded function signatures via interface merging?](#13-how-does-typescript-merge-overloaded-function-signatures-via-interface-merging)

---

### 1. What is an interface in TypeScript? 🟢

- A way to define the **shape** of an object — what properties and methods it must have, and their types. Purely a compile-time construct, erased at runtime.

```ts
interface User {
  id: number;
  name: string;
  isActive: boolean;
}
const user: User = { id: 1, name: 'Vaibhav', isActive: true };
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between `type` and `interface`? 🟢

- Both can describe object shapes, and for that purpose are mostly interchangeable. Key differences:
  - `interface` can be **re-opened and merged** (declaration merging); `type` cannot.
  - `type` can alias **anything** — unions, primitives, tuples — not just object shapes; `interface` only describes object-like shapes.
  - `interface extends` vs `type &` (intersection) behave similarly but report conflicting properties differently (see Q12).

```ts
interface Animal { name: string; }
type Pet = { name: string };       // type alias — equally valid for an object shape
type ID = string | number;          // only possible with `type`, not `interface`
```

[↑ Back to top](#table-of-contents)

---

### 3. What are optional properties? 🟢

- Marked with `?` — the property may be omitted entirely. Its type becomes `T | undefined`.

```ts
interface User {
  name: string;
  nickname?: string; // optional
}
const u: User = { name: 'Vaibhav' }; // valid — nickname omitted
```

[↑ Back to top](#table-of-contents)

---

### 4. What is a `readonly` property? 🟢

- Prevents reassignment **after the object is created** — a compile-time-only restriction (no runtime enforcement, unlike `Object.freeze()`).

```ts
interface Point { readonly x: number; readonly y: number; }
const p: Point = { x: 1, y: 2 };
p.x = 5; // Error: Cannot assign to 'x' because it is a read-only property
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the `enum` type, and when would you use it? 🟢

- A named set of related constant values — useful for representing a fixed set of options more readably than raw string/number literals.

```ts
enum Status {
  Active,   // 0
  Inactive, // 1
  Banned,   // 2
}
let s: Status = Status.Active;

enum Direction {
  Up = 'UP',
  Down = 'DOWN',
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why do many style guides prefer a union of string literals over `enum`?
> - What JavaScript code does a regular `enum` actually compile to?

[↑ Back to top](#table-of-contents)

---

### 6. How do you extend an interface? 🟡

- `extends` lets one interface inherit all members of another (or multiple), then add more.

```ts
interface Animal { name: string; }
interface Dog extends Animal {
  breed: string;
}
const d: Dog = { name: 'Rex', breed: 'Labrador' };
```

[↑ Back to top](#table-of-contents)

---

### 7. What is declaration merging? 🟡

- If you declare **two interfaces with the same name**, TypeScript merges their members into one combined interface, instead of one overwriting the other (which is what would happen with `type`).

```ts
interface Window {
  title: string;
}
interface Window {
  size: number;
}
// Merged: Window now has both `title` and `size`
const w: Window = { title: 'App', size: 100 };
```

> [!TIP]
> **Real-life example:** this is how you can add custom properties to global types like `Window` or `Express.Request` when writing type declarations for a library.

[↑ Back to top](#table-of-contents)

---

### 8. What are index signatures? 🟡

- Let you type an object whose exact property names aren't known ahead of time, but whose key/value **types** are known.

```ts
interface StringMap {
  [key: string]: string;
}
const colors: StringMap = { primary: 'blue', secondary: 'green' };
colors.tertiary = 'red'; // fine — any string key is allowed
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you type a function using an interface vs a type alias? 🟡

- Both work the same way for describing a callable shape — purely a style preference.

```ts
interface Add {
  (a: number, b: number): number;
}
type AddAlias = (a: number, b: number) => number;

const add: Add = (a, b) => a + b;
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between a `const enum` and a regular `enum`? 🟡

- `const enum` is fully **inlined at compile time** — no object is generated in the output JS, member references are replaced directly with their literal values. Slightly more performant, but can't be used with some tools (e.g. Babel's isolated-file compilation, since it needs cross-file type information).
- A regular `enum` compiles to an actual JS object that exists at runtime.

[↑ Back to top](#table-of-contents)

---

### 11. How would you model a discriminated union using interfaces? 🔴

- Give each variant interface a shared **literal property** (the "discriminant"), then TypeScript can narrow the union automatically based on checking that property.

```ts
interface Circle { kind: 'circle'; radius: number; }
interface Square { kind: 'square'; side: number; }
type Shape = Circle | Square;

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2; // TS knows it's Circle here
    case 'square': return shape.side ** 2;               // TS knows it's Square here
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between `interface` extension and `type` intersection (`&`)? 🔴

- `interface extends` **errors immediately** if the extending interface declares a conflicting property type with its parent.
- `type` intersection (`&`) **silently combines** conflicting types — if two intersected types have an incompatible property, that property's type collapses to `never` instead of raising an error at the point of intersection (the error often only surfaces later, when you actually try to use the property).

```ts
interface A { x: string; }
interface B extends A { x: number; } // Error immediately: incompatible with A.x

type C = { x: string };
type D = C & { x: number };           // No error here — x becomes type `never`
```

[↑ Back to top](#table-of-contents)

---

### 13. How does TypeScript merge overloaded function signatures via interface merging? 🔴

- When multiple interface declarations with the same name each declare a call signature, TypeScript merges them into a set of **overloads**, with **later declarations taking precedence** when resolving which overload matches a call.

```ts
interface Greeter {
  greet(name: string): string;
}
interface Greeter {
  greet(name: string, formal: boolean): string;
}
// Merged Greeter now has two overloaded `greet` signatures
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does declaration order matter when merging overloaded signatures?
> - How is this used in practice when augmenting third-party library types?

[↑ Back to top](#table-of-contents)

---
