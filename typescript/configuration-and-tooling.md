# Configuration & Tooling

_Part of [TypeScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is `tsconfig.json`, and what does it control?](#1-what-is-tsconfigjson-and-what-does-it-control)
- [2. What does the `target` compiler option do?](#2-what-does-the-target-compiler-option-do)
- [3. What does the `strict` flag enable?](#3-what-does-the-strict-flag-enable)

**🟡 Medium**
- [4. What's the difference between `noImplicitAny` and `strict`?](#4-whats-the-difference-between-noimplicitany-and-strict)
- [5. What does `esModuleInterop` do?](#5-what-does-esmoduleinterop-do)
- [6. How do you run TypeScript directly without a separate compile step?](#6-how-do-you-run-typescript-directly-without-a-separate-compile-step)
- [7. Why does `isolatedModules` matter when using a transpiler like Babel or esbuild?](#7-why-does-isolatedmodules-matter-when-using-a-transpiler-like-babel-or-esbuild)

**🔴 Hard**
- [8. What does `skipLibCheck` do, and what are the trade-offs of enabling it?](#8-what-does-skiplibcheck-do-and-what-are-the-trade-offs-of-enabling-it)
- [9. How do project references / composite projects help in large monorepos?](#9-how-do-project-references--composite-projects-help-in-large-monorepos)
- [10. What's the difference between `tsc --noEmit` and using TypeScript purely as a type-stripping transpiler?](#10-whats-the-difference-between-tsc---noemit-and-using-typescript-purely-as-a-type-stripping-transpiler)

---

### 1. What is `tsconfig.json`, and what does it control? 🟢

- The config file that tells the TypeScript compiler which files to include, which JS features/APIs to assume exist, how strict to be, and what kind of output to emit.

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What does the `target` compiler option do? 🟢

- Sets which JavaScript version the output is compiled **down to** — controls whether newer syntax (like optional chaining, classes, async/await) gets transformed into older equivalents, or left as-is for environments that already support it.

```json
{ "compilerOptions": { "target": "ES5" } } // transforms arrow functions, classes, etc. into ES5-compatible code
```

[↑ Back to top](#table-of-contents)

---

### 3. What does the `strict` flag enable? 🟢

- A shorthand that turns on a whole group of stricter type-checking flags at once: `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, `strictPropertyInitialization`, `noImplicitThis`, and more.
- Recommended for all real projects — without it, TypeScript is far more permissive and misses a lot of bugs it's otherwise capable of catching.

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between `noImplicitAny` and `strict`? 🟡

- `noImplicitAny`: just one specific flag — errors when TypeScript would otherwise silently infer `any` for something with no explicit type and no way to infer one.
- `strict`: the umbrella flag that **includes** `noImplicitAny` plus many other unrelated stricter checks (null safety, `this` typing, property initialization, etc.).

```ts
function greet(name) { // Error with noImplicitAny: 'name' implicitly has an 'any' type
  console.log(name);
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What does `esModuleInterop` do? 🟡

- Fixes interoperability issues between CommonJS modules and the ES module `import`/`export` syntax — without it, importing a CommonJS-only package's default export often requires the awkward `import * as React from 'react'` form instead of the cleaner `import React from 'react'`.
- Almost always should be enabled in modern projects; see also [Modules & Namespaces](modules-and-namespaces.md) for the deeper distinction from `allowSyntheticDefaultImports`.

[↑ Back to top](#table-of-contents)

---

### 6. How do you run TypeScript directly without a separate compile step? 🟡

- Tools like `ts-node` (Node.js, compiles on the fly), `tsx` (faster, esbuild-based), or `bun`/`deno` (native TS support) let you execute `.ts` files directly during development, without manually running `tsc` first.

```bash
npx ts-node src/index.ts
npx tsx src/index.ts
```

[↑ Back to top](#table-of-contents)

---

### 7. Why does `isolatedModules` matter when using a transpiler like Babel or esbuild? 🟡

- Tools like Babel/esbuild strip types **file by file**, with no cross-file type information — they can't see whether something imported is a type or a value, so certain TS features that require whole-program knowledge (like `const enum` inlining, or non-explicit re-exports of types) can't be transpiled correctly.
- `isolatedModules: true` makes `tsc` itself flag any code that wouldn't survive this kind of per-file transpilation, so you catch the incompatibility early instead of getting a confusing runtime bug from the actual build tool.

[↑ Back to top](#table-of-contents)

---

### 8. What does `skipLibCheck` do, and what are the trade-offs of enabling it? 🔴

- Skips type-checking of all `.d.ts` declaration files (including ones inside `node_modules`) — only your own source files are checked.
- **Pro**: significantly faster builds, and avoids errors caused by conflicting/incompatible type declarations between different libraries' dependencies (a common, hard-to-fix problem in large projects).
- **Con**: you lose the safety net of catching genuine type errors in declaration files — though in practice, this is rarely where real bugs live, so most large projects enable it.

[↑ Back to top](#table-of-contents)

---

### 9. How do project references / composite projects help in large monorepos? 🔴

- `references` in `tsconfig.json` let you split a large codebase into smaller TypeScript "projects" that depend on each other, each compiled (and incrementally cached) independently — instead of one giant compile of everything every time.
- Combined with `"composite": true`, `tsc --build` can detect which referenced projects actually changed and only rebuild those, dramatically speeding up builds in monorepos with many packages.

```json
// packages/app/tsconfig.json
{ "references": [{ "path": "../shared-utils" }] }
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between `tsc --noEmit` and using TypeScript purely as a type-stripping transpiler? 🔴

- `tsc --noEmit`: runs the **full type checker**, reports all type errors, but produces no JS output at all — used purely as a CI/editor type-checking step.
- A type-stripping transpiler (Babel's TS preset, esbuild, swc): does **zero type checking** — it just removes type annotations syntactically and emits JS as fast as possible, trusting that type errors were already caught elsewhere (like a separate `tsc --noEmit` step or your editor).
- This split (fast transpile for builds, separate `tsc --noEmit` for correctness) is the standard modern setup, since type-checking is the slow part of compilation and doesn't need to block every dev rebuild.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is type-checking so much slower than transpilation alone?
> - What CI step would you add to make sure type errors still get caught despite using a fast type-stripping build pipeline?

[↑ Back to top](#table-of-contents)

---
