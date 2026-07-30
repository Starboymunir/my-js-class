# Lesson 13 — Forms and Validation (+ localStorage)

> **Goal:** Handle the web's main way of collecting user data: forms. Read inputs properly, validate them, give friendly feedback — and save data with `localStorage` so it survives a refresh. This lesson ends with the course's first real app: the to-do list.

---

## 1. Form basics

A form groups inputs and has a special **submit** behavior:

```html
<form id="signup-form">
  <input id="username" type="text" placeholder="Username">
  <input id="email" type="email" placeholder="Email">
  <input id="age" type="number" placeholder="Age">

  <select id="level">
    <option value="">-- choose level --</option>
    <option value="beginner">Beginner</option>
    <option value="pro">Pro</option>
  </select>

  <input id="terms" type="checkbox"> I agree to the terms

  <button type="submit">Sign up</button>
</form>
<p id="message"></p>
```

Reading each kind of input:

```js
document.querySelector("#username").value    // text → string
document.querySelector("#age").value         // number input → STILL a string! Number() it.
document.querySelector("#level").value       // selected option's value ("" if the placeholder)
document.querySelector("#terms").checked     // checkbox → true/false (checked, not value!)
```

## 2. The `submit` event and `preventDefault`

Pressing the button (or Enter in a field) fires the form's `"submit"` event. But there's a trap: **by default, submitting reloads the entire page** — a leftover from the pre-JavaScript web. Your JS state would be wiped. So the first line of every submit handler:

```js
const form = document.querySelector("#signup-form");

form.addEventListener("submit", (event) => {
  event.preventDefault();          // ⭐ stop the reload — WE will handle this

  const username = document.querySelector("#username").value.trim();
  console.log(`Submitting: ${username}`);
});
```

Two habits shown here that should become automatic: `preventDefault()` first, and `.trim()` on every text field (users paste spaces constantly).

> Listen on the **form's** `"submit"` — not the button's `"click"`. Submit also catches the Enter key, which users expect to work.

---

## 3. Validation — never trust input ⭐

Users mistype. Users leave fields empty. Users type `"abc"` in the age box. **Validation** = checking data before accepting it, and telling the user clearly what to fix.

```js
const message = document.querySelector("#message");

function showError(text) {
  message.textContent = text;        // textContent — user-derived text, remember XSS!
  message.style.color = "red";
}

function showSuccess(text) {
  message.textContent = text;
  message.style.color = "green";
}

form.addEventListener("submit", (event) => {
  event.preventDefault();

  const username = document.querySelector("#username").value.trim();
  const email = document.querySelector("#email").value.trim();
  const age = Number(document.querySelector("#age").value);
  const agreed = document.querySelector("#terms").checked;

  // Guard clauses — early returns (Lesson 6!), one clear message per problem:
  if (username === "") {
    return showError("Username is required.");
  }
  if (username.length < 3) {
    return showError("Username must be at least 3 characters.");
  }
  if (!email.includes("@")) {
    return showError("That doesn't look like an email address.");
  }
  if (Number.isNaN(age) || age < 13) {
    return showError("Age must be a number, 13 or older.");
  }
  if (!agreed) {
    return showError("You must accept the terms.");
  }

  // All checks passed:
  showSuccess(`Welcome, ${username}! 🎉`);
  form.reset();                      // clear all fields
});
```

Notice the shape: *read → clean → validate with early returns → act on success*. It's Lesson 10's "validate and fail clearly" pattern wearing frontend clothes.

**Backend foreshadowing (important):** frontend validation is for *user convenience* — instant feedback. It provides **zero security**: anyone can bypass your page's JS entirely (DevTools makes it trivial). Real protection means the *server* re-validates everything it receives. When you learn backend, you will write these same checks again on the server — by design, not duplication. Rule of the industry: **never trust the client.**

*(Aside: HTML has built-in validation attributes — `required`, `min`, `maxlength`. Nice as a bonus layer, but learn the JS way first: real apps need custom rules and custom messages.)*

---

## 4. `localStorage` — data that survives refresh

Everything in variables dies on refresh. The browser offers a tiny persistent store per website: **localStorage**. It stores strings by key:

```js
localStorage.setItem("username", "amina");     // save
localStorage.getItem("username");              // "amina" — even after refresh, tomorrow, next week
localStorage.removeItem("username");           // delete
```

To store arrays/objects, convert to and from **JSON** (a string format for structured data — full story in Lesson 14):

```js
const tasks = [{ text: "Wash plates", done: false }];

localStorage.setItem("tasks", JSON.stringify(tasks));         // object → string → saved

const raw = localStorage.getItem("tasks");                     // string or null!
const loaded = raw ? JSON.parse(raw) : [];                     // string → object (ternary guards first visit)
```

That last line is a standard idiom: *if there's saved data, parse it; otherwise start empty.*

---

## 5. Capstone of Phase 2 — the To-Do List 📋

The classic first app, because it exercises **everything**: state, render, events, forms, validation, delegation, localStorage. Build it together in class, typing every line.

