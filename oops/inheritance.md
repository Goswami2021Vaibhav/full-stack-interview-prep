# Inheritance

_Part of [OOPs](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is inheritance?](#1-what-is-inheritance)
- [2. What is a superclass/parent class and a subclass/child class?](#2-what-is-a-superclassparent-class-and-a-subclasschild-class)
- [3. What is `super`, and what is it used for?](#3-what-is-super-and-what-is-it-used-for)

**🟡 Medium**
- [4. What is single inheritance vs. multiple inheritance?](#4-what-is-single-inheritance-vs-multiple-inheritance)
- [5. What is multilevel inheritance?](#5-what-is-multilevel-inheritance)
- [6. What is hierarchical inheritance?](#6-what-is-hierarchical-inheritance)
- [7. Why do many languages (Java, C#) not support multiple inheritance of classes?](#7-why-do-many-languages-java-c-not-support-multiple-inheritance-of-classes)
- [8. What is the diamond problem?](#8-what-is-the-diamond-problem)

**🔴 Hard**
- [9. How do interfaces solve the need for multiple inheritance without the diamond problem?](#9-how-do-interfaces-solve-the-need-for-multiple-inheritance-without-the-diamond-problem)
- [10. What is method resolution order (MRO)?](#10-what-is-method-resolution-order-mro)
- [11. Why is deep inheritance hierarchy considered an anti-pattern?](#11-why-is-deep-inheritance-hierarchy-considered-an-anti-pattern)
- [12. What is the Liskov Substitution Principle's relationship to inheritance?](#12-what-is-the-liskov-substitution-principles-relationship-to-inheritance)
- [13. What's the difference between inheritance and interface implementation?](#13-whats-the-difference-between-inheritance-and-interface-implementation)

---

### 1. What is inheritance? 🟢

- A mechanism letting one class (the subclass) **acquire** the properties and methods of another (the superclass) — enabling code reuse and modeling "is-a" relationships (a `Dog` is an `Animal`).

```java
class Animal { void eat() { ... } }
class Dog extends Animal { } // Dog inherits eat() from Animal
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a superclass/parent class and a subclass/child class? 🟢

- **Superclass (parent/base class)**: the class being inherited **from**.
- **Subclass (child/derived class)**: the class doing the inheriting, gaining the superclass's members and optionally adding/overriding its own.

[↑ Back to top](#table-of-contents)

---

### 3. What is `super`, and what is it used for? 🟢

- A reference to the **immediate superclass**, used to call its constructor or an overridden method's original implementation from within the subclass.

```java
class Dog extends Animal {
  Dog() { super(); }              // call the superclass's constructor
  void eat() { super.eat(); ... } // call the superclass's version of an overridden method
}
```

[↑ Back to top](#table-of-contents)

---

### 4. What is single inheritance vs. multiple inheritance? 🟡

- **Single inheritance**: a class extends **one** superclass only.
- **Multiple inheritance**: a class extends **more than one** superclass directly — supported in some languages (C++), but disallowed for classes in others (Java, C#) due to the diamond problem (Q8).

[↑ Back to top](#table-of-contents)

---

### 5. What is multilevel inheritance? 🟡

- A chain where a class inherits from a class that **itself** inherits from another — `C extends B`, and `B extends A`, so `C` transitively gains `A`'s members too.

[↑ Back to top](#table-of-contents)

---

### 6. What is hierarchical inheritance? 🟡

- **Multiple subclasses** inheriting from the **same single** superclass — e.g. `Dog extends Animal` and `Cat extends Animal`, both siblings sharing one common parent.

[↑ Back to top](#table-of-contents)

---

### 7. Why do many languages (Java, C#) not support multiple inheritance of classes? 🟡

- Primarily to avoid the diamond problem (Q8) and the added complexity it introduces — these languages instead support implementing **multiple interfaces**, which provides much of multiple inheritance's flexibility (inheriting multiple "contracts") without the ambiguity of inheriting **conflicting implementations**.

[↑ Back to top](#table-of-contents)

---

### 8. What is the diamond problem? 🟡

- Arises when a class inherits from two classes that **both** inherit from a common ancestor, and both override the **same** method — the resulting class has an **ambiguous** inheritance path for that method (which version does it actually get?). Named for the diamond shape the class hierarchy diagram forms.

```
      A
     / \
    B   C
     \ /
      D     <- which version of a method overridden in both B and C does D inherit?
```

[↑ Back to top](#table-of-contents)

---

### 9. How do interfaces solve the need for multiple inheritance without the diamond problem? 🔴

- An interface declares **method signatures** (a contract) without providing a conflicting **implementation** — a class can implement multiple interfaces safely, since there's no ambiguity about which *implementation* to inherit (the implementing class always provides its own). The diamond problem specifically arises from conflicting **implementations**, which interfaces (traditionally) don't carry.

[↑ Back to top](#table-of-contents)

---

### 10. What is method resolution order (MRO)? 🔴

- The specific, well-defined **algorithm** a language uses to determine which class's method implementation to use when multiple ancestors in a hierarchy could provide it — e.g. Python's C3 linearization algorithm gives a deterministic order to search through a class's multiple base classes, resolving exactly the kind of ambiguity the diamond problem raises, rather than disallowing multiple inheritance outright.

[↑ Back to top](#table-of-contents)

---

### 11. Why is deep inheritance hierarchy considered an anti-pattern? 🔴

- Each additional level adds **tight coupling** between subclass and superclass — a change anywhere up the chain can have cascading, hard-to-predict effects on every descendant; deep hierarchies become difficult to understand (you must trace through many levels to know a class's full behavior) and brittle to refactor. This is the core motivation behind "favor composition over inheritance" (see [Core Concepts](core-concepts.md#11-what-is-object-composition-and-how-does-it-differ-from-inheritance)).

[↑ Back to top](#table-of-contents)

---

### 12. What is the Liskov Substitution Principle's relationship to inheritance? 🔴

- States that a subclass object should be usable **anywhere** its superclass is expected, without breaking the calling code's expectations — inheritance is only modeling a *correct* "is-a" relationship if this substitutability actually holds. A classic violation: `Square extends Rectangle` overriding `setWidth()`/`setHeight()` to keep both sides equal, which breaks code that assumes setting a `Rectangle`'s width independently of its height is safe (see [SOLID](design-principles-solid.md)).

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between inheritance and interface implementation? 🔴

- **Inheritance** (`extends`): inherits **both** the contract (method signatures) **and** a concrete implementation from the superclass — represents a stronger "is-a" relationship.
- **Interface implementation** (`implements`): commits to a **contract only** (method signatures), with the implementing class providing its **own** complete implementation — represents more of a "can-do" / "behaves-as" relationship, and (in most languages) a class can implement many interfaces but extend only one class.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might "favor composition, supplemented by interfaces, over deep class inheritance" be better default advice than reaching for `extends` first?

[↑ Back to top](#table-of-contents)

---
