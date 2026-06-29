# Modules & Namespaces

_Part of [TypeScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you import and export in TypeScript?](#1-how-do-you-import-and-export-in-typescript)
- [2. What is a namespace, and how is it different from a module?](#2-what-is-a-namespace-and-how-is-it-different-from-a-module)

**🟡 Medium**
- [3. What's the difference between `import type` and a regular `import`?](#3-whats-the-difference-between-import-type-and-a-regular-import)
- [4. What are ambient declarations (`.d.ts` files), and when do you need them?](#4-what-are-ambient-declarations-dts-files-and-when-do-you-need-them)
- [5. How do you add types for an untyped JavaScript library?](#5-how-do-you-add-types-for-an-untyped-javascript-library)

**🔴 Hard**
- [6. What is module augmentation, and when would you use it?](#6-what-is-module-augmentation-and-when-would-you-use-it)
- [7. What's the difference between `esModuleInterop` and `allowSyntheticDefaultImports`?](#7-whats-the-difference-between-esmoduleinterop-and-allowsyntheticdefaultimports)
- [8. How does TypeScript resolve module paths (`moduleResolution`)?](#8-how-does-typescript-resolve-module-paths-moduleresolution)
- [9. How would you write a `.d.ts` file for a CommonJS module with both default and named exports?](#9-how-would-you-write-a-dts-file-for-a-commonjs-module-with-both-default-and-named-exports)

---

### 1. How do you import and export in TypeScript? 🟢

- Same ES module syntax as JavaScript — `import`/`export` — plus the ability to export types/interfaces alongside values.

```ts
// math.ts
export function add(a: number, b: number) { return a + b; }
export interface MathResult { value: number; }

// app.ts
import { add, MathResult } from './math';
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a namespace, and how is it different from a module? 🟢

- A **namespace** groups related code under a single global-ish name using `namespace Foo { ... }` — a TS-specific, pre-ES-modules way to organize code and avoid naming collisions.
- A **module** (ES module / `import`/`export`) is the modern standard, file-based approach. Namespaces are now largely legacy — prefer modules for new code; namespaces mainly still appear in older codebases or certain `.d.ts` declaration scenarios.

```ts
namespace Shapes {
  export class Circle {}
}
const c = new Shapes.Circle();
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between `import type` and a regular `import`? 🟡

- `import type` imports **only the type information**, guaranteed to be fully erased at compile time — useful for being explicit, and required in some setups (`isolatedModules`) where the compiler can't otherwise tell if an import is type-only or has runtime side effects.

```ts
import type { User } from './types'; // erased entirely, no runtime import
import { fetchUser } from './api';     // real runtime import
```

[↑ Back to top](#table-of-contents)

---

### 4. What are ambient declarations (`.d.ts` files), and when do you need them? 🟡

- Files that declare **types only**, with no actual implementation — used to describe the shape of existing JS code (a library, a global variable, a non-JS asset) to the TypeScript compiler.
- Needed when consuming a JS library that ships no types of its own, or when referencing globals/assets TypeScript doesn't natively understand (like importing a `.svg` file).

```ts
// global.d.ts
declare const APP_VERSION: string; // tells TS this global exists, defined elsewhere (e.g. via a build tool)

// svg.d.ts
declare module '*.svg' {
  const content: string;
  export default content;
}
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you add types for an untyped JavaScript library? 🟡

- Check `DefinitelyTyped` first — install community-maintained types via `npm install -D @types/<package-name>`.
- If none exist, write your own `.d.ts` file declaring the module's shape.

```ts
// types/some-untyped-lib.d.ts
declare module 'some-untyped-lib' {
  export function doSomething(value: string): number;
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What is module augmentation, and when would you use it? 🔴

- Lets you **add to** an existing module's types from outside that module — e.g. adding custom properties to a library's exported interface without modifying its source.

```ts
// express-augment.d.ts
import 'express';
declare module 'express' {
  interface Request {
    user?: { id: number; name: string }; // add a custom field used by your auth middleware
  }
}
```

> [!TIP]
> **Real-life example:** adding a `req.user` property after an auth middleware attaches it, so the rest of your Express route handlers get proper type-checking on `req.user` without `any`.

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between `esModuleInterop` and `allowSyntheticDefaultImports`? 🔴

- `allowSyntheticDefaultImports`: only affects **type-checking** — lets you write `import React from 'react'` even if the library doesn't have a real `default` export, without changing any emitted JS.
- `esModuleInterop`: changes the actual **emitted JavaScript** to correctly interoperate between CommonJS and ES module import semantics (handles the cases where Node's CommonJS default export doesn't line up with how ES modules expect a default import to work) — and implicitly enables `allowSyntheticDefaultImports` too.

[↑ Back to top](#table-of-contents)

---

### 8. How does TypeScript resolve module paths (`moduleResolution`)? 🔴

- Controlled by the `moduleResolution` compiler option, which mimics different runtime resolution algorithms:
  - `node` / `node10`: classic Node.js CommonJS-style resolution (walks `node_modules`, resolves `index.js`, etc.).
  - `bundler`: designed for modern bundlers (Vite, esbuild) — more lenient, matches how those tools actually resolve imports.
  - `nodenext` / `node16`: matches modern Node's actual ESM + CJS dual resolution rules, respecting `package.json`'s `exports` field and `.mts`/`.cts` extensions.
- Picking the wrong strategy is a common source of "works in dev, breaks at build" or vice versa import errors.

[↑ Back to top](#table-of-contents)

---

### 9. How would you write a `.d.ts` file for a CommonJS module with both default and named exports? 🔴

```ts
// for: module.exports = mainFn; module.exports.helper = helperFn;
declare module 'my-lib' {
  function mainFn(input: string): number;
  namespace mainFn {
    function helper(input: string): boolean;
  }
  export = mainFn;
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What's the difference between `export =` and `export default` in a declaration file?
> - Why does `export =` require `esModuleInterop` (or `import * as`) to consume correctly from an ES module file?

[↑ Back to top](#table-of-contents)

---
