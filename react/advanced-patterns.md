# Advanced Patterns

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What are Higher-Order Components (HOCs)?](#1-what-are-higher-order-components-hocs)
- [2. What are render props?](#2-what-are-render-props)
- [3. What is a wrapper component?](#3-what-is-a-wrapper-component)

**🟡 Medium**
- [4. What is `forwardRef`, and when do you need it?](#4-what-is-forwardref-and-when-do-you-need-it)
- [5. What are the limitations/downsides of HOCs?](#5-what-are-the-limitationsdownsides-of-hocs)
- [6. What problems can arise from using render props with Pure Components?](#6-what-problems-can-arise-from-using-render-props-with-pure-components)
- [7. Do Hooks replace render props and HOCs?](#7-do-hooks-replace-render-props-and-hocs)
- [8. What are styled-components, and what problem do they solve?](#8-what-are-styled-components-and-what-problem-do-they-solve)

**🔴 Hard**
- [9. What is the compound component pattern?](#9-what-is-the-compound-component-pattern)
- [10. What is render hijacking in React?](#10-what-is-render-hijacking-in-react)
- [11. What were React Mixins, and why were they deprecated?](#11-what-were-react-mixins-and-why-were-they-deprecated)
- [12. How would you implement a HOC using render props?](#12-how-would-you-implement-a-hoc-using-render-props)
- [13. What is a HOC factory implementation?](#13-what-is-a-hoc-factory-implementation)
- [14. Why do component libraries need extra care when using `forwardRef`?](#14-why-do-component-libraries-need-extra-care-when-using-forwardref)

---

### 1. What are Higher-Order Components (HOCs)? 🟢

- A function that takes a component and **returns a new, enhanced component** — a pattern for reusing component logic (e.g. adding auth checks, injecting data) without repeating it across multiple components.

```jsx
function withLogger(Component) {
  return function Wrapped(props) {
    console.log('Rendering with props:', props);
    return <Component {...props} />;
  };
}
const LoggedButton = withLogger(Button);
```

[↑ Back to top](#table-of-contents)

---

### 2. What are render props? 🟢

- A pattern where a component takes a **function as a prop** (often literally named `render`, or passed as `children`) that returns JSX — letting the consumer control exactly what gets rendered with the data/behavior the component provides.

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  return <div onMouseMove={(e) => setPosition({ x: e.clientX, y: e.clientY })}>{render(position)}</div>;
}
<MouseTracker render={({ x, y }) => <p>Mouse at {x}, {y}</p>} />
```

[↑ Back to top](#table-of-contents)

---

### 3. What is a wrapper component? 🟢

- A component whose main job is to **wrap** other content and add some shared structure/behavior around it (layout, styling, a modal shell) — `children` is typically the primary prop.

```jsx
function Card({ children }) {
  return <div className="card-wrapper">{children}</div>;
}
```

[↑ Back to top](#table-of-contents)

---

### 4. What is `forwardRef`, and when do you need it? 🟡

- By default, function components can't receive a `ref` directly (refs don't pass through as a normal prop). `forwardRef` lets a function component **forward** a ref it receives down to one of its own child elements (often a DOM node).

```jsx
const FancyInput = forwardRef((props, ref) => <input ref={ref} className="fancy" {...props} />);

function App() {
  const inputRef = useRef();
  return <FancyInput ref={inputRef} />; // ref reaches the actual <input>
}
```

[↑ Back to top](#table-of-contents)

---

### 5. What are the limitations/downsides of HOCs? 🟡

- **Wrapper hell**: nesting many HOCs makes the component tree deep and harder to debug (`withA(withB(withC(Component)))`).
- **Prop name collisions**: two HOCs might inject a prop with the same name, silently overwriting each other.
- **Unclear data source**: it's not always obvious, just by reading a component's usage, where a given prop actually came from (which wrapping HOC injected it).
- **Static methods aren't copied automatically**: must manually hoist them from the original component.

[↑ Back to top](#table-of-contents)

---

### 6. What problems can arise from using render props with Pure Components? 🟡

- Passing an inline function as a render prop creates a **new function reference every render** — this defeats `React.memo`/`PureComponent`'s shallow comparison, since the "render" prop never looks equal between renders, causing the component to re-render every time regardless.

```jsx
// New function every render — breaks memoization
<PureMouseTracker render={(pos) => <Display pos={pos} />} />
```

[↑ Back to top](#table-of-contents)

---

### 7. Do Hooks replace render props and HOCs? 🟡

- For the most common use case — **sharing stateful logic** between components — yes, custom Hooks are simpler and avoid wrapper-hell/prop-collision issues entirely (see [Hooks](hooks.md#14-what-are-custom-hooks-and-how-do-you-build-one)).
- They don't fully replace every use case though: HOCs can still wrap a component to inject **props** or intercept its **rendering** itself, which a Hook (which only adds logic *inside* a component, not around it) can't do on its own.

[↑ Back to top](#table-of-contents)

---

### 8. What are styled-components, and what problem do they solve? 🟡

- A CSS-in-JS library that lets you define component styles using tagged template literals, generating a uniquely-scoped class name automatically — solves CSS's global-namespace problem (class name collisions) without needing a separate naming convention like BEM.

```jsx
const Button = styled.button`
  background: ${(props) => (props.primary ? 'blue' : 'gray')};
  color: white;
`;
<Button primary>Click</Button>
```

[↑ Back to top](#table-of-contents)

---

### 9. What is the compound component pattern? 🔴

- Multiple components that work together as a cohesive unit, implicitly sharing state via Context, while giving the consumer flexible control over structure/layout — instead of one monolithic component with many configuration props.

```jsx
const TabsContext = createContext();

function Tabs({ children, defaultIndex = 0 }) {
  const [active, setActive] = useState(defaultIndex);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}
Tabs.Tab = function Tab({ index, children }) {
  const { active, setActive } = useContext(TabsContext);
  return <button onClick={() => setActive(index)} aria-selected={active === index}>{children}</button>;
};

<Tabs>
  <Tabs.Tab index={0}>First</Tabs.Tab>
  <Tabs.Tab index={1}>Second</Tabs.Tab>
</Tabs>
```

> [!TIP]
> **Real-life example:** this is how many component libraries (Radix UI, Chakra UI's `<Menu>`/`<Accordion>`) structure flexible, composable components.

[↑ Back to top](#table-of-contents)

---

### 10. What is render hijacking in React? 🔴

- A technique (mostly associated with older HOC patterns) where a HOC **intercepts and modifies** what a wrapped component renders — e.g. conditionally rendering a loading state instead of the wrapped component, or injecting/altering its returned elements — rather than just passing extra props through untouched.

```jsx
function withLoading(Component) {
  return function Wrapped({ isLoading, ...props }) {
    if (isLoading) return <Spinner />; // hijacks the render output entirely
    return <Component {...props} />;
  };
}
```

[↑ Back to top](#table-of-contents)

---

### 11. What were React Mixins, and why were they deprecated? 🔴

- A pre-ES6-classes mechanism for sharing behavior between **class** components by merging in extra methods/lifecycle hooks — deprecated because mixins could silently introduce naming collisions, hidden dependencies between a component and its mixins, and made it hard to trace where a given piece of behavior actually came from. HOCs, then Hooks, were introduced as cleaner replacements.

[↑ Back to top](#table-of-contents)

---

### 12. How would you implement a HOC using render props? 🔴

- Have the HOC's wrapped component accept a function as `children`/`render`, combining both patterns: the HOC handles the logic, the render prop gives the consumer full control of the output.

```jsx
function withMouse(render) {
  return function Wrapped() {
    const [pos, setPos] = useState({ x: 0, y: 0 });
    return (
      <div onMouseMove={(e) => setPos({ x: e.clientX, y: e.clientY })}>
        {render(pos)}
      </div>
    );
  };
}
const MouseDisplay = withMouse((pos) => <p>{pos.x}, {pos.y}</p>);
```

[↑ Back to top](#table-of-contents)

---

### 13. What is a HOC factory implementation? 🔴

- A HOC that itself accepts **configuration arguments** before being applied to a component — a "factory that returns a HOC," letting the same enhancement logic be customized per use.

```jsx
function withMinRole(requiredRole) {
  return function (Component) {
    return function Wrapped(props) {
      if (props.user.role !== requiredRole) return <AccessDenied />;
      return <Component {...props} />;
    };
  };
}
const AdminPanel = withMinRole('admin')(Panel);
```

[↑ Back to top](#table-of-contents)

---

### 14. Why do component libraries need extra care when using `forwardRef`? 🔴

- A library's exported component is used in countless unknown contexts — if it doesn't forward refs properly, consumers can't access the underlying DOM node for things like focus management, measurements, or integrating with non-React code, and there's no good workaround for the consumer (unlike in app-level code, where you can just restructure your own components).
- Libraries typically forward refs **all the way through** to the actual lowest-level DOM element, even through multiple layers of internal composition, so consumers get the behavior they'd expect from a plain native element.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can't a ref be passed as a regular named prop instead of needing `forwardRef` at all?

[↑ Back to top](#table-of-contents)

---
