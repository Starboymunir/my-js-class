# Lesson 02 — Variables and Data Types

> **Goal:** Learn how programs remember information (variables) and the kinds of information JavaScript can hold (data types). This is the foundation of *everything* — frontend and backend alike.

---

## 1. What is a variable?

A **variable** is a named container for a value. You give it a name, put something inside, and later you can use the name to get the value back or replace it.

Think of labeled boxes in a storeroom:

```
[ box labeled "age" ]        → contains 15
[ box labeled "userName" ]   → contains "Amina"
[ box labeled "isLoggedIn" ] → contains true
```

Programs are constantly remembering things — the user's name, the score in a game, the items in a cart. Variables are how.

### Creating a variable

```js
let age = 15;
```

Read this out loud as: *"Let there be a variable named `age`, and put the value `15` inside it."*

Breaking it down:

| Part | Meaning |
|------|---------|
| `let` | The keyword that creates (declares) a new variable |
| `age` | The name we chose |
| `=` | The **assignment operator** — "put the right side into the left side". **Not** "equals" like in math! |
| `15` | The value being stored |
| `;` | End of the instruction |

Now the name works anywhere below:

```js
let age = 15;
console.log(age);        // 15
console.log(age + 1);    // 16  (the box still contains 15 — we didn't change it)
console.log(age);        // 15  (see?)
```

### Changing a variable

```js
let score = 0;
console.log(score);   // 0

score = 10;           // no 'let' — the box already exists, we're replacing its contents
console.log(score);   // 10

score = score + 5;    // read the current value (10), add 5, store the result back
console.log(score);   // 15
```

That last line confuses every beginner, so let's slow down. `score = score + 5` is **not** a math equation (in math it would be impossible!). Remember: `=` means *assign*. JavaScript always works out the **right side first**:

1. Right side: `score + 5` → "what's in the box (10) plus 5" → `15`
2. Then assignment: put `15` into `score`.

This read-modify-store pattern appears in almost every program ever written. Make sure it feels natural before moving on.

---

## 2. `let` vs `const` (and why we avoid `var`)

JavaScript has three keywords for declaring variables:

### `const` — a value that will never be reassigned

```js
const birthYear = 2010;
birthYear = 2011;   // ❌ ERROR! "Assignment to constant variable"
```

`const` is short for *constant*. Use it when the value shouldn't change: a person's birth year, the number of days in a week, a website's name.

### `let` — a value that may change

```js
let score = 0;
score = 100;   // ✅ fine
```

Use `let` for things that will be updated: scores, counters, the text a user is typing.

### The professional habit (adopt it now)

> **Use `const` by default. Use `let` only when you know the value must change.**

This might seem backwards, but most values in real programs never get reassigned, and `const` protects you: if you accidentally try to overwrite something important, JavaScript stops you with a clear error instead of letting a silent bug through.

### What about `var`?

You will see `var` in old tutorials and old code:

```js
var name = "old style";   // works, but don't use it
```

