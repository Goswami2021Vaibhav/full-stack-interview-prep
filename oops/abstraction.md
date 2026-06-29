# Abstraction

_Part of [OOPs](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is abstraction?](#1-what-is-abstraction)
- [2. What is an abstract class?](#2-what-is-an-abstract-class)
- [3. What is an interface?](#3-what-is-an-interface)

**🟡 Medium**
- [4. What's the difference between an abstract class and an interface?](#4-whats-the-difference-between-an-abstract-class-and-an-interface)
- [5. Can an abstract class have a constructor? Why would it need one?](#5-can-an-abstract-class-have-a-constructor-why-would-it-need-one)
- [6. Why can't you instantiate an abstract class directly?](#6-why-cant-you-instantiate-an-abstract-class-directly)
- [7. What is a real-world analogy that illustrates abstraction well?](#7-what-is-a-real-world-analogy-that-illustrates-abstraction-well)

**🔴 Hard**
- [8. How do default/concrete methods in interfaces blur the line with abstract classes?](#8-how-do-defaultconcrete-methods-in-interfaces-blur-the-line-with-abstract-classes)
- [9. When should you use an abstract class vs. an interface?](#9-when-should-you-use-an-abstract-class-vs-an-interface)
- [10. How does abstraction reduce the impact of changes in a large system?](#10-how-does-abstraction-reduce-the-impact-of-changes-in-a-large-system)
- [11. What's the relationship between abstraction and the Dependency Inversion Principle?](#11-whats-the-relationship-between-abstraction-and-the-dependency-inversion-principle)

---

### 1. What is abstraction? 🟢

- Exposing only the **essential** behavior/interface of something while hiding the underlying implementation complexity — a driver uses a car's steering wheel and pedals without needing to understand the engine's internal combustion process.

[↑ Back to top](#table-of-contents)

---

### 2. What is an abstract class? 🟢

- A class that **cannot be instantiated directly**, meant to be subclassed — it can mix fully-implemented methods with **abstract methods** (declared but not implemented, see [Polymorphism](polymorphism.md#5-what-is-an-abstract-method-and-how-does-it-relate-to-polymorphism)) that subclasses must provide.

```java
abstract class Shape {
  abstract double area();              // must be implemented by subclasses
  void describe() { System.out.println("Area: " + area()); } // shared, concrete
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What is an interface? 🟢

- A purely abstract contract — declares method signatures **without** any implementation (traditionally), which any implementing class must fulfill. Defines "what a class can do," not "how."

```java
interface Drawable { void draw(); }
class Circle implements Drawable { public void draw() { ... } }
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between an abstract class and an interface? 🟡

- **Abstract class**: can have **both** concrete and abstract methods, instance fields, and constructors — a class can extend only **one**.
- **Interface**: traditionally only method signatures (modern languages now allow some default implementations too, Q8) — a class can implement **multiple** interfaces.

[↑ Back to top](#table-of-contents)

---

### 5. Can an abstract class have a constructor? Why would it need one? 🟡

- Yes — even though it can't be instantiated directly, its constructor runs when a **subclass** is instantiated (via `super()`), typically used to initialize fields shared by every subclass.

```java
abstract class Shape {
  String color;
  Shape(String color) { this.color = color; } // runs when any subclass is constructed
}
```

[↑ Back to top](#table-of-contents)

---

### 6. Why can't you instantiate an abstract class directly? 🟡

- It may contain **abstract methods with no implementation** — creating an instance would mean having an object that's missing required behavior, with no defined way to actually execute a call to that unimplemented method. The language enforces that only a **concrete** subclass (which supplies the missing implementations) can actually be instantiated.

[↑ Back to top](#table-of-contents)

---

### 7. What is a real-world analogy that illustrates abstraction well? 🟡

- A TV remote: you press "power" or "volume up" without knowing anything about the infrared signal encoding or the TV's internal circuitry — the remote's buttons are the **abstraction**, hiding all of that complexity behind a simple interface.

[↑ Back to top](#table-of-contents)

---

### 8. How do default/concrete methods in interfaces blur the line with abstract classes? 🔴

- Modern languages (Java 8+, with `default` methods) let interfaces provide an **actual implementation** for some methods, not just signatures — narrowing the historical gap between interfaces and abstract classes. The remaining hard distinction is usually: interfaces still can't hold **instance state** (fields) the way an abstract class can, and a class can implement many interfaces but extend only one abstract class.

[↑ Back to top](#table-of-contents)

---

### 9. When should you use an abstract class vs. an interface? 🔴

- **Abstract class**: when subclasses share significant **common implementation/state**, and there's a genuine "is-a" hierarchy (an `Animal` base class shared by `Dog`/`Cat`).
- **Interface**: when you're defining a **capability/contract** that unrelated classes might implement (`Comparable`, `Serializable`) — a `Dog` and a `String` might both be `Comparable`, despite having nothing else in common, which an abstract class hierarchy couldn't elegantly express.

[↑ Back to top](#table-of-contents)

---

### 10. How does abstraction reduce the impact of changes in a large system? 🔴

- Code that depends on an **abstraction** (an interface) rather than a concrete implementation can keep working unchanged even if the underlying implementation is completely rewritten — as long as the abstraction's contract stays the same. This decouples *how* something is done from *that* it's done, letting large systems evolve their internals independently of all the code that merely depends on the interface.

[↑ Back to top](#table-of-contents)

---

### 11. What's the relationship between abstraction and the Dependency Inversion Principle? 🔴

- The Dependency Inversion Principle states that high-level code should depend on **abstractions**, not concrete low-level implementations — abstraction (interfaces/abstract classes) is the literal mechanism that makes this possible: a high-level `OrderService` can depend on a `PaymentGateway` **interface**, with the actual `StripeGateway`/`PayPalGateway` implementation swapped in later, without `OrderService` ever needing to know which concrete class it's actually using (see [SOLID](design-principles-solid.md)).

> [!IMPORTANT]
> **Follow-up questions:**
> - If `OrderService` directly instantiated `new StripeGateway()` internally instead of depending on a `PaymentGateway` interface, what would break if you later needed to swap payment providers or write a unit test?

[↑ Back to top](#table-of-contents)

---
