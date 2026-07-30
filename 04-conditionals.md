# Lesson 04 — Conditionals: Making Decisions

> **Goal:** Make programs choose between different actions. Until now our code ran every line, top to bottom, always the same. Conditionals give programs a brain: *"IF this, do that; OTHERWISE, do something else."*

---

## 1. The `if` statement

```js
const age = 20;

if (age >= 18) {
  console.log("You can vote!");
}
```

Anatomy:

```js
if (condition) {
  // this "block" runs ONLY if the condition is true
}
```

- `condition` — any expression that produces `true` or `false` (everything from Lesson 3!)
- `{ ... }` — the **block**: the group of lines that run when the condition holds. Can be one line or many.
- If the condition is `false`, JavaScript skips the whole block and continues below it.

```js
const temperature = 35;

if (temperature > 30) {
  console.log("It's hot today!");
  console.log("Drink lots of water.");
}
console.log("Have a nice day.");   // outside the block — runs no matter what
```

**Indent the code inside the block** (VS Code does it for you with Tab). Indentation doesn't change how the code runs, but it shows the human reader what's inside what — never skip it.

---

## 2. `else` — the other path

```js
const age = 15;

if (age >= 18) {
  console.log("You can vote!");
} else {
  console.log("Too young to vote — for now.");
}
```

Exactly **one** of the two blocks runs. Never both, never neither. `if/else` is a fork in the road.

---

## 3. `else if` — many paths

For more than two possibilities, chain conditions:

```js
const score = 67;

if (score >= 70) {
  console.log("Grade: A");
} else if (score >= 60) {
  console.log("Grade: B");
} else if (score >= 50) {
  console.log("Grade: C");
} else if (score >= 40) {
  console.log("Grade: D");
} else {
  console.log("Grade: F");
}
// prints "Grade: B"
```

How JavaScript reads this — **top to bottom, first match wins**:

1. `score >= 70`? 67 ≥ 70 → false, skip.
2. `score >= 60`? 67 ≥ 60 → **true** → run this block → **skip everything else in the chain.**

Two important consequences:

- **Order matters.** If we checked `score >= 40` first, a 67 would get a D! Put the most specific/highest condition first.
- Only ONE block in a chain ever runs. The final `else` (optional) is the catch-all if nothing matched.

---

## 4. Nesting — decisions inside decisions

Blocks can contain anything — including more `if`s:

```js
const isLoggedIn = true;
const isAdmin = false;

if (isLoggedIn) {
  console.log("Welcome back!");
  if (isAdmin) {
    console.log("Admin panel unlocked.");
  } else {
    console.log("Regular user view.");
  }
} else {
  console.log("Please log in.");
}
```

Nesting is powerful but gets hard to read past 2–3 levels. Often you can flatten with `&&`:

```js
// Instead of nested ifs:
if (isLoggedIn && isAdmin) {
  console.log("Admin panel unlocked.");
}
```

---

## 5. Truthy and falsy — how JS bends the rules

The condition in an `if` doesn't *have* to be a boolean. JavaScript will convert whatever you give it. Every value is either **truthy** (converts to `true`) or **falsy** (converts to `false`).

**Memorize the falsy list — there are only 6:**

```js
false
0
""          // empty string
null
undefined
NaN
```

**Everything else is truthy** — including `"0"`, `"false"` (non-empty strings!), negative numbers, and empty arrays.

Why is this useful? It gives natural-sounding checks:

```js
const username = "";   // user didn't type anything

if (username) {
  console.log(`Hello, ${username}!`);
} else {
  console.log("Please enter your name.");   // ← this runs: "" is falsy
}
```

`if (username)` reads as *"if we have a username"* — checking for empty/missing values in one stroke. You'll see this idiom everywhere in real code (frontend form checks, backend request validation).

---

## 6. The ternary operator — a mini if/else inside an expression

Sometimes you just want to pick between two *values*. The **ternary** (three-part) operator does if/else in one line:

```js
condition ? valueIfTrue : valueIfFalse
```

```js
const age = 15;
const category = age >= 18 ? "adult" : "minor";
console.log(category);   // "minor"

// Perfect inside template literals:
console.log(`You ${age >= 18 ? "can" : "cannot"} vote.`);
```

Use it for *simple* either/or values. If the logic has multiple branches or side effects, use a real `if/else` — readability first.

---

## 7. `switch` — comparing one value against many options

