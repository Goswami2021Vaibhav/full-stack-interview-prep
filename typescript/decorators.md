# Decorators

_Part of [TypeScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are decorators in TypeScript?](#1-what-are-decorators-in-typescript)
- [2. How do you apply a class decorator?](#2-how-do-you-apply-a-class-decorator)

**🟡 Medium**
- [3. What are the different types of decorators?](#3-what-are-the-different-types-of-decorators)
- [4. What arguments does a method decorator receive?](#4-what-arguments-does-a-method-decorator-receive)
- [5. What is a decorator factory?](#5-what-is-a-decorator-factory)
- [6. What compiler flag is needed to use decorators?](#6-what-compiler-flag-is-needed-to-use-decorators)

**🔴 Hard**
- [7. How would you implement a simple logging decorator for a class method?](#7-how-would-you-implement-a-simple-logging-decorator-for-a-class-method)
- [8. How do decorators relate to metadata reflection (e.g. in NestJS/Angular dependency injection)?](#8-how-do-decorators-relate-to-metadata-reflection-eg-in-nestjsangular-dependency-injection)
- [9. What is the execution order when multiple decorators are applied to the same target?](#9-what-is-the-execution-order-when-multiple-decorators-are-applied-to-the-same-target)
- [10. What changed between TypeScript's experimental decorators and the newer Stage 3 ECMAScript decorators?](#10-what-changed-between-typescripts-experimental-decorators-and-the-newer-stage-3-ecmascript-decorators)

---

### 1. What are decorators in TypeScript? 🟢

- Functions that can **annotate and modify** classes, methods, properties, or parameters at definition time — a way to add reusable behavior (logging, validation, dependency injection) declaratively, without changing the target's own code.

```ts
function Logged(target: Function) {
  console.log(`Class defined: ${target.name}`);
}

@Logged
class MyService {}
// logs "Class defined: MyService" when the class is declared
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you apply a class decorator? 🟢

- Prefix the class with `@decoratorName` right above its declaration — the decorator function receives the class's constructor.

```ts
function Frozen(constructor: Function) {
  Object.freeze(constructor);
  Object.freeze(constructor.prototype);
}

@Frozen
class Config {
  static apiUrl = '/api';
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What are the different types of decorators? 🟡

- **Class decorators**: applied to the class itself.
- **Method decorators**: applied to a class method.
- **Property decorators**: applied to a class field.
- **Accessor decorators**: applied to a getter/setter.
- **Parameter decorators**: applied to a single constructor/method parameter.

[↑ Back to top](#table-of-contents)

---

### 4. What arguments does a method decorator receive? 🟡

- `target`: the class prototype (for instance methods) or constructor (for static methods).
- `propertyKey`: the method's name.
- `descriptor`: the method's property descriptor — lets you read/replace the actual function.

```ts
function LogCall(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return original.apply(this, args);
  };
}

class Calculator {
  @LogCall
  add(a: number, b: number) { return a + b; }
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What is a decorator factory? 🟡

- A function that **returns** a decorator — lets you pass configuration arguments to the decorator itself.

```ts
function MinLength(length: number) {
  return function (target: any, propertyKey: string) {
    console.log(`${propertyKey} must be at least ${length} characters`);
  };
}

class User {
  @MinLength(3)
  name: string = '';
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What compiler flag is needed to use decorators? 🟡

- Historically: `"experimentalDecorators": true` in `tsconfig.json`, paired with `"emitDecoratorMetadata": true` if using reflection-based metadata (see Q8).
- TypeScript 5.0+ also supports the newer Stage 3 ECMAScript decorators **without** that flag — but the two decorator systems have slightly different signatures (see Q10), so check which one a given codebase/library targets.

[↑ Back to top](#table-of-contents)

---

### 7. How would you implement a simple logging decorator for a class method? 🔴

```ts
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`[${propertyKey}] called with: ${JSON.stringify(args)}`);
    const result = original.apply(this, args);
    console.log(`[${propertyKey}] returned: ${JSON.stringify(result)}`);
    return result;
  };
  return descriptor;
}

class MathService {
  @Log
  multiply(a: number, b: number) { return a * b; }
}
new MathService().multiply(2, 3);
// logs: "[multiply] called with: [2,3]" then "[multiply] returned: 6"
```

[↑ Back to top](#table-of-contents)

---

### 8. How do decorators relate to metadata reflection (e.g. in NestJS/Angular dependency injection)? 🔴

- With `emitDecoratorMetadata` enabled, TypeScript emits extra type metadata (parameter types, return types) alongside decorated members, readable at runtime via the `reflect-metadata` library.
- Frameworks like **NestJS** and **Angular** use this to implement dependency injection: a `@Injectable()` class's constructor parameter types are inspected at runtime to automatically resolve and inject the right instances, without you manually wiring each dependency.

```ts
@Injectable()
class UserService {
  constructor(private db: DatabaseService) {} // framework reads this type via reflection, injects it automatically
}
```

[↑ Back to top](#table-of-contents)

---

### 9. What is the execution order when multiple decorators are applied to the same target? 🔴

- **Evaluation** of decorator expressions happens top-to-bottom, but **application** (actually calling them) happens bottom-to-top — similar to function composition.

```ts
function first() { console.log('first: evaluated'); return () => console.log('first: applied'); }
function second() { console.log('second: evaluated'); return () => console.log('second: applied'); }

class Example {
  @first()
  @second()
  method() {}
}
// Order: "first: evaluated", "second: evaluated", "second: applied", "first: applied"
```

[↑ Back to top](#table-of-contents)

---

### 10. What changed between TypeScript's experimental decorators and the newer Stage 3 ECMAScript decorators? 🔴

- **Experimental decorators** (TS-specific, predates the JS standard): method decorators receive `(target, propertyKey, descriptor)`, can directly mutate the descriptor, and rely on `emitDecoratorMetadata` for type reflection.
- **Stage 3 / ECMAScript decorators** (the actual TC39 standard TS 5.0+ also supports): a different, more restrictive signature `(value, context)`, no direct descriptor mutation, and designed to work consistently across JS engines (not just TypeScript) — but currently lacks the metadata reflection experimental decorators provided, which is why frameworks like NestJS/Angular still default to the experimental flag.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can't existing decorator-heavy frameworks easily switch to the new standard yet?
> - What does `context.kind` give you in the new decorator signature that the old one didn't have directly?

[↑ Back to top](#table-of-contents)

---
