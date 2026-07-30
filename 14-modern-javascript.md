# Lesson 14 — Modern JavaScript (ES6+): Writing Like a Pro

> **Goal:** Learn the modern syntax features that fill professional codebases: destructuring, spread/rest, optional chaining, modules, and a proper understanding of JSON. Nothing here adds *new powers* — these are sharper knives for things you can already do. But you must know them: every tutorial, every framework, every codebase (and Node.js backend code!) is written this way.

*"ES6" = ECMAScript 2015, the big modernization of JavaScript. Features kept coming yearly after that; people say "ES6+" or "modern JS" for the whole era. You already know several: `let`/`const`, arrow functions, template literals, default parameters.*

---

## 1. Destructuring — unpacking values

### From objects

Instead of repetitive extraction:

```js
const user = { name: "Amina", age: 15, city: "Kano" };

// old:
const name = user.name;
const age = user.age;

// modern — mirror the object's shape on the left:
const { name, age } = user;
console.log(name, age);        // "Amina" 15
```

Read `{ name, age } = user` as: *"reach into `user` and pull out its `name` and `age` properties into variables of the same names."*

Extras you'll meet:

```js
const { city: hometown } = user;        // rename while unpacking → hometown = "Kano"
const { country = "Nigeria" } = user;   // default if the property is missing
```

### In function parameters — extremely common

When a function takes an object, destructure right in the parameter list:

```js
function describe({ name, age }) {
  return `${name} is ${age}`;
}
describe(user);                        // "Amina is 15" — passed the whole object!
```

You will see this *constantly* in React and in Node.js code. Train your eyes now.

### From arrays

```js
const [first, second] = ["gold", "silver", "bronze"];
console.log(first);    // "gold" — unpacked by POSITION

const [x, y] = [y, x]; // (classic party trick: swap two variables)
```

---

## 2. Spread `...` — unpacking INTO things

The `...` operator spreads an array's (or object's) contents out into a new context:

```js
const arr1 = [1, 2, 3];
const arr2 = [4, 5];

const combined = [...arr1, ...arr2];        // [1, 2, 3, 4, 5]
const withExtra = [0, ...arr1, 99];         // [0, 1, 2, 3, 99]
const copy = [...arr1];                      // a real copy — remember Lesson 8's reference problem!

Math.max(...arr1);                           // 3 — spread as function arguments (mystery from L9 solved!)

// Objects too:
const user = { name: "Amina", age: 15 };
const updated = { ...user, age: 16 };        // copy user, override age
// { name: "Amina", age: 16 } — and user itself is untouched!
```

That last pattern — **copy-with-changes instead of mutating** — is the backbone of modern frontend state management. When you meet React, `setUser({ ...user, age: 16 })` will already read like home.

### Rest — the same dots, the opposite job

In a *parameter list* or on the *left* of destructuring, `...` **collects** leftovers instead of spreading:

```js
function sum(...numbers) {              // accept ANY number of arguments as an array
  let total = 0;
  for (const n of numbers) total += n;
  return total;
}
sum(1, 2);            // 3
sum(1, 2, 3, 4, 5);   // 15

const [winner, ...others] = ["A", "B", "C", "D"];   // winner="A", others=["B","C","D"]
```

Same syntax, direction depends on context: right-hand side = spread out; parameter/left-hand side = gather up.

---

## 3. Optional chaining `?.` and nullish coalescing `??`

Remember the most common error in JavaScript — `Cannot read properties of undefined`? Modern JS ships a seatbelt:

```js
const user = { name: "Amina", address: { city: "Kano" } };
const ghost = { name: "Nobody" };                 // no address!

console.log(user.address.city);    // "Kano"
console.log(ghost.address.city);   // ❌ TypeError — crash!

console.log(ghost.address?.city);  // undefined — no crash. ?. means:
                                   // "if what's before me is null/undefined, stop and give undefined"
```

Use it when data *might* legitimately be missing (API responses! optional fields!). Don't sprinkle it everywhere on data you control — that just hides real bugs.

And for defaults, `??` — like `||`, but only treats `null`/`undefined` as missing:

```js
const count = 0;
count || 10;    // 10  😱 — || sees 0 as falsy and replaces a real value!
count ?? 10;    // 0   ✅ — ?? only replaces null/undefined

const displayName = user.nickname ?? user.name ?? "Anonymous";
```

---

## 4. JSON — the language of data exchange ⭐

You met JSON in Lesson 13 with localStorage. Now the full picture, because **JSON is how frontend and backend talk to each other** — it's the format of essentially every API on the internet, so this section is direct preparation for your backend future.

**JSON** (JavaScript Object Notation) is a *text format* for structured data, borrowed from JS object syntax but stricter:

```json
{
  "name": "Amina",
  "age": 15,
  "isEnrolled": true,
  "subjects": ["math", "physics"],
  "address": { "city": "Kano" }
}
```

Rules that differ from JS objects — memorize them, they cause real errors:

- Keys **must** be in **double quotes**. Strings too — no single quotes, no backticks.
- No trailing commas. No comments. No functions, no `undefined` (use `null`).
- A JSON document is a string — data *packaged for shipping*, not live objects.

The two converters (you know them already — now you know why they matter):

```js
const obj = { name: "Amina", age: 15 };

const packed = JSON.stringify(obj);      // '{"name":"Amina","age":15}' — object → string (to SEND/STORE)
const unpacked = JSON.parse(packed);     // { name: "Amina", age: 15 } — string → object (after RECEIVING)
unpacked.age                             // 15 — a real, usable object again
```

The full life of data on the web, which you'll complete in Lesson 16:

