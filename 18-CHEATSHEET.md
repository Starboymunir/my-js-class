# 18 — JavaScript Cheat Sheet (Quick Reference)

Keep this open while coding. Lesson numbers in [brackets] point back to full explanations.

---

## Variables [L2]
```js
const name = "Amina";     // can't be reassigned — DEFAULT choice
let score = 0;            // can be reassigned
// var — old, don't use
```

## Types [L2]
```js
42, 3.14        // number
"hi", `hi ${x}` // string (backticks = template literal with ${expressions})
true, false     // boolean
undefined       // nothing was assigned
null            // intentionally empty
typeof x        // check a type
Number("42")    // string → number (NaN if it fails)
String(42)      // number → string
```

## Operators [L3]
```js
+ - * / %  **        // math (% = remainder: 7 % 2 === 1 → odd)
x += 5; x++; x--     // update shortcuts
=== !==              // compare (ALWAYS triple — never == !=)
> < >= <=
&& || !              // and, or, not
cond ? a : b         // ternary: pick a value [L4]
```

## Conditionals [L4]
```js
if (cond) { ... }
else if (cond) { ... }
else { ... }
// Falsy values (only 6): false, 0, "", null, undefined, NaN
switch (x) { case "a": ...; break; default: ... }
```

## Loops [L5]
```js
for (let i = 0; i < 5; i++) { ... }      // known count
while (cond) { ... }                     // unknown count
for (const item of array) { ... }        // each item [L7]
break;      // exit loop
continue;   // next iteration
// Accumulator: let total = 0; ...loop... total += x;
```

## Functions [L6]
```js
function add(a, b) { return a + b; }     // declaration
const add = (a, b) => a + b;             // arrow (implicit return, no braces)
const f = (x) => { return x * 2; };      // arrow with braces NEEDS return
add(2, 3)                                // call → 5
function greet(name = "friend") {...}    // default parameter
// return = give value back & exit. console.log = just display.
```

## Arrays [L7, L9]
```js
const a = [1, 2, 3];
a[0]; a[a.length - 1]           // first / last (indexes start at 0!)
a.push(x); a.pop()              // add/remove end
a.unshift(x); a.shift()         // add/remove start
a.includes(x); a.indexOf(x)
a.slice(1, 3)                   // copy portion (doesn't modify)
a.splice(i, 1)                  // remove at index (modifies!) [L13]
a.join(", "); "a,b".split(",")  // array ↔ string
a.sort((x, y) => x - y)         // numeric sort (never bare .sort() on numbers)

a.filter((x) => x > 2)          // keep matching → new array
a.map((x) => x * 2)             // transform each → new array
a.find((x) => x > 2)            // first match (or undefined)
a.some(f); a.every(f)           // at least one / all pass?
a.forEach((x) => ...)           // just loop
```

## Objects [L8]
```js
const user = { name: "Amina", age: 15 };
user.name                        // dot: known key
user[keyVariable]                // bracket: key in a variable
user.city = "Kano";              // add/change
const { name, age } = user;      // destructuring [L14]
const copy = { ...user, age: 16 }; // spread: copy + override [L14]
Object.keys(o); Object.values(o); Object.entries(o)
user.profile?.email ?? "none"    // safe access + default [L14]
// Methods & this:
const obj = { n: 0, inc() { this.n++; } };
// ⚠️ objects/arrays are REFERENCES: b = a shares, {..} === {..} is false
```

## Strings [L9]
```js
s.length; s[0]
s.toUpperCase(); s.toLowerCase()
s.includes("x"); s.startsWith("x"); s.endsWith("x")
s.slice(0, 5); s.trim(); s.repeat(3)
s.replace("a", "b"); s.replaceAll("a", "b")
// Strings never change in place — capture the return value!
```

## Math & numbers [L9]
```js
Math.round(4.5); Math.floor(4.9); Math.ceil(4.1)
Math.max(...arr); Math.min(...arr); Math.abs(-7)
Math.floor(Math.random() * N) + 1     // random int 1..N
(1234.567).toFixed(2)                  // "1234.57" (string!)
```

## Errors [L10]
```js
try { risky(); } catch (error) { console.log(error.message); }
throw new Error("what went wrong");
// SyntaxError: broken grammar | ReferenceError: unknown name (typo?)
// TypeError "of undefined/null": the thing LEFT of the dot doesn't exist
console.log({ x }); console.table(arr)   // debug helpers
```

## DOM [L11]
```js
document.querySelector("#id")       // one ( "#id" ".class" "tag" )
document.querySelectorAll(".cls")   // all → loop with for...of
el.textContent = "hi";              // text (SAFE — use for user data)
el.innerHTML = "<b>hi</b>";         // parses HTML (never with user input — XSS)
input.value                         // form field text (a STRING — Number() it!)
checkbox.checked                    // true/false
el.classList.add/remove/toggle/contains("cls")
el.style.color = "red";             // inline style (prefer classList)
document.createElement("li"); parent.appendChild(el); el.remove()
el.dataset.index                    // data-index="..." attribute [L13]
// <script src="app.js"></script> at the BOTTOM of <body>
```

## Events [L12]
```js
el.addEventListener("click", () => { ... });   // pass fn — DON'T call it: fn not fn()
// types: click, input (each keystroke), change, submit, keydown
input.addEventListener("keydown", (e) => e.key === "Enter" && go());
list.addEventListener("click", (e) => e.target)   // delegation: which child?
setTimeout(fn, ms); setInterval(fn, ms); clearInterval(id)
// PATTERN: state variable → render() → handlers change state + re-render
```

## Forms & storage [L13]
```js
form.addEventListener("submit", (e) => {
  e.preventDefault();                     // stop the page reload — ALWAYS
  const v = input.value.trim();
  if (v === "") return showError("Required!");   // guard clauses
  ...
});
localStorage.setItem("k", JSON.stringify(data));
const data = JSON.parse(localStorage.getItem("k")) || fallback;
```

## Modern JS [L14]
```js
const { a, b: renamed, c = defaultVal } = obj;   // destructure
const [first, ...rest] = arr;                    // array destructure + rest
function f({ name, age }) {...}                  // destructured params
const merged = [...arr1, ...arr2];               // spread
function sum(...nums) {...}                      // rest params
import { add } from "./utils.js";                // + export, and
export function add() {...}                      // <script type="module">
```

## Async [L15]
```js
async function load() {
  try {
    const data = await slowThing();      // pauses THIS function only
    use(data);
  } catch (err) { handle(err); }
}
promise.then((v) => ...).catch((e) => ...)
await Promise.all([p1, p2])              // parallel
// Code AFTER an async call does NOT wait — use the result INSIDE then/await.
```

## Fetch [L16]
```js
const res = await fetch(url);
if (!res.ok) throw new Error(`HTTP ${res.status}`);   // fetch won't throw on 404!
const data = await res.json();                        // second await!

await fetch(url, {                                    // POST
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(obj),
});
// 2xx ok | 4xx your request wrong | 5xx server broke
// Every screen: loading → success | error
```

---

## The 10 commandments of this course

1. `const` by default, `let` when needed, `var` never.
2. `===` always, `==` never.
3. Indexes start at 0; last is `length - 1`.
4. Functions **return** values; `console.log` is only for peeking.
5. `input.value` is always a string — `Number()` before math.
6. "Cannot read properties of undefined" → look LEFT of the dot, log it.
7. State lives in variables; the page is just a rendering of it.
8. Never put user text in `innerHTML`.
9. `e.preventDefault()` first in every submit handler.
10. When stuck: read the error → log the suspects → check MDN → then ask.
