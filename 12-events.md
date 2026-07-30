# Lesson 12 — Events: Responding to the User

> **Goal:** Make pages react — to clicks, typing, and more. Last lesson, *we* changed the page from code. This lesson, the **user's actions** trigger our code. Events + DOM = interactivity. This is where it starts feeling like real frontend.

---

## 1. The event model

The browser constantly notices things happening: clicks, key presses, scrolling, a page finishing loading. Each of these is an **event**. Normally they pass by silently — unless you register a **listener**: *"when THIS event happens on THIS element, run THIS function."*

```html
<button id="greet-btn">Greet</button>
```

```js
const button = document.querySelector("#greet-btn");

button.addEventListener("click", () => {
  console.log("Button was clicked!");
});
```

Click the button — the message appears. Every click, again. Breaking down `addEventListener`:

- **On this element** (`button`) —
- **listen for** `"click"` (the event type, as a string) —
- **and when it happens, call this function** (the arrow function).

That function is called an **event handler** (or *callback* — Lesson 9 vocabulary!). Crucial mental shift:

> **We are not calling the function. We're handing it to the browser to call LATER — zero, one, or many times — whenever the event fires.**

This is why we pass the function itself, not the result of calling it (see mistakes). Your program stops being a top-to-bottom script and becomes a set of *reactions waiting to happen*. That's what makes it feel alive — and this "register a callback for later" idea is exactly how backend servers work too (*"when a request arrives, run this function"*).

---

## 2. Events you'll actually use

| Event | Fires when... |
|-------|---------------|
| `"click"` | Element is clicked (works on ANY element, not just buttons) |
| `"input"` | An input's value changes — fires on every keystroke ⭐ |
| `"change"` | An input's value is committed (checkbox toggled, dropdown picked, text field left) |
| `"submit"` | A form is submitted — next lesson's star |
| `"keydown"` | A keyboard key goes down |
| `"mouseover"` / `"mouseout"` | Pointer enters / leaves an element |

Live-typing example — the "instant feedback" pattern used in search boxes everywhere:

```html
<input id="name-input" placeholder="Your name">
<p id="preview"></p>
```

```js
const input = document.querySelector("#name-input");
const preview = document.querySelector("#preview");

input.addEventListener("input", () => {
  preview.textContent = `Hello, ${input.value}!`;
});
```

Type and watch the greeting build letter by letter. Notice the anatomy — it's ALWAYS this trio:

1. **Select** the elements involved.
2. **Listen** for the event.
3. In the handler: **read state → compute → update the page.**

---

## 3. The event object — details about what happened

The browser passes your handler an object full of information about the event. Declare a parameter (conventionally `event` or `e`) to receive it:

```js
input.addEventListener("keydown", (event) => {
  console.log(`You pressed: ${event.key}`);
  if (event.key === "Enter") {
    console.log("Submitted!");
  }
});
```

