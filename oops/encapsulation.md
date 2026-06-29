# Encapsulation

_Part of [OOPs](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is encapsulation?](#1-what-is-encapsulation)
- [2. What are access modifiers?](#2-what-are-access-modifiers)
- [3. What is a getter/setter?](#3-what-is-a-gettersetter)

**🟡 Medium**
- [4. Why use private fields with public getters/setters instead of just making fields public?](#4-why-use-private-fields-with-public-getterssetters-instead-of-just-making-fields-public)
- [5. What's the difference between `private`, `protected`, and `public`?](#5-whats-the-difference-between-private-protected-and-public)
- [6. What is data hiding, and how does it relate to encapsulation?](#6-what-is-data-hiding-and-how-does-it-relate-to-encapsulation)
- [7. What is an invariant, and how does encapsulation help enforce one?](#7-what-is-an-invariant-and-how-does-encapsulation-help-enforce-one)

**🔴 Hard**
- [8. What's the difference between encapsulation and abstraction?](#8-whats-the-difference-between-encapsulation-and-abstraction)
- [9. How does encapsulation support maintainability when requirements change?](#9-how-does-encapsulation-support-maintainability-when-requirements-change)
- [10. Can encapsulation be broken even with private fields? How?](#10-can-encapsulation-be-broken-even-with-private-fields-how)
- [11. How does encapsulation relate to the concept of a module's "public API"?](#11-how-does-encapsulation-relate-to-the-concept-of-a-modules-public-api)

---

### 1. What is encapsulation? 🟢

- Bundling an object's data together with the methods that operate on it, and **restricting direct access** to that data from outside — external code interacts with the object only through its defined methods, not by reaching in and modifying its internals directly.

[↑ Back to top](#table-of-contents)

---

### 2. What are access modifiers? 🟢

- Keywords controlling **visibility** of a class's members from outside code — typically `public` (accessible from anywhere), `private` (only within the class itself), and `protected` (accessible within the class and its subclasses).

[↑ Back to top](#table-of-contents)

---

### 3. What is a getter/setter? 🟢

- Public methods that provide **controlled** access to a private field — a getter reads its value, a setter updates it (optionally validating the new value before accepting it).

```java
class Account {
  private double balance;
  public double getBalance() { return balance; }
  public void setBalance(double amount) {
    if (amount < 0) throw new IllegalArgumentException("Balance can't be negative");
    this.balance = amount;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 4. Why use private fields with public getters/setters instead of just making fields public? 🟡

- A public field can be set to **any** value from anywhere, with no validation — a setter lets you **enforce rules** (e.g. rejecting a negative balance) at the single point of entry, and lets you later change the internal representation without breaking external code that only ever interacted through the getter/setter, not the raw field.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `private`, `protected`, and `public`? 🟡

- **`private`**: accessible only within the **same class**.
- **`protected`**: accessible within the same class **and its subclasses** (and often the same package, depending on the language).
- **`public`**: accessible from **anywhere**.

[↑ Back to top](#table-of-contents)

---

### 6. What is data hiding, and how does it relate to encapsulation? 🟡

- **Data hiding** is the specific *mechanism* (using access modifiers to restrict visibility) — **encapsulation** is the broader *principle/goal* it serves (bundling data+behavior and controlling access to maintain consistency). Data hiding is how encapsulation is actually implemented in code.

[↑ Back to top](#table-of-contents)

---

### 7. What is an invariant, and how does encapsulation help enforce one? 🟡

- An **invariant** is a condition that must always hold true for an object to be in a valid state (e.g. "balance can never go negative"). Encapsulation enforces invariants by funneling all state changes through controlled methods (setters, dedicated methods like `withdraw()`) that can validate the change **before** applying it — direct, unchecked field access would let any code violate the invariant accidentally.

```java
public void withdraw(double amount) {
  if (amount > balance) throw new IllegalStateException("Insufficient funds");
  balance -= amount; // invariant (balance >= 0) is protected
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between encapsulation and abstraction? 🔴

- **Encapsulation**: about **restricting access** to an object's internal state — a mechanism (private fields, access modifiers).
- **Abstraction**: about **hiding complexity** by exposing only relevant, simplified behavior — a design concept (an interface, a simplified API). They're complementary and often used together, but encapsulation is concerned with *protecting* internal data, while abstraction is concerned with *simplifying* what's exposed (see [Abstraction](abstraction.md) for the deeper distinction).

[↑ Back to top](#table-of-contents)

---

### 9. How does encapsulation support maintainability when requirements change? 🔴

- Since external code only depends on a class's **public interface**, not its internal implementation, you can freely change *how* a class stores/computes its data internally (switch a field's type, change a calculation) without breaking any code that consumes it — as long as the public method signatures and behavior stay the same.

[↑ Back to top](#table-of-contents)

---

### 10. Can encapsulation be broken even with private fields? How? 🔴

- Yes — if a getter returns a **mutable** internal object (an array, a list, a mutable date) **by reference** rather than a copy, the caller can mutate that returned object directly, silently changing the original object's internal state from outside, bypassing any validation the class intended. Fix by returning a defensive **copy**, or an immutable view.

```java
// Vulnerable: caller can do account.getTransactions().clear() and corrupt internal state
public List<String> getTransactions() { return transactions; }

// Safer: returns a copy, caller's mutations don't affect internal state
public List<String> getTransactions() { return new ArrayList<>(transactions); }
```

[↑ Back to top](#table-of-contents)

---

### 11. How does encapsulation relate to the concept of a module's "public API"? 🔴

- The same principle applied at a **larger scale** — a well-designed module/library exposes a deliberate, minimal public API (its "public" surface) while keeping internal implementation details (helper functions, internal data structures) hidden/unexported, exactly mirroring how a class hides its private fields. Changing hidden internals freely (without breaking consumers) at the module level depends on the same discipline as a class's encapsulation.

> [!IMPORTANT]
> **Follow-up questions:**
> - If a library exports an internal helper function "just in case someone needs it," what encapsulation principle does that violate, and what risk does it create for future changes?

[↑ Back to top](#table-of-contents)

---
