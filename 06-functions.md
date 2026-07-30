# Lesson 06 — Functions ⭐ (The Most Important Lesson)

> **Goal:** Learn to package code into reusable, named machines. Functions are the single most important concept in this course. All frontend code, all backend code, all of professional programming is organized as functions. Take this lesson slowly and do every exercise.

---

## 1. What is a function?

A **function** is a named block of code that you can run (**call**) whenever you want, as many times as you want.

You've already been *calling* functions: `console.log(...)`, `alert(...)`, `Number(...)` — someone else wrote those machines; you just pressed their button. Today you build your own.

### Defining and calling

```js
// DEFINE the function (build the machine — nothing runs yet!)
function greet() {
  console.log("Hello!");
  console.log("Welcome to the program.");
}

// CALL the function (press the button — NOW it runs)
greet();
greet();
greet();
```

Output: the two lines print three times.

Two separate moments — this trips up every beginner:

1. **Definition** (`function greet() { ... }`) — teaches JavaScript the recipe. **No code inside runs yet.**
2. **Call** (`greet()`) — the parentheses are the "run it" command. Without `()`, nothing happens.

```js
console.log(greet);     // prints the function itself (the recipe) — didn't run it
console.log(greet());   // runs it
```

### Why functions? Three reasons

1. **Reuse** — write once, use everywhere. Fix a bug once, it's fixed everywhere.
2. **Organization** — a big program becomes small named pieces: `validateEmail()`, `calculateTotal()`, `showError()`. You can read what a program does from its function names.
3. **Thinking tool** — functions let you *ignore* details. Once `calculateTotal()` works, you never think about how again — you just use it. This is how humans manage big programs.

---

## 2. Parameters — giving the machine input

A greeting machine that can only say "Hello!" is boring. **Parameters** let the caller pass information in:

```js
function greet(name) {           // name is a PARAMETER — a placeholder variable
  console.log(`Hello, ${name}!`);
}

greet("Amina");    // "Hello, Amina!"   — "Amina" is the ARGUMENT
greet("Bola");     // "Hello, Bola!"
```

Think of a parameter as an empty slot in the machine. Each call fills the slot with a specific value (the **argument**), and inside the function the parameter behaves like a normal variable holding that value.

> **Vocabulary:** *parameter* = the name in the definition. *argument* = the actual value you pass when calling. People mix them up constantly; you don't have to — but do know both words, documentation uses them.

### Multiple parameters

```js
function introduce(name, age, city) {
  console.log(`${name} is ${age} years old and lives in ${city}.`);
}

introduce("Amina", 15, "Kano");     // matched by POSITION: name="Amina", age=15, city="Kano"
introduce(15, "Kano", "Amina");     // 🙃 works but nonsense — order matters!
```

### Default values

```js
function greet(name = "friend") {
  console.log(`Hello, ${name}!`);
}
greet("Sam");   // "Hello, Sam!"
greet();        // "Hello, friend!"  — missing argument → default kicks in
```

---

## 3. `return` — getting output back ⭐⭐

So far our functions only print. But printing is a dead end — you can't do further math with something that went to the console. Real functions **hand a value back to their caller** with `return`:

```js
function add(a, b) {
  return a + b;        // compute the value and SEND IT BACK
}

const result = add(3, 4);
console.log(result);            // 7
console.log(add(10, 20) + 1);   // 31 — the call itself BECOMES the value 7... er, 30!
```

Mental model: **when a function returns, the entire call expression is replaced by the returned value.** `add(3, 4)` *becomes* `7` right there in the code. That's why you can store it, print it, pass it to another function, or use it in more math:

```js
const bigger = add(add(1, 2), add(3, 4));   // add(3, 7) → 10
```

### `console.log` vs `return` — the classic confusion

