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

- Debouncing delays running a function until a certain amount of time has passed **since the last time it was called**. Every new call resets the wait — so if the function keeps getting called rapidly (e.g. on every keystroke), it never actually runs until things go quiet for the full delay period.
- Think of it like an elevator that waits a few extra seconds before closing its doors each time someone new steps in — as long as people keep arriving, it keeps waiting.
- Common uses: search-as-you-type (don't call the API on every keystroke — wait until the user actually pauses), window resize handlers (don't recalculate a layout 100 times a second while the user is dragging the window edge).

> [!TIP]
> **Real-life example:** a search box that only fires the API request 300ms after the user stops typing, instead of on every keystroke.

[↑ Back to top](#table-of-contents)

---

### 2. What is throttling, and how is it different from debouncing? 🟢

- Throttling guarantees a function runs **at most once** per fixed time interval (say, once every 200ms), no matter how many times it gets triggered in between. Unlike debounce, it doesn't care whether calls have "stopped" — it just enforces a steady rate.
- The key difference from debounce: debounce waits for a **pause** before running once at the end; throttle runs **periodically the whole time**, at a controlled rate, even while events keep firing continuously.
- Common uses: scroll event handlers (you don't need to react to every single pixel scrolled), tracking mouse movement, or rate-limiting a button so it can't be spam-clicked faster than once per second.

```js
// Rough mental model:
// debounce: -----x-----x--x---------[fires once here, after the pause]
// throttle: -----x-----x--x--[fires]--x--x--[fires]--x--[fires]  (runs periodically, not just at the end)
```

[↑ Back to top](#table-of-contents)

---

### 3. What causes memory leaks in JavaScript? 🟢

- A memory leak happens when something keeps a reference to a piece of memory **alive longer than it's actually needed**, which stops the garbage collector (Q4) from ever being able to free it — over time, this unused memory piles up and can slow down or crash the page.
- Common causes: event listeners or timers that are set up but never removed/cleared, global variables that keep accumulating data (e.g. pushing to an array that's never cleared), closures that unintentionally hold on to large objects even after they're no longer needed (see [Closures & Scope](closures-and-scope.md)), and DOM elements that were removed from the page but are still referenced by a JS variable somewhere ("detached" nodes — see Q7 for a deeper look).

[↑ Back to top](#table-of-contents)

---

### 4. What is garbage collection? 🟢

- Garbage collection (GC) is the JS engine's automatic background process for freeing up memory. Unlike languages like C, where you must manually allocate and `free()` memory yourself, JS handles this for you — it periodically looks for objects that are no longer needed and reclaims their memory.
- "No longer needed" specifically means no longer **reachable** — there's no path of references leading to that object starting from a "root" (things like global variables, or variables currently in scope in a running function). If nothing points to an object anymore, directly or indirectly, it's safe to remove.
- An important nuance: "reachable" is not the same as "still being used." An object stays alive in memory as long as *something*, somewhere, still holds a reference to it — even if your code never actually reads or uses that object again. This is exactly why forgotten references (Q3) cause leaks: the object is technically still "reachable," so GC correctly leaves it alone, even though it's effectively dead weight.

[↑ Back to top](#table-of-contents)

---

### 5. How would you implement a `debounce` function from scratch? 🟡

- `debounce(fn, delay)` returns a new "wrapped" function. Every time the wrapped function is called, it cancels any previously scheduled call and schedules a fresh one `delay` milliseconds in the future — so `fn` only actually runs once things have gone quiet for the full `delay`.
- The `timeoutId` variable is remembered between calls thanks to a **closure** (the returned function keeps access to variables from the outer `debounce` scope even after `debounce` itself has finished running — see [Closures & Scope](closures-and-scope.md)).
- `fn.apply(this, args)` makes sure the wrapped call preserves both the original `this` context and all the arguments passed in, so it behaves like a normal, direct call to `fn`.

```js
function debounce(fn, delay) {
  let timeoutId; // remembered across calls via closure
  return function (...args) {
    clearTimeout(timeoutId);                          // cancel the previous pending call
    timeoutId = setTimeout(() => fn.apply(this, args), delay); // schedule a new one
  };
}

const search = debounce((query) => console.log('Searching for', query), 300);
input.addEventListener('input', (e) => search(e.target.value));
// typing "hi" fast only logs once, 300ms after the last keystroke — not once per letter
```

[↑ Back to top](#table-of-contents)

---

### 6. How would you implement a `throttle` function from scratch? 🟡

- `throttle(fn, interval)` returns a wrapped function that remembers the timestamp of the last time `fn` actually ran (`lastCall`, kept alive between calls via closure — see Q5).
- Every time the wrapped function is called, it checks how much time has passed since the last run. If at least `interval` milliseconds have gone by, it runs `fn` and updates `lastCall`; otherwise, the call is simply ignored (dropped), since we're still "cooling down" from the last run.
- This is what gives throttle its "at most once per interval" guarantee, in contrast to debounce, which only fires after a full pause (Q2).

```js
function throttle(fn, interval) {
  let lastCall = 0; // timestamp of the last time fn actually ran
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= interval) { // enough time has passed — allowed to run
      lastCall = now;
      fn.apply(this, args);
    }
    // else: still within the cooldown window — call is dropped
  };
}

const onScroll = throttle(() => console.log('scroll handled'), 200);
window.addEventListener('scroll', onScroll);
// even if scroll fires 60 times/sec, "scroll handled" logs at most every 200ms
```

[↑ Back to top](#table-of-contents)

---

### 7. What are the most common real-world causes of memory leaks? 🟡

Building on Q3's general idea (something keeps a reference alive longer than needed), here are the specific patterns that trip people up in real apps:

- **Detached DOM nodes**: an element is removed from the visible page (e.g. via `.remove()` or re-rendering), but some JS variable still holds a reference to it. The browser can't reclaim that memory because, from the garbage collector's point of view, the node is still "reachable" — even though it's no longer on screen.
- **Forgotten timers/intervals**: a `setInterval` that's never `clearInterval`-ed keeps running forever, and its callback function (along with anything it closes over via closure — e.g. `largeData` below) stays alive in memory the entire time, even if the feature that started it is long gone.
- **Event listeners never removed**: especially common in single-page apps, where UI components get created and destroyed repeatedly. If a listener is attached to something long-lived and shared (like `window` or `document`) and never removed when the component goes away, both the listener *and* everything it references stay in memory.
- **Growing caches/global arrays**: pushing data into a global array, object, or Map with no limit or cleanup strategy means it just keeps growing for the lifetime of the page, holding onto every item ever added.

```js
// Leak: interval keeps running and holding `largeData` in memory
// even after whatever started polling is gone, because it's never cleared
function startPolling(largeData) {
  setInterval(() => console.log(largeData.length), 1000); // never cleared — runs and holds largeData forever
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why specifically are detached DOM nodes hard to spot in a memory profiler?
> - How would you clean up properly in a React component (`useEffect` cleanup function) or similar lifecycle hook?

[↑ Back to top](#table-of-contents)

---

### 8. How do you measure and profile JavaScript performance? 🟡

- **Browser DevTools** — the go-to visual tools:
  - **Performance tab**: records a timeline of what the CPU is doing over time (script execution, rendering, painting), so you can spot exactly which function or frame is slow.
  - **Memory tab**: lets you take heap snapshots (a picture of everything currently in memory) and compare them over time to spot leaks (see Q11), or record allocation timelines to see what's being created and never freed.
  - **Lighthouse**: an automated audit tool built into DevTools that scores overall page performance, accessibility, and best practices, and suggests concrete fixes.
- **Performance API** — for precise, code-level timing that doesn't require DevTools to be open: `performance.now()` gives a high-precision timestamp, and `performance.mark()`/`performance.measure()` let you label points in time and measure the duration between them, directly from your own code (useful for logging real-user performance metrics in production).

```js
performance.mark('start');          // record a timestamp labeled "start"
expensiveOperation();
performance.mark('end');            // record a timestamp labeled "end"
performance.measure('operation', 'start', 'end'); // compute the duration between the two marks
console.log(performance.getEntriesByName('operation')[0].duration); // e.g. 42.3 (milliseconds)
```

[↑ Back to top](#table-of-contents)

---

### 9. What is lazy loading, and how does it improve performance? 🟡

- Lazy loading means deferring the loading of a resource — an image, a UI component, a whole page/route, a JS module — until the moment it's actually needed, instead of loading everything upfront when the page first opens.
- This directly improves performance because the initial page load only has to fetch and process what's needed to show the very first screen. Everything else loads later, in the background or on demand, so the user sees something useful sooner.
- Common examples: `<img loading="lazy">` tells the browser to skip loading an image until it's about to scroll into view; dynamic `import()` lets you split your JS bundle so a route's code only loads when the user actually navigates to it, instead of being bundled into the initial download (see [ES6+ Features](es6-plus-features.md)).

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between mark-and-sweep and reference-counting garbage collection? 🔴

These are two different strategies an engine can use to decide what memory is safe to free (see Q4 for what garbage collection is):

- **Reference counting**: every object keeps a count of how many references point to it. Each time a new reference is made, the count goes up; each time one is removed, it goes down. Once the count hits zero, nobody can reach the object anymore, so it's safe to free. The catch: it completely fails on **circular references** — if object A references object B and B references A, but nothing outside the pair references either of them, their counts never drop to zero, so they leak forever even though they're genuinely unreachable from the rest of the program.
- **Mark-and-sweep** (what all modern JS engines actually use): instead of counting references, it periodically starts from a set of "roots" (global variables, currently-running functions' local variables) and walks every reachable reference outward, "marking" each object it finds along the way. Once it's explored everything reachable, anything left unmarked is, by definition, unreachable from any root — those objects get "swept" (freed). This correctly handles circular references too, since a cycle that's isolated from the roots simply never gets marked, and gets cleaned up in the sweep.

[↑ Back to top](#table-of-contents)

---

### 11. How would you detect a memory leak in a running application? 🔴

- The general approach: perform the *same* user action repeatedly (e.g. open and close the same modal 10 times, or navigate to the same page and back 10 times) — an action that should leave memory usage roughly unchanged once it's done, since nothing should permanently accumulate from one modal open/close cycle. Then check whether memory actually returns to roughly where it started, or whether it keeps climbing higher after every repetition. Steady climbing that never comes back down is the signature of a leak.
- Concretely, in Chrome DevTools' **Memory tab**: take a heap snapshot (a full picture of everything currently allocated), perform the action several times, then take another snapshot. Use the comparison view between the two snapshots to see which object types grew in count when they shouldn't have (e.g. 50 extra detached DOM nodes after opening/closing a modal 10 times is a red flag).
- Once you've spotted what's growing, trace its **retainers** — the chain of references that's keeping each leaked object reachable (shown in DevTools as a tree). Following that chain back usually leads you straight to the offending code: an event listener that was never removed, a timer that was never cleared, or a variable holding onto stale data.

[↑ Back to top](#table-of-contents)

---

### 12. How do V8 optimizations like hidden classes and inline caching affect real-world performance decisions? 🔴

V8 (the JS engine behind Chrome and Node.js) applies clever internal tricks to make property access fast — but these tricks only work if your code follows a predictable, consistent pattern. Here's how they work and why that matters:

- **Hidden classes**: JS objects are dynamic (you can add/remove properties anytime), which normally would make property lookups slow, since the engine can't assume a fixed memory layout the way it can for a fixed `struct` in a language like C. To get around this, V8 secretly tracks the "shape" of an object — which properties it has, and in what order they were added — and assigns objects with the *same* shape to a shared internal "hidden class." This lets V8 optimize property access the way it would for a fixed, known layout. But if two objects that are supposed to represent "the same kind of thing" end up with properties added in a different order, V8 has to create a *different* hidden class for each, silently losing the optimization.
- **Inline caching**: at a specific spot in the code where a property is accessed (e.g. `obj.x` inside a loop), V8 remembers what hidden class it saw on the previous call and caches the fast path for accessing that property. If the object at that same call site is always the same hidden class, the cached fast path keeps getting reused — very fast. But if that call site keeps seeing objects of many different hidden classes (called going "megamorphic"), the cache stops being useful and V8 falls back to a slower, general-purpose lookup every time.
- **Practical takeaway**: always initialize all of an object's properties inside the constructor, in the same order, every time — rather than conditionally adding properties later (e.g. only setting `obj.extra = 1` sometimes). Keeping object shapes consistent across instances of "the same kind of thing" is what lets V8's hidden classes and inline caching actually kick in and speed up your code.

```js
// Inconsistent shapes — hurts optimization:
function makePoint(x, y) {
  const p = { x };
  if (y) p.y = y; // sometimes has y, sometimes doesn't — different hidden classes result
  return p;
}

// Consistent shape — V8-friendly:
function makePoint(x, y) {
  return { x, y: y ?? 0 }; // always the same properties, same order, same hidden class
}
```

[↑ Back to top](#table-of-contents)

---
