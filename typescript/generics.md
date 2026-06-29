# Generics

_Part of [TypeScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are generics, and why do we need them?](#1-what-are-generics-and-why-do-we-need-them)
- [2. How do you define a generic function?](#2-how-do-you-define-a-generic-function)
- [3. What is a generic constraint?](#3-what-is-a-generic-constraint)

**🟡 Medium**
- [4. How do you use multiple generic type parameters?](#4-how-do-you-use-multiple-generic-type-parameters)
- [5. How do you write a generic interface or class?](#5-how-do-you-write-a-generic-interface-or-class)
- [6. What are default generic types?](#6-what-are-default-generic-types)
- [7. How does `keyof` work with generics?](#7-how-does-keyof-work-with-generics)
- [8. Why use generics instead of `any` for reusable code?](#8-why-use-generics-instead-of-any-for-reusable-code)

**🔴 Hard**
- [9. What are conditional types, and how do they combine with generics?](#9-what-are-conditional-types-and-how-do-they-combine-with-generics)
- [10. What is the `infer` keyword used for?](#10-what-is-the-infer-keyword-used-for)
- [11. How would you implement a type-safe `Pick<T, K>` from scratch?](#11-how-would-you-implement-a-type-safe-pickt-k-from-scratch)
- [12. What are variadic tuple types, and how do they interact with generics?](#12-what-are-variadic-tuple-types-and-how-do-they-interact-with-generics)

---

### 1. What are generics, and why do we need them? 🟢

- Generics let you write reusable functions/types/classes that work with **multiple types** while still preserving type information — instead of using `any` (which loses all type safety) for each input.

```ts
function identity<T>(value: T): T {
  return value;
}
identity('hello'); // T inferred as string, return type is string
identity(42);       // T inferred as number, return type is number
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you define a generic function? 🟢

- Add a type parameter in angle brackets (`<T>`) right after the function name, then use `T` like any other type within the function signature/body.

```ts
function firstElement<T>(arr: T[]): T {
  return arr[0];
}
firstElement([1, 2, 3]);     // number
firstElement(['a', 'b']);     // string
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a generic constraint? 🟢

- `extends` restricts what types are allowed for a generic parameter — guarantees the type has at least certain properties/methods, so you can safely use them inside the function.

```ts
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}
getLength('hello');   // OK — strings have .length
getLength([1, 2, 3]); // OK — arrays have .length
getLength(42);          // Error — number has no .length
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you use multiple generic type parameters? 🟡

- Just declare more than one inside the angle brackets, separated by commas.

```ts
function merge<T, U>(a: T, b: U): T & U {
  return { ...a, ...b };
}
const result = merge({ name: 'Vaibhav' }, { age: 25 });
// result is typed as { name: string } & { age: number }
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you write a generic interface or class? 🟡

```ts
interface Box<T> {
  value: T;
}
const stringBox: Box<string> = { value: 'hello' };

class Stack<T> {
  #items: T[] = [];
  push(item: T) { this.#items.push(item); }
  pop(): T | undefined { return this.#items.pop(); }
}
const numberStack = new Stack<number>();
```

[↑ Back to top](#table-of-contents)

---

### 6. What are default generic types? 🟡

- A generic parameter can have a fallback type used when the caller doesn't explicitly specify one.

```ts
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
}
const res: ApiResponse = { data: 'anything', status: 200 }; // T defaults to unknown
const typedRes: ApiResponse<string> = { data: 'hello', status: 200 };
```

[↑ Back to top](#table-of-contents)

---

### 7. How does `keyof` work with generics? 🟡

- `keyof T` produces a union of all the **property names** of `T` as string literal types — commonly combined with generics to write functions that safely access an object's properties by key.

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { name: 'Vaibhav', age: 25 };
getProperty(user, 'name'); // OK, returns string
getProperty(user, 'email'); // Error — 'email' is not a key of `user`
```

[↑ Back to top](#table-of-contents)

---

### 8. Why use generics instead of `any` for reusable code? 🟡

- `any` makes a function reusable across types, but at the cost of **all** type safety — both inputs and outputs become untyped, and TS won't catch mismatches.
- Generics preserve the **relationship** between input and output types, so the compiler still checks usage correctly, while still allowing the function to work with any type.

```ts
function identityAny(val: any): any { return val; }
const x = identityAny('hello');
x.toFixed(2); // No error from TS, but this is a runtime crash — `any` hid the mistake

function identityGeneric<T>(val: T): T { return val; }
const y = identityGeneric('hello');
y.toFixed(2); // Error: Property 'toFixed' does not exist on type 'string' — caught!
```

[↑ Back to top](#table-of-contents)

---

### 9. What are conditional types, and how do they combine with generics? 🔴

- A type-level `T extends U ? X : Y` — chooses between two types based on whether `T` is assignable to `U`. Almost always used together with generics to build flexible utility types.

```ts
type IsString<T> = T extends string ? 'yes' : 'no';
type A = IsString<'hello'>; // 'yes'
type B = IsString<42>;       // 'no'
```

[↑ Back to top](#table-of-contents)

---

### 10. What is the `infer` keyword used for? 🔴

- Used inside a conditional type to **extract** and capture a type from within a larger type, binding it to a new type variable you can reuse.

```ts
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() { return { name: 'Vaibhav' }; }
type User = ReturnTypeOf<typeof getUser>; // { name: string }
```

> [!TIP]
> **Real-life example:** TypeScript's built-in `ReturnType<T>` and `Parameters<T>` utility types are implemented using exactly this pattern.

[↑ Back to top](#table-of-contents)

---

### 11. How would you implement a type-safe `Pick<T, K>` from scratch? 🔴

- Use a **mapped type**: iterate over the keys in `K` (constrained to be keys of `T`), and for each, copy the corresponding property type from `T`.

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

interface User { id: number; name: string; email: string; }
type UserPreview = MyPick<User, 'id' | 'name'>; // { id: number; name: string }
```

[↑ Back to top](#table-of-contents)

---

### 12. What are variadic tuple types, and how do they interact with generics? 🔴

- Tuple types that can spread/capture an arbitrary number of elements using `...T[]`, similar to JS rest/spread but at the type level — lets generic functions accurately type variable-length argument lists.

```ts
function concat<T extends unknown[], U extends unknown[]>(arr1: [...T], arr2: [...U]): [...T, ...U] {
  return [...arr1, ...arr2];
}
const result = concat([1, 2], ['a', 'b']); // type: [number, number, string, string]
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How does this pattern help type a function like `bind()` or `curry()` accurately?
> - How do variadic tuples improve typings for something like Redux's `connect` or `compose`?

[↑ Back to top](#table-of-contents)

---