When you're comparing the *same variable* against many exact values, `switch` can be cleaner than a long else-if chain:

```js
const day = "saturday";

switch (day) {
  case "saturday":
  case "sunday":
    console.log("Weekend! 🎉");
    break;
  case "friday":
    console.log("Almost there...");
    break;
  default:
    console.log("Regular work day.");
}
```

Rules to remember:

- Comparison is strict (`===`).
- **`break` is mandatory** at the end of each case — without it, execution "falls through" into the next case (a notorious bug!). Stacking two cases with no code between them (like saturday/sunday above) is the one *intentional* use of fall-through.
- `default` is the catch-all, like `else`.

Honestly: you can live without `switch` (else-if does everything it does), but you must be able to *read* it — it's common in other people's code.

---

## 8. Worked example — a login checker

```js
const storedPassword = "secret123";
const enteredPassword = "secret124";
const attemptsLeft = 2;

if (enteredPassword === storedPassword) {
  console.log("✅ Welcome back!");
} else if (attemptsLeft > 0) {
  console.log(`❌ Wrong password. ${attemptsLeft} attempts left.`);
} else {
  console.log("🔒 Account locked. Try again in 15 minutes.");
}
```

This tiny structure — check credentials, branch on the result — is the exact skeleton of real backend login code you'll write one day.

---

## ⚠️ Common mistakes

```js
// 1. Assignment in the condition
if (score = 100) { ... }        // ❌ always truthy! Wanted ===

// 2. Semicolon after the condition
if (age >= 18); {               // ❌ that ; ends the if! The block always runs.
  console.log("adult");
}

// 3. Repeating the variable in compound conditions
if (day === "sat" || "sun") { ... }   // ❌ "sun" is truthy → always true!
if (day === "sat" || day === "sun") { ... }  // ✅ each side must be complete

// 4. Ranges in the wrong order in else-if chains
if (score >= 40) { grade = "D"; }      // ❌ catches 95 too!
else if (score >= 70) { grade = "A"; } // never reached

// 5. Forgetting break in switch
```

Mistake #3 deserves a highlight: `||` and `&&` join **complete questions**, not halves of one. English lets you say "if day is sat or sun" — JavaScript needs "if day is sat OR day is sun."

---

## ✅ Classwork

1. **Bouncer program:** `const age = ...` — print "Enter" if 18+, otherwise "Sorry, adults only." Test with 3 different ages.
2. **Grade converter:** build the full A/B/C/D/F chain from section 3, then deliberately reorder two branches and observe the bug.
3. **Truthy quiz:** predict `if` behavior for: `"hello"`, `""`, `0`, `"0"`, `null`, `42`, `NaN`. Check each with `if (value) console.log("truthy"); else console.log("falsy");`
4. Rewrite with a ternary: 
   ```js
   let message;
   if (stock > 0) { message = "In stock"; } else { message = "Sold out"; }
   ```
5. **Weekend detector** with `switch`, using a `day` variable.

## 📝 Homework

1. **Traffic light:** `const light = "yellow";` → print "Go", "Slow down", or "Stop". Handle an unknown color with a message like "Invalid light!" (Try it twice: once with else-if, once with switch.)
2. **BMI commentator:** given `weight` (kg) and `height` (m), compute `bmi = weight / (height * height)`, then print "Underweight" (<18.5), "Normal" (18.5–24.9), "Overweight" (25–29.9), or "Obese" (30+). Include the BMI value, rounded — try `bmi.toFixed(1)`.
3. **Ticket pricing:** kids under 5: free; ages 5–12: 1000; teens 13–17: 1500; adults 18–64: 2500; seniors 65+: 1200. Given an `age`, print the price. *Think carefully about the order of your conditions.*
4. **Challenge — leap year:** a year is a leap year if divisible by 4, EXCEPT century years (divisible by 100), UNLESS also divisible by 400. So 2024 ✅, 1900 ❌, 2000 ✅. Write it with `%`, `&&`, `||`. Test all three examples.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **conditional** | Code that runs only when a condition holds |
| **block** | Lines grouped in `{ }` |
| **branch** | One of the possible paths through if/else |
| **truthy / falsy** | How non-boolean values behave in conditions (6 falsy values!) |
| **ternary `? :`** | One-line if/else that produces a value |
| **`switch` / `case` / `break`** | Multi-way comparison of one value |
