# Testing

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is Jest, and why is it commonly used with React?](#1-what-is-jest-and-why-is-it-commonly-used-with-react)
- [2. Can you give a simple example of a Jest test case?](#2-can-you-give-a-simple-example-of-a-jest-test-case)
- [3. What is snapshot testing?](#3-what-is-snapshot-testing)

**🟡 Medium**
- [4. What is the Shallow Renderer, and when would you use it?](#4-what-is-the-shallow-renderer-and-when-would-you-use-it)
- [5. What is React Testing Library, and how does its philosophy differ from shallow rendering?](#5-what-is-react-testing-library-and-how-does-its-philosophy-differ-from-shallow-rendering)
- [6. How do you mock a function or module in a Jest test?](#6-how-do-you-mock-a-function-or-module-in-a-jest-test)
- [7. How do you test a component that fetches data asynchronously?](#7-how-do-you-test-a-component-that-fetches-data-asynchronously)

**🔴 Hard**
- [8. What is the `TestRenderer` package used for?](#8-what-is-the-testrenderer-package-used-for)
- [9. How do you test a custom Hook in isolation?](#9-how-do-you-test-a-custom-hook-in-isolation)
- [10. What are the advantages of Jest over older test runners like Jasmine?](#10-what-are-the-advantages-of-jest-over-older-test-runners-like-jasmine)

---

### 1. What is Jest, and why is it commonly used with React? 🟢

- A JS testing framework (test runner + assertion library + mocking utilities, all in one) created by Facebook/Meta — the default choice for React projects, including Create React App, since it requires minimal setup and integrates well with JSX/Babel.

[↑ Back to top](#table-of-contents)

---

### 2. Can you give a simple example of a Jest test case? 🟢

```jsx
function sum(a, b) { return a + b; }

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

[↑ Back to top](#table-of-contents)

---

### 3. What is snapshot testing? 🟢

- Renders a component, serializes its output to a text "snapshot" file, and on future test runs compares the new output against the saved one — flags the test if the rendered output **unexpectedly** changed.

```jsx
import { render } from '@testing-library/react';

test('renders correctly', () => {
  const { container } = render(<Greeting name="Vaibhav" />);
  expect(container).toMatchSnapshot();
});
```

> [!IMPORTANT]
> **Follow-up questions:**
> - What's the risk of relying too heavily on snapshot tests?

[↑ Back to top](#table-of-contents)

---

### 4. What is the Shallow Renderer, and when would you use it? 🟡

- Renders a component **one level deep** — child components render as placeholders, not their actual output — useful for testing a component's own logic/output in isolation, without triggering child components' behavior/side effects too.
- Historically provided by Enzyme's `shallow()`; React's own `react-test-renderer/shallow` offers a lower-level version. Less commonly used today in favor of React Testing Library's approach (Q5).

[↑ Back to top](#table-of-contents)

---

### 5. What is React Testing Library, and how does its philosophy differ from shallow rendering? 🟡

- A testing utility built around one guiding principle: **test components the way a user actually interacts with them** (find elements by visible text/role, fire real click/type events) — rather than reaching into component internals (props, state, instance methods) the way Enzyme's shallow rendering encouraged.
- This makes tests more resilient to internal refactors (switching class to function component, renaming internal state) since they only break if the actual user-facing behavior breaks.

```jsx
import { render, screen, fireEvent } from '@testing-library/react';

test('increments counter on click', () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you mock a function or module in a Jest test? 🟡

```jsx
jest.mock('./api'); // auto-mocks every export in ./api
import { fetchUser } from './api';

test('shows user name', async () => {
  fetchUser.mockResolvedValue({ name: 'Vaibhav' });
  // ...render component that calls fetchUser, assert it shows "Vaibhav"
});
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you test a component that fetches data asynchronously? 🟡

- Mock the data-fetching call, render the component, then use an async query (`findBy...`) or `waitFor` to wait until the UI updates after the promise resolves, before asserting.

```jsx
import { render, screen } from '@testing-library/react';

test('shows fetched user', async () => {
  render(<UserProfile userId={1} />);
  expect(await screen.findByText('Vaibhav')).toBeInTheDocument(); // waits for async update
});
```

[↑ Back to top](#table-of-contents)

---

### 8. What is the `TestRenderer` package used for? 🔴

- `react-test-renderer` renders React components to plain JS objects (a JSON-like tree) **without** needing a real DOM or browser — used internally for snapshot testing, and useful for testing React Native components, which have no DOM at all.

[↑ Back to top](#table-of-contents)

---

### 9. How do you test a custom Hook in isolation? 🔴

- Use `renderHook` from React Testing Library (`@testing-library/react`) — it mounts a tiny throwaway component internally just to execute the Hook, giving you back its return value and a way to trigger re-renders.

```jsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('increments count', () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

[↑ Back to top](#table-of-contents)

---

### 10. What are the advantages of Jest over older test runners like Jasmine? 🔴

- Built-in mocking, snapshot testing, and parallel test execution out of the box (Jasmine needs extra libraries for mocking/spies).
- Zero-config for most projects, faster watch mode (only re-runs tests related to changed files), and tightly integrated with the React/JS ecosystem (Babel/TS transforms, coverage reports).

> [!IMPORTANT]
> **Follow-up questions:**
> - Why has the React community largely moved from Enzyme to React Testing Library over the past few years?

[↑ Back to top](#table-of-contents)

---
