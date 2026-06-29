# Performance Optimization

_Part of [React](README.md) interview notes._

> `React.memo`, `useMemo`, and `useCallback` mechanics are covered in [Components & JSX](components-and-jsx.md) and [Hooks](hooks.md). This file focuses on broader app-level performance strategies.

## Table of Contents

**🟢 Easy**
- [1. What is code-splitting, and why does it help performance?](#1-what-is-code-splitting-and-why-does-it-help-performance)
- [2. What is `React.lazy`, and how do you use it with `Suspense`?](#2-what-is-reactlazy-and-how-do-you-use-it-with-suspense)
- [3. What is the windowing/virtualization technique?](#3-what-is-the-windowingvirtualization-technique)

**🟡 Medium**
- [4. How do you prevent unnecessary re-renders in a React app?](#4-how-do-you-prevent-unnecessary-re-renders-in-a-react-app)
- [5. Why can inline arrow functions/objects in JSX hurt performance?](#5-why-can-inline-arrow-functionsobjects-in-jsx-hurt-performance)
- [6. How do you implement route-based code splitting?](#6-how-do-you-implement-route-based-code-splitting)
- [7. What is the React Profiler, and how do you use it?](#7-what-is-the-react-profiler-and-how-do-you-use-it)

**🔴 Hard**
- [8. What is a dynamic import, and how does it relate to code splitting?](#8-what-is-a-dynamic-import-and-how-does-it-relate-to-code-splitting)
- [9. What are loadable components, and how do they compare to `React.lazy`?](#9-what-are-loadable-components-and-how-do-they-compare-to-reactlazy)
- [10. How does concurrent rendering improve perceived performance?](#10-how-does-concurrent-rendering-improve-perceived-performance)
- [11. What's the cost of overusing `useMemo`/`useCallback`?](#11-whats-the-cost-of-overusing-usememousecallback)
- [12. How would you debug an unexpected/excessive re-render in a large app?](#12-how-would-you-debug-an-unexpectedexcessive-re-render-in-a-large-app)
- [13. How do you reduce overall JS bundle size in a React app?](#13-how-do-you-reduce-overall-js-bundle-size-in-a-react-app)
- [14. What is the impact of "prop drilling deeply nested objects" on performance, and how does it relate to memoization?](#14-what-is-the-impact-of-prop-drilling-deeply-nested-objects-on-performance-and-how-does-it-relate-to-memoization)

---

### 1. What is code-splitting, and why does it help performance? 🟢

- Breaking a single large JS bundle into smaller chunks that load **on demand**, instead of forcing the user to download the entire app's code upfront — reduces initial load time, especially for large apps.

[↑ Back to top](#table-of-contents)

---

### 2. What is `React.lazy`, and how do you use it with `Suspense`? 🟢

- `React.lazy(() => import(...))` defines a component that's loaded **only when first rendered**. `<Suspense>` wraps it to show a fallback (like a spinner) while the chunk is still downloading.

```jsx
const Settings = React.lazy(() => import('./Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Settings />
    </Suspense>
  );
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What is the windowing/virtualization technique? 🟢

- Rendering only the items currently **visible in the viewport** (plus a small buffer) for a long list, instead of all thousands of items at once — drastically reduces DOM node count and render cost. Libraries: `react-window`, `react-virtualized`.

> [!TIP]
> **Real-life example:** a chat app with 10,000 messages only renders the ~20 visible on screen at any time, recycling DOM nodes as the user scrolls.

[↑ Back to top](#table-of-contents)

---

### 4. How do you prevent unnecessary re-renders in a React app? 🟡

- Memoize components (`React.memo`) and the props passed to them (`useMemo`/`useCallback`) so references stay stable.
- Split large components into smaller ones, so a state change only re-renders the specific part that actually needs it, not a large shared parent.
- Avoid putting frequently-changing state high up the tree if only a small leaf component actually needs it.

[↑ Back to top](#table-of-contents)

---

### 5. Why can inline arrow functions/objects in JSX hurt performance? 🟡

- A new function/object literal is created on **every render**, even if its logic/values never actually change — this breaks `React.memo`'s shallow prop comparison on the child receiving it, causing the child to re-render every time regardless of memoization.

```jsx
// New function created every render — breaks memoization on Child
<MemoizedChild onClick={() => doSomething(id)} />

// Stable reference via useCallback — memoization actually works
const handleClick = useCallback(() => doSomething(id), [id]);
<MemoizedChild onClick={handleClick} />
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you implement route-based code splitting? 🟡

- Lazily load each route's component, so users only download the code for the page they're actually visiting.

```jsx
const Home = React.lazy(() => import('./pages/Home'));
const Profile = React.lazy(() => import('./pages/Profile'));

<Suspense fallback={<Spinner />}>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/profile" element={<Profile />} />
  </Routes>
</Suspense>
```

[↑ Back to top](#table-of-contents)

---

### 7. What is the React Profiler, and how do you use it? 🟡

- A React DevTools panel (and a `<Profiler>` component you can embed in code) that records render timings — shows which components rendered, how long each took, and why (which prop/state change triggered it) — the primary tool for actually measuring rather than guessing at performance issues.

[↑ Back to top](#table-of-contents)

---

### 8. What is a dynamic import, and how does it relate to code splitting? 🔴

- `import('./module')` (a function call, not the static `import` statement) loads a module asynchronously at runtime, returning a Promise. Bundlers (Webpack, Vite) treat each dynamic import as a **split point**, generating a separate chunk for it — this is the underlying mechanism `React.lazy` is built on.

[↑ Back to top](#table-of-contents)

---

### 9. What are loadable components, and how do they compare to `React.lazy`? 🔴

- `@loadable/component` is a third-party library offering code-splitting similar to `React.lazy`, but with extra capabilities `React.lazy` lacks out of the box: **SSR support** (React.lazy + Suspense for data fetching during SSR has historically been limited), preloading hints, and named (not just default) exports.

[↑ Back to top](#table-of-contents)

---

### 10. How does concurrent rendering improve perceived performance? 🔴

- Lets React **deprioritize** non-urgent rendering work (via `useTransition`/`useDeferredValue`, see [Hooks](hooks.md)) so urgent updates — like responding to a keystroke — aren't blocked behind a slower re-render (like re-filtering a large list). The total work isn't necessarily less, but the **app stays responsive**, which is what users actually perceive as "fast."

[↑ Back to top](#table-of-contents)

---

### 11. What's the cost of overusing `useMemo`/`useCallback`? 🔴

- Each memoization call still has overhead — comparing dependency arrays and storing cached values consumes memory and CPU cycles. For cheap computations or components that rarely re-render anyway, the memoization bookkeeping cost can outweigh the (negligible) cost of just recomputing.
- Overusing them also adds noise and an extra surface for dependency-array bugs (stale closures), without measurable benefit — best applied where profiling actually shows a problem, not by default everywhere.

[↑ Back to top](#table-of-contents)

---

### 12. How would you debug an unexpected/excessive re-render in a large app? 🔴

- Use the React DevTools Profiler (or the "highlight updates when components render" setting) to visually see which components re-rendered and why.
- Temporarily add `console.log` or `why-did-you-render` (a debugging library) to a suspect component to see which specific prop/state changed.
- Check for unstable references (new objects/functions/arrays created inline each render) being passed down, and context providers re-rendering all consumers unnecessarily.

[↑ Back to top](#table-of-contents)

---

### 13. How do you reduce overall JS bundle size in a React app? 🔴

- Code-split routes/heavy components, tree-shake unused code (use ES modules, avoid importing whole libraries for one function), analyze the bundle (`source-map-explorer`, `webpack-bundle-analyzer`) to find unexpectedly large dependencies, and replace heavy libraries with lighter alternatives where possible.

[↑ Back to top](#table-of-contents)

---

### 14. What is the impact of prop-drilling deeply nested objects on performance, and how does it relate to memoization? 🔴

- Passing a large object several levels deep means every intermediate component re-renders whenever **any** part of that object changes, even if a given intermediate component doesn't actually use the changed field — because object identity changes wholesale.
- Mitigated by splitting state more granularly (pass only the specific fields actually needed, not the whole object), or using `React.memo` + stable references at each level so an unrelated change doesn't cascade through components that don't care about it.

> [!IMPORTANT]
> **Follow-up questions:**
> - How does this interact with Context's "re-renders all consumers" behavior mentioned in [State Management](state-management.md)?

[↑ Back to top](#table-of-contents)

---