`var` is the original (1995) way. It has quirky scoping behavior that causes subtle bugs (we'll see exactly why in the functions lesson). Modern JavaScript replaced it with `let`/`const` in 2015. **Recognize it when you see it; never write it.**

---

## 3. Naming variables

Rules (JavaScript enforces these):

- Can contain letters, digits, `_`, `$`
- **Cannot start with a digit**: `2players` ❌, `players2` ✅
- Cannot be a reserved word: `let let = 5;` ❌
- Case-sensitive: `name`, `Name`, and `NAME` are three different variables!

Conventions (humans enforce these — follow them so your code reads like everyone else's):

- Use **camelCase** for multi-word names: `firstName`, `totalPrice`, `isLoggedIn`. First word lowercase, each next word capitalized.
- Names should **describe the contents**: `userAge` is good; `x`, `thing`, `data1` are bad. Code is read far more often than it is written — name things for the reader.
- Booleans often start with `is`/`has`: `isOpen`, `hasPaid`.

```js
// Bad — what do these even hold?
let a = "Lagos";
let b = true;

// Good — the code explains itself
let cityName = "Lagos";
let isRaining = true;
```

---

## 4. Data types — the kinds of values

Every value in JavaScript has a **type**. The type determines what you can do with the value (you can multiply numbers; you can't sensibly multiply names). JavaScript has **7 primitive types** — but 5 of them do 99% of the work for a beginner:

### 4.1 Number

All numbers — whole or decimal, positive or negative. (Unlike some languages, JS has just one number type.)

```js
const price = 250;
const temperature = -4;
const pi = 3.14159;

console.log(price * 2);     // 500
console.log(10 / 3);        // 3.3333333333333335
```

### 4.2 String — text

Any text, wrapped in quotes. All three quote styles work:

```js
const name1 = "Amina";      // double quotes
const name2 = 'Bola';       // single quotes
const name3 = `Chidi`;      // backticks (special powers — see below)
```

A string can hold anything typeable — letters, digits, spaces, emoji:

```js
const message = "I have 3 cats!";
const empty = "";              // an empty string is still a string
```

**Careful:** `"42"` (a string) and `42` (a number) are **completely different values**. One is text that happens to look like a number; the other is a number you can do math with. This distinction will matter constantly — especially in frontend, because *everything a user types into a form arrives as a string*.

#### Joining strings (concatenation)

The `+` operator glues strings together:

```js
const firstName = "Amina";
const lastName = "Yusuf";
const fullName = firstName + " " + lastName;
console.log(fullName);   // "Amina Yusuf"
```

(Notice the `" "` — without it we'd get `"AminaYusuf"`. The computer does exactly what you say!)

#### Template literals — the modern, better way

Backtick strings can **embed values** directly using `${...}`:

```js
const name = "Amina";
const age = 15;

console.log(`My name is ${name} and I am ${age} years old.`);
// "My name is Amina and I am 15 years old."

console.log(`Next year I will be ${age + 1}.`);   // you can put any expression inside ${ }
```

Compare with the old way: `"My name is " + name + " and I am " + age + " years old."` — easy to mess up the spaces and quotes. **Prefer template literals whenever you're building a string from pieces.**

#### The homework mystery from Lesson 1, solved

```js
console.log(2 + 3);       // 5      — both numbers → addition
console.log("2" + "3");   // "23"   — both strings → gluing!
console.log("2" + 3);     // "23"   — string + number → JS converts 3 to "3" and glues
```

When `+` sees a string on either side, it glues instead of adding. This is a classic source of bugs. Keep numbers as numbers.

### 4.3 Boolean — true or false

Just two possible values. Named after logician George Boole.

```js
const isRaining = true;
const hasFinishedHomework = false;
```

Booleans look tiny but they power all decision-making in programs ("IF the user is logged in, show the dashboard"). Next two lessons are basically all about them.

### 4.4 Undefined — "nothing was put here"

A variable declared without a value automatically holds `undefined`:

```js
let winner;
console.log(winner);   // undefined
```

It means: *this box exists but nothing has been placed in it yet.* You'll rarely write `undefined` yourself, but you'll **see** it a lot — usually as a hint that you forgot to assign something or misspelled a name.

### 4.5 Null — "intentionally empty"

`null` means *deliberately nothing*. A programmer sets it on purpose to say "no value, and that's intentional":

```js
let selectedItem = null;   // the user hasn't selected anything yet — and we know it
```

**undefined vs null:** `undefined` = "nobody set this" (usually accidental). `null` = "set to empty on purpose". Subtle, but the distinction shows up in real code and in job interviews.

### 4.6 The other two (just so you've heard of them)

- **BigInt** — for astronomically large whole numbers. Rare.
- **Symbol** — unique identifiers for advanced use cases. Rare.

Ignore both for now.

### Checking a type: `typeof`

```js
console.log(typeof 42);          // "number"
console.log(typeof "hello");     // "string"
console.log(typeof true);        // "boolean"
console.log(typeof undefined);   // "undefined"
console.log(typeof null);        // "object"  ← a famous 25-year-old bug in JS itself! null is NOT an object.
```

`typeof` is handy when debugging: "why isn't my math working?" → `console.log(typeof price)` → `"string"` → aha, forgot to convert.

---

## 5. Converting between types

Because user input arrives as strings, converting is an everyday task:

```js
// String → Number
const input = "42";
const n = Number(input);       // 42 (a real number)
console.log(n + 1);            // 43 ✅

const bad = Number("hello");   // NaN — "Not a Number": the conversion failed
console.log(typeof NaN);       // "number" (weird but true — NaN is a special number value meaning "invalid number")

// Number → String
const s = String(99);          // "99"

// Anything → Boolean (preview — details in Lesson 4)
Boolean("hello");   // true
Boolean("");        // false
Boolean(0);         // false
Boolean(42);        // true
```

---

## 6. A worked example — putting it all together

```js
// A small profile program
const firstName = "Amina";
const lastName = "Yusuf";
const birthYear = 2010;
const currentYear = 2026;

let age = currentYear - birthYear;
const isTeenager = true;

const bio = `${firstName} ${lastName} is ${age} years old.`;
console.log(bio);                      // "Amina Yusuf is 15 years old."
console.log(`Teenager? ${isTeenager}`); // "Teenager? true"

// A birthday happens!
age = age + 1;
console.log(`After her birthday she is ${age}.`);  // 16
```

Walk through this line by line and predict each output before checking.

---

## ⚠️ Common beginner mistakes

```js
// 1. Using a variable before declaring it
console.log(city);       // ❌ ReferenceError: city is not defined
let city = "Abuja";

// 2. Re-declaring with let
let x = 5;
let x = 10;              // ❌ SyntaxError — the box already exists; just write: x = 10;

// 3. Reassigning a const
const limit = 100;
limit = 200;             // ❌ TypeError: Assignment to constant variable

// 4. Quotes around things that shouldn't be strings
const age = "15";
console.log(age + 1);    // "151" 😱 — string gluing, not math!

// 5. Case confusion
let userName = "Bola";
console.log(username);   // ❌ ReferenceError — lowercase n ≠ capital N
```

Getting these errors is *good* — read the message, find the line, fix it. That loop is the job.

---

## ✅ Classwork

1. Declare variables for: your name (`const`), your favorite number (`const`), your current mood (`let` — moods change!). Print all three with one `console.log` using a template literal.
2. Create `let count = 0;` then, in three separate steps, add 5, add 10, subtract 3 — printing after each step. Predict every output first.
3. Run `typeof` on: `"33"`, `33`, `true`, `undefined`, and a variable you declared but never assigned.
4. Fix this broken code (three bugs!):
   ```js
   const myName = "Sam";
   let 2ndPlace = "Jo";
   myName = "Samuel";
   console.log(myname);
   ```
5. Predict, then run: `console.log("10" + 5)` and `console.log(Number("10") + 5)`.

## 📝 Homework

1. **Mini bio program:** declare `name`, `country`, `birthYear`, `favoriteFood`. Calculate `age` from the birth year. Print a paragraph about yourself using ONE template literal spanning those variables.
2. **Shopping math:** a book costs 1500, a pen costs 200. Store both in constants. Calculate and print: the total for 2 books and 3 pens, using a template literal like `` `Total: ${...}` ``.
3. **Type detective:** for each value, write down your *prediction* of its `typeof`, then check in the console: `"false"`, `false`, `null`, `"null"`, `NaN`, `1.5`.
4. In your own words (write it as comments in your file): what's the difference between `let` and `const`? When would you use each?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **variable** | A named container for a value |
| **declare** | Create a variable (`let` / `const`) |
| **assign** | Put a value into a variable (`=`) |
| **string** | Text in quotes |
| **number** | Numeric value (math works on it) |
| **boolean** | `true` or `false` |
| **undefined** | "Nothing was put here" |
| **null** | "Intentionally empty" |
| **template literal** | Backtick string with `${}` slots |
| **`typeof`** | Tells you a value's type |
| **NaN** | "Not a Number" — result of failed number conversion |
