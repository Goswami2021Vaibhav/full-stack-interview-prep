# Design Patterns

_Part of [OOPs](README.md) interview notes._

> For concrete JavaScript implementations of the Observer and Strategy patterns, see [JavaScript › OOPs](../javascript/oops.md). This file covers the broader catalog conceptually, language-agnostically.

## Table of Contents

**🟢 Easy**
- [1. What is a design pattern, and why use one?](#1-what-is-a-design-pattern-and-why-use-one)
- [2. What are the three main categories of GoF design patterns?](#2-what-are-the-three-main-categories-of-gof-design-patterns)
- [3. What is the Singleton pattern?](#3-what-is-the-singleton-pattern)

**🟡 Medium**
- [4. What is the Factory pattern?](#4-what-is-the-factory-pattern)
- [5. What is the Builder pattern?](#5-what-is-the-builder-pattern)
- [6. What is the Adapter pattern?](#6-what-is-the-adapter-pattern)
- [7. What is the Decorator pattern?](#7-what-is-the-decorator-pattern)
- [8. What is the Strategy pattern?](#8-what-is-the-strategy-pattern)
- [9. What is the Observer pattern?](#9-what-is-the-observer-pattern)

**🔴 Hard**
- [10. What is the Facade pattern?](#10-what-is-the-facade-pattern)
- [11. What's the difference between Factory Method and Abstract Factory?](#11-whats-the-difference-between-factory-method-and-abstract-factory)
- [12. Why is Singleton often considered an anti-pattern despite being popular?](#12-why-is-singleton-often-considered-an-anti-pattern-despite-being-popular)
- [13. What is the Command pattern?](#13-what-is-the-command-pattern)
- [14. How would you choose between the Strategy pattern and a simple `if`/`else`, or between Decorator and inheritance?](#14-how-would-you-choose-between-the-strategy-pattern-and-a-simple-ifelse-or-between-decorator-and-inheritance)

---

### 1. What is a design pattern, and why use one? 🟢

- A **reusable, proven solution** to a recurring software design problem — not specific code you copy-paste, but a general approach/structure you adapt to your situation. Using established patterns gives a shared vocabulary among developers and avoids reinventing solutions to well-understood problems.

[↑ Back to top](#table-of-contents)

---

### 2. What are the three main categories of GoF design patterns? 🟢

- **Creational**: concerned with **object creation** (Singleton, Factory, Builder).
- **Structural**: concerned with how objects/classes are **composed** into larger structures (Adapter, Decorator, Facade).
- **Behavioral**: concerned with how objects **interact and communicate** (Observer, Strategy, Command).

[↑ Back to top](#table-of-contents)

---

### 3. What is the Singleton pattern? 🟢

- Ensures a class has **only one** instance throughout the application, providing a single global access point to it — commonly used for shared resources like a configuration object or a connection pool.

```java
class Config {
  private static Config instance;
  private Config() {}
  public static Config getInstance() {
    if (instance == null) instance = new Config();
    return instance;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 4. What is the Factory pattern? 🟡

- Delegates object **creation** to a dedicated method/class, rather than calling `new` directly wherever an object is needed — the calling code asks for "a shape" without needing to know which **concrete** class gets instantiated.

```java
class ShapeFactory {
  static Shape create(String type) {
    if (type.equals("circle")) return new Circle();
    if (type.equals("square")) return new Square();
    throw new IllegalArgumentException();
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What is the Builder pattern? 🟡

- Constructs a complex object **step by step**, via a fluent chain of method calls, instead of one constructor with an unwieldy number of parameters — especially useful when many fields are optional.

```java
User user = new UserBuilder()
  .setName("Vaibhav")
  .setEmail("v@x.com")
  .setRole("admin")
  .build();
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the Adapter pattern? 🟡

- Wraps an object with an **incompatible interface**, translating calls into a form the client code expects — lets two otherwise-incompatible interfaces work together without modifying either one's original code.

```java
class LegacyPrinter { void oldPrint(String s) { ... } }
class PrinterAdapter implements ModernPrinter {
  LegacyPrinter legacy;
  public void print(String s) { legacy.oldPrint(s); } // translates the call
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What is the Decorator pattern? 🟡

- **Wraps** an object to add new behavior/responsibilities **dynamically**, without modifying its original class or relying on subclassing — multiple decorators can be stacked to combine behaviors flexibly at runtime.

```java
Coffee coffee = new MilkDecorator(new SugarDecorator(new BasicCoffee()));
// Each decorator adds its own cost/description on top of the wrapped object
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the Strategy pattern? 🟡

- Defines a **family of interchangeable algorithms**, each encapsulated behind a common interface, letting the algorithm used be selected/swapped **at runtime** without changing the code that uses it — see [JavaScript › OOPs](../javascript/oops.md) for a concrete implementation.

[↑ Back to top](#table-of-contents)

---

### 9. What is the Observer pattern? 🟡

- Lets one or more "observer" objects **subscribe** to and get automatically notified of changes/events in a "subject" object, without the subject needing to know any specifics about its observers — the foundation behind event systems (`EventEmitter`, DOM events). See [JavaScript › OOPs](../javascript/oops.md) for a concrete implementation.

[↑ Back to top](#table-of-contents)

---

### 10. What is the Facade pattern? 🔴

- Provides a **simplified, unified interface** in front of a complex set of underlying subsystems/classes — the client interacts only with the facade, never needing to understand or directly coordinate the multiple components working behind it.

```java
class OrderFacade {
  void placeOrder() {
    inventory.reserve(); payment.charge(); shipping.schedule(); // coordinates 3 subsystems
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between Factory Method and Abstract Factory? 🔴

- **Factory Method**: a **single method** (often overridden by subclasses) that creates **one** type of object — focused on deferring the decision of *which concrete class* to instantiate.
- **Abstract Factory**: an interface for creating a **whole family of related objects** (e.g. a `UIFactory` that produces matching `Button`, `Checkbox`, and `Scrollbar` objects for a consistent theme) — a higher-level pattern, often implemented **using** multiple Factory Methods internally.

[↑ Back to top](#table-of-contents)

---

### 12. Why is Singleton often considered an anti-pattern despite being popular? 🔴

- It introduces **hidden global state**, making code that depends on it harder to **unit test** (you can't easily swap in a mock/different instance per test) and creates **implicit coupling** between any class that reaches for the singleton directly, rather than receiving it as an explicit, injectable dependency (see [SOLID › Dependency Inversion](design-principles-solid.md#6-what-is-the-dependency-inversion-principle-dip)). Often, a single shared instance managed via dependency injection achieves the same goal with far less hidden coupling.

[↑ Back to top](#table-of-contents)

---

### 13. What is the Command pattern? 🔴

- Encapsulates a **request/action** itself as an object (with an `execute()` method), rather than calling the action directly — lets you queue, log, undo/redo, or pass around "an action to be performed later" as a first-class value.

```java
interface Command { void execute(); void undo(); }
class AddTextCommand implements Command {
  void execute() { document.append(text); }
  void undo() { document.removeLast(text.length()); }
}
```

[↑ Back to top](#table-of-contents)

---

### 14. How would you choose between the Strategy pattern and a simple `if`/`else`, or between Decorator and inheritance? 🔴

- **Strategy vs. `if`/`else`**: a handful of stable, rarely-changing branches is fine as plain conditionals — reach for Strategy when the set of behaviors is **expected to grow** over time, needs to be swapped at runtime, or each branch is complex enough to deserve its own dedicated class (avoiding the Open/Closed Principle violation of editing one big conditional every time a new case appears).
- **Decorator vs. inheritance**: use inheritance for a **fixed, small** set of variations known upfront; reach for Decorator when you need to **combine multiple behaviors flexibly at runtime** (e.g. any combination of milk/sugar/whip on a coffee) — inheritance alone would require a combinatorial explosion of subclasses for every possible combination.

> [!IMPORTANT]
> **Follow-up questions:**
> - If you used inheritance to model "coffee with milk," "coffee with sugar," and "coffee with milk and sugar" as three separate subclasses, what happens to that hierarchy once a fourth optional ingredient is introduced?

[↑ Back to top](#table-of-contents)

---
