# Lesson 12 — React Effects and Data Fetching: Connecting to Your API

> **Goal:** Learn `useEffect` for side effects, fetch data from *your own* Express API (Lesson 9) with loading/error states, handle login with tokens in React, and extract reusable logic into custom hooks. Both halves of the diagram are now yours: a React frontend talking to your Node backend.

---

## 1. Side effects and `useEffect`

Rendering should be pure: given props and state, return JSX. But apps also need to *do things* that aren't rendering — fetch data, save to localStorage, set a document title, start a timer. These are **side effects**, and React gives them a home: `useEffect`.

```jsx
import { useState, useEffect } from "react";

function Clock() {
  const [now, setNow] = useState(new Date());

  useEffect(() => {
    const id = setInterval(() => setNow(new Date()), 1000);   // the effect: start a timer
    return () => clearInterval(id);                            // cleanup: stop it when the component leaves
  }, []);                                                      // dependencies: [] = run once, after first render

  return <p>{now.toLocaleTimeString()}</p>;
}
```

Anatomy: `useEffect(effectFunction, dependencies)`.

- The **effect** runs *after* React has updated the DOM.
- The **dependency array** decides *when* it runs again:
  - `[]` → once, after the first render (mount).
  - `[a, b]` → after the first render, and again whenever `a` or `b` changes.
  - omitted → after *every* render (rarely what you want).
- The optional **cleanup** function returned from the effect runs before the effect re-runs and when the component is removed (unmount). Use it to cancel timers, subscriptions, and stale requests.

### Syncing state to localStorage

```jsx
const [tasks, setTasks] = useState(() => JSON.parse(localStorage.getItem("tasks")) ?? []);

useEffect(() => {
  localStorage.setItem("tasks", JSON.stringify(tasks));
}, [tasks]);                                   // whenever tasks changes, save it
```

Read "whenever `tasks` changes, save it" — that's the effect's dependency array in English. Compare with Part 1 L13 where you had to remember to call `save()` in every handler. Here, one effect covers every change forever.

> **Dev-mode double run:** in development, `<StrictMode>` runs every effect *twice* on mount to help catch missing cleanups. You'll see two fetches / two logs. It's intentional and doesn't happen in production. Don't remove StrictMode — write proper cleanups.

---

## 2. Fetching data in an effect ⭐

The three-state pattern (loading / success / error) from Part 1 L16, now as state:

```jsx
function TaskList() {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;                                   // guard against updates after unmount

    async function load() {
      try {
        setLoading(true);
        const res = await fetch("http://localhost:3000/api/tasks");
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data = await res.json();
        if (!cancelled) setTasks(data);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }
    load();

    return () => { cancelled = true; };
  }, []);

  if (loading) return <p>Loading…</p>;
  if (error) return <p className="error">Error: {error}</p>;
  if (tasks.length === 0) return <p>No tasks yet.</p>;
  return <ul>{tasks.map((t) => <li key={t.id}>{t.text}</li>)}</ul>;
}
```

Note: the effect function itself can't be `async` (it must return a cleanup or nothing), so define an async function inside and call it. The `cancelled` flag prevents "setState on an unmounted component" if the user navigates away mid-request.

### CORS — the error you will hit

Your React dev server is on `localhost:5173`; the API is on `localhost:3000`. Different ports = different **origins**, and browsers block cross-origin requests unless the *server* allows them. That's why Lesson 8 installed `cors`:

```js
// in the API
app.use(cors({ origin: "http://localhost:5173" }));     // or process.env.CLIENT_ORIGIN
```

"Blocked by CORS policy" in the console → fix it on the server, not the client. (In Lesson 17 you'll see Next.js sidesteps this entirely by putting frontend and backend on one origin.)

---

## 3. An API client module

Fetch calls scattered across components get messy. Centralize them — a pattern you'll keep in every project:

```js
// src/api.js
const BASE = import.meta.env.VITE_API_URL ?? "http://localhost:3000";   // Vite env vars: VITE_ prefix, in .env

function getToken() {
  return localStorage.getItem("token");
}

export async function apiFetch(path, options = {}) {
  const token = getToken();
  const res = await fetch(`${BASE}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token && { Authorization: `Bearer ${token}` }),     // Lesson 8's header, automatically
      ...options.headers,
    },
  });

  if (res.status === 204) return null;
  const data = await res.json().catch(() => null);
  if (!res.ok) throw new Error(data?.error ?? data?.errors?.join(", ") ?? `HTTP ${res.status}`);
  return data;
}

