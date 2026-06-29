# Design Principles (SOLID)

_Part of [OOPs](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What does SOLID stand for?](#1-what-does-solid-stand-for)
- [2. What is the Single Responsibility Principle (SRP)?](#2-what-is-the-single-responsibility-principle-srp)
- [3. What is the Open/Closed Principle (OCP)?](#3-what-is-the-openclosed-principle-ocp)

**🟡 Medium**
- [4. What is the Liskov Substitution Principle (LSP)?](#4-what-is-the-liskov-substitution-principle-lsp)
- [5. What is the Interface Segregation Principle (ISP)?](#5-what-is-the-interface-segregation-principle-isp)
- [6. What is the Dependency Inversion Principle (DIP)?](#6-what-is-the-dependency-inversion-principle-dip)
- [7. How do you identify an SRP violation in existing code?](#7-how-do-you-identify-an-srp-violation-in-existing-code)
- [8. What's a classic example of an LSP violation?](#8-whats-a-classic-example-of-an-lsp-violation)

**🔴 Hard**
- [9. How do OCP and polymorphism work together in practice?](#9-how-do-ocp-and-polymorphism-work-together-in-practice)
- [10. What's the difference between dependency inversion and dependency injection?](#10-whats-the-difference-between-dependency-inversion-and-dependency-injection)
- [11. How does violating ISP lead to "fat interfaces"?](#11-how-does-violating-isp-lead-to-fat-interfaces)
- [12. Can over-applying SOLID principles itself become a problem?](#12-can-over-applying-solid-principles-itself-become-a-problem)
- [13. How would you refactor a class violating both SRP and OCP?](#13-how-would-you-refactor-a-class-violating-both-srp-and-ocp)

---

### 1. What does SOLID stand for? 🟢

- **S**ingle Responsibility, **O**pen/Closed, **L**iskov Substitution, **I**nterface Segregation, **D**ependency Inversion — five guidelines for writing maintainable, flexible object-oriented code, popularized by Robert C. Martin.

[↑ Back to top](#table-of-contents)

---

### 2. What is the Single Responsibility Principle (SRP)? 🟢

- A class should have **one** reason to change — i.e. one well-defined responsibility. A class handling both "calculate an invoice total" and "send the invoice email" has two unrelated reasons to change, and should likely be split into two classes.

[↑ Back to top](#table-of-contents)

---

### 3. What is the Open/Closed Principle (OCP)? 🟢

- Code should be **open for extension** (you can add new behavior) but **closed for modification** (you shouldn't need to edit existing, working code to do so) — typically achieved via polymorphism/abstraction (see [Polymorphism](polymorphism.md#11-how-does-polymorphism-support-the-openclosed-principle)).

[↑ Back to top](#table-of-contents)

---

### 4. What is the Liskov Substitution Principle (LSP)? 🟡

- Subtypes must be **substitutable** for their base type without breaking the correctness of code that uses the base type — if code works correctly with a `Shape`, it should keep working correctly when given any specific subclass of `Shape`, with no surprises (see [Inheritance](inheritance.md#12-what-is-the-liskov-substitution-principles-relationship-to-inheritance)).

[↑ Back to top](#table-of-contents)

---

### 5. What is the Interface Segregation Principle (ISP)? 🟡

- Clients shouldn't be forced to depend on methods they **don't use** — prefer several small, focused interfaces over one large, general-purpose interface that forces every implementer to provide methods irrelevant to them.

```java
// Violates ISP: every Worker must implement eat(), even a Robot that doesn't eat
interface Worker { void work(); void eat(); }

// Better: split into focused interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
```

[↑ Back to top](#table-of-contents)

---

### 6. What is the Dependency Inversion Principle (DIP)? 🟡

- High-level code should depend on **abstractions**, not concrete low-level implementations — and those abstractions shouldn't depend on implementation details either; it's the implementations that depend on (conform to) the abstraction (see [Abstraction](abstraction.md#11-whats-the-relationship-between-abstraction-and-the-dependency-inversion-principle)).

[↑ Back to top](#table-of-contents)

---

### 7. How do you identify an SRP violation in existing code? 🟡

- Ask: "what would force this class to change?" — if you can list **multiple, unrelated** reasons (a change in business calculation logic, a change in email formatting, a change in database schema), it's likely doing too much. A class name like `UserManagerAndEmailSenderAndLogger` is a strong tell as well.

[↑ Back to top](#table-of-contents)

---

### 8. What's a classic example of an LSP violation? 🟡

- `Square extends Rectangle`, where `Square` overrides `setWidth()`/`setHeight()` to force both dimensions equal — code that calls `setWidth(5)` on a `Rectangle` reference and then expects `getHeight()` to be unaffected breaks silently if the actual object is a `Square`, since setting its width also changed its height. The subtype changed the **expected behavior** of the base type's contract.

[↑ Back to top](#table-of-contents)

---

### 9. How do OCP and polymorphism work together in practice? 🔴

- A function written against an abstraction (e.g. a `PaymentMethod` interface) stays completely **closed for modification** — adding a new payment type (`CryptoPayment`) just means writing a new class implementing that interface, polymorphically slotting into existing code with **zero edits** to that existing code (see [Polymorphism](polymorphism.md#12-whats-the-practical-benefit-of-polymorphism-in-a-large-codebase)).

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between dependency inversion and dependency injection? 🔴

- **Dependency Inversion** (DIP): the **design principle** — depend on abstractions, not concrete implementations.
- **Dependency Injection**: a **technique** for actually *providing* a class with its dependencies (typically interfaces, per DIP) from the outside — via a constructor, a setter, or a DI framework/container — rather than having the class construct its own dependencies internally. DI is one common (though not the only) way to **achieve** DIP in practice.

```java
// Violates both DIP and uses no DI: OrderService creates its own concrete dependency
class OrderService { PaymentGateway gateway = new StripeGateway(); }

// Follows DIP, uses constructor injection
class OrderService {
  PaymentGateway gateway;
  OrderService(PaymentGateway gateway) { this.gateway = gateway; } // injected from outside
}
```

[↑ Back to top](#table-of-contents)

---

### 11. How does violating ISP lead to "fat interfaces"? 🔴

- As an interface accumulates more and more methods over time (to satisfy each new use case as it comes up), every implementer is forced to provide (or stub out with exceptions/no-ops) methods irrelevant to them — bloating implementations with dead/meaningless code and tightly coupling unrelated clients to an ever-growing, unfocused contract. Splitting into smaller, role-specific interfaces (Q5) keeps each implementer honest about what it actually does.

[↑ Back to top](#table-of-contents)

---

### 12. Can over-applying SOLID principles itself become a problem? 🔴

- Yes — aggressively splitting every class for hypothetical future flexibility (excessive SRP), or introducing an interface/abstraction for every single class "just in case" (excessive DIP/OCP) can produce **unnecessary indirection and complexity** for a system that doesn't actually need that flexibility yet. SOLID principles are guidelines for managing complexity as it arises, not a checklist to maximize regardless of context — over-applying them trades one kind of complexity (a messy class) for another (an over-abstracted maze of tiny classes/interfaces).

[↑ Back to top](#table-of-contents)

---

### 13. How would you refactor a class violating both SRP and OCP? 🔴

- First **split** the class along its distinct responsibilities (addressing SRP) — e.g. separate `InvoiceCalculator` from `InvoiceEmailer`. Then, for the part likely to need new variants over time (different calculation rules, different notification channels), introduce an **interface/abstract base** that each variant implements, so adding a new variant later doesn't require modifying the existing dispatching code (addressing OCP) — effectively combining single-responsibility extraction with a polymorphic extension point.

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you recognize, in a code review, that introducing an interface for OCP's sake is premature versus genuinely justified by the system's actual needs?

[↑ Back to top](#table-of-contents)

---
