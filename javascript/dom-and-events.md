# DOM & Events

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between `window` and `document`?](#1-whats-the-difference-between-window-and-document)
- [2. What's the difference between `querySelector()` and `getElementById()`?](#2-whats-the-difference-between-queryselector-and-getelementbyid)
- [3. What's the difference between an attribute and a property?](#3-whats-the-difference-between-an-attribute-and-a-property)
- [4. How do you add and remove an event listener?](#4-how-do-you-add-and-remove-an-event-listener)
- [5. What's the difference between `innerHTML` and `textContent`?](#5-whats-the-difference-between-innerhtml-and-textcontent)

**🟡 Medium**
- [6. What is event bubbling and event capturing?](#6-what-is-event-bubbling-and-event-capturing)
- [7. What is event delegation, and why is it useful?](#7-what-is-event-delegation-and-why-is-it-useful)
- [8. What's the difference between `event.target` and `event.currentTarget`?](#8-whats-the-difference-between-eventtarget-and-eventcurrenttarget)
- [9. What's the difference between `preventDefault()` and `stopPropagation()`?](#9-whats-the-difference-between-preventdefault-and-stoppropagation)
- [10. What's the difference between the `load` and `DOMContentLoaded` events?](#10-whats-the-difference-between-the-load-and-domcontentloaded-events)

**🔴 Hard**
- [11. What's the difference between the `async` and `defer` script attributes?](#11-whats-the-difference-between-the-async-and-defer-script-attributes)
- [12. What are reflow and repaint, and how do you minimize them?](#12-what-are-reflow-and-repaint-and-how-do-you-minimize-them)
- [13. How does event delegation interact with dynamically-added elements?](#13-how-does-event-delegation-interact-with-dynamically-added-elements)
- [14. How would you implement a custom pub-sub event system without using the DOM?](#14-how-would-you-implement-a-custom-pub-sub-event-system-without-using-the-dom)

---

### 1. What's the difference between `window` and `document`? 🟢

- `window`: the global object representing the entire browser window/tab — holds globals, timers, `location`, `history`, and `document` itself.
- `document`: represents the loaded **HTML page** specifically — the entry point for querying and manipulating the DOM tree.

```js
window.document === document; // true — document is a property of window
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between `querySelector()` and `getElementById()`? 🟢

- `getElementById(id)`: fast, simple, looks up by ID only.
- `querySelector(cssSelector)`: more flexible — accepts any CSS selector (class, attribute, descendant combinators), but slightly slower since it must parse the selector.
- `querySelectorAll()` returns a static `NodeList` of **all** matches; `getElementsByClassName`/`TagName` return a **live** `HTMLCollection` that auto-updates as the DOM changes.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between an attribute and a property? 🟢

- **Attribute**: defined in the HTML markup itself (the initial value) — accessed via `getAttribute()`/`setAttribute()`.
- **Property**: the corresponding value on the live DOM **object** in JS — can drift from the attribute after user interaction or script changes.

```js
const input = document.querySelector('input');
// user types "hello" into the input
input.getAttribute('value'); // '' (original HTML attribute, unchanged)
input.value;                  // 'hello' (current live property)
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you add and remove an event listener? 🟢

```js
function handleClick() { console.log('clicked'); }

button.addEventListener('click', handleClick);
button.removeEventListener('click', handleClick); // must pass the same function reference
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why does `removeEventListener` silently fail if you pass an anonymous/inline function instead of a named reference?

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between `innerHTML` and `textContent`? 🟢

- `innerHTML`: parses the string as **HTML**, can inject elements — but is a common **XSS risk** if it includes unsanitized user input.
- `textContent`: treats the string as **plain text only**, automatically escaping any HTML — safer when you just need to display text.

```js
el.innerHTML = '<b>bold</b>';   // renders bold text
el.textContent = '<b>bold</b>'; // renders the literal string "<b>bold</b>"
```

[↑ Back to top](#table-of-contents)

---

### 6. What is event bubbling and event capturing? 🟡

- **Capturing** (rarely used): the event travels **down** from the `window`/root to the actual target element first.
- **Bubbling** (default): after reaching the target, the event travels **up** from the target through each ancestor.
- `addEventListener(type, handler, true)` listens during the capture phase; omitting it (or `false`) listens during the bubble phase.

[↑ Back to top](#table-of-contents)

---

### 7. What is event delegation, and why is it useful? 🟡

- Attach a **single** listener to a common parent, and use `event.target` inside it to figure out which child actually triggered the event — relying on bubbling.
- Avoids attaching (and managing) a separate listener on every individual child, and automatically works for children added **later**.

```js
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log('Clicked item:', e.target.textContent);
  }
});
```

> [!TIP]
> **Real-life example:** a todo list where items can be added/removed dynamically — one listener on the `<ul>` handles clicks for every `<li>`, present or future.

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `event.target` and `event.currentTarget`? 🟡

- `event.target`: the actual element that **triggered** the event (e.g. the specific `<li>` clicked).
- `event.currentTarget`: the element the listener is **attached to** (e.g. the parent `<ul>`, in a delegation setup) — stays constant for a given handler regardless of where the event originated.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `preventDefault()` and `stopPropagation()`? 🟡

- `preventDefault()`: stops the browser's **default behavior** for that event (e.g. a link navigating, a form submitting) — does **not** stop the event from bubbling/capturing further.
- `stopPropagation()`: stops the event from **continuing to travel** up (or down) to other listeners — does **not** prevent the default browser action.

```js
form.addEventListener('submit', (e) => {
  e.preventDefault(); // stop the page from reloading
});
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between the `load` and `DOMContentLoaded` events? 🟡

- `DOMContentLoaded`: fires once the HTML is fully parsed and the DOM tree is built — **doesn't** wait for images, stylesheets, or iframes to finish loading.
- `load`: fires only after **everything** (images, CSS, subframes, etc.) has fully loaded — comes later.

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between the `async` and `defer` script attributes? 🔴

- Without either: script downloads and **executes immediately**, blocking HTML parsing.
- `async`: downloads in parallel with parsing, but executes **as soon as it's ready** — may run before parsing finishes, and multiple `async` scripts can execute out of order.
- `defer`: downloads in parallel too, but execution is **deferred** until after parsing completes, and scripts run **in document order** — generally the safer default for scripts that depend on the DOM or each other.

[↑ Back to top](#table-of-contents)

---

### 12. What are reflow and repaint, and how do you minimize them? 🔴

- **Reflow** (layout): the browser recalculates element positions/sizes — triggered by changes affecting layout (resizing, adding/removing elements, changing `width`/`margin`).
- **Repaint**: the browser redraws pixels — triggered by visual-only changes (`color`, `background`, `visibility`) that don't affect layout. Reflow always triggers a repaint, but not vice versa.
- Minimize by: batching DOM changes (e.g. build off-DOM then insert once), using `transform`/`opacity` for animations (GPU-accelerated, skip layout entirely), and avoiding reading layout properties (`offsetHeight`, etc.) interleaved with writes (causes "layout thrashing").

[↑ Back to top](#table-of-contents)

---

### 13. How does event delegation interact with dynamically-added elements? 🔴

- Since the listener lives on a stable **ancestor** (not the individual children), it automatically works for elements added to the DOM **after** the listener was attached — no need to re-bind anything.
- This is one of delegation's biggest advantages over attaching listeners directly to each child, which would require re-attaching on every DOM update.

[↑ Back to top](#table-of-contents)

---

### 14. How would you implement a custom pub-sub event system without using the DOM? 🔴

```js
class EventEmitter {
  #listeners = new Map();

  on(event, callback) {
    if (!this.#listeners.has(event)) this.#listeners.set(event, []);
    this.#listeners.get(event).push(callback);
  }

  off(event, callback) {
    const callbacks = this.#listeners.get(event) || [];
    this.#listeners.set(event, callbacks.filter((cb) => cb !== callback));
  }

  emit(event, ...args) {
    (this.#listeners.get(event) || []).forEach((cb) => cb(...args));
  }
}

const bus = new EventEmitter();
bus.on('userLoggedIn', (user) => console.log(`${user} logged in`));
bus.emit('userLoggedIn', 'Vaibhav'); // "Vaibhav logged in"
```

> [!TIP]
> **Real-life example:** this is essentially how Node's built-in `EventEmitter` works, and the underlying pattern behind many state-management/event-bus libraries.

[↑ Back to top](#table-of-contents)

---