Useful goodies: `event.key` (which key), `event.target` (the element the event happened on — see section 5), and `event.preventDefault()` (stop the browser's default behavior — vital for forms, next lesson).

---

## 4. State + events — building a real widget ⭐

Real apps keep their data (**state**) in variables, and events modify it. Golden rule:

> **The data lives in JavaScript. The page is just a *display* of the data.**
> Events change the data → then re-render the display. Never treat the page itself as your storage.

The counter — a tiny app with the full architecture:

```html
<h1 id="count-display">0</h1>
<button id="minus">−</button>
<button id="plus">+</button>
<button id="reset">Reset</button>
```

```js
// STATE — the single source of truth
let count = 0;

// ELEMENTS
const display = document.querySelector("#count-display");

// RENDER — draw the state onto the page
function render() {
  display.textContent = count;
  display.style.color = count < 0 ? "red" : "black";   // display logic lives here too
}

// EVENTS — change state, then re-render
document.querySelector("#plus").addEventListener("click", () => {
  count++;
  render();
});

document.querySelector("#minus").addEventListener("click", () => {
  count--;
  render();
});

document.querySelector("#reset").addEventListener("click", () => {
  count = 0;
  render();
});

render();   // initial draw
```

Study the shape: **state → render() → event handlers that mutate state and call render()**. Small as it is, this is the architecture of every frontend app ever built — React's whole job is automating the "call render after state changes" part. Internalize it here, by hand.

---

## 5. Events on dynamic elements — delegation

A to-do app creates `<li>` items *at runtime*. How do you listen for clicks on elements that don't exist yet when the page loads?

Naive approach: add a listener to every `li` as you create it. Works, but there's a slicker, very common trick — **event delegation**: put ONE listener on the *parent* and use `event.target` to see which child was actually clicked:

```html
<ul id="task-list">
  <li>Wash plates</li>
  <li>Do homework</li>
</ul>
```

```js
const list = document.querySelector("#task-list");

list.addEventListener("click", (event) => {
  // event.target = the exact element that was clicked
  if (event.target.tagName === "LI") {
    event.target.classList.toggle("completed");
  }
});
```

Now *any* `li` — including ones you `push`+render later — responds to clicks, with a single listener. (It works because events "bubble" up from the clicked element to its ancestors; the parent hears them all.)

---

## 6. Bonus: `setTimeout` and `setInterval` — time as an event

Not user events, but the same "call my function later" idea:

```js
setTimeout(() => {
  console.log("3 seconds passed!");
}, 3000);                              // milliseconds — runs ONCE, later

const timerId = setInterval(() => {
  console.log("tick");
}, 1000);                              // runs EVERY second...

setTimeout(() => clearInterval(timerId), 5500);   // ...until cancelled
```

Great for clocks, countdowns, slideshows — and a first taste of *asynchronous* thinking (code that runs "later, not now"), which Lesson 15 dives into.

---

## ⚠️ Common mistakes

```js
// 1. CALLING the handler instead of PASSING it — the #1 events bug:
button.addEventListener("click", greet());   // ❌ runs greet NOW, once, passes its return value
button.addEventListener("click", greet);     // ✅ passes the function itself, for later
// (With inline arrows this can't happen — one reason beginners love them here.)

// 2. Reading input.value ONCE at the top instead of inside the handler:
const name = input.value;                    // ❌ frozen at page load: ""
button.addEventListener("click", () => {
  console.log(input.value);                  // ✅ read it AT CLICK TIME
});

// 3. Updating state but forgetting to call render() — "my variable changed but the page didn't!"
//    The page NEVER updates itself. You must re-render.

// 4. Listener on a container's children added before they exist → use delegation (section 5)

// 5. Math on input.value without Number() — count = count + input.value → "12345" strings again!
```

---

## ✅ Classwork

1. **Click counter:** build the counter app from section 4, typing it yourself. Then add: the display turns green above 10.
2. **Live preview:** the input mirror from section 2. Extend: show the number of characters typed, and turn the text red past 20 characters.
3. **Key spy:** listen for `keydown` on the whole `document` and log `event.key`. Find out what key names Space, Enter, and the arrows have.
4. **Show/hide:** a button that toggles a paragraph's `hidden` class. The button's own label must switch between "Show" and "Hide" too (state or `classList.contains` — your choice).
5. **Delegation demo:** section 5's list — add a "completed" toggle, then use JS to append a NEW `li` and confirm clicking it works with no extra listener.

## 📝 Homework

1. **Dice roller:** a button + a big `<h1>`. Each click shows a random 1–6 (Lesson 9's formula). Double roll = show "🎲 DOUBLE!" — wait, one die can't double; make it TWO dice and detect equal values.
2. **Dark mode toggle:** a button that toggles class `dark` on `document.body`; CSS makes `.dark` a dark background with light text. Button label flips between "🌙 Dark" and "☀️ Light".
3. **Countdown timer:** an input (seconds) + Start button + display. On Start: read the number, tick down every second with `setInterval`, show "⏰ Time up!" at zero and stop the interval. Handle nonsense input (empty / not a number) with a message. *(This one combines everything — budget real time for it.)*
4. **Mini quiz — trace, don't run:** what does the console show, and why?
   ```js
   let n = 0;
   const btn = document.querySelector("button");
   btn.addEventListener("click", () => n++);
   btn.addEventListener("click", () => console.log(n));
   // user clicks the button 3 times
   ```
   *(Yes — an element can have several listeners; they run in order.)*

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **event** | Something that happens (click, keypress, timer...) |
| **listener** | Registration: "when X happens on Y, call Z" |
| **handler / callback** | The function the browser calls when the event fires |
| **`addEventListener(type, fn)`** | Pass the function — don't call it! |
| **event object (`e`)** | Details: `e.key`, `e.target`, `e.preventDefault()` |
| **state** | Your app's data, living in JS variables — the source of truth |
| **render** | Redraw the page from state after every change |
| **delegation** | One listener on the parent handles clicks on all (even future) children |
| **`setTimeout` / `setInterval`** | Run a function later / repeatedly |