| | `console.log(x)` | `return x` |
|---|---|---|
| Purpose | Show a value to the **human** (for debugging) | Hand a value to the **calling code** |
| Can the program use the value afterward? | ❌ No | ✅ Yes |
| Analogy | Saying the answer out loud | Writing the answer on the exam paper |

```js
function addBroken(a, b) {
  console.log(a + b);            // prints 7... but returns nothing
}

const x = addBroken(3, 4);       // prints 7 (from the log)
console.log(x);                  // undefined 😱 — nothing was returned!
```

A function with no `return` (or a bare `return;`) gives back `undefined`. If you ever see `undefined` where you expected a number, check: *did my function actually return?*

**Rule of thumb: functions should `return` their result; `console.log` is only for peeking.** This matters enormously for backend work later — server code returns data; it doesn't print it.

### `return` exits immediately

The moment `return` runs, the function is DONE — nothing below it executes:

```js
function checkAge(age) {
  if (age < 0) {
    return "Invalid age!";       // early exit — the rest is skipped
  }
  if (age >= 18) {
    return "Adult";
  }
  return "Minor";                // only reached if neither return above fired
}

console.log(checkAge(-5));   // "Invalid age!"
console.log(checkAge(25));   // "Adult"
console.log(checkAge(12));   // "Minor"
```

This "early return" pattern — handle the bad/special cases first and bail out — keeps functions flat and readable. Professionals use it constantly.

---

## 4. Scope — where variables live

Variables declared **inside** a function exist only inside it:

```js
function calculate() {
  const secret = 42;          // local variable
  console.log(secret);        // 42 — visible in here
}

calculate();
console.log(secret);          // ❌ ReferenceError: secret is not defined
```

This is called **scope**, and it's a feature, not a limitation: each function is a private workshop. Two functions can both use a variable named `total` without ever colliding.

The rules:

```js
const appName = "SuperApp";        // GLOBAL scope — visible everywhere

function outer() {
  const a = 1;                     // visible in outer AND in blocks/functions inside it
  if (true) {
    const b = 2;                   // block scope — visible only inside these { }
    console.log(appName, a, b);    // ✅ inner code can see outward
  }
  console.log(b);                  // ❌ outer code cannot see into inner blocks
}
```

**Inner can see out; outer cannot see in.** (This one-way glass is also why `var` is banned in this course: `var` ignores block scope and leaks out of `{ }`, which breaks these rules and causes ghost bugs. `let`/`const` behave properly.)

Practical advice: **prefer local variables.** Globals that everyone can modify become impossible to track in big programs. Pass data *in* via parameters, get it *out* via return.

---

## 5. Other ways to write functions

JavaScript has three syntaxes for creating functions. Same machine, different packaging — you must recognize all three because real code uses all three.

### 5.1 Function declaration (what we've done so far)

```js
function double(x) {
  return x * 2;
}
```

### 5.2 Function expression — storing a function in a variable

```js
const double = function (x) {
  return x * 2;
};

double(5);   // 10 — called exactly the same way
```

Wait — a function stored in a *variable*? Yes! **In JavaScript, functions are values**, just like numbers and strings. They can be stored, passed as arguments, even returned from other functions. Park that thought — it becomes hugely important in Lesson 9 (array methods) and Lesson 12 (events).

### 5.3 Arrow function — the modern shorthand ⭐

```js
const double = (x) => {
  return x * 2;
};
```

Read `=>` as "goes to": *"x goes to x times 2."* And when the body is a single expression, it shrinks beautifully:

```js
const double = (x) => x * 2;         // no braces, no 'return' — the expression IS the return value
const add = (a, b) => a + b;
const sayHi = () => console.log("Hi!");   // no parameters → empty ()
```

⚠️ The #1 arrow mistake: adding `{ }` but forgetting `return`:

```js
const double = (x) => { x * 2 };   // ❌ returns undefined! With braces you MUST write return.
const double = (x) => x * 2;       // ✅ or drop the braces
```

