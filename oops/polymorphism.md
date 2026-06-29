# Polymorphism

_Part of [OOPs](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is polymorphism?](#1-what-is-polymorphism)
- [2. What is compile-time (static) polymorphism?](#2-what-is-compile-time-static-polymorphism)
- [3. What is runtime (dynamic) polymorphism?](#3-what-is-runtime-dynamic-polymorphism)

**🟡 Medium**
- [4. How does method overriding enable runtime polymorphism?](#4-how-does-method-overriding-enable-runtime-polymorphism)
- [5. What is an abstract method, and how does it relate to polymorphism?](#5-what-is-an-abstract-method-and-how-does-it-relate-to-polymorphism)
- [6. What is dynamic method dispatch?](#6-what-is-dynamic-method-dispatch)
- [7. What is duck typing, and how does it relate to polymorphism?](#7-what-is-duck-typing-and-how-does-it-relate-to-polymorphism)

**🔴 Hard**
- [8. How is runtime polymorphism typically implemented internally (vtables)?](#8-how-is-runtime-polymorphism-typically-implemented-internally-vtables)
- [9. What is parametric polymorphism, and how does it relate to generics?](#9-what-is-parametric-polymorphism-and-how-does-it-relate-to-generics)
- [10. Why does calling an overridden method from inside a constructor behave unexpectedly?](#10-why-does-calling-an-overridden-method-from-inside-a-constructor-behave-unexpectedly)
- [11. How does polymorphism support the Open/Closed Principle?](#11-how-does-polymorphism-support-the-openclosed-principle)
- [12. What's the practical benefit of polymorphism in a large codebase?](#12-whats-the-practical-benefit-of-polymorphism-in-a-large-codebase)

---

### 1. What is polymorphism? 🟢

- The ability for objects of **different types** to be treated through a **common interface**, with each type providing its own specific behavior for the same method call — "many forms," literally.

[↑ Back to top](#table-of-contents)

---

### 2. What is compile-time (static) polymorphism? 🟢

- Resolved by the **compiler**, before the program runs — method **overloading** (see [Core Concepts](core-concepts.md#8-what-is-method-overloading)) is the classic example, where the compiler picks the right method based on the argument types at the call site.

[↑ Back to top](#table-of-contents)

---

### 3. What is runtime (dynamic) polymorphism? 🟢

- Resolved while the program is **running**, based on the object's **actual** type rather than its declared type — method **overriding** is the classic example.

```java
Animal a = new Dog(); // declared type: Animal, actual type: Dog
a.speak(); // calls Dog's speak() — decided at runtime, based on the actual object
```

[↑ Back to top](#table-of-contents)

---

### 4. How does method overriding enable runtime polymorphism? 🟡

- When a method is called on a reference of a **superclass** type but the underlying object is actually a **subclass** instance, the subclass's overridden version runs — the same line of code (`a.speak()`) produces different behavior depending on which concrete subclass `a` actually points to at runtime.

[↑ Back to top](#table-of-contents)

---

### 5. What is an abstract method, and how does it relate to polymorphism? 🟡

- A method declared **without** an implementation in a superclass/interface, forcing every concrete subclass to provide its **own** — guarantees every subtype honors the same method signature while allowing each to behave differently, the foundation that makes polymorphic calls through that shared method meaningful.

```java
abstract class Shape { abstract double area(); } // no implementation here
class Circle extends Shape { double area() { return Math.PI * r * r; } }
```

[↑ Back to top](#table-of-contents)

---

### 6. What is dynamic method dispatch? 🟡

- The mechanism by which the **specific method implementation to run** is determined at runtime based on the object's actual type, rather than the reference's declared type — the underlying machinery that makes runtime polymorphism (Q3) actually work.

[↑ Back to top](#table-of-contents)

---

### 7. What is duck typing, and how does it relate to polymorphism? 🟡

- "If it walks like a duck and quacks like a duck, it's a duck" — in dynamically/loosely-typed languages, an object can be used polymorphically as long as it **happens to support** the needed methods/properties, with **no** requirement that it formally implements a shared interface or inherits from a common ancestor. Achieves a form of polymorphism based purely on **behavior compatibility**, rather than a declared type relationship.

[↑ Back to top](#table-of-contents)

---

### 8. How is runtime polymorphism typically implemented internally (vtables)? 🔴

- Many languages (C++, Java conceptually) implement it via a **virtual method table (vtable)** — each class has a table of pointers to its actual method implementations; an object carries a hidden pointer to its class's vtable. A polymorphic call looks up the correct implementation through this table **at runtime**, which is why dynamic dispatch carries a small performance cost compared to a direct, statically-resolved call.

[↑ Back to top](#table-of-contents)

---

### 9. What is parametric polymorphism, and how does it relate to generics? 🔴

- Writing code that works **uniformly across many types**, parameterized by a type variable, without the code itself caring what that type actually is — generics (`List<T>`, TypeScript's `<T>`) are the direct implementation of parametric polymorphism, letting one function/class definition work correctly for `List<String>`, `List<Number>`, etc., without rewriting it per type.

[↑ Back to top](#table-of-contents)

---

### 10. Why does calling an overridden method from inside a constructor behave unexpectedly? 🔴

- If a superclass constructor calls a method that's **overridden** in the subclass, the **subclass's** version runs (dynamic dispatch, Q6) — but at that point, the **subclass's own fields haven't been initialized yet** (the subclass constructor hasn't run its body yet), so the overridden method may operate on incomplete/default state, producing subtle bugs. General guidance: avoid calling overridable methods from within a constructor.

[↑ Back to top](#table-of-contents)

---

### 11. How does polymorphism support the Open/Closed Principle? 🔴

- The Open/Closed Principle says code should be **open for extension, closed for modification** — polymorphism lets you add **new** subtypes implementing a shared interface/abstract method, extending the system's behavior, **without** touching any existing code that depends on that interface (which simply calls the polymorphic method, unaware of the new subtype). See [SOLID](design-principles-solid.md) for the full principle.

[↑ Back to top](#table-of-contents)

---

### 12. What's the practical benefit of polymorphism in a large codebase? 🔴

- Code written against an **abstraction** (an interface, a base class) doesn't need to know about every concrete implementation — new types can be introduced later and just "plug in" anywhere that abstraction is used, with **zero changes** to existing calling code. This is what makes plugin systems, strategy patterns (see [Design Patterns](design-patterns.md)), and extensible frameworks practical at scale.

> [!IMPORTANT]
> **Follow-up questions:**
> - If a function takes a parameter typed as an interface rather than a concrete class, what specifically does that buy you when new implementations of that interface are added later?

[↑ Back to top](#table-of-contents)

---