```html
<h1>My Tasks</h1>
<form id="task-form">
  <input id="task-input" placeholder="What needs doing?">
  <button type="submit">Add</button>
</form>
<p id="form-error"></p>
<ul id="task-list"></ul>
<p id="stats"></p>
```

```css
.done { text-decoration: line-through; color: gray; }
.delete-btn { margin-left: 8px; }
```

```js
// ===== STATE =====
let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
// each task: { text: "Wash plates", done: false }

// ===== ELEMENTS =====
const form = document.querySelector("#task-form");
const input = document.querySelector("#task-input");
const list = document.querySelector("#task-list");
const stats = document.querySelector("#stats");
const formError = document.querySelector("#form-error");

// ===== PERSISTENCE =====
function save() {
  localStorage.setItem("tasks", JSON.stringify(tasks));
}

// ===== RENDER =====
function render() {
  list.innerHTML = "";

  tasks.forEach((task, index) => {
    const li = document.createElement("li");
    li.textContent = task.text;
    li.dataset.index = index;              // stash the index on the element itself
    if (task.done) li.classList.add("done");

    const del = document.createElement("button");
    del.textContent = "✖";
    del.classList.add("delete-btn");
    li.appendChild(del);

    list.appendChild(li);
  });

  const doneCount = tasks.filter((t) => t.done).length;
  stats.textContent = `${doneCount} of ${tasks.length} done`;
}

// ===== EVENTS =====
// Add a task
form.addEventListener("submit", (event) => {
  event.preventDefault();
  const text = input.value.trim();

  if (text === "") {
    formError.textContent = "Task cannot be empty!";
    return;
  }
  formError.textContent = "";

  tasks.push({ text: text, done: false });
  input.value = "";
  save();
  render();
});

// Toggle / delete — one delegated listener for all (current AND future) items
list.addEventListener("click", (event) => {
  const li = event.target.closest("li");
  if (!li) return;
  const index = Number(li.dataset.index);

  if (event.target.classList.contains("delete-btn")) {
    tasks.splice(index, 1);               // remove 1 item at that index
  } else {
    tasks[index].done = !tasks[index].done;   // flip the boolean
  }
  save();
  render();
});

// ===== START =====
render();
```

New bits worth pausing on:

- **`dataset`** — `li.dataset.index = i` writes a `data-index="0"` attribute; reading it back tells the delegated handler *which* task was clicked. Custom data on elements: an everyday frontend tool.
- **`closest("li")`** — from whatever was clicked (maybe the ✖ button inside the li), walk up to the nearest `<li>`.
- **`splice(index, 1)`** — remove items from an array by position (this one *mutates*).
- The rhythm of every handler: **change `tasks` → `save()` → `render()`**. State first, display follows. Always.

Refresh the page — tasks are still there. That's a real app. 🎉

---

## ⚠️ Common mistakes

```js
// 1. Forgetting preventDefault → page reloads and everything "mysteriously resets"

// 2. Listening to the button's click instead of the form's submit → Enter key doesn't work

// 3. Math/comparisons on .value without Number() — "9" > "10" is true (string compare)!

// 4. checkbox.value instead of checkbox.checked

// 5. Storing objects in localStorage without JSON.stringify → "[object Object]" garbage

// 6. JSON.parse(null) crash on first visit → guard with the || [] idiom

// 7. Editing the DOM as storage — e.g. counting <li>s to know task count. The ARRAY is the truth!
```

---

## ✅ Classwork

1. Build the signup form from sections 1–3 exactly, then extend validation: username may not contain spaces (`includes(" ")`), and level must be chosen (its value isn't `""`).
2. Save the successful signup's data as an object to localStorage; on page load, if it exists, greet the user by name instead of showing the form (toggle `hidden` classes).
3. **Build the to-do list together, live** — teacher types nothing the student can type. Expect this to take a full session. Test: add, complete, delete, refresh, empty-input error.

## 📝 Homework

1. **Extend the to-do app** (pick at least two):
   - A "Clear completed" button (`filter`!)
   - Task counter in the tab title (`document.title`)
   - Filter buttons: All / Active / Done (hint: a `filterMode` state variable that `render()` respects)
   - Edit a task by double-clicking it (`dblclick` event, `prompt()` is acceptable)
2. **Guest book:** a form (name + message), entries rendered newest-first and persisted in localStorage. Validation: both fields required, message ≤ 200 chars with a live character counter (`input` event — Lesson 12!).
3. **Written (comments):** why must servers re-validate data even when the frontend already validated it? Why do we keep tasks in an array instead of just reading the page's `<li>` elements when we need them?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **`submit` event** | Fires on button press AND Enter — listen on the form |
| **`preventDefault()`** | Stop the built-in page reload |
| **validation** | Check + clean input, fail with helpful messages |
| **guard clause** | Early `return` on invalid input |
| **`checked`** | A checkbox's true/false (not `.value`) |
| **`localStorage`** | Per-site persistent string storage |
| **`JSON.stringify` / `JSON.parse`** | Object → string / string → object |
| **`dataset`** | Custom `data-*` attributes on elements |
| **"never trust the client"** | Servers must re-validate everything — frontend checks are UX, not security |
