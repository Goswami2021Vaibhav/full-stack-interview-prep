# Lifecycle & Rendering

_Part of [React](README.md) interview notes._

> Hook-based lifecycle equivalents (`useEffect`, `useLayoutEffect`) are covered in [Hooks](hooks.md). This file focuses on class component lifecycle methods and the render/commit model underneath both.

## Table of Contents

**🟢 Easy**
- [1. What are the different phases of a component's lifecycle?](#1-what-are-the-different-phases-of-a-components-lifecycle)
- [2. What are the main lifecycle methods of a class component?](#2-what-are-the-main-lifecycle-methods-of-a-class-component)
- [3. What is the order of lifecycle methods during mounting?](#3-what-is-the-order-of-lifecycle-methods-during-mounting)

**🟡 Medium**
- [4. What is `shouldComponentUpdate`, and what problem does it solve?](#4-what-is-shouldcomponentupdate-and-what-problem-does-it-solve)
- [5. What is `getDerivedStateFromProps`, and when would you use it?](#5-what-is-getderivedstatefromprops-and-when-would-you-use-it)
- [6. What is `getSnapshotBeforeUpdate`, and when would you use it?](#6-what-is-getsnapshotbeforeupdate-and-when-would-you-use-it)
- [7. What are error boundaries, and which lifecycle methods implement them?](#7-what-are-error-boundaries-and-which-lifecycle-methods-implement-them)
- [8. What is the `componentDidCatch` method signature?](#8-what-is-the-componentdidcatch-method-signature)
- [9. In which scenarios do error boundaries not catch errors?](#9-in-which-scenarios-do-error-boundaries-not-catch-errors)

**🔴 Hard**
- [10. What's the difference between the render phase and the commit phase?](#10-whats-the-difference-between-the-render-phase-and-the-commit-phase)
- [11. What is the recommended placement for error boundaries in an app?](#11-what-is-the-recommended-placement-for-error-boundaries-in-an-app)
- [12. What is the behavior of uncaught errors in React 16+?](#12-what-is-the-behavior-of-uncaught-errors-in-react-16)
- [13. Which lifecycle methods are considered legacy/unsafe, and why?](#13-which-lifecycle-methods-are-considered-legacyunsafe-and-why)
- [14. What is `getDerivedStateFromError`, and how does it differ from `componentDidCatch`?](#14-what-is-getderivedstatefromerror-and-how-does-it-differ-from-componentdidcatch)

---

### 1. What are the different phases of a component's lifecycle? 🟢

- **Mounting**: the component is being created and inserted into the DOM for the first time.
- **Updating**: the component re-renders due to changed props/state.
- **Unmounting**: the component is being removed from the DOM.

[↑ Back to top](#table-of-contents)

---

### 2. What are the main lifecycle methods of a class component? 🟢

- **Mounting**: `constructor` → `getDerivedStateFromProps` → `render` → `componentDidMount`.
- **Updating**: `getDerivedStateFromProps` → `shouldComponentUpdate` → `render` → `getSnapshotBeforeUpdate` → `componentDidUpdate`.
- **Unmounting**: `componentWillUnmount`.

[↑ Back to top](#table-of-contents)

---

### 3. What is the order of lifecycle methods during mounting? 🟢

1. `constructor()`
2. `static getDerivedStateFromProps()`
3. `render()`
4. `componentDidMount()`

```jsx
class Example extends React.Component {
  constructor(props) { super(props); console.log('1. constructor'); }
  static getDerivedStateFromProps() { console.log('2. getDerivedStateFromProps'); return null; }
  render() { console.log('3. render'); return <div />; }
  componentDidMount() { console.log('4. componentDidMount'); }
}
```

[↑ Back to top](#table-of-contents)

---

### 4. What is `shouldComponentUpdate`, and what problem does it solve? 🟡

- Lets a class component **opt out** of re-rendering by returning `false`, based on comparing current vs. next props/state — a manual performance optimization to skip unnecessary renders. `React.PureComponent` implements this automatically with a shallow comparison.

```jsx
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.value !== this.props.value; // skip re-render if `value` didn't change
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What is `getDerivedStateFromProps`, and when would you use it? 🟡

- A **static** method called right before every render (both mounting and updating), letting state be derived/synced from incoming props. Rare in practice — usually a sign you could instead compute the value directly during render, or lift the state up.

```jsx
static getDerivedStateFromProps(props, state) {
  if (props.userId !== state.prevUserId) {
    return { prevUserId: props.userId, data: null }; // reset derived state when userId changes
  }
  return null; // no state change
}
```

[↑ Back to top](#table-of-contents)

---

### 6. What is `getSnapshotBeforeUpdate`, and when would you use it? 🟡

- Called right **before** the DOM is updated, letting you capture some info from the current DOM (e.g. scroll position) before it changes — that captured value is then passed as the third argument to `componentDidUpdate`.

```jsx
getSnapshotBeforeUpdate(prevProps) {
  if (prevProps.list.length < this.props.list.length) {
    return this.listRef.current.scrollHeight; // capture scroll height before new items render
  }
  return null;
}
componentDidUpdate(prevProps, prevState, snapshot) {
  if (snapshot !== null) {
    this.listRef.current.scrollTop += this.listRef.current.scrollHeight - snapshot;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 7. What are error boundaries, and which lifecycle methods implement them? 🟡

- Class components that catch JavaScript errors thrown anywhere in their **child** component tree during rendering, log them, and display a fallback UI instead of crashing the whole app.
- Implemented via `static getDerivedStateFromError()` (to update state and render a fallback) and/or `componentDidCatch()` (to log the error). Only class components can be error boundaries — there's no Hook equivalent.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info) { logErrorToService(error, info); }
  render() {
    return this.state.hasError ? <h1>Something went wrong.</h1> : this.props.children;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the `componentDidCatch` method signature? 🟡

- `componentDidCatch(error, info)` — `error` is the actual thrown error object; `info` is an object containing `componentStack`, a string describing which component in the tree threw it.

[↑ Back to top](#table-of-contents)

---

### 9. In which scenarios do error boundaries not catch errors? 🟡

- Errors inside: **event handlers** (use a regular `try`/`catch` there instead), **asynchronous code** (`setTimeout`, promises), **server-side rendering**, and errors thrown in the **error boundary itself** (rather than its children).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why doesn't React consider event handler errors part of the "rendering" process error boundaries protect?

[↑ Back to top](#table-of-contents)

---

### 10. What's the difference between the render phase and the commit phase? 🔴

- **Render phase**: React calls component functions/render methods to figure out what *should* change — pure, has no visible side effects, and (under concurrent rendering) **can be paused, aborted, or restarted**.
- **Commit phase**: React actually applies the calculated changes to the real DOM, and runs `componentDidMount`/`componentDidUpdate`/`useLayoutEffect`/`useEffect` — this phase is **synchronous and cannot be interrupted**, since the user shouldn't see a half-applied UI update.

[↑ Back to top](#table-of-contents)

---

### 11. What is the recommended placement for error boundaries in an app? 🔴

- Wrap them around **logical sections** likely to fail independently (a specific widget, a route, a third-party-data-dependent panel) rather than just one single boundary around the entire app — so one broken section shows a fallback without taking down the whole page.

[↑ Back to top](#table-of-contents)

---

### 12. What is the behavior of uncaught errors in React 16+? 🔴

- If an error is thrown during rendering and **no** error boundary catches it anywhere up the tree, React will **unmount the entire component tree** (white screen) — a deliberate change from React 15, where leaving a partially-broken UI on screen was considered worse than a controlled full unmount.

[↑ Back to top](#table-of-contents)

---

### 13. Which lifecycle methods are considered legacy/unsafe, and why? 🔴

- `componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate` (renamed with `UNSAFE_` prefixes) — deprecated because they can be called **multiple times** per single update under concurrent rendering (since render-phase work can be paused/restarted), making any side effects inside them unreliable and bug-prone.
- Replaced by `getDerivedStateFromProps` and `getSnapshotBeforeUpdate`, which are designed to be safely re-callable.

[↑ Back to top](#table-of-contents)

---

### 14. What is `getDerivedStateFromError`, and how does it differ from `componentDidCatch`? 🔴

- `getDerivedStateFromError(error)`: a **static** method called during the render phase — used to compute new state (e.g. a `hasError` flag) so the next render can show fallback UI. Must be pure, no side effects.
- `componentDidCatch(error, info)`: called during the commit phase — the right place for actual **side effects**, like logging the error to an external service.
- In practice, error boundaries typically implement both: `getDerivedStateFromError` to drive the fallback UI, `componentDidCatch` to report it.

[↑ Back to top](#table-of-contents)

---
