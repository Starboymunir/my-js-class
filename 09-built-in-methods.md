# Lesson 09 — Built-in Methods: JavaScript's Toolbox

> **Goal:** Learn the everyday tools JavaScript ships with — string methods, number formatting, `Math`, and the three superstar array methods (`map`, `filter`, `reduce`-lite) that will transform how you write code. Plus: how to look up methods you don't know (a superpower).

---

## 1. You don't memorize the toolbox — you learn to reach into it

JavaScript ships with hundreds of built-in methods. Nobody memorizes all of them. What professionals actually do:

1. Know the ~25 workhorses by heart (this lesson).
2. For anything else: **search "mdn string uppercase"** → developer.mozilla.org tells you the method, its inputs, and examples.

Learning to *read documentation* is a course goal in itself. When a homework question below says "find the method," that's on purpose.

---

## 2. String methods

Strings come with a toolbelt. Important: **strings are immutable** — no method changes the original; they all *return a new string*:

```js
const name = "Amina Yusuf";

name.length                    // 11 — (a property, not a method — no parentheses)
name.toUpperCase()             // "AMINA YUSUF"
name.toLowerCase()             // "amina yusuf"
console.log(name);             // "Amina Yusuf" — original untouched! Store results if you need them:
const loud = name.toUpperCase();

name.includes("Yusuf")         // true
name.startsWith("Am")          // true
name.endsWith("uf")            // true

name.indexOf("Y")              // 6 — position of first match (-1 if absent)
name[0]                        // "A" — index into a string like an array!
name.slice(0, 5)               // "Amina" — copy characters 0 up-to-not-including 5
name.slice(6)                  // "Yusuf" — from 6 to the end

name.replace("Yusuf", "Bello") // "Amina Bello" (first match only)
name.replaceAll("a", "@")      // "Amin@ Yusuf" — wait, why not the capital A? Case-sensitive!

"   messy input   ".trim()     // "messy input" — strips surrounding whitespace (ESSENTIAL for form input!)
"ha".repeat(3)                 // "hahaha"

"a,b,c".split(",")             // ["a", "b", "c"] — string → array
["a","b","c"].join("-")        // "a-b-c"        — array → string (they're a matched pair)
```

Real-life combo — cleaning up user input:

```js
const rawEmail = "  Amina.Y@GMAIL.com ";
const email = rawEmail.trim().toLowerCase();   // "amina.y@gmail.com"
```

Note the **chaining**: `trim()` returns a string… which has methods… so you can call the next one immediately. Read chains left to right like a pipeline.

---

## 3. Number methods and `Math`

```js
const price = 1234.5678;

price.toFixed(2)          // "1234.57" — round to 2 decimals. ⚠️ returns a STRING (fine for display)

Number("42")              // 42     — string → number (from Lesson 2)
parseInt("42px")          // 42     — reads leading digits, ignores the rest (handy!)
parseFloat("3.14m")       // 3.14
Number.isInteger(4.0)     // true

// The Math object — a bundle of math utilities:
Math.round(4.5)           // 5
Math.floor(4.9)           // 4  — always DOWN
Math.ceil(4.1)            // 5  — always UP
Math.abs(-7)              // 7
Math.max(3, 9, 2)         // 9
Math.min(3, 9, 2)         // 2
Math.sqrt(64)             // 8

// Random numbers! Math.random() gives a random decimal from 0 up to (not including) 1:
Math.random()                          // e.g. 0.7263...
Math.floor(Math.random() * 6) + 1      // random die roll: 1–6 🎲
```

That die-roll formula is worth understanding, not memorizing: `random()` → 0–0.999, `* 6` → 0–5.999, `floor` → 0–5, `+ 1` → 1–6. General recipe for a random integer from 1 to N: `Math.floor(Math.random() * N) + 1`.

### Dates (a taste)

```js
const now = new Date();
now.getFullYear();     // 2026
now.getMonth();        // 0-11 (January is 0! — a famous gotcha)
now.getDate();         // day of month
now.getHours();        // 0-23
```

Enough for "display today's date"; deeper date work can wait.

---

## 4. The superstar array methods ⭐⭐ — `map`, `filter`, `find`

Recall from Lesson 6: **functions are values** — they can be passed as arguments. These methods are where that pays off. Each one takes a function and applies it to every item, replacing an entire loop pattern with one readable line.

### `filter` — keep the items that pass a test

The Lesson 7 way:

```js
const prices = [1200, 350, 4999, 89, 2400];
const cheap = [];
for (const p of prices) {
  if (p < 1000) cheap.push(p);
}
```

The `filter` way — same result:

```js
const cheap = prices.filter((p) => p < 1000);
console.log(cheap);    // [350, 89]
```

How it works: `filter` calls your arrow function once per item. The function returns `true` (keep) or `false` (drop). `filter` collects the keepers into a **new array**; the original is untouched.

```js
const products = [
  { name: "Laptop", price: 450000, inStock: true },
  { name: "Mouse", price: 8000, inStock: true },
  { name: "Monitor", price: 120000, inStock: false },
];

const available = products.filter((p) => p.inStock);
const affordable = products.filter((p) => p.price < 100000);
```

### `map` — transform every item

"Give me a new array where each item is (something computed from) the old item":

```js
const nums = [1, 2, 3, 4];
const doubled = nums.map((n) => n * 2);          // [2, 4, 6, 8]

const names = products.map((p) => p.name);       // ["Laptop", "Mouse", "Monitor"]
const labels = products.map((p) => `${p.name}: ₦${p.price}`);
// ["Laptop: ₦450000", "Mouse: ₦8000", "Monitor: ₦120000"]
```

