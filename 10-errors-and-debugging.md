# Lesson 10 — Errors and Debugging: Thinking Like a Detective

> **Goal:** Stop fearing red text. Learn to read error messages, hunt bugs methodically, and handle errors gracefully with `try/catch`. This is the lesson that separates people who *quit* programming from people who *get good* — and error handling is a huge part of backend work later.

---

## 1. Errors are your friend (really)

An error message is not the computer being mean. It is the computer giving you a **precise report**: what went wrong, and *where*. Professional developers see dozens of errors a day. The skill isn't avoiding errors — it's reading them fast.

Every error has three parts. Given this code:

```js
const user = { name: "Amina" };
console.log(user.profile.email);
```

Chrome shows:

```
Uncaught TypeError: Cannot read properties of undefined (reading 'email')
    at app.js:2
```

| Part | Here | Meaning |
|------|------|---------|
| **Type** | `TypeError` | The category of problem |
| **Message** | `Cannot read properties of undefined (reading 'email')` | What exactly happened |
| **Location** | `app.js:2` | File and line — **click it** in DevTools to jump there! |

Translation: *on line 2, something was `undefined`, and you asked it for `.email`.* What was undefined? The thing before `.email` → `user.profile`. Why? `user` has no `profile` property. Mystery solved, in seconds — because we read the message instead of panicking.

---

## 2. The big three error types

### `SyntaxError` — "I can't even read this"

Broken grammar: a missing bracket, stray comma, typo in a keyword. The program **doesn't run at all**.

```js
if (age > 18 {          // SyntaxError: missing )
console.log("hi";       // SyntaxError: missing )
functoin greet() {}     // SyntaxError: unexpected identifier
```

Fix: go to the reported line — though note the *real* mistake is sometimes a line or two **above** where JS noticed (e.g. an unclosed brace confuses everything after it). VS Code's red squiggles catch most of these before you even run.

### `ReferenceError` — "I've never heard of that name"

Using a variable that doesn't exist — almost always a typo or a scope problem:

```js
let userName = "Amina";
console.log(username);       // ReferenceError: username is not defined  (capital N!)
```

### `TypeError` — "That value can't do that"

The name exists, but the value can't do what you asked. The #1 flavor, which you will see hundreds of times in your career:

> **`Cannot read properties of undefined (reading 'x')`**

Something to the left of `.x` was `undefined`. Common causes: a missing object property, an array index past the end, a function that forgot to `return`, a misspelled property name. Method: look left of the dot, log that thing, find out why it's undefined.

Also in this family:

```js
const n = 42;
n.toUpperCase();     // TypeError: n.toUpperCase is not a function (numbers don't have it)
```

"...is not a function" usually means: typo in the method name, or the value isn't the type you think it is.

---

## 3. Debugging — the method

A **bug** is when the program runs but does the wrong thing. No red text, just wrong answers — sneakier than errors! Debugging is detective work, and it has a method:

