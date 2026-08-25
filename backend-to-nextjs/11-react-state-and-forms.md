# Lesson 11 — React State, Events, and Forms

> **Goal:** Make components interactive with `useState`, handle events, build controlled forms, and share state between components by "lifting" it. By the end you'll rebuild the to-do app in React — and it will be shorter and clearer than the Part 1 version.

---

## 1. `useState` — memory that triggers re-rendering

In Part 1, state was a plain variable plus a manual `render()` call. In React, **state is a special variable that, when changed, makes the component re-render automatically.**

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);        // [current value, function to change it], starting at 0

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>−</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

`useState(0)` returns a pair (array destructuring!): the value and a setter. Calling the setter **schedules a re-render** with the new value. React calls `Counter()` again, `count` is now the new number, the JSX is recomputed, and React updates just the `<h1>` text in the DOM.

Compare with Part 1 Lesson 12's counter: same state, same buttons — but no `render()` function and no `display.textContent = count`. That is React's entire job: *you change state, it updates the page.*

### Rules of hooks

`useState` is a **hook** (functions starting with `use`). Two rules, enforced by React:

1. Call hooks only at the **top level** of a component — not inside `if`, loops, or nested functions.
2. Call hooks only inside **components** (or other hooks).

React tracks hooks by call order, so they must run in the same order every render.

### State is immutable — replace, don't mutate ⭐

```jsx
const [user, setUser] = useState({ name: "Amina", age: 15 });

user.age = 16;                          // ❌ mutation — React doesn't notice, no re-render
setUser({ ...user, age: 16 });          // ✅ a NEW object — React sees a change

const [tasks, setTasks] = useState([]);
tasks.push(newTask);                    // ❌ same array reference — no re-render
setTasks([...tasks, newTask]);          // ✅ new array
```

React compares the old and new value by reference (Part 1 L8!). Mutating in place keeps the same reference, so React thinks nothing changed. **Every state update creates a new object/array** — this is where Part 1 Lesson 14's spread patterns (`{ ...obj, field }`, `[...arr, item]`, `arr.filter(...)`, `arr.map(...)`) become daily tools:

| Operation | Immutable way |
|-----------|---------------|
| Add item | `setTasks([...tasks, task])` |
| Remove item | `setTasks(tasks.filter((t) => t.id !== id))` |
| Update item | `setTasks(tasks.map((t) => (t.id === id ? { ...t, done: !t.done } : t)))` |
| Update object field | `setUser({ ...user, name: "New" })` |

### Updates based on the previous value

When the new state depends on the old, pass a function — it's guaranteed the latest value even if several updates are queued:

```jsx
setCount((prev) => prev + 1);
setTasks((prev) => [...prev, task]);
```

Habit: use the function form whenever you're computing from the current state.

---

## 2. Events in React

Same events as Part 1, camelCased, receiving the same event object:

```jsx
<button onClick={handleClick}>Save</button>
<input onChange={(e) => setName(e.target.value)} />
<form onSubmit={handleSubmit}>
<div onMouseEnter={() => setHover(true)}>
<input onKeyDown={(e) => e.key === "Enter" && submit()} />
```

Pass the function, don't call it. To pass an argument, wrap in an arrow: `onClick={() => deleteTask(task.id)}`.

---

## 3. Controlled forms ⭐

In Part 1, the input held the text and you read `input.value` when needed. In React, the preferred pattern is **controlled inputs**: React state is the single source of truth, and the input just displays it.

```jsx
function AddTaskForm() {
  const [text, setText] = useState("");
  const [error, setError] = useState("");

  function handleSubmit(e) {
    e.preventDefault();                                   // still needed! (Part 1 L13)
    const trimmed = text.trim();
    if (trimmed === "") {
      setError("Task cannot be empty");
      return;
    }
    console.log("Would add:", trimmed);
    setText("");                                          // clearing state clears the input
    setError("");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}                                      // input shows state...
        onChange={(e) => setText(e.target.value)}         // ...and every keystroke updates state
        placeholder="What needs doing?"
      />
      <button type="submit">Add</button>
      {error && <p className="error">{error}</p>}
    </form>
  );
}
```