`map` always returns a new array of **the same length**, each item transformed. In real frontend work you will `map` arrays of data into pieces of page constantly — this method is the beating heart of frontend frameworks like React.

### `find` — get the first match (one item, not an array)

```js
const mouse = products.find((p) => p.name === "Mouse");
console.log(mouse.price);        // 8000
const unicorn = products.find((p) => p.price < 5);   // undefined if nothing matches
```

### Friends of the family

```js
nums.some((n) => n > 3);          // true  — does AT LEAST ONE pass?
nums.every((n) => n > 0);         // true  — do ALL pass?
nums.forEach((n) => console.log(n));   // just loop (no new array; like for...of in method form)

// And the sort fix promised in Lesson 7 — give sort a comparison function:
const ns = [1, 10, 2];
ns.sort((a, b) => a - b);         // [1, 2, 10] ✅ numeric ascending
ns.sort((a, b) => b - a);         // [10, 2, 1]    descending
```

### Chaining — the pipeline style

Because these return arrays, they chain into readable data pipelines:

```js
const inStockNames = products
  .filter((p) => p.inStock)          // keep available products
  .map((p) => p.name);               // extract just the names

console.log(inStockNames);           // ["Laptop", "Mouse"]
```

Read it top-to-bottom like a sentence. This style dominates modern JavaScript — frontend and backend alike.

### `reduce` — honorable mention

`reduce` boils an array down to one value (sum, max, anything). It's powerful but confusing early on:

```js
const total = prices.reduce((sum, p) => sum + p, 0);   // 9038 — like the accumulator loop in one line
```

Recognize it when you see it; keep using loops for this until the others feel easy. There is no prize for using `reduce` — clarity wins.

---

## 5. Which tool when? (loops still matter!)

| You want to... | Reach for |
|----------------|-----------|
| A new array with some items removed | `filter` |
| A new array with every item transformed | `map` |
| The first item matching a condition | `find` |
| Just do something per item | `for...of` or `forEach` |
| Combine items into one value (sum, max…) | loop with accumulator (or `reduce` later) |
| Complex logic, early exits, index arithmetic | classic `for` loop |

`map`/`filter` don't replace loops — they replace *specific patterns* of loops with clearer names. Under the hood they ARE loops.

---

## ⚠️ Common mistakes

```js
// 1. Expecting string methods to modify the string
let s = "hello";
s.toUpperCase();
console.log(s);        // "hello" — you threw the result away! s = s.toUpperCase();

// 2. map/filter without using the return value — they NEVER modify the original
nums.map((n) => n * 2);
console.log(nums);     // unchanged. const doubled = nums.map(...)

// 3. Arrow braces without return (Lesson 6 flashback) — array of undefineds:
nums.map((n) => { n * 2 });    // ❌   nums.map((n) => n * 2) ✅

// 4. filter returning the value instead of a test
nums.filter((n) => n * 2);     // keeps everything truthy — wanted a CONDITION like n > 2

// 5. Forgetting sort's compare function for numbers → [1, 10, 2]
```

---

## ✅ Classwork

1. Take `"  JavaScript Is FUN  "` and, in one chain, trim it and lowercase it. Then check whether it includes `"fun"`.
2. Random practice: roll two dice 5 times (a loop!), printing both values and their sum each time.
3. `const words = ["apple", "fig", "banana", "kiwi", "watermelon"];`
   - `filter` the words longer than 4 letters
   - `map` all words to UPPERCASE
   - `find` the first word starting with `"b"`
   - chain: words longer than 4 letters, uppercased
4. Rewrite Lesson 7 homework #2 (the word filter) using `filter` — compare line counts!
5. Sort `[45, 3, 120, 7]` ascending. First try plain `.sort()` and observe the disaster.

## 📝 Homework

1. **Username generator:** write `makeUsername(fullName)` → takes `"Amina Yusuf"`, returns `"amina.yusuf87"` (lowercase, space→dot via `replace`, plus a random 2-digit number). Methods needed: `toLowerCase`, `replace`, `Math.random`, `Math.floor`.
2. **Price tags:** given the products array from the lesson, use `map` to produce `["Laptop — ₦450,000.00", ...]` (hint: research `toLocaleString()` on MDN — this is a documentation-reading exercise!).
3. **Report pipeline:** `const scores = [55, 90, 33, 78, 62, 47, 85];`
   - How many passed (≥ 50)? (`filter` + `.length`)
   - List of passing scores as strings like `"78%"` (`filter` + `map`)
   - Highest score (`Math.max(...scores)` — the `...` is a preview of Lesson 14!)
   - Average (your `getAverage` function from Lesson 7 — reuse it!)
4. **Email checker v1:** `isValidEmail(email)` returns true only if the string includes `"@"`, includes `"."` after the @ (think about `indexOf`), and has no spaces (after trimming? you decide — write a comment justifying your choice).
5. **MDN treasure hunt:** using ONLY developer.mozilla.org, find and demo (a) a string method that pads a string to a given length, and (b) an array method that flattens nested arrays. One example each.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **immutable** | Can't be changed — string methods always return a new string |
| **chaining** | `x.trim().toLowerCase()` — feed each result to the next call |
| **callback** | A function you pass to another function (e.g. into `map`) |
| **`filter`** | New array: items passing a test |
| **`map`** | New array: every item transformed |
| **`find`** | First matching item (or `undefined`) |
| **`Math.random()`** | Random decimal 0–0.999…; scale + floor for integers |
| **MDN** | developer.mozilla.org — THE JavaScript reference. Bookmark it. |
