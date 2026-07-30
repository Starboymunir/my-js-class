# Lesson 11 — The DOM: JavaScript Meets the Web Page

> **Goal:** Welcome to Phase 2 — the frontend! Everything from Lessons 1–10 now gets a visible playground: the web page itself. Today you learn to select parts of a page and change them with code.
>
> **Prerequisite check:** you should be comfortable with functions, arrays, objects, and loops. If not, review first — DOM code *uses* all of them constantly.

---

## 0. A 15-minute HTML refresher

JavaScript manipulates HTML, so you need to read basic HTML. Crash summary:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Shown in the browser tab</title>
  </head>
  <body>
    <h1>A big heading</h1>
    <p>A paragraph of text.</p>
    <button>Click me</button>
    <input type="text" placeholder="Type here...">
    <ul>
      <li>List item one</li>
      <li>List item two</li>
    </ul>
    <div>A generic container box</div>
    <img src="cat.jpg" alt="a cat">

    <script src="app.js"></script>
  </body>
</html>
```

- HTML is made of **elements**: `<p>content</p>` — opening tag, content, closing tag.
- Elements can have **attributes** inside the opening tag: `<input type="text">`, `<img src="cat.jpg">`.
- Two attributes matter enormously for us:
  - **`id`** — a *unique* name for ONE element: `<p id="greeting">`. One id per page.
  - **`class`** — a label shared by *many* elements: `<li class="task">`. Used for styling and for selecting groups.
- The `<script>` tag goes at the **bottom of `<body>`** — so the page's elements exist before our code tries to touch them.

If HTML is totally new to the student, spend one session building a small page by hand first (a heading, paragraphs, a list, a button, an input) before continuing.

---

## 1. What is the DOM?

When the browser loads your HTML, it builds an in-memory model of the page: every element becomes an **object** (Lesson 8!) with properties and methods. This tree of objects is the **DOM** — the *Document Object Model*.

```
document
└── html
    ├── head
    └── body
        ├── h1
        ├── p
        └── ul
            ├── li
            └── li
```

The magic: **the DOM is live.** Change the objects → the page on screen instantly changes. That's all frontend interactivity is: JavaScript editing the DOM while the user watches.

Your entry point is a global object called `document`. Try it right now: open DevTools on any website and type `document.title` — then set it: `document.title = "I changed this!"` and watch the tab.

---

## 2. Selecting elements

Before changing an element, you must grab it. The modern way: **`querySelector`**, which finds elements using CSS selector syntax:

```html
<h1 id="main-title">My Page</h1>
<p class="note">First note</p>
<p class="note">Second note</p>
<button>Save</button>
```

```js
// By id — # means id
const title = document.querySelector("#main-title");

// By class — . means class (returns the FIRST match)
const firstNote = document.querySelector(".note");

// By tag name
const button = document.querySelector("button");

// ALL matches, not just the first:
const allNotes = document.querySelectorAll(".note");   // a list of elements
console.log(allNotes.length);                          // 2
```

- `querySelector(...)` → the **first** matching element (or `null` if none — see mistakes below!)
- `querySelectorAll(...)` → **all** matches, as a NodeList — loop it with `for...of` or `forEach`.

The string inside is a CSS selector: `"#id"`, `".class"`, `"tag"`, and combinations like `"ul .task"` (elements with class task inside a ul). Your CSS knowledge doubles as selection skill.

(Legacy alternatives you'll meet in tutorials: `getElementById("main-title")`, `getElementsByClassName(...)`. They work fine — `querySelector` just does everything with one consistent syntax.)

---

## 3. Reading and changing content

Selected elements are objects. Their most useful property:

```js
const title = document.querySelector("#main-title");

console.log(title.textContent);      // "My Page" — read the text
title.textContent = "Welcome back!"; // change it — LOOK AT THE PAGE. It changed. 🎉
```

There's also `innerHTML`, which interprets HTML tags in the string:

```js
title.innerHTML = "Hello <em>world</em>";   // "world" renders in italics
```

⚠️ **Safety rule:** never put user-typed text into `innerHTML` — a malicious user could inject a `<script>` and attack your page (this is called **XSS**, a top real-world vulnerability). User text goes in `textContent`, always, which treats everything as plain text. Get this habit right from day one.

### Inputs are different!

Form fields hold their content in `.value`, not `.textContent`:

```js
const input = document.querySelector("input");
console.log(input.value);       // whatever the user has typed
input.value = "";               // clear the box
```

And remember Lesson 2: **`.value` is always a string** — `Number(input.value)` before doing math!

---

## 4. Changing appearance

### Inline styles

```js
const note = document.querySelector(".note");
note.style.color = "red";
note.style.backgroundColor = "yellow";    // CSS background-color → JS backgroundColor (camelCase!)
note.style.fontSize = "24px";
```

CSS property names with dashes become camelCase in JS. Values are strings, units included.

### The better way — toggling classes

Professionals define looks in CSS and use JS only to switch classes on/off:

```css
/* style.css */
.completed { text-decoration: line-through; color: gray; }
.hidden { display: none; }
```

```js
note.classList.add("completed");       // strike it through
note.classList.remove("completed");    // back to normal
note.classList.toggle("hidden");       // on if off, off if on — perfect for show/hide
note.classList.contains("completed");  // true/false
```

Why better? Separation of jobs: CSS describes *what "completed" looks like*; JS decides *when* something is completed. Change the design later without touching a line of JS.

---

## 5. Creating and removing elements

Pages aren't static — think of a chat where messages keep appearing. You can build new elements from scratch:

```js
// 1. Create (exists only in memory so far)
const li = document.createElement("li");

