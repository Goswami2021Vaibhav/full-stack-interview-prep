# Advanced Types

_Part of [TypeScript](README.md) interview notes._

> Generic-specific advanced patterns (conditional types, `infer`, variadic tuples) are covered in [Generics](generics.md). This file focuses on union/intersection types, mapped types, and utility types.

## Table of Contents

**🟢 Easy**
- [1. What is a union type?](#1-what-is-a-union-type)
- [2. What is an intersection type?](#2-what-is-an-intersection-type)
- [3. What are literal types?](#3-what-are-literal-types)

**🟡 Medium**
- [4. What is a discriminated union, and why is it useful?](#4-what-is-a-discriminated-union-and-why-is-it-useful)
- [5. What are mapped types?](#5-what-are-mapped-types)
- [6. What are the common built-in utility types (`Partial`, `Pick`, `Omit`, etc.)?](#6-what-are-the-common-built-in-utility-types-partial-pick-omit-etc)
- [7. What's the difference between `as` type assertions and actual type conversion?](#7-whats-the-difference-between-as-type-assertions-and-actual-type-conversion)

**🔴 Hard**
- [8. What are template literal types?](#8-what-are-template-literal-types)
- [9. What's the difference between mapped types and conditional types?](#9-whats-the-difference-between-mapped-types-and-conditional-types)
- [10. How would you implement a custom `DeepPartial<T>` utility type?](#10-how-would-you-implement-a-custom-deeppartialt-utility-type)
- [11. What is the `satisfies` operator, and how is it different from a type annotation?](#11-what-is-the-satisfies-operator-and-how-is-it-different-from-a-type-annotation)

---

### 1. What is a union type? 🟢

- A value that can be **one of several types**, written with `|`. The compiler only lets you use members common to all the possibilities, unless you narrow it first.

```ts
let id: string | number;
id = 'abc123'; // OK
id = 42;        // OK
id = true;      // Error
```

[↑ Back to top](#table-of-contents)

---

### 2. What is an intersection type? 🟢

- Combines multiple types into one that must satisfy **all** of them at once, written with `&`.

```ts
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged; // must have both `name` and `age`

const p: Person = { name: 'Vaibhav', age: 25 };
```

[↑ Back to top](#table-of-contents)

---

### 3. What are literal types? 🟢

- Instead of the general type (`string`), you can narrow to one **specific value** as a type — useful for flags, modes, and discriminants.

```ts
let direction: 'up' | 'down' | 'left' | 'right';
direction = 'up';    // OK
direction = 'diag';  // Error — not one of the allowed literals
```

[↑ Back to top](#table-of-contents)

---

### 4. What is a discriminated union, and why is it useful? 🟡

- A union of object types that share a common **literal property** (the discriminant), letting TypeScript automatically narrow which variant you're dealing with after checking that property.
- Useful because it gives you exhaustive, type-safe handling of each variant (e.g. a `switch` over the discriminant) without manual type assertions.

```ts
type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };

function handle(result: Result) {
  if (result.status === 'success') {
    console.log(result.data); // narrowed to the success variant
  } else {
    console.log(result.message); // narrowed to the error variant
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What are mapped types? 🟡

- Build a new type by transforming each property of an existing type, using `[K in keyof T]` syntax — the foundation behind utility types like `Partial`/`Readonly`.

```ts
type Optional<T> = { [K in keyof T]?: T[K] };
type ReadonlyVersion<T> = { readonly [K in keyof T]: T[K] };

interface User { id: number; name: string; }
type PartialUser = Optional<User>; // { id?: number; name?: string }
```

[↑ Back to top](#table-of-contents)

---

### 6. What are the common built-in utility types (`Partial`, `Pick`, `Omit`, etc.)? 🟡

- `Partial<T>`: makes all properties optional.
- `Required<T>`: makes all properties required (opposite of `Partial`).
- `Readonly<T>`: makes all properties `readonly`.
- `Pick<T, K>`: keeps only the listed keys.
- `Omit<T, K>`: removes the listed keys, keeps the rest.
- `Record<K, V>`: builds an object type with keys `K` all mapped to value type `V`.

```ts
interface User { id: number; name: string; email: string; }

type UserUpdate = Partial<User>;          // all optional, for PATCH requests
type UserPreview = Pick<User, 'id' | 'name'>;
type PublicUser = Omit<User, 'email'>;
type UsersById = Record<number, User>;
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `as` type assertions and actual type conversion? 🟡

- `as` **does not convert** anything at runtime — it just tells the compiler "trust me, treat this value as this type," purely a compile-time instruction. If you're wrong, you get a runtime bug, not a compile error.
- Actual conversion (e.g. `Number(str)`, `String(num)`) genuinely transforms the **value** at runtime.

```ts
const input = document.getElementById('name') as HTMLInputElement;
// no runtime check happened — if the element isn't actually an input, this crashes later

const num = Number('42'); // genuinely converts the string '42' to the number 42
```

[↑ Back to top](#table-of-contents)

---

### 8. What are template literal types? 🔴

- Build new string literal types by combining other literal types and types, using template-literal-like syntax at the **type** level.

```ts
type Direction = 'top' | 'bottom';
type Side = 'left' | 'right';
type Corner = `${Direction}-${Side}`;
// 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right'

type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<'click'>; // 'onClick'
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between mapped types and conditional types? 🔴

- **Mapped types**: transform every property of an **existing object type** uniformly (e.g. make them all optional/readonly) — operate on a type's keys.
- **Conditional types**: choose between two types based on a `extends` check (`T extends U ? X : Y`) — operate on the relationship between types, not on iterating an object's keys.
- They're often combined: a mapped type can apply a conditional type to each property individually.

```ts
// Mapped + conditional together: only keep properties whose value is a function
type FunctionProperties<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K];
};
```

[↑ Back to top](#table-of-contents)

---

### 10. How would you implement a custom `DeepPartial<T>` utility type? 🔴

- A mapped type that recursively applies itself to any nested object property, instead of stopping at the first level like the built-in `Partial<T>`.

```ts
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

interface Config {
  server: { host: string; port: number };
  debug: boolean;
}
type PartialConfig = DeepPartial<Config>;
// { server?: { host?: string; port?: number }; debug?: boolean }
```

[↑ Back to top](#table-of-contents)

---

### 11. What is the `satisfies` operator, and how is it different from a type annotation? 🔴

- `satisfies` checks that an expression **matches** a type, while still letting TypeScript infer the **most specific** literal type for the value — unlike an annotation (`: T`), which widens the value to exactly `T` and loses literal specificity.

```ts
type Palette = Record<'red' | 'green', string | number>;

// With annotation — value is widened to Palette, loses specific literal info
const colors1: Palette = { red: '#ff0000', green: '#00ff00' };
colors1.red.toUpperCase(); // Error — TS thinks `red` could be string | number

// With satisfies — checked against Palette, but keeps the inferred literal types
const colors2 = { red: '#ff0000', green: '#00ff00' } satisfies Palette;
colors2.red.toUpperCase(); // OK — TS knows `red` is specifically a string here
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why would you choose `satisfies` over a plain type annotation in a config object?
> - What TypeScript version introduced `satisfies`?

[↑ Back to top](#table-of-contents)

---
