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

- `window` is the **top-level global object** in a browser — it represents the entire browser tab/window itself. All global variables and functions you create automatically become properties of it, and it also holds browser-level features like timers (`setTimeout`), the current URL (`location`), navigation history (`history`), and the loaded page (`document`).
- `document` represents specifically the **loaded HTML page** inside that window — it's the entry point for reading and changing what's actually on the page (the DOM tree: elements, text, attributes). Anything to do with "find this element" or "change this element's content" goes through `document`.
- Think of `window` as the whole browser tab (including its address bar, tab controls, and timers) and `document` as just the webpage rendered inside it.

```js
window.document === document; // true — document is a property of window
```

[↑ Back to top](#table-of-contents)

---

### 2. What's the difference between `querySelector()` and `getElementById()`? 🟢

- `getElementById(id)` does exactly one thing: find the single element with a matching `id` attribute. Because the browser can look this up very directly (IDs are meant to be unique), it's fast and simple.
- `querySelector(cssSelector)` is much more flexible — you give it any valid **CSS selector** (a class like `.item`, an attribute like `[data-active]`, a combination like `ul li.active`, etc.) and it returns the first matching element. That flexibility costs a little speed, since the browser has to parse and evaluate the selector, but in practice this difference is rarely noticeable.
- One more distinction worth knowing: `querySelectorAll()` returns a **static** `NodeList` — a fixed snapshot of matches taken at the time you called it, which won't update if the DOM changes afterward. In contrast, `getElementsByClassName()`/`getElementsByTagName()` return a **live** `HTMLCollection`, which automatically updates in real time as matching elements are added or removed from the page.

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between an attribute and a property? 🟢

- An **attribute** is what's written in the HTML markup itself — it represents the element's **initial** state as parsed from the page source. You read/write attributes with `getAttribute()`/`setAttribute()`.
- A **property** is a field on the **live JavaScript object** the browser creates to represent that element in memory (every DOM element in JS is an object with properties, like any other object). This reflects the element's **current** state.
- For many things, the attribute and property start out in sync and stay that way. But for things that change through user interaction — like typing into an `<input>` — the **property** updates to reflect what's currently true, while the **attribute** stays frozen at its original HTML value unless you explicitly change it with `setAttribute()`. This is exactly why the example below shows two different values for the same input.

```js
const input = document.querySelector('input');
// user types "hello" into the input
input.getAttribute('value'); // '' (original HTML attribute, unchanged)
input.value;                  // 'hello' (current live property)
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you add and remove an event listener? 🟢

- `addEventListener(type, handler)` registers a function to run whenever the given event type (like `'click'`) happens on that element. You can attach multiple listeners for the same event without them overwriting each other.
- `removeEventListener(type, handler)` un-registers a previously added listener — but only works if you pass **the exact same function reference** that was originally used with `addEventListener`. This is a common gotcha: if you pass a new anonymous function (even one that looks identical), the browser has no way to know it's "the same" listener, so nothing gets removed.
- That's why, if you intend to remove a listener later, you should always store the handler in a named variable/function (as below) rather than writing it inline.

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

- `innerHTML` takes the string you assign to it and **parses it as HTML**, creating real elements from any tags it finds. This is powerful (you can inject entire chunks of markup at once), but dangerous: if any part of that string comes from user input and isn't sanitized first, an attacker could inject a `<script>` tag or malicious markup — this is a classic **XSS (Cross-Site Scripting)** vulnerability.
- `textContent` treats whatever you assign to it as **plain text, always** — even if the string contains things that look like HTML tags, they're displayed literally as text rather than being turned into actual elements. This makes it the safer choice whenever you just want to show text and have no need to insert markup.
- Simple rule: if you're just displaying text, use `textContent`. Only use `innerHTML` when you specifically need to insert markup, and never with untrusted/unsanitized input.

```js
el.innerHTML = '<b>bold</b>';   // renders bold text
el.textContent = '<b>bold</b>'; // renders the literal string "<b>bold</b>"
```

[↑ Back to top](#table-of-contents)

---

### 6. What is event bubbling and event capturing? 🟡

- When you click an element, that click doesn't just happen at that one element — the event actually travels through the DOM tree in stages, and both capturing and bubbling describe two different "directions" of that journey.
- **Capturing** (the first stage, rarely used directly): the event starts at the outermost ancestor (`window`) and travels **downward** through each nested parent element until it reaches the actual element that was clicked (the target).
- **Bubbling** (the second stage, and the default behavior most code relies on): after reaching the target, the event then travels back **upward**, from the target through each ancestor element, all the way back out to `window`.
- By default, `addEventListener(type, handler)` listens during the bubbling phase. Passing `true` as the third argument — `addEventListener(type, handler, true)` — instead makes that listener fire during the earlier capturing phase.

[↑ Back to top](#table-of-contents)

---

### 7. What is event delegation, and why is it useful? 🟡

- **Event delegation** means attaching just **one** event listener to a common parent element, instead of attaching a separate listener to every individual child. It works because of **bubbling** (Q6): a click on a child bubbles up to the parent, so the parent's single listener still gets notified.
- Inside that one listener, you check `event.target` (the actual element that was clicked, see Q8) to figure out which specific child was responsible, and respond accordingly.
- Why it's useful:
  - Far less memory/setup overhead than managing potentially hundreds of individual listeners.
  - It automatically works for children added to the page **later**, after the listener was set up — since the listener lives on the parent, not the child, there's nothing new to wire up when new children appear.

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

- `event.target` is the **specific, innermost element that originally triggered** the event — e.g. if you click on a `<li>` inside a `<ul>`, `target` is that exact `<li>`, no matter which ancestor's listener is currently running.
- `event.currentTarget` is **whichever element the currently-executing listener was actually attached to** — e.g. in an event delegation setup where the listener lives on the `<ul>`, `currentTarget` is always the `<ul>`, even though the click physically happened on one of its `<li>` children.
- In other words: `target` answers "what did the user actually interact with?" while `currentTarget` answers "which element's listener is running right now?" They're often different in a delegation setup, but identical when a listener is attached directly to the element it's meant to handle.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `preventDefault()` and `stopPropagation()`? 🟡

- These two methods control two completely independent things, which is why they're often confused:
  - `preventDefault()` stops the browser's **built-in default action** for that event — e.g. stopping a clicked `<a>` link from navigating, or a form's `submit` event from actually reloading the page. It has **no effect** on whether the event keeps bubbling/capturing to other listeners — those still run normally.
  - `stopPropagation()` stops the event from **continuing to travel** through the DOM to other elements' listeners (whether bubbling up or capturing down). It has **no effect** on the browser's default behavior — that still happens unless you also call `preventDefault()`.
- Rule of thumb: use `preventDefault()` when you want to override what the browser would normally do; use `stopPropagation()` when you want to stop other listeners (e.g. from a parent in a delegation setup) from also reacting to this same event. You can call both together if you need both effects.

```js
form.addEventListener('submit', (e) => {
  e.preventDefault(); // stop the page from reloading
});
```

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between the `load` and `DOMContentLoaded` events? 🟡

- `DOMContentLoaded` fires as soon as the browser has finished reading and parsing the HTML and has built the full DOM tree (the structure of elements) in memory. Crucially, it **doesn't wait** for slower resources like images, stylesheets, or embedded iframes to actually finish downloading — so it fires relatively early, and is usually the right event to wait for if your script just needs to find/manipulate elements on the page.
- `load` fires later, only once **everything** on the page has fully finished loading — every image is downloaded and rendered, every stylesheet applied, every iframe loaded, etc. Use this if your code genuinely depends on something like an image's final dimensions.
- Simple rule: if you just need the DOM elements to exist so you can query/modify them, wait for `DOMContentLoaded` (it's faster). Only wait for `load` if you need external resources to be fully ready too.

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between the `async` and `defer` script attributes? 🔴

- These attributes control how `<script src="...">` tags interact with the browser's HTML parsing, and both exist to avoid the default behavior being slow.
- **Without either attribute**: when the browser's HTML parser reaches a `<script>` tag, it **stops parsing the rest of the page**, downloads the script (if external), and executes it immediately, before resuming parsing. This can visibly slow down page rendering if the script is large or the network is slow.
- **`async`**: the script is downloaded **in parallel** while parsing continues (so parsing isn't blocked while waiting for the download) — but as soon as the download finishes, parsing pauses and the script **executes immediately**, whenever that happens to be. This means: it might run before the HTML is fully parsed, and if you have multiple `async` scripts, they can finish downloading (and thus execute) in a different order than they appear in the HTML — whichever finishes downloading first, runs first.
- **`defer`**: also downloads in parallel without blocking parsing, but execution is **postponed until after the entire HTML document has been parsed**. Multiple `defer` scripts are guaranteed to execute **in the order they appear** in the document, regardless of download speed.
- Practical takeaway: `defer` is generally the safer default for scripts that need the full DOM to exist, or that depend on each other running in a specific order. `async` is better suited to independent scripts (like analytics) that don't touch the DOM and don't care about order.

[↑ Back to top](#table-of-contents)

---

### 12. What are reflow and repaint, and how do you minimize them? 🔴

- After the browser builds the DOM, it has to figure out how to actually draw it on screen. This happens in stages, and two of the most performance-sensitive stages are reflow and repaint.
- **Reflow** (also called "layout"): the browser recalculates the **position and size** of elements on the page. This is triggered by anything that could change the page's geometry — resizing the window, adding/removing elements, or changing layout-affecting CSS like `width`, `height`, or `margin`. Reflow is expensive because changing one element's size/position can ripple outward and force recalculating other elements too.
- **Repaint**: the browser redraws the actual pixels for an element, without needing to recalculate any layout — triggered by purely visual changes like `color`, `background-color`, or `visibility`, which don't affect where anything sits on the page.
- A reflow always causes a repaint afterward (since positions changed, pixels must be redrawn), but a repaint can happen on its own without a reflow (if only appearance changed, not layout) — so reflow is generally the more expensive of the two.
- Ways to minimize them:
  - **Batch DOM changes**: instead of making many small changes to the live page one at a time (each potentially causing its own reflow), build the new structure off-screen (e.g. in a `DocumentFragment`) and insert it all at once.
  - **Animate with `transform`/`opacity`** instead of properties like `top`/`left`/`width` — these two properties can be handled by the GPU and skip the layout step entirely, making animations much smoother.
  - **Avoid "layout thrashing"**: don't interleave reading a layout property (like `element.offsetHeight`) with writing one in a loop — reading forces the browser to immediately finish any pending layout work to give you an accurate answer, so alternating reads and writes forces many reflows back-to-back instead of one batched reflow.

[↑ Back to top](#table-of-contents)

---

### 13. How does event delegation interact with dynamically-added elements? 🔴

- Recall from Q7 that event delegation works by placing **one listener on a stable ancestor element**, and relying on event bubbling to detect which child was actually interacted with (via `event.target`).
- Because the listener never lives on the children themselves, it doesn't matter *when* those children were added to the page — even elements created and inserted into the DOM **after** the listener was originally set up will still trigger it, since their events still bubble up to the same ancestor, the same as always. There's no need to "notice" the new elements or re-attach anything.
- This is one of delegation's biggest practical advantages: without it, you'd need to manually attach a new listener every single time a new child element is added (e.g. every time a new row is added to a table), and remember to remove it when that element is deleted — a maintenance burden and a common source of memory leaks. Delegation sidesteps all of that by design.

[↑ Back to top](#table-of-contents)

---

### 14. How would you implement a custom pub-sub event system without using the DOM? 🔴

- **Pub-sub** (publish-subscribe) is a pattern where different parts of your code can communicate without directly referencing each other: one part "publishes" (emits) a named event with some data, and any number of other parts that have "subscribed" to that same event name get notified and run their callback.
- To build this ourselves (without relying on DOM elements' built-in event system), we just need a small class that keeps track of, per event name, a list of callback functions to call:
  - `on(event, callback)`: registers a callback to run whenever `event` is emitted — stored in a `Map` keyed by event name, with an array of callbacks as the value.
  - `off(event, callback)`: removes a specific callback so it stops being called for that event.
  - `emit(event, ...args)`: looks up all callbacks registered for that event name and calls each one, forwarding along any extra arguments.
- This gives you the same core "subscribe to a named event, get notified when it happens" behavior the DOM provides via `addEventListener`, but usable anywhere in your code — not just on DOM elements.

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