// 2. Fill
li.textContent = "Buy groceries";
li.classList.add("task");

// 3. Attach to the page — NOW it appears
const list = document.querySelector("ul");
list.appendChild(li);

// Removing:
li.remove();
```

The create → configure → append rhythm becomes second nature quickly.

### Rendering an array of data ⭐ — the most important frontend pattern

Here it comes — the payoff of Lessons 7, 8, 9 combined. Take an array of objects, produce page elements:

```html
<ul id="product-list"></ul>
```

```js
const products = [
  { name: "Laptop", price: 450000 },
  { name: "Mouse", price: 8000 },
  { name: "Monitor", price: 120000 },
];

const list = document.querySelector("#product-list");

for (const product of products) {
  const li = document.createElement("li");
  li.textContent = `${product.name} — ₦${product.price}`;
  list.appendChild(li);
}
```

Three items appear on the page. Change the array, refresh, the page follows. **Data drives the page.** This exact idea — *render UI from data* — is the entire philosophy of React, Vue, and every modern framework. You've just written it by hand, which means you'll actually understand what frameworks automate.

A useful refinement — wrap rendering in a function you can call anytime the data changes:

```js
function render(products) {
  list.innerHTML = "";                   // clear old content (our own strings only — safe)
  for (const product of products) {
    const li = document.createElement("li");
    li.textContent = `${product.name} — ₦${product.price}`;
    list.appendChild(li);
  }
}

render(products);                        // initial draw
products.push({ name: "Keyboard", price: 15000 });
render(products);                        // redraw — keyboard appears
```

---

## ⚠️ Common mistakes

```js
// 1. THE classic: script runs before the element exists
//    <script> in <head> + querySelector → null → "Cannot read properties of null"
//    Fix: keep <script> at the bottom of <body> (or use the defer attribute).

// 2. Forgetting the # or .
document.querySelector("main-title")     // looks for a <main-title> TAG → null
document.querySelector("#main-title")    // ✅

// 3. Not checking for null
const el = document.querySelector("#typo-id");
el.textContent = "hi";                   // TypeError: Cannot read properties of null
// When you see "of null" — your selector found nothing. Check the spelling and the HTML.

// 4. .textContent on an input (wanted .value) — or vice versa

// 5. querySelector when you needed ALL of them (only the first changes!)

// 6. Doing math with input.value without Number()
```

---

## ✅ Classwork

*(Build one `index.html` with: an `h1`, three `p.note` paragraphs, a `button`, a text `input`, and an empty `ul` — then do everything in `app.js` + the console.)*

1. Select the `h1` and change its text to your name. Change its color to purple via `.style`.
2. Select ALL the notes with `querySelectorAll` and loop over them, setting each one's text to `"Note 1"`, `"Note 2"`, `"Note 3"` (use the index).
3. Type something in the input box, then read it from the console with `.value`. Then clear it from code.
4. Create a `.hidden { display: none; }` CSS class. From the console, toggle it on and off on the button — watch it vanish and return.
5. Build the products render from section 5. Add a fourth product to the array and re-render.

## 📝 Homework

1. **Profile page:** HTML with empty `<h1 id="name">`, `<p id="bio">`, `<ul id="hobbies">`. In JS: a `me` object (from Lesson 8 homework!) — write `renderProfile(person)` that fills the h1 and p from properties and creates an `li` per hobby. Everything on the page must come from the object.
2. **Score board:** an array of `{ player, points }` objects. Render as a list sorted by points, highest first (Lesson 9's `sort` with a compare function!). The top player's `li` should get a class `champion` that styles it gold.
3. **Word counter (static):** put a paragraph of text in your HTML. In JS: read its `textContent`, split it into words, and set another element's text to `"This paragraph has N words."`
4. Break-it lab: move your `<script>` tag into `<head>`, refresh, and read the error you get. Write a comment explaining exactly why it happens, then move it back.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **DOM** | The live object model of the page; change it → page changes |
| **`document`** | The global entry-point object to the DOM |
| **`querySelector` / `querySelectorAll`** | Find element(s) with CSS selectors (`#id`, `.class`, `tag`) |
| **`textContent`** | An element's text (safe for any content) |
| **`innerHTML`** | HTML content — never feed it user input (XSS!) |
| **`.value`** | What's typed in an input (always a string!) |
| **`classList`** | `add` / `remove` / `toggle` CSS classes — the pro way to change looks |
| **`createElement` / `appendChild`** | Build and attach new elements |
| **render** | Draw page content from data — THE core frontend pattern |