The loop: keystroke → `onChange` → `setText` → re-render → `value={text}` shows the new text. It feels circular but it means the state *always* matches the screen, live validation is trivial (it's just state), and clearing/resetting is `setText("")`.

Other inputs:

```jsx
<input type="checkbox" checked={done} onChange={(e) => setDone(e.target.checked)} />
<select value={priority} onChange={(e) => setPriority(e.target.value)}>
  <option value="low">Low</option>
  <option value="high">High</option>
</select>
<textarea value={notes} onChange={(e) => setNotes(e.target.value)} />
```

Multiple fields — one object state:

```jsx
const [form, setForm] = useState({ email: "", password: "" });

function handleChange(e) {
  const { name, value } = e.target;
  setForm((prev) => ({ ...prev, [name]: value }));     // computed key — Part 1 L8 bracket notation!
}

<input name="email" value={form.email} onChange={handleChange} />
<input name="password" type="password" value={form.password} onChange={handleChange} />
```

---

## 4. Lifting state up — sharing between components ⭐

`AddTaskForm` collects a task; `TaskList` displays tasks. They're siblings — how does the form's data reach the list? **Move the state to their closest common parent** and pass it down as props, with callback props for changes:

```
        App            ← owns `tasks` state
       /    \
AddTaskForm  TaskList  ← form calls onAdd(text); list receives tasks + onToggle/onDelete
                 |
              TaskItem
```

```jsx
function App() {
  const [tasks, setTasks] = useState([]);

  function addTask(text) {
    setTasks((prev) => [...prev, { id: Date.now(), text, done: false }]);
  }
  function toggleTask(id) {
    setTasks((prev) => prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t)));
  }
  function deleteTask(id) {
    setTasks((prev) => prev.filter((t) => t.id !== id));
  }

  return (
    <div>
      <h1>My Tasks</h1>
      <AddTaskForm onAdd={addTask} />
      <TaskList tasks={tasks} onToggle={toggleTask} onDelete={deleteTask} />
      <p>{tasks.filter((t) => t.done).length} of {tasks.length} done</p>
    </div>
  );
}

function AddTaskForm({ onAdd }) {
  const [text, setText] = useState("");
  function handleSubmit(e) {
    e.preventDefault();
    if (!text.trim()) return;
    onAdd(text.trim());                                    // tell the parent
    setText("");
  }
  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button>Add</button>
    </form>
  );
}

function TaskList({ tasks, onToggle, onDelete }) {
  if (tasks.length === 0) return <p>Nothing to do 🎉</p>;
  return (
    <ul>
      {tasks.map((task) => (
        <TaskItem key={task.id} task={task} onToggle={onToggle} onDelete={onDelete} />
      ))}
    </ul>
  );
}

function TaskItem({ task, onToggle, onDelete }) {
  return (
    <li>
      <input type="checkbox" checked={task.done} onChange={() => onToggle(task.id)} />
      <span style={{ textDecoration: task.done ? "line-through" : "none" }}>{task.text}</span>
      <button onClick={() => onDelete(task.id)}>✖</button>
    </li>
  );
}
```

**Data down, events up.** Props carry state downward; callback props carry user actions upward to whoever owns the state. This is the React architecture, and it's Part 1's to-do app with the `render()` plumbing removed.

Notice what disappeared from the Part 1 version: `querySelector`, `createElement`, `appendChild`, `innerHTML = ""`, `dataset.index`, event delegation, `render()` calls. What stayed: the state shape, the immutable array operations, `preventDefault`, validation, the `filter` for stats. The *thinking* transferred; the plumbing vanished.

### Where should state live?

Put it in the **lowest** component that still lets everyone who needs it reach it. Form text: local to the form. The task list: the parent. Logged-in user: near the top. If you find yourself passing props through many layers that don't use them ("prop drilling"), that's a signal for **Context** (a React feature for sharing data tree-wide — look it up when you feel the pain; not needed for this course's projects).

---

## 5. Persistence with localStorage (bridge to next lesson)

To save tasks across refreshes, you need to run code *when state changes* and *when the component first appears*. That's a side effect — `useEffect`, next lesson. Preview: initialize state from storage lazily:

```jsx
const [tasks, setTasks] = useState(() => JSON.parse(localStorage.getItem("tasks")) ?? []);
```

(Passing a function to `useState` runs it only on the first render — Part 1 L13's `|| []` idiom, React edition.) Saving on change is next lesson.

---

## ⚠️ Common mistakes

```
1. Mutating state (push, obj.x = ...) → no re-render. Always create new objects/arrays.
2. Reading state right after setting it → still the old value in this render. State updates apply on the NEXT render.
     setCount(count + 1); console.log(count);   // old value — by design
3. Hooks inside conditions/loops → "Rendered more hooks than during the previous render".
4. Input without value/onChange pair → "changing an uncontrolled input to controlled" warning. Always both.
5. onChange={setText(e.target.value)} → called during render → infinite loop. Wrap in an arrow.
6. Date.now() keys colliding when adding fast → fine for practice; the server will assign real ids (next lesson).
7. Forgetting e.preventDefault() in onSubmit → page reload, state gone.
```

---

## ✅ Classwork

1. Build `Counter`. Add a `step` input (controlled, Number()!) that controls how much +/− change. Show "Negative!" in red when below 0.
2. Build `AddTaskForm` standalone with validation (empty → error; > 100 chars → error; live character count).
3. Build the full to-do app from section 4, typing every component. Add the stats line and an empty state.
4. Refactor: pass a `filter` state ("all" / "active" / "done") from `App` into `TaskList` with three buttons in a `FilterBar` component. Which component owns `filter`? Why?
5. **Trace:** add `console.log("App rendered")` at the top of `App`. Type in the form — does `App` re-render? Toggle a task — does it? Explain.

## 📝 Homework

1. **To-do extensions:** edit-in-place (double-click text → becomes an input; Enter saves, Escape cancels — local `isEditing` state in `TaskItem`), "Clear completed", and a "Mark all done" toggle.
2. **Signup form:** the Part 1 Lesson 13 signup (username, email, age, level select, terms checkbox) as a controlled React form with a single `form` object state, per-field errors shown under each field, and a disabled submit button until valid.
3. **Shopping cart:** `products` array (static) + `cart` state (`[{ productId, qty }]`). `ProductCard` has "Add to cart"; `Cart` shows lines with +/− qty, remove, and total. All updates immutable. Lift state to `App`.
4. Written: explain "data down, events up" with the to-do app as the example. Why must state updates create new objects? What happens if you call a hook inside an `if`?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **`useState`** | Hook: `[value, setValue]`; setting triggers re-render |
| **hook** | `use*` function; call at top level of components only |
| **re-render** | React calls your component again with new state and updates the DOM |
| **immutable update** | New object/array via spread/map/filter — never mutate state |
| **functional update** | `setX((prev) => ...)` when the next value depends on the previous |
| **controlled input** | `value={state}` + `onChange={...}` — state is the source of truth |
| **lifting state** | Move shared state to the closest common parent |
| **callback prop** | A function passed down so a child can report events up (`onAdd`) |
| **data down, events up** | The React data-flow rule |
