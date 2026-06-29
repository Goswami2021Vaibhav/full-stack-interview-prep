# Core Concepts

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is React?](#1-what-is-react)
- [2. What is JSX?](#2-what-is-jsx)
- [3. What's the difference between an Element and a Component?](#3-whats-the-difference-between-an-element-and-a-component)
- [4. What's the difference between React and ReactDOM?](#4-whats-the-difference-between-react-and-reactdom)
- [5. What are the advantages and limitations of React?](#5-what-are-the-advantages-and-limitations-of-react)
- [6. Does the browser understand JSX directly?](#6-does-the-browser-understand-jsx-directly)

**🟡 Medium**
- [7. What is the Virtual DOM, and how does it work?](#7-what-is-the-virtual-dom-and-how-does-it-work)
- [8. What is reconciliation?](#8-what-is-reconciliation)
- [9. What is the diffing algorithm, and what rules does it follow?](#9-what-is-the-diffing-algorithm-and-what-rules-does-it-follow)
- [10. What is React Fiber, and what problem does it solve?](#10-what-is-react-fiber-and-what-problem-does-it-solve)
- [11. What's the difference between the Shadow DOM and the Virtual DOM?](#11-whats-the-difference-between-the-shadow-dom-and-the-virtual-dom)
- [12. What's the difference between imperative and declarative programming, in React's context?](#12-whats-the-difference-between-imperative-and-declarative-programming-in-reacts-context)
- [13. How does JSX prevent injection attacks?](#13-how-does-jsx-prevent-injection-attacks)

**🔴 Hard**
- [14. What is concurrent rendering, and how is it different from the legacy/sync mode?](#14-what-is-concurrent-rendering-and-how-is-it-different-from-the-legacysync-mode)
- [15. How does React batch multiple state updates?](#15-how-does-react-batch-multiple-state-updates)
- [16. What is React hydration?](#16-what-is-react-hydration)
- [17. What is Strict Mode, and why does it render components twice in development?](#17-what-is-strict-mode-and-why-does-it-render-components-twice-in-development)
- [18. What are React Server Components?](#18-what-are-react-server-components)

---

### 1. What is React? 🟢

- A JavaScript library (not a full framework) for building user interfaces out of reusable, composable **components** — manages what gets rendered based on state, using a declarative model instead of manually touching the DOM.

[↑ Back to top](#table-of-contents)

---

### 2. What is JSX? 🟢

- A syntax extension that lets you write HTML-like markup directly inside JavaScript — compiles down to `React.createElement()` calls (or a newer automatic JSX transform), it isn't actually HTML and isn't understood by browsers natively.

```jsx
const element = <h1>Hello, {name}</h1>;
// compiles roughly to:
const element = React.createElement('h1', null, 'Hello, ', name);
```

[↑ Back to top](#table-of-contents)

---

### 3. What's the difference between an Element and a Component? 🟢

- An **Element** is a plain, immutable object describing what should appear on screen (`{ type: 'div', props: {...} }`) — cheap to create, just a description.
- A **Component** is a function or class that **returns** elements — the reusable blueprint that produces elements when rendered.

```jsx
const element = <div>Hi</div>;       // an element — just a description
function Greeting() { return <div>Hi</div>; } // a component — produces elements
```

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between React and ReactDOM? 🟢

- `react`: the core library — component model, hooks, JSX element creation — platform-agnostic.
- `react-dom`: the renderer that takes React elements and actually mounts/updates them in the **browser DOM**. React Native uses a different renderer instead of `react-dom`, while reusing the same `react` core.

```jsx
import ReactDOM from 'react-dom/client';
ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why was rendering logic split out of the `react` package in the first place?

[↑ Back to top](#table-of-contents)

---

### 5. What are the advantages and limitations of React? 🟢

- **Advantages**: component reusability, Virtual DOM for efficient updates, huge ecosystem/community, one-way data binding makes data flow predictable, usable for web + native (via React Native).
- **Limitations**: just the view layer (routing/state-management/etc. need extra libraries), JSX has a learning curve, fast pace of ecosystem changes, SEO requires extra setup (SSR) since it's client-rendered by default.

[↑ Back to top](#table-of-contents)

---

### 6. Does the browser understand JSX directly? 🟢

- No — JSX must be **compiled** (by Babel, the TypeScript compiler, or a bundler) into plain `React.createElement()`/JS function calls before it reaches the browser.

[↑ Back to top](#table-of-contents)

---

### 7. What is the Virtual DOM, and how does it work? 🟡

- A lightweight, in-memory JS representation of the actual DOM tree. When state changes, React builds a **new** virtual tree, **diffs** it against the previous one, and applies only the minimal set of real DOM updates needed — avoiding expensive direct DOM manipulation for every change.

[↑ Back to top](#table-of-contents)

---

### 8. What is reconciliation? 🟡

- The overall **algorithm/process** React uses to figure out what changed between two virtual DOM trees and update the real DOM accordingly — diffing is the comparison step within this larger reconciliation process.

[↑ Back to top](#table-of-contents)

---

### 9. What is the diffing algorithm, and what rules does it follow? 🟡

- Instead of a generic (and expensive, O(n³)) tree-diff, React uses heuristics that make comparisons close to O(n):
  1. **Different element types** at the same position → tear down the old subtree entirely, build a new one (no attempt to preserve children).
  2. **Same element type** → keep the DOM node, just update changed attributes/props.
  3. **Lists of children** are matched using the `key` prop, not just position — letting React correctly track which item moved/added/removed instead of mismatching them.

[↑ Back to top](#table-of-contents)

---

### 10. What is React Fiber, and what problem does it solve? 🟡

- React's internal reconciliation engine (since React 16), rewritten so rendering work can be **split into units, paused, prioritized, and resumed** — instead of the old "stack reconciler," which rendered the whole tree synchronously and could block the main thread on large updates.
- Fiber is what makes concurrent features (Suspense, time-slicing, `useTransition`) possible.

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between the Shadow DOM and the Virtual DOM? 🟡

- **Shadow DOM**: a real **browser** API for encapsulating a subtree's markup/styles so they don't leak in or out (used by native Web Components) — it's a DOM feature, not React-specific.
- **Virtual DOM**: a React-specific (and not browser-native) in-memory abstraction purely for efficient diffing/updates — not about style/markup encapsulation at all.

[↑ Back to top](#table-of-contents)

---

### 12. What's the difference between imperative and declarative programming, in React's context? 🟡

- **Imperative** (vanilla DOM manipulation): you write the exact step-by-step instructions for *how* to update the UI (`element.classList.add(...)`, `element.appendChild(...)`).
- **Declarative** (React): you describe *what* the UI should look like for a given state, and React figures out how to make the real DOM match it.

```js
// Imperative
const el = document.createElement('div');
el.textContent = 'Hello';
document.body.appendChild(el);

// Declarative (React)
function App() { return <div>Hello</div>; }
```

[↑ Back to top](#table-of-contents)

---

### 13. How does JSX prevent injection attacks? 🟡

- React escapes any value embedded in JSX via `{ }` before rendering it as text — so even if a value contains HTML/script-like content, it's rendered as a literal string, not executed. (This protection doesn't apply if you explicitly bypass it with `dangerouslySetInnerHTML`.)

```jsx
const userInput = '<img src=x onerror="alert(1)">';
return <div>{userInput}</div>; // rendered as literal escaped text, not executed
```

[↑ Back to top](#table-of-contents)

---

### 14. What is concurrent rendering, and how is it different from the legacy/sync mode? 🔴

- In legacy mode, once React starts rendering an update, it runs to completion **synchronously**, blocking the main thread.
- **Concurrent rendering** (enabled via `createRoot` in React 18+) lets React **interrupt, pause, abandon, or prioritize** render work — e.g. it can pause a low-priority render to handle an urgent user interaction first, then resume. This is what powers `useTransition` and `useDeferredValue`.

[↑ Back to top](#table-of-contents)

---

### 15. How does React batch multiple state updates? 🔴

- React groups multiple `setState`/state-setter calls that happen within the same event handler (or, since React 18, within promises/timeouts/native event handlers too — "automatic batching") into a **single** re-render, instead of re-rendering once per call.

```jsx
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // Only ONE re-render happens, with both updates applied together
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - How would you opt out of batching if you needed two separate renders (`flushSync`)?
> - Why couldn't `setTimeout`/promise callbacks be batched before React 18?

[↑ Back to top](#table-of-contents)

---

### 16. What is React hydration? 🔴

- The process of taking server-rendered static HTML (from SSR) and **attaching** React's event listeners and internal state to it on the client, instead of throwing it away and re-rendering everything from scratch.
- Faster perceived load (HTML is visible immediately) while still ending up with a fully interactive React app.

[↑ Back to top](#table-of-contents)

---

### 17. What is Strict Mode, and why does it render components twice in development? 🔴

- `<StrictMode>` is a development-only wrapper that helps surface potential problems: deprecated APIs, unexpected side effects, and unsafe lifecycle usage.
- It intentionally **double-invokes** component render functions (and effects) in development only — to help you catch code that isn't pure (e.g. side effects during render) by making the impact of accidentally running twice visible early, before it causes hard-to-debug bugs in production under concurrent rendering.

[↑ Back to top](#table-of-contents)

---

### 18. What are React Server Components? 🔴

- A component type that renders **only on the server**, never shipped to the client as JS — it can directly access backend resources (databases, file systems) and sends only the rendered output (not component code) to the browser, reducing bundle size.
- Different from traditional SSR: SSR still ships the component's JS to hydrate on the client; Server Components for non-interactive parts ship **zero** JS for those components.

> [!IMPORTANT]
> **Follow-up questions:**
> - Can a Server Component import and render a Client Component, and vice versa?
> - Why can't Server Components use hooks like `useState`?

[↑ Back to top](#table-of-contents)

---
