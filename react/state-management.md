# State Management

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Why shouldn't you mutate state directly?](#1-why-shouldnt-you-mutate-state-directly)
- [2. What is prop drilling, and what problem does it cause?](#2-what-is-prop-drilling-and-what-problem-does-it-cause)
- [3. What is Redux, and what problem does it solve?](#3-what-is-redux-and-what-problem-does-it-solve)
- [4. What are the three core principles of Redux?](#4-what-are-the-three-core-principles-of-redux)

**🟡 Medium**
- [5. How do you correctly update a nested object in state?](#5-how-do-you-correctly-update-a-nested-object-in-state)
- [6. How do you correctly update an array in state?](#6-how-do-you-correctly-update-an-array-in-state)
- [7. What is the Immer library, and how does it simplify immutable updates?](#7-what-is-the-immer-library-and-how-does-it-simplify-immutable-updates)
- [8. What's the difference between `mapStateToProps` and `mapDispatchToProps`?](#8-whats-the-difference-between-mapstatetoprops-and-mapdispatchtoprops)
- [9. What is a Redux reducer, and why must it be a pure function?](#9-what-is-a-redux-reducer-and-why-must-it-be-a-pure-function)
- [10. What is an action in Redux?](#10-what-is-an-action-in-redux)
- [11. What's the difference between React Context and Redux?](#11-whats-the-difference-between-react-context-and-redux)
- [12. Should you keep all component state in a global store?](#12-should-you-keep-all-component-state-in-a-global-store)

**🔴 Hard**
- [13. What's the difference between Redux Thunk and Redux Saga?](#13-whats-the-difference-between-redux-thunk-and-redux-saga)
- [14. What are Redux selectors, and why use them?](#14-what-are-redux-selectors-and-why-use-them)
- [15. How do you add multiple middlewares to a Redux store?](#15-how-do-you-add-multiple-middlewares-to-a-redux-store)
- [16. What are the differences between Redux and MobX?](#16-what-are-the-differences-between-redux-and-mobx)

---

### 1. Why shouldn't you mutate state directly? 🟢

- React detects changes by comparing references (`oldState !== newState`), not by deeply inspecting values. Mutating the existing object/array keeps the same reference, so React won't detect a change and **won't re-render**.

```jsx
// Wrong — mutates in place, same reference, React won't re-render
state.items.push(newItem);
setItems(state.items);

// Right — creates a new array, new reference
setItems([...state.items, newItem]);
```

[↑ Back to top](#table-of-contents)

---

### 2. What is prop drilling, and what problem does it cause? 🟢

- Passing a piece of data through several layers of components via props, just so a deeply nested component can use it — components in between that don't need the data still have to receive and forward it, adding noise and coupling.
- Solved with [Context API](context-api.md) or a state-management library (Redux, Zustand) for data needed broadly across the tree.

[↑ Back to top](#table-of-contents)

---

### 3. What is Redux, and what problem does it solve? 🟢

- A predictable, centralized state container — all application state lives in **one store**, updated only via dispatched actions processed by pure reducer functions.
- Solves the problem of scattered, hard-to-track state changes across many components, especially state shared/needed across unrelated parts of the tree.

[↑ Back to top](#table-of-contents)

---

### 4. What are the three core principles of Redux? 🟢

1. **Single source of truth**: the entire app state lives in one store.
2. **State is read-only**: the only way to change it is by dispatching an action — never mutate it directly.
3. **Changes are made with pure functions (reducers)**: given the same state and action, a reducer always produces the same new state.

[↑ Back to top](#table-of-contents)

---

### 5. How do you correctly update a nested object in state? 🟡

- Spread every level that needs to change, creating new objects up the chain — don't just spread the top level and mutate inside it.

```jsx
setUser((prev) => ({
  ...prev,
  address: { ...prev.address, city: 'New City' },
}));
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you correctly update an array in state? 🟡

- Use non-mutating array methods/patterns (`map`, `filter`, spread) instead of mutating ones (`push`, `splice`, `sort` in place).

```jsx
// Add
setItems((prev) => [...prev, newItem]);
// Remove
setItems((prev) => prev.filter((item) => item.id !== idToRemove));
// Update one
setItems((prev) => prev.map((item) => (item.id === id ? { ...item, done: true } : item)));
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is `array.sort()` risky to use directly on state, even though it "looks" like it just reorders?

[↑ Back to top](#table-of-contents)

---

### 7. What is the Immer library, and how does it simplify immutable updates? 🟡

- Lets you write code that **looks like** direct mutation, but produces a new immutable object behind the scenes (via a temporary "draft" proxy) — removes the need for manual nested spreading.

```jsx
import { produce } from 'immer';

const nextState = produce(state, (draft) => {
  draft.user.address.city = 'New City'; // looks like mutation, but Immer makes it immutable
});
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the difference between `mapStateToProps` and `mapDispatchToProps`? 🟡

- `mapStateToProps(state)`: selects which slices of the Redux store to inject as props into a connected component.
- `mapDispatchToProps(dispatch)`: defines which action-dispatching functions get injected as props, so the component can trigger state changes without importing `dispatch` directly.

```jsx
const mapStateToProps = (state) => ({ count: state.counter.value });
const mapDispatchToProps = (dispatch) => ({ increment: () => dispatch({ type: 'INCREMENT' }) });
export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```

[↑ Back to top](#table-of-contents)

---

### 9. What is a Redux reducer, and why must it be a pure function? 🟡

- A function `(state, action) => newState` that computes the next state based on the current state and a dispatched action.
- Must be pure (no side effects, no mutation, same input → same output) so Redux's predictability guarantees hold — time-travel debugging, replaying actions, and `React.memo`-style optimizations all depend on reducers behaving deterministically.

[↑ Back to top](#table-of-contents)

---

### 10. What is an action in Redux? 🟡

- A plain object describing **what happened** — must have a `type` field, plus optional payload data — dispatched to the store to trigger a state update via the reducer.

```js
{ type: 'cart/addItem', payload: { id: 1, name: 'Book' } }
```

[↑ Back to top](#table-of-contents)

---

### 11. What's the difference between React Context and Redux? 🟡

- **Context**: built into React, great for low-frequency-update, broadly-needed values (theme, auth user) — but re-renders **every** consumer on any value change, and has no built-in dev tools/middleware.
- **Redux**: external library, built for larger/more complex state with frequent updates — offers fine-grained subscriptions (only components actually using a changed slice re-render via selectors), middleware, time-travel debugging.

[↑ Back to top](#table-of-contents)

---

### 12. Should you keep all component state in a global store? 🟡

- No — purely local UI state (an input's current value, whether a dropdown is open) usually belongs in `useState`/`useReducer` **within** the component that needs it.
- Reach for a global store only for state genuinely shared across distant parts of the tree, or that needs to persist across navigation/unmounts. Overusing global state adds unnecessary complexity and coupling.

[↑ Back to top](#table-of-contents)

---

### 13. What's the difference between Redux Thunk and Redux Saga? 🔴

- **Redux Thunk**: lets action creators return a **function** (instead of a plain object) that receives `dispatch`/`getState` — simple way to handle async logic (like API calls) inline.
- **Redux Saga**: uses **generator functions** to manage complex async flows (sequencing, cancellation, retries, debouncing) declaratively, as a separate background "process" listening for actions — more powerful but a steeper learning curve.

```js
// Thunk
const fetchUser = (id) => async (dispatch) => {
  const res = await fetch(`/api/users/${id}`);
  dispatch({ type: 'user/loaded', payload: await res.json() });
};

// Saga (simplified)
function* fetchUserSaga(action) {
  const data = yield call(fetch, `/api/users/${action.payload}`);
  yield put({ type: 'user/loaded', payload: data });
}
```

[↑ Back to top](#table-of-contents)

---

### 14. What are Redux selectors, and why use them? 🔴

- Functions that **derive/extract** specific data from the store, instead of components reaching directly into `state.some.deeply.nested.field`.
- Benefits: centralizes derived-data logic, makes refactoring the store shape easier (only the selector needs updating), and (combined with a memoizing library like Reselect) avoids recomputing expensive derived values unless their inputs actually changed.

```js
const selectCartTotal = (state) =>
  state.cart.items.reduce((sum, item) => sum + item.price, 0);
```

[↑ Back to top](#table-of-contents)

---

### 15. How do you add multiple middlewares to a Redux store? 🔴

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import logger from 'redux-logger';

const store = createStore(rootReducer, applyMiddleware(thunk, logger));
```

[↑ Back to top](#table-of-contents)

---

### 16. What are the differences between Redux and MobX? 🔴

- **Redux**: single immutable store, explicit actions/reducers, unidirectional and very predictable — more boilerplate, but easier to reason about and debug at scale.
- **MobX**: uses observable, **mutable** state directly — components automatically re-render when an observed value changes, via transparent reactive tracking (similar in spirit to Vue). Less boilerplate, but the "magic" reactivity can make data flow less explicit/traceable than Redux's.

> [!IMPORTANT]
> **Follow-up questions:**
> - Which approach tends to scale better on large teams, and why?
> - How does MobX detect which components depend on which observables?

[↑ Back to top](#table-of-contents)

---
