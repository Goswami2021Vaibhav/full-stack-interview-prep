# Performance & Memory

_Part of [JavaScript](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is debouncing, and when would you use it?](#1-what-is-debouncing-and-when-would-you-use-it)
- [2. What is throttling, and how is it different from debouncing?](#2-what-is-throttling-and-how-is-it-different-from-debouncing)
- [3. What causes memory leaks in JavaScript?](#3-what-causes-memory-leaks-in-javascript)
- [4. What is garbage collection?](#4-what-is-garbage-collection)

**🟡 Medium**
- [5. How would you implement a `debounce` function from scratch?](#5-how-would-you-implement-a-debounce-function-from-scratch)
- [6. How would you implement a `throttle` function from scratch?](#6-how-would-you-implement-a-throttle-function-from-scratch)
- [7. What are the most common real-world causes of memory leaks?](#7-what-are-the-most-common-real-world-causes-of-memory-leaks)
- [8. How do you measure and profile JavaScript performance?](#8-how-do-you-measure-and-profile-javascript-performance)
- [9. What is lazy loading, and how does it improve performance?](#9-what-is-lazy-loading-and-how-does-it-improve-performance)

**🔴 Hard**
- [10. What's the difference between mark-and-sweep and reference-counting garbage collection?](#10-whats-the-difference-between-mark-and-sweep-and-reference-counting-garbage-collection)
- [11. How would you detect a memory leak in a running application?](#11-how-would-you-detect-a-memory-leak-in-a-running-application)
- [12. How do V8 optimizations like hidden classes and inline caching affect real-world performance decisions?](#12-how-do-v8-optimizations-like-hidden-classes-and-inline-caching-affect-real-world-performance-decisions)

---

### 1. What is debouncing, and when would you use it? 🟢

- Delays running a function until a certain amount of time has passed **since the last time it was called** — if called again before the delay elapses, the timer resets.
- Used for: search-as-you-type (wait until the user pauses typing before firing the API call), resize handlers.

> [!TIP]
> **Real-life example:** a search box that only fires the API request 300ms after the user stops typing, instead of on every keystroke.

[↑ Back to top](#table-of-contents)

---

### 2. What is throttling, and how is it different from debouncing? 🟢

- Throttling guarantees a function runs **at most once** per fixed time interval, no matter how many times it's triggered — unlike debounce, it doesn't wait for calls to stop.
- Used for: scroll handlers, mouse-move tracking, button-click rate limiting — anything that should run periodically, not just once at the end.

[↑ Back to top](#table-of-contents)

---

### 3. What causes memory leaks in JavaScript? 🟢

- Anything that keeps a reference alive longer than needed, preventing garbage collection: forgotten event listeners/timers, global variables that accumulate data, closures unintentionally holding large objects (see [Closures & Scope](closures-and-scope.md)), and detached DOM nodes still referenced from JS.

[↑ Back to top](#table-of-contents)

---

### 4. What is garbage collection? 🟢

- The JS engine's automatic process of freeing memory used by objects that are no longer **reachable** from any root reference (global object, currently-executing functions' variables) — you never manually `free()` memory in JS.
- "Reachable" doesn't mean "currently unused" — an object stays alive as long as *something* still references it, even if your code never accesses it again.

[↑ Back to top](#table-of-contents)

---

### 5. How would you implement a `debounce` function from scratch? 🟡

```js
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce((query) => console.log('Searching for', query), 300);
input.addEventListener('input', (e) => search(e.target.value));
```

[↑ Back to top](#table-of-contents)

---

### 6. How would you implement a `throttle` function from scratch? 🟡

```js
function throttle(fn, interval) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= interval) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}

const onScroll = throttle(() => console.log('scroll handled'), 200);
window.addEventListener('scroll', onScroll);
```

[↑ Back to top](#table-of-contents)

---

### 7. What are the most common real-world causes of memory leaks? 🟡

- **Detached DOM nodes**: removed from the page but still referenced by a JS variable, so they can't be collected.
- **Forgotten timers/intervals**: `setInterval` that's never cleared keeps its callback (and anything it closes over) alive forever.
- **Event listeners never removed**: especially in SPAs where components mount/unmount repeatedly but listeners on shared/global targets (like `window`) are never cleaned up.
- **Growing caches/global arrays**: data pushed into a global structure with no eviction strategy.

```js
// Leak: interval keeps running and holding `largeData` even after the component is gone
function startPolling(largeData) {
  setInterval(() => console.log(largeData.length), 1000); // never cleared
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why specifically are detached DOM nodes hard to spot in a memory profiler?
> - How would you clean up properly in a React component (`useEffect` cleanup function) or similar lifecycle hook?

[↑ Back to top](#table-of-contents)

---

### 8. How do you measure and profile JavaScript performance? 🟡

- **Browser DevTools**: Performance tab (records CPU/paint/script timeline), Memory tab (heap snapshots, allocation timelines to spot leaks), Lighthouse (overall page audits).
- **Performance API** (`performance.now()`, `performance.mark()`/`measure()`): precise, code-level timing without DevTools attached.

```js
performance.mark('start');
expensiveOperation();
performance.mark('end');
performance.measure('operation', 'start', 'end');
console.log(performance.getEntriesByName('operation')[0].duration);
```

[↑ Back to top](#table-of-contents)

---

### 9. What is lazy loading, and how does it improve performance? 🟡

- Deferring the loading of resources (images, components, routes, modules) until they're actually needed, rather than upfront.
- Reduces initial bundle size and load time — e.g. `<img loading="lazy">` for offscreen images, or dynamic `import()` for route-level code-splitting (see [ES6+ Features](es6-plus-features.md)).

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between mark-and-sweep and reference-counting garbage collection? 🔴

- **Reference counting**: tracks how many references point to an object; collects it once the count hits zero. Fails on **circular references** (two objects referencing each other but unreachable from anywhere else never get collected).
- **Mark-and-sweep** (what modern JS engines use): starting from root references, "marks" everything reachable; anything left unmarked is "swept" (collected) — correctly handles circular references, since unreachable cycles get cleaned up too.

[↑ Back to top](#table-of-contents)

---

### 11. How would you detect a memory leak in a running application? 🔴

- Take repeated **heap snapshots** in DevTools (Memory tab) while performing the same user action multiple times (e.g. opening/closing a modal 10 times) — if memory keeps climbing and never returns to baseline, that's a leak.
- Use the snapshot comparison view to see which object types are growing unexpectedly between snapshots, and trace their **retainers** (what's still holding a reference to them) to find the leaking code.

[↑ Back to top](#table-of-contents)

---

### 12. How do V8 optimizations like hidden classes and inline caching affect real-world performance decisions? 🔴

- **Hidden classes**: objects with the same shape (same properties, added in the same order) share an internal hidden class, letting V8 optimize property access. Adding properties in inconsistent order across similar objects forces V8 to create extra hidden classes, hurting performance.
- **Inline caching**: V8 caches the result of a property lookup at a specific call site, assuming the object shape stays the same next time — breaks down (goes "megamorphic") if the same code path sees many different object shapes.
- Practical takeaway: initialize all of an object's properties in the constructor (in the same order) rather than adding them conditionally later, to keep hidden classes consistent and let V8's optimizations actually kick in.

[↑ Back to top](#table-of-contents)

---
