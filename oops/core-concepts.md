# Core Concepts

_Part of [OOPs](README.md) interview notes._

> Examples in this topic use Java/TypeScript-style syntax to stay language-agnostic. For JavaScript-specific OOP mechanics (prototypes, classes-as-sugar), see [JavaScript › OOPs](../javascript/oops.md).

## Table of Contents

**🟢 Easy**
- [1. What is Object-Oriented Programming?](#1-what-is-object-oriented-programming)
- [2. What are the four pillars of OOP?](#2-what-are-the-four-pillars-of-oop)
- [3. What is a class, and what is an object?](#3-what-is-a-class-and-what-is-an-object)
- [4. What is a constructor?](#4-what-is-a-constructor)

**🟡 Medium**
- [5. What's the difference between procedural and object-oriented programming?](#5-whats-the-difference-between-procedural-and-object-oriented-programming)
- [6. What is the difference between a class variable and an instance variable?](#6-what-is-the-difference-between-a-class-variable-and-an-instance-variable)
- [7. What is `this`/`self`, and what does it refer to?](#7-what-is-thisself-and-what-does-it-refer-to)
- [8. What is method overloading?](#8-what-is-method-overloading)
- [9. What is method overriding?](#9-what-is-method-overriding)

**🔴 Hard**
- [10. What's the difference between a class and a struct (conceptually)?](#10-whats-the-difference-between-a-class-and-a-struct-conceptually)
- [11. What is object composition, and how does it differ from inheritance?](#11-what-is-object-composition-and-how-does-it-differ-from-inheritance)
- [12. What is coupling and cohesion, and why do they matter in OOP design?](#12-what-is-coupling-and-cohesion-and-why-do-they-matter-in-oop-design)
- [13. What is the difference between "is-a" and "has-a" relationships?](#13-what-is-the-difference-between-is-a-and-has-a-relationships)

---

### 1. What is Object-Oriented Programming? 🟢

- A programming paradigm that models software as a collection of interacting **objects** — each bundling together data (state) and the behavior (methods) that operates on it — rather than a sequence of procedures acting on separate data.

[↑ Back to top](#table-of-contents)

---

### 2. What are the four pillars of OOP? 🟢

- **Encapsulation**: bundling data and behavior together, hiding internal details.
- **Abstraction**: exposing only what's necessary, hiding implementation complexity.
- **Inheritance**: deriving new classes from existing ones, reusing/extending behavior.
- **Polymorphism**: the same interface behaving differently depending on the actual underlying type.

[↑ Back to top](#table-of-contents)

---

### 3. What is a class, and what is an object? 🟢

- A **class** is a blueprint/template defining what properties and behaviors its instances will have.
- An **object** is a concrete **instance** of that class, with its own actual values for those properties.

```java
class Car { String color; void drive() { ... } }
Car myCar = new Car(); // myCar is an object — a specific instance of Car
```

[↑ Back to top](#table-of-contents)

---

### 4. What is a constructor? 🟢

- A special method automatically called when an object is created, typically used to initialize its initial state.

```java
class Car {
  String color;
  Car(String color) { this.color = color; } // constructor
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between procedural and object-oriented programming? 🟡

- **Procedural**: a program is a sequence of functions/procedures operating on data passed between them — data and behavior are separate.
- **Object-oriented**: data and the behavior that operates on it are bundled **together** into objects — better suited for modeling complex systems with many interacting, stateful entities, and generally easier to extend/maintain as a system grows.

[↑ Back to top](#table-of-contents)

---

### 6. What is the difference between a class variable and an instance variable? 🟡

- **Class variable** (`static`): shared across **every** instance of the class — one copy total, regardless of how many objects exist.
- **Instance variable**: each object gets its **own** separate copy.

```java
class Counter {
  static int totalCount = 0;  // shared across all instances
  int id;                       // unique per instance
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What is `this`/`self`, and what does it refer to? 🟡

- A reference to the **current object instance** a method is being called on — used to access that specific instance's own fields/methods, particularly to disambiguate from a same-named parameter or local variable.

```java
class Car {
  String color;
  Car(String color) { this.color = color; } // this.color = the field; color = the parameter
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is method overloading? 🟡

- Defining **multiple methods with the same name** in the same class, distinguished by their **parameter list** (different number/types of parameters) — resolved at **compile time** based on the arguments passed.

```java
void print(String s) { ... }
void print(int i) { ... }    // overloaded — different parameter type
```

[↑ Back to top](#table-of-contents)

---

### 9. What is method overriding? 🟡

- A **subclass** providing its own implementation of a method already defined in its **superclass**, with the **same signature** — resolved at **runtime**, based on the actual object's type (see [Polymorphism](polymorphism.md)).

```java
class Animal { void speak() { System.out.println("..."); } }
class Dog extends Animal { void speak() { System.out.println("Woof"); } } // overridden
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between a class and a struct (conceptually)? 🔴

- In languages that distinguish them (C++, C#): a `struct` is traditionally a simpler, lightweight, typically **value-type** construct meant for grouping plain data without much behavior, while a `class` is a full-featured, typically **reference-type** construct supporting inheritance, access control, and richer behavior. The exact distinction is language-specific — some languages (Java, JavaScript) have no separate `struct` concept at all.

[↑ Back to top](#table-of-contents)

---

### 11. What is object composition, and how does it differ from inheritance? 🔴

- **Composition**: building a class by **including instances of other classes** as fields ("has-a" relationship) — a `Car` *has an* `Engine`.
- **Inheritance**: building a class by **extending** another ("is-a" relationship) — a `Car` *is a* `Vehicle`. The well-known design guideline "favor composition over inheritance" reflects that composition tends to produce more flexible, less tightly-coupled designs than deep inheritance hierarchies (see [Design Patterns](design-patterns.md)).

[↑ Back to top](#table-of-contents)

---

### 12. What is coupling and cohesion, and why do they matter in OOP design? 🔴

- **Coupling**: how much one class/module **depends on** the internal details of another — lower coupling means classes can change independently without breaking each other.
- **Cohesion**: how closely related and focused a class's responsibilities are — high cohesion means a class does **one** well-defined thing, rather than many unrelated things.
- Good OOP design aims for **low coupling, high cohesion** — classes that are self-contained and focused, interacting through clean, minimal interfaces.

[↑ Back to top](#table-of-contents)

---

### 13. What is the difference between "is-a" and "has-a" relationships? 🔴

- **"Is-a"**: modeled via **inheritance** — a `Dog` is-a `Animal`. Implies the subtype should be usable anywhere the supertype is expected (see [Polymorphism](polymorphism.md) and the Liskov Substitution Principle in [SOLID](design-principles-solid.md)).
- **"Has-a"**: modeled via **composition** — a `Car` has-a `Engine`. The `Car` doesn't *become* an `Engine`; it simply **owns/uses** one. Misusing inheritance for a relationship that's really "has-a" (e.g. making `Car extends Engine`) is a common OOP design mistake.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is "favor composition over inheritance" considered good practice, even though inheritance is one of OOP's core pillars?

[↑ Back to top](#table-of-contents)

---
