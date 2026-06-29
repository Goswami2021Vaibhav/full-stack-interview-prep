# Context API

_Part of [React](README.md) interview notes._

> Basic `useContext` mechanics are covered in [Hooks](hooks.md). This file focuses on Context-specific setup, pitfalls, and patterns.

## Table of Contents

**🟢 Easy**
- [1. How do you create and provide a Context?](#1-how-do-you-create-and-provide-a-context)
- [2. What is the purpose of the default value passed to `createContext`?](#2-what-is-the-purpose-of-the-default-value-passed-to-createcontext)
- [3. What is a Consumer, and when would you use one instead of `useContext`?](#3-what-is-a-consumer-and-when-would-you-use-one-instead-of-usecontext)

**🟡 Medium**
- [4. How do you use Context in a class component?](#4-how-do-you-use-context-in-a-class-component)
- [5. Can you use multiple Contexts in one component?](#5-can-you-use-multiple-contexts-in-one-component)
- [6. What's a common pitfall when a Context value is an object created inline?](#6-whats-a-common-pitfall-when-a-context-value-is-an-object-created-inline)
- [7. What context value do you get if there's no matching Provider above a component?](#7-what-context-value-do-you-get-if-theres-no-matching-provider-above-a-component)

**🔴 Hard**
- [8. How do you combine `useReducer` with `useContext` for simple global state?](#8-how-do-you-combine-usereducer-with-usecontext-for-simple-global-state)
- [9. How do you solve Context's "re-renders every consumer" performance problem?](#9-how-do-you-solve-contexts-re-renders-every-consumer-performance-problem)
- [10. How do you keep a user authenticated across a page refresh using Context?](#10-how-do-you-keep-a-user-authenticated-across-a-page-refresh-using-context)

---

### 1. How do you create and provide a Context? 🟢

```jsx
const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What is the purpose of the default value passed to `createContext`? 🟢

- `createContext(defaultValue)` — used **only** when a component consumes the context but has **no** matching `Provider` anywhere above it in the tree. Useful for testing components in isolation, or providing a sensible fallback.

```jsx
const ThemeContext = createContext('light'); // fallback if no Provider wraps the consumer
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a Consumer, and when would you use one instead of `useContext`? 🟢

- `<Context.Consumer>` is the older, render-prop-based way to read a context value — still needed inside **class components** (which can't use Hooks), or in the rare case you need to consume context inside a render-prop pattern in a function component.

```jsx
<ThemeContext.Consumer>
  {(theme) => <div className={theme}>...</div>}
</ThemeContext.Consumer>
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you use Context in a class component? 🟡

- Set `static contextType = MyContext` on the class, then read `this.context` anywhere in the component.

```jsx
class Toolbar extends React.Component {
  static contextType = ThemeContext;
  render() {
    return <div className={this.context}>...</div>;
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 5. Can you use multiple Contexts in one component? 🟡

- Yes — call `useContext` once per context, or nest multiple `<Context.Consumer>`s in class components.

```jsx
const theme = useContext(ThemeContext);
const user = useContext(UserContext);
```

[↑ Back to top](#table-of-contents)

---

### 6. What's a common pitfall when a Context value is an object created inline? 🟡

- Passing `value={{ user, setUser }}` directly creates a **new object every render** of the Provider, causing every consumer to re-render even if `user` itself didn't change. Memoize the value with `useMemo` instead.

```jsx
// Bug: new object every render of the Provider
<UserContext.Provider value={{ user, setUser }}>

// Fix: stable reference unless user/setUser actually change
const value = useMemo(() => ({ user, setUser }), [user]);
<UserContext.Provider value={value}>
```

[↑ Back to top](#table-of-contents)

---

### 7. What context value do you get if there's no matching Provider above a component? 🟡

- The **default value** passed to `createContext(defaultValue)` — see Q2. If no default was given either, you get `undefined`.

[↑ Back to top](#table-of-contents)

---

### 8. How do you combine `useReducer` with `useContext` for simple global state? 🔴

- Provide both the state and `dispatch` through context, giving any descendant component access to read state and dispatch actions — a lightweight, dependency-free alternative to Redux for small-to-medium apps.

```jsx
const StateContext = createContext();
const DispatchContext = createContext();

function AppProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>{children}</DispatchContext.Provider>
    </StateContext.Provider>
  );
}
// Elsewhere: const state = useContext(StateContext); const dispatch = useContext(DispatchContext);
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you solve Context's "re-renders every consumer" performance problem? 🔴

- Split a large context into **multiple smaller contexts** by concern, so a change to one slice doesn't re-render consumers only interested in another slice.
- Memoize the provided value (Q6) so unrelated re-renders of the Provider don't cascade.
- For frequently-changing, performance-sensitive state, reach for a library (Redux, Zustand, Jotai) that supports fine-grained, selector-based subscriptions instead of Context's all-or-nothing notification model.

[↑ Back to top](#table-of-contents)

---

### 10. How do you keep a user authenticated across a page refresh using Context? 🔴

- Context state alone doesn't persist across reloads (it resets to its initial value) — pair it with persistent storage (e.g. a token in `localStorage`, or an httpOnly cookie set by the server) and **rehydrate** the context's state from that storage on initial mount.

```jsx
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      fetchCurrentUser(token).then(setUser).finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {loading ? <Spinner /> : children}
    </AuthContext.Provider>
  );
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is it important to show a loading state instead of immediately rendering protected routes while this check is in progress?

[↑ Back to top](#table-of-contents)

---
