# Core Concepts

_Part of [TypeScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is TypeScript, and how does it differ from JavaScript?](#1-what-is-typescript-and-how-does-it-differ-from-javascript)
- [2. What does it mean that TypeScript is a "superset" of JavaScript?](#2-what-does-it-mean-that-typescript-is-a-superset-of-javascript)
- [3. How do you compile TypeScript into JavaScript?](#3-how-do-you-compile-typescript-into-javascript)
- [4. What are the basic types available in TypeScript?](#4-what-are-the-basic-types-available-in-typescript)
- [5. What is type inference?](#5-what-is-type-inference)

**🟡 Medium**
- [6. What is structural typing, and how does TypeScript use it?](#6-what-is-structural-typing-and-how-does-typescript-use-it)
- [7. What's the difference between `any`, `unknown`, and `never`?](#7-whats-the-difference-between-any-unknown-and-never)
- [8. What is type narrowing?](#8-what-is-type-narrowing)
- [9. Does TypeScript provide any runtime type safety?](#9-does-typescript-provide-any-runtime-type-safety)
- [10. What does `strict` mode in `tsconfig.json` do, and why enable it?](#10-what-does-strict-mode-in-tsconfigjson-do-and-why-enable-it)

**🔴 Hard**
- [11. What does it mean that TypeScript's types are "erased" at compile time?](#11-what-does-it-mean-that-typescripts-types-are-erased-at-compile-time)
- [12. What's the difference between nominal and structural typing, and why did TypeScript choose structural?](#12-whats-the-difference-between-nominal-and-structural-typing-and-why-did-typescript-choose-structural)
- [13. What are excess property checks, and why do they only apply to object literals?](#13-what-are-excess-property-checks-and-why-do-they-only-apply-to-object-literals)

---

### 1. What is TypeScript, and how does it differ from JavaScript? 🟢

- A statically-typed language built on top of JavaScript — adds a type system and compile-time checks, then **compiles down to plain JavaScript** that runs anywhere JS runs.
- JS itself doesn't change at runtime; TypeScript's value is catching type-related bugs **before** the code ever runs.

[↑ Back to top](#table-of-contents)

---

### 2. What does it mean that TypeScript is a "superset" of JavaScript? 🟢

- Every valid JavaScript file is also valid TypeScript — TS only **adds** syntax (types, interfaces, generics) on top, it doesn't remove or change JS semantics.
- This makes migration incremental: you can rename a `.js` file to `.ts` and it'll mostly work as-is (errors will surface gradually as you enable stricter checks).

[↑ Back to top](#table-of-contents)

---

### 3. How do you compile TypeScript into JavaScript? 🟢

- The TypeScript compiler (`tsc`) reads `.ts` files and a `tsconfig.json` config, type-checks them, then emits plain `.js` files.
- In practice, most projects use a bundler (Vite, esbuild, Babel) to **strip types** quickly during development, and run `tsc` separately (often with `--noEmit`) just for type-checking.

```bash
tsc app.ts          # compiles app.ts -> app.js
tsc --noEmit         # type-check only, don't output JS
```

[↑ Back to top](#table-of-contents)

---

### 4. What are the basic types available in TypeScript? 🟢

- Primitives: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
- Plus TS-specific ones: `any`, `unknown`, `never`, `void`, and array/tuple/object shapes.

```ts
let name: string = 'Vaibhav';
let age: number = 25;
let isActive: boolean = true;
let scores: number[] = [1, 2, 3];
```

[↑ Back to top](#table-of-contents)

---

### 5. What is type inference? 🟢

- TypeScript can often figure out a variable's type automatically from its initial value, without an explicit annotation.

```ts
let name = 'Vaibhav'; // inferred as `string`, no annotation needed
name = 42;             // Error: Type 'number' is not assignable to type 'string'
```

[↑ Back to top](#table-of-contents)

---

### 6. What is structural typing, and how does TypeScript use it? 🟡

- TypeScript compares types by their **shape** (what properties/methods they have), not by name or explicit declaration — often called "duck typing": if it has the right shape, it's compatible.

```ts
interface Point { x: number; y: number; }

function logPoint(p: Point) { console.log(p.x, p.y); }

const obj = { x: 1, y: 2, z: 3 }; // not declared as `Point`...
logPoint(obj); // ...but works fine — shape matches (extra props are fine on existing variables)
```

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `any`, `unknown`, and `never`? 🟡

- `any`: opts **out** of type checking entirely — you can do anything with it, no safety at all.
- `unknown`: also accepts anything, but you **must narrow** it (check its type) before using it — the type-safe alternative to `any`.
- `never`: represents a value that **can never occur** — used for functions that always throw, or exhaustiveness checks.

```ts
let a: any = 'hello';
a.toUpperCase(); // allowed, even if `a` is actually a number — no safety net

let u: unknown = 'hello';
u.toUpperCase(); // Error: Object is of type 'unknown'
if (typeof u === 'string') u.toUpperCase(); // OK, narrowed

function fail(): never {
  throw new Error('Always throws');
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is type narrowing? 🟡

- The process of refining a broader type (like a union) to a more specific one within a code branch, based on runtime checks (`typeof`, `instanceof`, truthiness, discriminant properties).

```ts
function printId(id: string | number) {
  if (typeof id === 'string') {
    console.log(id.toUpperCase()); // TS knows `id` is `string` here
  } else {
    console.log(id.toFixed(2));    // TS knows `id` is `number` here
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 9. Does TypeScript provide any runtime type safety? 🟡

- **No** — types are purely a compile-time/development-time tool. They're completely erased when compiled to JS, so there's zero runtime overhead, but also zero runtime protection.
- Data from outside your TS code (API responses, `JSON.parse`, user input) isn't actually type-checked — it's only as safe as you (or a runtime validation library like Zod) make it.

```ts
interface User { name: string; }
const res = await fetch('/api/user');
const user: User = await res.json(); // TS trusts you — no actual runtime check happens
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you actually validate external data matches a type at runtime?
> - Why is `JSON.parse(json) as User` considered unsafe?

[↑ Back to top](#table-of-contents)

---

### 10. What does `strict` mode in `tsconfig.json` do, and why enable it? 🟡

- A single flag that turns on a whole bundle of stricter checks at once: `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, `strictPropertyInitialization`, and more.
- Without it, TypeScript is much more lenient (e.g. `null`/`undefined` are assignable to everything), which defeats much of the purpose of using TS in the first place. Strongly recommended for all real projects.

[↑ Back to top](#table-of-contents)

---

### 11. What does it mean that TypeScript's types are "erased" at compile time? 🔴

- All type annotations, interfaces, and generic parameters exist only for compile-time checking — none of it exists in the emitted JavaScript. You can't check a generic type parameter at runtime, because it simply isn't there anymore.

```ts
function isString<T>(val: T): boolean {
  return typeof val === 'string'; // fine — `typeof` checks the *value*, not `T` itself
}
// Compiled output has no trace of `<T>` at all
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can't you do `if (value instanceof T)` with a generic type parameter `T`?
> - How do libraries like `class-validator` work around type erasure to do runtime checks?

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between nominal and structural typing, and why did TypeScript choose structural? 🔴

- **Nominal typing** (Java, C#): two types are compatible only if they're explicitly declared as related (same class, or one extends/implements the other) — compatibility is about the type's **name/identity**.
- **Structural typing** (TypeScript): two types are compatible if they have the **same shape**, regardless of name or declared relationship.
- TS chose structural typing because it layers naturally on top of JavaScript's existing dynamic, duck-typed nature — JS objects were never tied to explicit type declarations, so a nominal system would have fought against how the language already works.

[↑ Back to top](#table-of-contents)

---

### 13. What are excess property checks, and why do they only apply to object literals? 🔴

- When you assign an **object literal directly** to a typed variable/parameter, TypeScript additionally checks that it has **no extra properties** beyond what the type declares — stricter than normal structural compatibility.
- This extra check only triggers for literals (not variables) because a literal has no other purpose — if it has a typo'd or unexpected property, that's almost certainly a mistake, since there's no other code relying on that literal's extra shape.

```ts
interface Config { width: number; }

function setup(config: Config) {}

setup({ width: 100, heigth: 200 }); // Error: 'heigth' does not exist — literal, excess check applies

const obj = { width: 100, heigth: 200 };
setup(obj); // OK — no excess check on a pre-existing variable (structural typing applies normally)
```

[↑ Back to top](#table-of-contents)

---