```
Server object → stringify → travels over the internet as text → parse → your frontend object
```

Pretty-printing trick for debugging: `JSON.stringify(obj, null, 2)` — indented, readable output.

---

## 5. Modules — splitting code across files ⭐

Real projects aren't one giant `app.js`. **Modules** let each file export what it offers and import what it needs:

```js
// 📄 mathUtils.js
export function add(a, b) {
  return a + b;
}
export function multiply(a, b) {
  return a * b;
}
export const PI = 3.14159;
```

```js
// 📄 app.js
import { add, PI } from "./mathUtils.js";

console.log(add(2, 3));       // 5
console.log(PI);
```

To use modules in the browser, the script tag needs one attribute:

```html
<script type="module" src="app.js"></script>
```

Notes:

- Import names in `{ }` must match the exported names (it's destructuring-flavored syntax). The path needs the `./` and the `.js`.
- A file can also have ONE `export default something`, imported without braces: `import anything from "./file.js"`. You'll see both styles everywhere.
- Modules require a server to load — Live Server counts, double-clicking the HTML file doesn't. (If you see a CORS error opening the file directly, that's why.)
- **Backend note:** Node.js uses this same `import`/`export` system (plus an older `require()` style you'll learn to recognize). Organizing code into modules is *the* structure of backend projects — one more thing you're learning once, using twice.

The principle behind modules is the same as functions, one level up: **small pieces, clear names, explicit connections.** `mathUtils.js`, `validation.js`, `render.js`, `app.js` — a program you can navigate.

---

## 6. Reading modern code — a decoder exercise

You now know enough to read real-world modern JS. Decode this together, feature by feature:

```js
import { fetchUsers } from "./api.js";

const ADMIN_DEFAULTS = { role: "admin", active: true };

function summarize({ name, posts = [] }) {
  return `${name}: ${posts.length} posts`;
}

const admins = users
  .filter((u) => u.role === "admin")
  .map((u) => ({ ...u, ...ADMIN_DEFAULTS, lastSeen: u.lastSeen ?? "never" }));

console.log(admins.map(summarize).join("\n"));
```

Every single piece — import, destructured params with defaults, filter/map chains, spread-merge, `??` — is from this course. If you can narrate this file aloud, you read modern JavaScript. *(One syntax note: `(u) => ({ ... })` — an arrow returning an object literal needs parentheses around the braces, or JS thinks `{` starts a function body.)*

---

## ⚠️ Common mistakes

```js
// 1. Destructuring a name that doesn't exist — silent undefined, not an error:
const { nmae } = user;          // typo → undefined. Watch spelling!

// 2. Spread makes a SHALLOW copy — nested objects are still shared references:
const copy = { ...user };
copy.address.city = "Lagos";    // user.address.city changed too! (address itself is a reference)

// 3. JSON with single quotes / trailing commas / unquoted keys → SyntaxError in JSON.parse

// 4. Forgetting type="module" → "Cannot use import statement outside a module"

// 5. Arrow returning an object without wrapping parens: (u) => { name: u.n }  // undefined!
```

---

## ✅ Classwork

1. Given `const movie = { title: "Inception", year: 2010, director: "Nolan" };` — destructure title and year in one line; rename director to `dir`; add a default for a missing `rating`.
2. Two arrays of names — merge them, spread-style, with `"--- break ---"` between. Then copy the result and prove (with push + logging both) that the copy is independent.
3. Rewrite: `function greet(person) { return "Hi " + person.name + " from " + person.city; }` using destructured parameters and a template literal.
4. Write an object with nesting, `JSON.stringify` it (pretty-printed), then parse it back and access a nested property. Then feed `JSON.parse` a deliberately broken string inside try/catch (Lesson 10!).
5. Split your Lesson 13 to-do app: move `save`/`load` into `storage.js`, validation into its own function file, import both into `app.js`. Confirm it still works (with `type="module"`).

## 📝 Homework

1. **Refactor sweep:** take your Lesson 9 "report pipeline" homework and modernize it: destructuring where sensible, spread instead of any copying, `??` for any defaults, split into two files with import/export.
2. **safeGet practice:** given `const school = { name: "Unity High", principal: { name: "Mrs. Bello" } }` — write expressions using `?.`/`??` that safely produce: the principal's name, the vice-principal's name (should give `"none appointed"`, not a crash), the principal's phone (`undefined` is fine).
3. **Immutable updates** (React warm-up): with `const settings = { theme: "light", fontSize: 14, lang: "en" };` — using ONLY spread (never modifying `settings`), create: `darkSettings` (theme flipped), `biggerSettings` (fontSize + 2), and `resetSettings` (a plain copy). Prove `settings` never changed.
4. **JSON detective:** each of these strings breaks `JSON.parse` — predict why BEFORE testing: `"{name: 'Amina'}"`, `'{"age": 15,}'`, `'{"fn": function(){}}'`. Fix each into valid JSON.
5. **Rest practice:** write `average(...nums)` accepting any number of arguments. `average(10, 20)` → 15, `average(1,2,3,4)` → 2.5, `average()` → 0 (guard!).

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **destructuring** | Unpack object/array values into variables: `const { name } = user` |
| **spread `...`** | Expand contents into a new array/object/call — great for copies |
| **rest `...`** | Gather leftovers into an array (params, destructuring) |
| **`?.` optional chaining** | Safe property access — `undefined` instead of a crash |
| **`??` nullish coalescing** | Default ONLY for null/undefined (unlike `\|\|`) |
| **JSON** | Strict text format for data exchange — the language of APIs |
| **module / `import` / `export`** | Split code across files with explicit connections |
| **shallow copy** | Spread copies one level; nested objects stay shared |
