# Lesson 10 — React Fundamentals: Components, JSX, Props

> **Goal:** Begin Phase B. Learn React — the UI library that Next.js is built on. You already invented React's core idea by hand in Part 1 (data → `render()` → page). React makes that idea automatic, composable, and fast. Today: components, JSX, props, and rendering lists.

---

## 1. What React is, and why

Remember Part 1 Lesson 12's rule: *state lives in JS; the page is a rendering of it; on every change, call `render()`.* Your to-do app had a `render()` that wiped the list and rebuilt it with `createElement`/`appendChild`.

That approach hits limits fast:

- Rebuilding everything on each change is slow for big pages — and you lose focus/scroll/typing state when elements get replaced.
- With 30 pieces of state and 30 parts of the page, *which parts to update* becomes a nightmare of manual bookkeeping.
- Reusing UI (a "card" that appears 50 times with different data) means copying code.

**React** solves this with one idea: **you describe what the UI should look like for a given state, and React figures out the minimal DOM changes when the state changes.** You never call `appendChild` again. You write *"the page looks like this"* and React keeps the real DOM in sync.

Plus **components**: reusable, self-contained UI pieces with their own state — functions that return UI. A page becomes a tree of components, like HTML is a tree of elements.

Next.js is React + a server + routing + tooling. Everything today is 100% applicable there.

---

## 2. Setup with Vite

**Vite** is the standard tool for starting a React project (fast dev server, bundling):

```bash
npm create vite@latest tasks-ui -- --template react
cd tasks-ui
npm install
npm run dev
```

Open the printed URL (usually `http://localhost:5173`). You'll see a demo page. The files that matter:

```
tasks-ui/
├── index.html          # ONE html file, with <div id="root"></div>
├── src/
│   ├── main.jsx        # attaches React to #root
│   ├── App.jsx         # your top-level component
│   └── App.css / index.css
├── package.json
└── vite.config.js
```

```jsx
// src/main.jsx — you rarely touch this
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

Read it with Part 1 eyes: `document.getElementById("root")` — a DOM selection! React takes over that one element and renders your `App` inside it. Everything else on the page is React's.

Replace `App.jsx` with:

```jsx
function App() {
  return <h1>Hello, React!</h1>;
}

export default App;
```

Save — the browser updates instantly (hot reload). That's a **component**: a function that returns UI.

---

## 3. JSX — HTML-looking JavaScript

That `<h1>` inside a JavaScript function is **JSX**. It's not HTML; it's syntax that Vite compiles into function calls:

```jsx
<h1 className="title">Hello</h1>
// becomes roughly:
createElement("h1", { className: "title" }, "Hello")
```

— which is Part 1 Lesson 11's `document.createElement` at heart, just declarative. JSX rules that differ from HTML:

```jsx
function Profile() {
  const name = "Amina";
  const age = 15;
  const isOnline = true;

  return (
    <div className="card">                       {/* class → className (class is a JS keyword) */}
      <h2>{name}</h2>                             {/* { } = embed any JS expression, like ${} in template literals */}
      <p>Age next year: {age + 1}</p>
      <p>Status: {isOnline ? "🟢 online" : "⚫ offline"}</p>   {/* ternary for either/or */}
      <label htmlFor="bio">Bio</label>           {/* for → htmlFor */}
      <input id="bio" placeholder="..." />       {/* every tag closes: <input />, <img />, <br /> */}
      <button onClick={() => alert("hi")}>Wave</button>   {/* events: camelCase, pass a FUNCTION */}
    </div>
  );
}
```

- A component returns **one root element** (wrap siblings in a `<div>` or an empty **fragment** `<>...</>`).
- `{ }` holds *expressions* — values. Not statements: no `if`, no `for` inside braces. (Use ternaries, `&&`, or compute before the `return`.)
- `style` takes an object: `style={{ color: "red", fontSize: 18 }}` (double braces: outer = JSX slot, inner = object literal).
- Comments inside JSX: `{/* like this */}`.

### Conditional rendering

```jsx
function Greeting({ user }) {
  if (!user) return <p>Please log in.</p>;          // early return — plain JS before the JSX

  return (
    <div>
      <h2>Welcome, {user.name}</h2>
      {user.isAdmin && <span className="badge">Admin</span>}    {/* && = "render if true" */}
      {user.tasks.length === 0 ? <p>No tasks</p> : <p>{user.tasks.length} tasks</p>}
    </div>
  );
}
```

`cond && <El />` renders `<El />` when `cond` is truthy, nothing otherwise. ⚠️ If `cond` is `0`, React renders the `0`! Use `count > 0 && ...` rather than `count && ...`.

---

## 4. Components and props ⭐

Components are functions; **props** are their parameters. Make a `TaskItem` and use it three times:

```jsx
function TaskItem({ text, done }) {                // destructured props (Part 1 L14 — told you it'd matter)
  return (
    <li style={{ textDecoration: done ? "line-through" : "none" }}>
      {text}
    </li>
  );
}