// Typed-looking helpers for each resource
export const tasksApi = {
  list: () => apiFetch("/api/tasks"),
  create: (text) => apiFetch("/api/tasks", { method: "POST", body: JSON.stringify({ text }) }),
  update: (id, data) => apiFetch(`/api/tasks/${id}`, { method: "PATCH", body: JSON.stringify(data) }),
  remove: (id) => apiFetch(`/api/tasks/${id}`, { method: "DELETE" }),
};

export const authApi = {
  login: (email, password) => apiFetch("/api/auth/login", { method: "POST", body: JSON.stringify({ email, password }) }),
  register: (data) => apiFetch("/api/auth/register", { method: "POST", body: JSON.stringify(data) }),
  me: () => apiFetch("/api/auth/me"),
};
```

⚠️ Vite exposes env vars with the `VITE_` prefix to the browser — so **never** put secrets in a frontend `.env`. Only public config (API URL) belongs there. Part 1 rule, still true: *anything in the browser is public.*

---

## 4. The connected to-do app

State now lives on the server; React state is a **cache** of it. Each handler: call the API → on success, update local state to match.

```jsx
import { useState, useEffect } from "react";
import { tasksApi } from "./api";

function TasksPage() {
  const [tasks, setTasks] = useState([]);
  const [status, setStatus] = useState("loading");     // "loading" | "ready" | "error"
  const [error, setError] = useState("");

  useEffect(() => {
    tasksApi.list()
      .then((data) => { setTasks(data); setStatus("ready"); })
      .catch((err) => { setError(err.message); setStatus("error"); });
  }, []);

  async function addTask(text) {
    const created = await tasksApi.create(text);                // server assigns the real id
    setTasks((prev) => [created, ...prev]);
  }

  async function toggleTask(task) {
    const updated = await tasksApi.update(task.id, { done: !task.done });
    setTasks((prev) => prev.map((t) => (t.id === task.id ? updated : t)));
  }

  async function deleteTask(id) {
    await tasksApi.remove(id);
    setTasks((prev) => prev.filter((t) => t.id !== id));
  }

  if (status === "loading") return <p>Loading…</p>;
  if (status === "error") return <p className="error">{error}</p>;

  return (
    <>
      <AddTaskForm onAdd={addTask} />
      <TaskList tasks={tasks} onToggle={toggleTask} onDelete={deleteTask} />
    </>
  );
}
```

Wrap the handlers' `await`s in try/catch and surface errors (a toast, an inline message) — homework. For extra polish, **optimistic updates**: update local state first, call the API, roll back on failure. Feels instant.

### Login in React

```jsx
function LoginForm({ onLoggedIn }) {
  const [form, setForm] = useState({ email: "", password: "" });
  const [error, setError] = useState("");

  async function handleSubmit(e) {
    e.preventDefault();
    try {
      const { token, user } = await authApi.login(form.email, form.password);
      localStorage.setItem("token", token);
      onLoggedIn(user);
    } catch (err) {
      setError(err.message);                          // "Invalid email or password" from your API
    }
  }
  // ...controlled inputs as in Lesson 11
}

function App() {
  const [user, setUser] = useState(null);
  const [checking, setChecking] = useState(true);

  useEffect(() => {                                   // on load: do we have a valid token?
    authApi.me()
      .then(setUser)
      .catch(() => localStorage.removeItem("token"))
      .finally(() => setChecking(false));
  }, []);

  function logout() {
    localStorage.removeItem("token");
    setUser(null);
  }

  if (checking) return <p>Loading…</p>;
  if (!user) return <LoginForm onLoggedIn={setUser} />;
  return (
    <>
      <header>Hi {user.name} <button onClick={logout}>Log out</button></header>
      <TasksPage />
    </>
  );
}
```

*(Tokens in localStorage are fine for learning. Production apps often prefer httpOnly cookies, which JavaScript can't read — XSS protection. You'll do that in Next.js, Lesson 17.)*

---

## 5. Custom hooks — reusing logic

The loading/error/data trio appears in every fetching component. Extract it. A **custom hook** is just a function that uses hooks:

```jsx
// src/hooks/useFetch.js
import { useState, useEffect } from "react";
import { apiFetch } from "../api";