**Which should you use?** In this course: function declarations for standalone named functions, arrows for short functions passed to other functions (coming in Lesson 9). Both are everywhere in real code.

---

## 6. Functions calling functions — composing

Real programs are functions built out of other functions:

```js
function calculateSubtotal(price, quantity) {
  return price * quantity;
}

function calculateTax(amount) {
  return amount * 0.075;                 // 7.5% VAT
}

function calculateTotal(price, quantity) {
  const subtotal = calculateSubtotal(price, quantity);
  const tax = calculateTax(subtotal);
  return subtotal + tax;
}

console.log(calculateTotal(1000, 3));    // 3225
```

Notice how readable `calculateTotal` is — it reads like a description of the process. Each function does **one job** and does it well. That's the design principle: *small functions, one responsibility, good names.* It applies identically to the backend code you'll write later.

---

## ⚠️ Common mistakes

```js
// 1. Defining but never calling — "why doesn't my code run?"
function doStuff() { console.log("hi"); }
// ...nothing happens without: doStuff();

// 2. Calling without parentheses
doStuff;          // does nothing (just refers to the function)

// 3. Expecting a value from a function that only logs
const r = printSum(2, 3);   // undefined if printSum has no return

// 4. Code after return (never runs)
function f() {
  return 5;
  console.log("unreachable");   // dead code
}

// 5. Using a local variable outside its function → ReferenceError

// 6. Arrow with braces but no return → undefined (see above)
```

---

## ✅ Classwork

1. Write `square(n)` that **returns** n². Test: `console.log(square(4) + square(3))` → should be 25. (If it prints `NaN` or `undefined`, you logged instead of returned!)
2. Write `isEven(n)` → returns `true`/`false`. *One-liner hint: `return n % 2 === 0;`*
3. Write `getGrade(score)` using your Lesson 4 else-if chain, but **returning** the grade instead of printing. Then: `console.log(getGrade(67));`
4. Convert these to arrow functions: `function triple(x) { return x * 3; }` and `function shout(word) { return word + "!!!"; }`
5. **Predict then run:**
   ```js
   function mystery(a, b) {
     if (a > b) return a;
     return b;
   }
   console.log(mystery(3, 8));
   console.log(mystery(10, 2));
   ```
   What does `mystery` do? What would be a better name for it?

## 📝 Homework

1. **Temperature converter:** `celsiusToFahrenheit(c)` returns `c * 9/5 + 32`, and `fahrenheitToCelsius(f)` does the reverse. Test both directions: 0°C→32°F, 100°C→212°F.
2. **Calculator:** write `add`, `subtract`, `multiply`, `divide` (all returning). In `divide`, if the divisor is 0, return the string "Cannot divide by zero" (early return!).
3. **Greeting machine:** `makeGreeting(name, timeOfDay)` returns `"Good morning, Amina!"` etc. Give `timeOfDay` a default of `"day"`.
4. **Compose:** write `max2(a, b)` (returns the bigger of two), then use it to write `max3(a, b, c)` — *without* new if statements. Hint: `max2(a, max2(b, c))`.
5. **Refactor:** take your FizzBuzz homework from Lesson 5 and wrap the "decide what to print for one number" logic into a function `fizzbuzzWord(n)` that returns `"Fizz"`, `"Buzz"`, `"FizzBuzz"`, or the number. The loop then just calls it. Notice how much cleaner it reads.
6. In comments, your own words: what's the difference between a parameter and an argument? Between `return` and `console.log`?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **function** | A named, reusable block of code |
| **call / invoke** | Run a function: `name()` |
| **parameter** | Input slot in the definition |
| **argument** | Actual value passed in a call |
| **return** | Hand a value back to the caller (and exit) |
| **scope** | Where a variable is visible; inner sees out, outer can't see in |
| **local / global** | Declared inside a function / outside everything |
| **arrow function** | Compact modern syntax: `(x) => x * 2` |
