# Hooks

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are Hooks?](#1-what-are-hooks)
- [2. What rules must Hooks follow?](#2-what-rules-must-hooks-follow)
- [3. How does `useState` work?](#3-how-does-usestate-work)
- [4. How does `useEffect` work?](#4-how-does-useeffect-work)
- [5. How do you fetch data with Hooks?](#5-how-do-you-fetch-data-with-hooks)

**🟡 Medium**
- [6. What's the difference between `useEffect` and `useLayoutEffect`?](#6-whats-the-difference-between-useeffect-and-uselayouteffect)
- [7. How do you prevent infinite loops with `useEffect`'s dependency array?](#7-how-do-you-prevent-infinite-loops-with-useeffects-dependency-array)
- [8. What is `useRef`, and what are its common use cases?](#8-what-is-useref-and-what-are-its-common-use-cases)
- [9. What's the difference between `useState` and `useRef`?](#9-whats-the-difference-between-usestate-and-useref)
- [10. How does `useContext` work?](#10-how-does-usecontext-work)
- [11. What is `useReducer`, and when would you prefer it over `useState`?](#11-what-is-usereducer-and-when-would-you-prefer-it-over-usestate)
- [12. What's the difference between `useMemo` and `useCallback`?](#12-whats-the-difference-between-usememo-and-usecallback)
- [13. Does `useMemo` prevent re-rendering of child components?](#13-does-usememo-prevent-re-rendering-of-child-components)
- [14. What are custom Hooks, and how do you build one?](#14-what-are-custom-hooks-and-how-do-you-build-one)
- [15. Can Hooks be used in class components?](#15-can-hooks-be-used-in-class-components)

**🔴 Hard**
- [16. How does `useState` work internally under the hood?](#16-how-does-usestate-work-internally-under-the-hood)
- [17. Is the `dispatch` function from `useReducer` synchronous?](#17-is-the-dispatch-function-from-usereducer-synchronous)
- [18. What is `useImperativeHandle`, and when would you use it?](#18-what-is-useimperativehandle-and-when-would-you-use-it)
- [19. What is `useTransition`, and how is it different from `useDeferredValue`?](#19-what-is-usetransition-and-how-is-it-different-from-usedeferredvalue)
- [20. What is `useSyncExternalStore` used for?](#20-what-is-usesyncexternalstore-used-for)
- [21. What is `useInsertionEffect` used for?](#21-what-is-useinsertioneffect-used-for)
- [22. What is `useId`, and when should you use it?](#22-what-is-useid-and-when-should-you-use-it)

---

### 1. What are Hooks? 🟢

- Functions (all prefixed `use...`) that let function components "hook into" React features — state, lifecycle-like effects, context, refs — without needing a class component.

```jsx
function Counter() {
  const [count, setCount] = useState(0); // hooking into state
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What rules must Hooks follow? 🟢

1. **Only call Hooks at the top level** — never inside loops, conditions, or nested functions.
2. **Only call Hooks from React function components or custom Hooks** — not regular JS functions.
- These rules exist because React tracks Hooks **by call order** between renders — breaking the order desyncs which piece of state belongs to which `useState` call. The `eslint-plugin-react-hooks` package enforces these automatically.

[↑ Back to top](#table-of-contents)

---

### 3. How does `useState` work? 🟢

- Returns a `[value, setter]` pair. Calling the setter schedules a re-render with the new value — the state persists across renders, tied to that specific Hook call's position.

```jsx
const [name, setName] = useState('Vaibhav');
setName('New name'); // triggers re-render with updated value
```

[↑ Back to top](#table-of-contents)

---

### 4. How does `useEffect` work? 🟢

- Runs a side-effect function **after** the component renders/commits to the DOM. Re-runs whenever any value in its dependency array changes; runs once (on mount) with an empty array `[]`; runs after every render with no array at all.

```jsx
useEffect(() => {
  console.log('Ran after every render where `count` changed');
}, [count]);
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you fetch data with Hooks? 🟢

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => { if (!cancelled) setUser(data); });
    return () => { cancelled = true; }; // cleanup: avoid setting state after unmount
  }, [userId]);

  return user ? <p>{user.name}</p> : <p>Loading...</p>;
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What's the difference between `useEffect` and `useLayoutEffect`? 🟡

- `useEffect`: runs **asynchronously**, after the browser has painted — won't block visual updates.
- `useLayoutEffect`: runs **synchronously**, right after DOM mutations but **before** the browser paints — use it only when you need to measure/adjust the DOM (e.g. element size) before the user sees a flash of the wrong layout.
- Overusing `useLayoutEffect` for non-layout work hurts performance, since it blocks painting.

[↑ Back to top](#table-of-contents)

---

### 7. How do you prevent infinite loops with `useEffect`'s dependency array? 🟡

- An infinite loop usually happens when the effect itself **updates a value that's also in its dependency array**, on every run, with a *new* reference each time (like creating a new object/array inline).
- Fix by: only updating state conditionally inside the effect, memoizing objects/functions used as dependencies (`useMemo`/`useCallback`), or removing unnecessary dependencies.

```jsx
// Bug: `options` is a new object every render -> effect re-runs forever
useEffect(() => { doSomething(options); }, [{ ...options }]);

// Fix: depend on stable primitive values instead
useEffect(() => { doSomething(options); }, [options.id]);
```

[↑ Back to top](#table-of-contents)

---

### 8. What is `useRef`, and what are its common use cases? 🟡

- Returns a mutable `{ current: value }` object that **persists across renders** without causing a re-render when changed (unlike state).
- Common uses: accessing a DOM node directly, storing a previous value, holding a mutable value (like a timer ID) that doesn't need to trigger UI updates.

```jsx
function TextInput() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);
  return <input ref={inputRef} />;
}
```

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `useState` and `useRef`? 🟡

- `useState`: updating it triggers a **re-render**; value is "snapshotted" per render.
- `useRef`: updating `.current` does **not** trigger a re-render; the same mutable object persists, and reading it always gives the latest value, even inside stale closures.

[↑ Back to top](#table-of-contents)

---

### 10. How does `useContext` work? 🟡

- Subscribes a component to the nearest matching `<Context.Provider>` above it in the tree, returning its current value — avoids manually passing props down through every intermediate level (see [Context API](context-api.md)).

```jsx
const ThemeContext = createContext('light');
function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>...</div>;
}
```

[↑ Back to top](#table-of-contents)

---

### 11. What is `useReducer`, and when would you prefer it over `useState`? 🟡

- Manages state via a `(state, action) => newState` reducer function, similar to Redux — preferable when state updates are complex, involve multiple related fields, or the next state depends heavily on the action type rather than a simple new value.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'reset': return { count: 0 };
    default: return state;
  }
}
const [state, dispatch] = useReducer(reducer, { count: 0 });
dispatch({ type: 'increment' });
```

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between `useMemo` and `useCallback`? 🟡

- `useMemo(fn, deps)`: memoizes the **return value** of calling `fn`.
- `useCallback(fn, deps)`: memoizes the **function itself** (doesn't call it) — equivalent to `useMemo(() => fn, deps)`.
- Both skip recomputing/recreating unless a dependency changes.

```jsx
const expensiveValue = useMemo(() => computeExpensive(a, b), [a, b]);
const stableCallback = useCallback(() => doSomething(a), [a]);
```

[↑ Back to top](#table-of-contents)

---

### 13. Does `useMemo` prevent re-rendering of child components? 🟡

- **No, by itself it doesn't.** `useMemo` only avoids recomputing a value. To actually skip a child's re-render, the child must also be wrapped in `React.memo`, **and** the memoized value/function must be passed as its prop (so the reference stays stable across renders).

```jsx
const data = useMemo(() => computeData(input), [input]);
return <MemoizedChild data={data} />; // only skips re-render if MemoizedChild itself is memoized
```

[↑ Back to top](#table-of-contents)

---

### 14. What are custom Hooks, and how do you build one? 🟡

- A function (prefixed `use...`) that calls other Hooks internally to encapsulate and **share reusable stateful logic** between components — without duplicating code or resorting to render props/HOCs.

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);
  return width;
}
// Usage: const width = useWindowWidth();
```

[↑ Back to top](#table-of-contents)

---

### 15. Can Hooks be used in class components? 🟡

- **No** — Hooks only work inside function components (or other custom Hooks). Class components must continue using lifecycle methods and `this.state`. You can, however, mix the two in the same app/tree (e.g. a class component rendering a function component that uses Hooks).

[↑ Back to top](#table-of-contents)

---

### 16. How does `useState` work internally under the hood? 🔴

- React maintains a **linked list of "hook" entries** per component instance (stored on its Fiber node). Each `useState` call, by call order, reads/writes its own slot in that list.
- Calling the setter doesn't mutate state immediately — it schedules an update; React re-renders the component, and during that re-render, `useState` returns the new value, from that same ordered slot. This call-order dependency is exactly why the [Rules of Hooks](#2-what-rules-must-hooks-follow) exist.

[↑ Back to top](#table-of-contents)

---

### 17. Is the `dispatch` function from `useReducer` synchronous? 🔴

- `dispatch` itself runs synchronously (the reducer function executes immediately to compute the new state), but the resulting **re-render** is scheduled, not immediate — similar to `useState`'s setter, reading state right after calling `dispatch` in the same render still gives you the **old** value, since the component hasn't re-rendered yet.

```jsx
function handleClick() {
  dispatch({ type: 'increment' });
  console.log(state.count); // still the OLD value — re-render hasn't happened yet
}
```

[↑ Back to top](#table-of-contents)

---

### 18. What is `useImperativeHandle`, and when would you use it? 🔴

- Lets a component **customize** what's exposed when a parent attaches a `ref` to it via `forwardRef` — instead of exposing the raw DOM node, you control exactly which methods/values are accessible.

```jsx
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(), // only this method is exposed, not the whole DOM node
  }));
  return <input ref={inputRef} />;
});
```

[↑ Back to top](#table-of-contents)

---

### 19. What is `useTransition`, and how is it different from `useDeferredValue`? 🔴

- `useTransition`: wraps a **state update** you trigger, marking it as low-priority ("non-urgent") so React can interrupt it in favor of more urgent updates (like typing) — gives you an `isPending` flag.
- `useDeferredValue`: wraps a **value** (not an update you control), telling React it's okay to show a stale/previous version of that value briefly while a more urgent re-render happens — useful when the value comes from somewhere you don't control (e.g. a prop).

```jsx
const [isPending, startTransition] = useTransition();
function handleChange(value) {
  setInput(value);            // urgent — keep input responsive
  startTransition(() => setResults(filterList(value))); // low-priority — can be interrupted
}
```

[↑ Back to top](#table-of-contents)

---

### 20. What is `useSyncExternalStore` used for? 🔴

- Lets you safely subscribe a component to an **external** (non-React) data store — like a browser API or a state library outside React's own state — in a way that's correctly synchronized with React's concurrent rendering, avoiding subtle "tearing" bugs that manual `useEffect`-based subscriptions could cause.

```jsx
const isOnline = useSyncExternalStore(
  (callback) => { window.addEventListener('online', callback); window.addEventListener('offline', callback); return () => {/* cleanup */}; },
  () => navigator.onLine
);
```

> [!TIP]
> **Real-life example:** state-management libraries like Redux and Zustand use `useSyncExternalStore` internally to safely connect their external store to React components.

[↑ Back to top](#table-of-contents)

---

### 21. What is `useInsertionEffect` used for? 🔴

- Runs **before** any DOM mutations happen for the current render — even earlier than `useLayoutEffect`. Designed specifically for CSS-in-JS libraries to inject `<style>` tags before layout is read/measured by other effects, avoiding layout thrashing. Not intended for general application logic.

[↑ Back to top](#table-of-contents)

---

### 22. What is `useId`, and when should you use it? 🔴

- Generates a unique, stable ID string for accessibility attributes (`htmlFor`/`id` pairs on labels and inputs) that's **consistent between server and client renders** — using `Math.random()` or an incrementing counter for IDs would mismatch between SSR and client hydration.

```jsx
function Field() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Name</label>
      <input id={id} />
    </>
  );
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why specifically does `Math.random()` break under server-side rendering + hydration?

[↑ Back to top](#table-of-contents)

---