export function useFetch(path) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    apiFetch(path)
      .then((d) => { if (!cancelled) { setData(d); setError(null); } })
      .catch((e) => { if (!cancelled) setError(e.message); })
      .finally(() => { if (!cancelled) setLoading(false); });
    return () => { cancelled = true; };
  }, [path]);                                          // re-fetch if the path changes

  return { data, loading, error };
}
```

```jsx
function ProjectList() {
  const { data: projects, loading, error } = useFetch("/api/projects");
  if (loading) return <p>Loading…</p>;
  if (error) return <p>{error}</p>;
  return <ul>{projects.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

Components become tiny; the logic lives once. Custom hooks are how React code gets organized at scale — `useAuth`, `useLocalStorage`, `useDebounce`... (Real projects often use a library like **TanStack Query** for server data — caching, refetching, mutations. Know it exists; hand-rolling first teaches what it automates.)

---

## ⚠️ Common mistakes

```
1. Missing dependency array → effect runs after EVERY render → if it sets state → infinite loop.
2. Async effect function: useEffect(async () => ...) ❌ — define an inner async function.
3. CORS error → fix on the SERVER (cors middleware with your dev origin).
4. Secrets in VITE_ env vars → shipped to every browser. Public config only.
5. Setting state after unmount → the cancelled flag / cleanup.
6. Effect depends on a value but the array is [] → stale data. Include what you use (the linter warns — listen).
7. Updating local state before checking the API succeeded → UI lies. Await, then update (or do optimistic + rollback deliberately).
8. Two fetches in dev → StrictMode. Not a bug.
```

---

## ✅ Classwork

1. Build `Clock` with cleanup. Remove the cleanup and toggle the component on/off with a button — watch the console for timers piling up. Restore it.
2. Add localStorage sync to the Lesson 11 to-do via `useEffect`. Refresh — tasks persist.
3. Start your `tasks-api` (Lesson 9) locally, set CORS, and build `TaskList` fetching from it with loading/error/empty states. Stop the API and watch the error state appear.
4. Write `api.js` with `apiFetch` and `tasksApi`, then wire the full connected to-do (add, toggle, delete). Watch the Network tab: every action is a request you built in Phase A.
5. Extract `useFetch`; use it for projects.

## 📝 Homework

1. **Auth flow:** `LoginForm`, `RegisterForm` (toggle between them), `App` checking the token on load, logout. Show API validation errors under the form. Header shows the user's name.
2. **Error handling & UX:** wrap every mutation in try/catch and show a dismissible error banner. Disable the Add button while the request is in flight (`submitting` state). Implement optimistic toggle with rollback on failure (test by stopping the API mid-session).
3. **Deployed API:** point `VITE_API_URL` at your Render URL (add the Vite dev origin to `CLIENT_ORIGIN`). Your React app now talks to your live backend. Then deploy the React app itself (Vercel or Netlify — `npm run build`, upload/connect the repo; set `VITE_API_URL` there) and update `CLIENT_ORIGIN` to the deployed frontend's URL. **Both halves live.**
4. **Custom hook:** `useLocalStorage(key, initial)` returning `[value, setValue]` that behaves like `useState` but persists. Use it for the filter setting.
5. Written: what's a side effect? What do `[]`, `[x]`, and no array mean? Why does CORS exist and why is it the server's job? Where do tokens live in this app, and what's the risk?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **side effect** | Anything beyond returning JSX: fetch, timers, storage, document title |
| **`useEffect(fn, deps)`** | Run `fn` after render, when `deps` change |
| **cleanup** | Function returned from an effect; runs before re-run and on unmount |
| **mount / unmount** | Component appears / disappears |
| **StrictMode** | Dev-only double-invocation to catch bugs |
| **origin / CORS** | scheme+host+port; server grants cross-origin access |
| **`VITE_` env vars** | Public config exposed to the browser — never secrets |
| **API client module** | One place for fetch + headers + error handling |
| **optimistic update** | Update UI first, roll back on failure |
| **custom hook** | A `useX` function composing other hooks for reuse |