function App() {
  return (
    <ul>
      <TaskItem text="Learn React" done={false} />     {/* attributes = props. Strings in quotes; anything else in { } */}
      <TaskItem text="Build the UI" done={true} />
      <TaskItem text="Deploy" done={false} />
    </ul>
  );
}
```

Rules:

- Component names are **Capitalized** — that's how React tells `<TaskItem>` (component) from `<li>` (HTML).
- Props flow **down** from parent to child. A child can't change its props (they're read-only, like a `const` parameter) — it only *renders* them.
- Any value can be a prop: strings, numbers, objects, arrays, functions (next lesson: passing callbacks up).
- `children` is a special prop — whatever you put *between* the tags:

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      {children}
    </div>
  );
}

<Card title="Today">
  <p>Anything in here becomes props.children</p>
</Card>
```

Components compose like Part 1 Lesson 6's functions calling functions: small pieces, one job each, clear names, assembled into bigger pieces. `App` → `TaskList` → `TaskItem`.

---

## 5. Rendering lists — `map` and `key` ⭐

The most important pattern in React, and it's Part 1 Lesson 9's `map`:

```jsx
const tasks = [
  { id: 1, text: "Learn React", done: false },
  { id: 2, text: "Build the UI", done: true },
  { id: 3, text: "Deploy", done: false },
];

function TaskList({ tasks }) {
  if (tasks.length === 0) return <p>No tasks yet.</p>;

  return (
    <ul>
      {tasks.map((task) => (
        <TaskItem key={task.id} text={task.text} done={task.done} />
      ))}
    </ul>
  );
}

function App() {
  return <TaskList tasks={tasks} />;
}
```

`tasks.map(task => <TaskItem ... />)` produces an *array of elements*, and JSX renders arrays. Your Part 1 `for…of` + `createElement` + `appendChild` loop became one expression.

### `key`

Every element in a mapped list needs a unique, stable `key` prop. React uses it to match old and new items when the list changes, so it can update the *right* DOM nodes instead of rebuilding all of them (and so typing in one item's input doesn't jump to another).

- ✅ Use a stable id from your data: `key={task.id}`.
- ❌ Don't use the array index (`key={i}`) — it breaks when items are reordered or removed.
- Missing key → React warns in the console. Fix it every time.

---

## 6. Where's the state? (preview)

Everything today was static data. Real apps change: the user adds a task, toggles it. In Part 1 that was `let tasks = []` + `render()`. In React, it's the `useState` hook — next lesson. The mental model stays identical: **state changes → UI re-renders from state.** React just does the re-rendering for you.

---

## ⚠️ Common mistakes

```
1. Lowercase component name → React treats it as an unknown HTML tag. Capitalize.
2. `class=` in JSX → warning; use className. Same for `for` → htmlFor.
3. Returning two sibling elements → wrap in <>...</>.
4. if/for inside { } → only expressions. Compute above the return or use ternary/&&/map.
5. onClick={doThing()} → calls it immediately (Part 1 L12 déjà vu). onClick={doThing} or onClick={() => doThing(x)}.
6. Missing key in a list → console warning; buggy updates later.
7. Forgetting to import a component / wrong relative path → "X is not defined" or module not found.
8. Editing index.html expecting content → the page is rendered by React from src/. Edit App.jsx.
```

---

## ✅ Classwork

1. Create the Vite project, run it, replace `App` with a hello. Add a second component `Header` that shows the app name and today's date (`new Date().toDateString()`), and use it inside `App`.
2. Build `Profile` from section 3 with your own data. Add a `style` object and a button with an `onClick` that logs to the console.
3. Build `TaskItem` + `TaskList` + `App` with a hardcoded array of 5 tasks. Remove the `key` and read the warning; add it back. Change one `done` to `true` and see the strikethrough.
4. Make a `Card` with `children`, and render two cards with different content.
5. Convert your Part 1 "profile page" homework (name, bio, hobbies list) into React components: `ProfilePage` → `HobbyList` → `HobbyItem`.

## 📝 Homework

1. **Product grid:** an array of 6 product objects `{ id, name, price, inStock, image }` (use placeholder image URLs like `https://picsum.photos/200?random=1`). Components: `ProductGrid` → `ProductCard`. Out-of-stock cards show a "Sold out" badge and a grayed style; prices formatted with `toLocaleString()`. Styling via `className` and a CSS file.
2. **Scoreboard:** array of `{ player, points }`; render sorted by points (do the sort *before* mapping — `[...players].sort(...)` — why the copy? Part 1 L8/L14: don't mutate props). Top player's row gets a `champion` class. Show "No players" when empty.
3. **Prop drilling exercise:** `App` holds a `user` object; pass it down `App → Layout → Sidebar → UserBadge` where only `UserBadge` uses it. Note in a comment how it feels (this pain motivates later tools; for now just feel it).
4. Written: in your own words — what's a component? What's a prop? What does `key` do? How does React's approach differ from your Part 1 `render()` function?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **React** | UI library: describe the UI for a state; React updates the DOM |
| **Vite** | Dev server + build tool for React projects |
| **component** | A function that returns UI (Capitalized name) |
| **JSX** | HTML-like syntax compiled to `createElement` calls |
| **`{ }`** | Embed a JS expression in JSX |
| **props** | Inputs to a component, passed as attributes, read-only |
| **`children`** | Content placed between a component's tags |
| **fragment `<>`** | Group siblings without an extra DOM element |
| **`key`** | Stable unique id for each item in a rendered list |
| **conditional rendering** | `cond ? a : b`, `cond && el`, early return |