1. **Reproduce** — make the bug happen on demand. What input triggers it?
2. **Read** — the error message (if any), and the code around the suspect line. *Slowly.*
3. **Isolate** — find where reality diverges from expectation. Main tool: `console.log`.
4. **Fix** — change ONE thing. Re-run. (Change three things and you won't know which mattered.)
5. **Verify** — does the original problem case now work? Did you break anything else?

### `console.log` debugging — the developer's flashlight

The core move: **check your assumptions by printing what's actually there.**

```js
function getAverage(numbers) {
  console.log("input is:", numbers);              // is it what I think?
  console.log("type:", typeof numbers);
  let sum = 0;
  for (const n of numbers) {
    sum += n;
    console.log("after adding", n, "sum is", sum);   // watch it evolve
  }
  return sum / numbers.length;
}
```

Tips that multiply its power:

```js
console.log("total:", total);        // LABEL your logs — five naked numbers help nobody
console.log({ total });              // trick: wraps it as {total: 9038} — auto-labeled!
console.table(products);             // arrays of objects as a beautiful table — try it!
```

A story in one line: you expect `price + 10` to be `60` but get `"5010"`. Log `typeof price` → `"string"`. The bug was never in the math — it was in the data. **Most "impossible" bugs are wrong assumptions about what a value actually is.**

### Rubber duck debugging (yes, it's a real technique)

Explain your code, line by line, out loud, to anything — a rubber duck, a younger sibling, an empty chair. Forcing the explanation makes your brain actually *look* at each line instead of skimming what you *meant* to write. It's uncanny how often you find the bug at line three of your own explanation.

### The debugger (a preview)

DevTools → **Sources** tab → click a line number to set a **breakpoint**. The program pauses there and you can inspect every variable and step line by line. `console.log` covers you for now, but know this exists — we'll use it naturally once we're doing DOM work.

---

## 4. `try/catch` — handling errors gracefully

Sometimes errors aren't bugs — they're *reality*. The network fails. The user types garbage. A file is missing. In those cases you don't want the whole program to die with red text; you want to **catch** the problem and respond sensibly.

```js
try {
  // risky code goes here
  const data = JSON.parse(userInput);      // throws if userInput isn't valid JSON
  console.log("Parsed!", data);
} catch (error) {
  // runs ONLY if something in try threw an error
  console.log("That wasn't valid data. Details:", error.message);
}

console.log("Program continues either way.");
```

How it flows: run the `try` block; the instant anything throws, jump to `catch` (with the error object in hand); either way, life goes on afterward.

### Throwing your own errors

Your functions can refuse bad input:

```js
function withdraw(balance, amount) {
  if (amount <= 0) {
    throw new Error("Amount must be positive");
  }
  if (amount > balance) {
    throw new Error("Insufficient funds");
  }
  return balance - amount;
}

try {
  const newBalance = withdraw(1000, 5000);
} catch (error) {
  console.log(`Transaction failed: ${error.message}`);   // "Insufficient funds"
}
```

This pattern — *validate, throw on bad input, catch at the boundary, report cleanly* — is **the** shape of robust code. Backend spoiler: half of server programming is exactly this (bad requests, database failures, invalid logins). Learn it now, thank yourself later.

**When to use `try/catch`:** around genuinely risky operations (parsing, network — Lesson 16). **Not** as tape over your own bugs — if your code throws because of a typo, fix the typo.

---

## 5. Reading a stack trace

When an error happens deep inside function calls, the error shows the whole chain:

```
Uncaught Error: Insufficient funds
    at withdraw (app.js:7)          ← where it was thrown
    at processPayment (app.js:15)   ← who called withdraw
    at checkout (app.js:22)         ← who called processPayment
```

Read top-down: line 1 is where it exploded; the rest is the trail of calls that led there. Usually your bug is at the topmost line that's in *your* file.

---

## ⚠️ Debugging anti-patterns (don't do these)

- **Staring** at code hoping the bug reveals itself. Log something. Get *evidence*.
- **Shotgun changes** — changing 5 things at once. Now you have 2 bugs and no idea what did what.
- **"It must be the computer/browser being weird."** It's your code. It's always the code. (This acceptance stage comes to every programmer. 😄)
- **Deleting and rewriting from scratch** on the first frustration. Debugging IS the skill you're training.
- **Leaving 30 dead `console.log`s** in finished code. Sweep them out when the case is closed.

---

## ✅ Classwork

1. **Error safari** — type each of these, predict the error TYPE before running, then read the real message aloud and explain it:
   ```js
   console.log(myVar);
   const x = 5; x = 6;
   "hello".push("!");
   const obj = {}; obj.a.b;
   if (true { console.log("hi"); }
   ```
2. **Bug hunt** — this runs without errors but gives the wrong answer. Use `console.log` to find both bugs, then fix:
   ```js
   function averageAbove(scores, threshold) {
     let sum = 0;
     let count = 0;
     for (let i = 0; i <= scores.length; i++) {
       if (scores[i] > threshold);
       {
         sum += scores[i];
         count++;
       }
     }
     return sum / count;
   }
   console.log(averageAbove([10, 80, 90, 20], 50));   // expected 85
   ```
   *(Hint: there are two classic mistakes from earlier lessons in there.)*
3. Wrap `JSON.parse("{bad json}")` in try/catch and print a friendly message instead of dying.
4. Rubber duck: explain your Lesson 9 homework #1 solution to your teacher line by line, out loud.

## 📝 Homework

1. **Safe divide:** `safeDivide(a, b)` that throws `"Cannot divide by zero"` when b is 0. Call it twice inside try/catch — once fine, once with 0 — printing a clean message each time.
2. **Validator:** `validateUser(user)` that throws a *specific, descriptive* error if: `user.name` is missing/empty, `user.age` is not a number, or `user.age` is negative. Test with 4 different broken users and one good one, catching and printing each message. (Descriptive errors = kindness to your future self.)
3. **Plant three bugs:** take any working homework from Lessons 5–9, deliberately introduce three different bugs (one SyntaxError, one ReferenceError, one silent logic bug), and write down the exact error message (or wrong output) each produces. Next session, your teacher will plant bugs for YOU to find — this is practice for that.
4. In comments: what are the 5 steps of the debugging method? What does `try/catch` do that `if/else` can't?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **SyntaxError** | Broken grammar — code can't even start |
| **ReferenceError** | Unknown variable name (usually a typo) |
| **TypeError** | Value can't do what you asked (`undefined.x`, "not a function") |
| **bug** | Runs, but does the wrong thing |
| **stack trace** | The chain of calls that led to an error |
| **breakpoint** | A DevTools pause-point for stepping through code |
| **`try/catch`** | Run risky code; jump to `catch` if it throws |
| **`throw new Error("...")`** | Raise your own error on bad input |
