# Components & JSX

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What's the difference between a class component and a function component?](#1-whats-the-difference-between-a-class-component-and-a-function-component)
- [2. What are props, and how do they differ from state?](#2-what-are-props-and-how-do-they-differ-from-state)
- [3. What is the `children` prop?](#3-what-is-the-children-prop)
- [4. Why must component names start with a capital letter?](#4-why-must-component-names-start-with-a-capital-letter)
- [5. What's the difference between controlled and uncontrolled components?](#5-whats-the-difference-between-controlled-and-uncontrolled-components)
- [6. How do you write comments inside JSX?](#6-how-do-you-write-comments-inside-jsx)

**🟡 Medium**
- [7. What is the `key` prop, and why does it matter for lists?](#7-what-is-the-key-prop-and-why-does-it-matter-for-lists)
- [8. What's the impact of using array indexes as keys?](#8-whats-the-impact-of-using-array-indexes-as-keys)
- [9. What are Fragments, and why use them over a wrapper `<div>`?](#9-what-are-fragments-and-why-use-them-over-a-wrapper-div)
- [10. What are Pure Components, and how do they differ from `React.memo`?](#10-what-are-pure-components-and-how-do-they-differ-from-reactmemo)
- [11. How do you apply validation to props (PropTypes)?](#11-how-do-you-apply-validation-to-props-proptypes)
- [12. How do you conditionally render content in JSX?](#12-how-do-you-conditionally-render-content-in-jsx)
- [13. Why do you need to be careful spreading props onto a DOM element?](#13-why-do-you-need-to-be-careful-spreading-props-onto-a-dom-element)

**🔴 Hard**
- [14. What are portals, and when would you use one?](#14-what-are-portals-and-when-would-you-use-one)
- [15. Why can't you mutate props directly?](#15-why-cant-you-mutate-props-directly)
- [16. What is "lifting state up," and when is it necessary?](#16-what-is-lifting-state-up-and-when-is-it-necessary)

---

### 1. What's the difference between a class component and a function component? 🟢

- **Class components**: use `this.state`, lifecycle methods (`componentDidMount`, etc.), more boilerplate.
- **Function components**: plain functions returning JSX, use Hooks (`useState`, `useEffect`) for state/lifecycle — the modern default; class components are now mostly legacy, kept for older codebases or specific cases like error boundaries (class-only, see [Advanced Patterns](advanced-patterns.md)).

```jsx
// Class
class Greeting extends React.Component {
  render() { return <h1>Hi, {this.props.name}</h1>; }
}

// Function
function Greeting({ name }) {
  return <h1>Hi, {name}</h1>;
}
```

[↑ Back to top](#table-of-contents)

---

### 2. What are props, and how do they differ from state? 🟢

- **Props**: data passed **into** a component from its parent — read-only from the receiving component's perspective.
- **State**: data owned and managed **internally** by the component itself — can change over time and triggers a re-render when updated.

```jsx
function Welcome(props) { return <h1>Hello, {props.name}</h1>; } // props — external, read-only
function Counter() {
  const [count, setCount] = useState(0); // state — internal, mutable via setter
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

[↑ Back to top](#table-of-contents)

---

### 3. What is the `children` prop? 🟢

- A special prop automatically populated with whatever is nested **between** a component's opening and closing tags — lets components wrap/compose arbitrary content.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
<Card><p>This is the card's content</p></Card>
```

[↑ Back to top](#table-of-contents)

---

### 4. Why must component names start with a capital letter? 🟢

- JSX uses capitalization to distinguish a **custom component** (`<Greeting />`) from a **built-in HTML tag** (`<greeting />` would be treated as a literal `<greeting>` DOM element, not your component) — this is a JSX convention/requirement, not just style.

[↑ Back to top](#table-of-contents)

---

### 5. What's the difference between controlled and uncontrolled components? 🟢

- **Controlled**: the form input's value is driven entirely by React state — every change goes through `onChange` and updates state, which then re-renders the input.
- **Uncontrolled**: the DOM itself manages the input's value — React reads it on demand via a `ref`, instead of tracking every keystroke in state.

```jsx
// Controlled
function Controlled() {
  const [value, setValue] = useState('');
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}

// Uncontrolled
function Uncontrolled() {
  const inputRef = useRef();
  return <input ref={inputRef} defaultValue="initial" />;
}
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you write comments inside JSX? 🟢

- Use standard JS comment syntax, but wrapped in curly braces since you're inside a JSX expression context.

```jsx
return (
  <div>
    {/* This is a JSX comment */}
    <p>Visible text</p>
  </div>
);
```

[↑ Back to top](#table-of-contents)

---

### 7. What is the `key` prop, and why does it matter for lists? 🟡

- A special prop that gives each element in a list a **stable identity** across renders, letting React's diffing algorithm correctly match old elements to new ones (see [Core Concepts](core-concepts.md#9-what-is-the-diffing-algorithm-and-what-rules-does-it-follow)) — without it, React may unnecessarily re-create or mismatch DOM nodes when the list changes.

```jsx
{items.map((item) => <li key={item.id}>{item.name}</li>)}
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the impact of using array indexes as keys? 🟡

- Works fine if the list is **static** (never reordered/filtered/inserted in the middle). But if items are added/removed/reordered, index-based keys can mismatch — React might preserve the wrong DOM node's internal state (e.g. focus, input values) against the wrong data, or cause unnecessary re-renders/remounts.

```jsx
// Risky if `items` can be reordered or filtered
{items.map((item, index) => <li key={index}>{item.name}</li>)}
// Safer — tied to the actual data, not position
{items.map((item) => <li key={item.id}>{item.name}</li>)}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - When is it actually safe to use an index as a key?
> - Do keys need to be globally unique across the whole app, or just among siblings?

[↑ Back to top](#table-of-contents)

---

### 9. What are Fragments, and why use them over a wrapper `<div>`? 🟡

- `<>...</>` (or `<React.Fragment>`) lets a component return multiple sibling elements **without** adding an extra real DOM node — avoids unnecessary wrapper `<div>`s that can break CSS layout (e.g. flex/grid expecting direct children) or add meaningless nesting.

```jsx
function Row() {
  return (
    <>
      <td>One</td>
      <td>Two</td>
    </>
  );
}
```

[↑ Back to top](#table-of-contents)

---

### 10. What are Pure Components, and how do they differ from `React.memo`? 🟡

- `React.PureComponent` (class components): automatically implements `shouldComponentUpdate` with a **shallow prop/state comparison** — skips re-rendering if nothing shallowly changed.
- `React.memo` (function components): the equivalent concept for functions — wraps a component so it skips re-rendering when props are shallowly equal to the previous render.

```jsx
const MemoizedList = React.memo(function List({ items }) {
  return items.map((i) => <li key={i.id}>{i.name}</li>);
});
```

[↑ Back to top](#table-of-contents)

---

### 11. How do you apply validation to props (PropTypes)? 🟡

- The `prop-types` package lets you declare expected prop types on a component — logs a console warning (not a hard error) in development if a prop doesn't match.

```jsx
import PropTypes from 'prop-types';

function Greeting({ name, age }) { /* ... */ }
Greeting.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
};
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why might you prefer TypeScript over PropTypes for prop validation?

[↑ Back to top](#table-of-contents)

---

### 12. How do you conditionally render content in JSX? 🟡

- Common patterns: `&&` for "render or nothing," ternary for either/or, or an early `return null`.

```jsx
{isLoggedIn && <LogoutButton />}
{isLoggedIn ? <LogoutButton /> : <LoginButton />}
if (!data) return null;
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What happens if you use `count && <Component />` and `count` is `0`?

[↑ Back to top](#table-of-contents)

---

### 13. Why do you need to be careful spreading props onto a DOM element? 🟡

- Spreading an object of unknown/extra props directly onto a native element (`<div {...props} />`) can leak props React doesn't recognize as valid DOM attributes, causing console warnings, or unintentionally overriding important attributes (like `className` or event handlers) if they're already set elsewhere.

```jsx
function Card({ highlighted, ...rest }) {
  return <div {...rest} />; // destructure out non-DOM props first
}
```

[↑ Back to top](#table-of-contents)

---

### 14. What are portals, and when would you use one? 🔴

- `createPortal(child, domNode)` renders a component's output into a **different** part of the actual DOM tree than its parent component — while it still behaves like a normal child for React's context/event bubbling purposes.

```jsx
import { createPortal } from 'react-dom';

function Modal({ children }) {
  return createPortal(children, document.getElementById('modal-root'));
}
```

> [!TIP]
> **Real-life example:** modals, tooltips, and dropdowns commonly use portals to render above all other content, avoiding `overflow: hidden`/`z-index` clipping issues from ancestor elements.

[↑ Back to top](#table-of-contents)

---

### 15. Why can't you mutate props directly? 🔴

- Props are **owned by the parent** — they flow one-way, down the tree. Mutating them in a child would silently change data the parent (and possibly other siblings using the same data) doesn't expect to change, breaking React's predictable data flow and making bugs much harder to trace.
- React doesn't runtime-enforce this for plain JS objects, but mutating props is considered undefined behavior in React's model — if you need to change something derived from a prop, copy it into local state instead.

[↑ Back to top](#table-of-contents)

---

### 16. What is "lifting state up," and when is it necessary? 🔴

- When two or more sibling components need to share/sync the same piece of state, move that state to their **closest common parent**, then pass it down as props (and pass down setter callbacks for children to update it).

```jsx
function Parent() {
  const [value, setValue] = useState('');
  return (
    <>
      <Input value={value} onChange={setValue} />
      <Display value={value} />
    </>
  );
}
```

[↑ Back to top](#table-of-contents)

---
