# Routing

_Part of [React](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is React Router, and why is it needed?](#1-what-is-react-router-and-why-is-it-needed)
- [2. How do you get query parameters in React Router?](#2-how-do-you-get-query-parameters-in-react-router)
- [3. How do you implement a default/NotFound page?](#3-how-do-you-implement-a-defaultnotfound-page)

**🟡 Medium**
- [4. What changed in React Router v6 compared to earlier versions?](#4-what-changed-in-react-router-v6-compared-to-earlier-versions)
- [5. How do you programmatically navigate in React Router?](#5-how-do-you-programmatically-navigate-in-react-router)
- [6. How do you read route params inside a component?](#6-how-do-you-read-route-params-inside-a-component)
- [7. How do you implement nested routes?](#7-how-do-you-implement-nested-routes)
- [8. What's the relationship between React Router and the `history` library?](#8-whats-the-relationship-between-react-router-and-the-history-library)

**🔴 Hard**
- [9. How do you implement a protected/private route?](#9-how-do-you-implement-a-protectedprivate-route)
- [10. Why might you get a "Router may have only one child element" warning?](#10-why-might-you-get-a-router-may-have-only-one-child-element-warning)
- [11. How do you perform an automatic redirect after a successful login?](#11-how-do-you-perform-an-automatic-redirect-after-a-successful-login)
- [12. How would you track page views for analytics on route changes?](#12-how-would-you-track-page-views-for-analytics-on-route-changes)

---

### 1. What is React Router, and why is it needed? 🟢

- The standard client-side routing library for React — React itself has no built-in router. It maps URL paths to components, enabling navigation between "pages" in a single-page app **without** a full page reload.

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
  </Routes>
</BrowserRouter>
```

[↑ Back to top](#table-of-contents)

---

### 2. How do you get query parameters in React Router? 🟢

- Use `useSearchParams` (v6) — returns a `URLSearchParams`-like object and a setter.

```jsx
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams] = useSearchParams();
  const query = searchParams.get('q'); // for /search?q=react
  return <p>Searching for: {query}</p>;
}
```

[↑ Back to top](#table-of-contents)

---

### 3. How do you implement a default/NotFound page? 🟢

- Add a route with path `*` as the last route — matches any URL that didn't match an earlier, more specific route.

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

[↑ Back to top](#table-of-contents)

---

### 4. What changed in React Router v6 compared to earlier versions? 🟡

- `<Switch>` replaced by `<Routes>`; route matching is no longer order-dependent exclusive-match-first, instead it picks the **best** match automatically.
- `component`/`render` props on `<Route>` replaced by a single `element` prop (`<Route element={<Home />} />`).
- New Hooks (`useNavigate` replacing `useHistory`, `useParams`, `useSearchParams`) instead of injecting `history`/`match`/`location` as props.
- Built-in support for relative, nested routes without manually prefixing paths.

[↑ Back to top](#table-of-contents)

---

### 5. How do you programmatically navigate in React Router? 🟡

- `useNavigate()` (v6) returns a function to navigate imperatively (e.g. after form submission), instead of relying only on `<Link>` clicks.

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  function handleSubmit() {
    login().then(() => navigate('/dashboard'));
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you read route params inside a component? 🟡

- `useParams()` returns an object of the dynamic segments matched in the current route's path.

```jsx
<Route path="/users/:userId" element={<UserProfile />} />

function UserProfile() {
  const { userId } = useParams();
  return <p>Viewing user {userId}</p>;
}
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you implement nested routes? 🟡

- Nest `<Route>` elements; the parent renders an `<Outlet />` where its matched child route should appear.

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route path="settings" element={<Settings />} />
  <Route path="profile" element={<Profile />} />
</Route>

function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <Outlet /> {/* renders Settings or Profile here, depending on the URL */}
    </div>
  );
}
```

[↑ Back to top](#table-of-contents)

---

### 8. What's the relationship between React Router and the `history` library? 🟡

- React Router is built on top of the `history` library, which provides the low-level abstraction for managing session history (push/replace/go) across different environments (browser, hash-based URLs, in-memory for tests/native) — React Router wraps it with components/Hooks tailored for React's component model.

[↑ Back to top](#table-of-contents)

---

### 9. How do you implement a protected/private route? 🔴

- Create a wrapper component that checks auth state and either renders the intended content (often via `<Outlet />` for nested protected routes) or redirects to a login page.

```jsx
function ProtectedRoute({ isAuthenticated }) {
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
}

<Route element={<ProtectedRoute isAuthenticated={isLoggedIn} />}>
  <Route path="/dashboard" element={<Dashboard />} />
</Route>
```

[↑ Back to top](#table-of-contents)

---

### 10. Why might you get a "Router may have only one child element" warning? 🔴

- A `<BrowserRouter>`/`<Router>` expects exactly **one** root child — usually thrown when you accidentally render multiple top-level elements directly inside it instead of wrapping them in a single `<Routes>` (or a single parent element).

```jsx
// Wrong
<BrowserRouter><Header /><Routes>...</Routes></BrowserRouter>

// Right — single child
<BrowserRouter>
  <>
    <Header />
    <Routes>...</Routes>
  </>
</BrowserRouter>
```

[↑ Back to top](#table-of-contents)

---

### 11. How do you perform an automatic redirect after a successful login? 🔴

- Combine `useNavigate` with the login flow's success callback, optionally redirecting back to the page the user originally tried to visit (captured via `location.state` before redirecting to `/login`).

```jsx
function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from?.pathname || '/dashboard';

  async function handleLogin() {
    await login();
    navigate(from, { replace: true });
  }
}
```

[↑ Back to top](#table-of-contents)

---

### 12. How would you track page views for analytics on route changes? 🔴

- Use `useLocation()` inside a `useEffect` at a high level in the app — since it changes on every navigation, the effect fires on each route change, letting you send a page-view event.

```jsx
function AnalyticsTracker() {
  const location = useLocation();
  useEffect(() => {
    sendPageView(location.pathname);
  }, [location]);
  return null;
}
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Where in the component tree should this tracker live so it reliably fires on every route change?

[↑ Back to top](#table-of-contents)

---
