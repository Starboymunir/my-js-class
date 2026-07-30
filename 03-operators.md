# Lesson 03 — Operators and Expressions

> **Goal:** Learn the symbols JavaScript uses to calculate, compare, and combine values. These are the "verbs" of the language and appear on nearly every line of real code.

---

## 1. Expressions — code that produces a value

An **expression** is any piece of code that produces a value:

```js
5 + 3            // produces 8
age * 2          // produces a number (whatever age holds, doubled)
"Hi " + name     // produces a string
age >= 18        // produces true or false
```

Operators are what build expressions. Wherever JavaScript expects a value, you can put an expression — inside `console.log()`, on the right of `=`, inside `${}` in a template literal, anywhere.

---

## 2. Arithmetic operators (math)

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | addition | `5 + 2` | `7` |
| `-` | subtraction | `5 - 2` | `3` |
| `*` | multiplication | `5 * 2` | `10` |
| `/` | division | `5 / 2` | `2.5` |
| `%` | **remainder (modulo)** | `5 % 2` | `1` |
| `**` | exponent (power) | `5 ** 2` | `25` |

Most are familiar from school. The new one is `%`:

### The remainder operator `%` — surprisingly useful

`a % b` gives the **remainder** after dividing a by b:

```js
console.log(10 % 3);   // 1   (10 ÷ 3 = 3 remainder 1)
console.log(12 % 4);   // 0   (divides evenly)
console.log(7 % 2);    // 1
console.log(8 % 2);    // 0
```

Why care? Two everyday tricks:

```js
// Is a number even? Even numbers have remainder 0 when divided by 2.
console.log(6 % 2 === 0);   // true  → even
console.log(7 % 2 === 0);   // false → odd

// Wrap-around: what time is it 5 hours after 22:00?
console.log((22 + 5) % 24); // 3  (3 AM)
```

You will use `%` in loops, games, calendars, and pagination — it comes up more than you'd expect.

### Order of operations

Same as math class: `*`, `/`, `%` before `+`, `-`. Use parentheses to control or clarify:

```js
console.log(2 + 3 * 4);     // 14, not 20
console.log((2 + 3) * 4);   // 20
```

**Tip:** even when parentheses aren't required, use them if they make the intent clearer. Code is for humans first.

---

## 3. Assignment shortcuts

Updating a variable based on its current value is so common that there are shortcuts:

```js
let score = 10;

score = score + 5;   // the long way
score += 5;          // the short way — same thing

score -= 3;          // score = score - 3
score *= 2;          // score = score * 2
score /= 4;          // score = score / 4
```

### Increment and decrement

Adding/subtracting exactly 1 is even more common (counting!), so it gets its own operators:

```js
let count = 0;
count++;             // count is now 1
count++;             // 2
count--;             // 1
```

You'll see `++` constantly in loops (next lessons).

---

## 4. Comparison operators — asking questions

Comparisons ask a question about two values and answer with a **boolean** (`true`/`false`):

| Operator | Question | Example | Result |
|----------|----------|---------|--------|
| `>` | greater than? | `7 > 3` | `true` |
| `<` | less than? | `7 < 3` | `false` |
| `>=` | greater or equal? | `5 >= 5` | `true` |
| `<=` | less or equal? | `4 <= 3` | `false` |
| `===` | equal? (strict) | `5 === 5` | `true` |
| `!==` | not equal? (strict) | `5 !== 3` | `true` |

```js
const age = 15;
console.log(age >= 18);        // false — not an adult yet
const canVote = age >= 18;     // you can store the answer in a variable!
console.log(canVote);          // false
```

That last pattern — storing a comparison's result in a well-named boolean — makes code very readable: `const isExpensive = price > 10000;`

### ⚠️ `=` vs `===` — the classic beginner trap

- `=` **assigns** ("put this in the box")
- `===` **compares** ("are these the same?")

```js
let x = 5;      // assignment
x === 5;        // comparison → true
```

Writing `if (x = 5)` when you meant `if (x === 5)` is one of the most common beginner bugs. If you see a comparison behaving strangely, count your equals signs.

### ⚠️ `===` vs `==` — always use three

JavaScript also has a two-equals version, `==`. The difference:

- `===` (**strict equality**) — equal value AND equal type. Predictable. ✅
- `==` (**loose equality**) — converts types before comparing, with confusing rules. ❌

```js
console.log(5 === "5");   // false — number vs string, different types. Sensible.
console.log(5 == "5");    // true  — JS silently converted... surprise bugs live here.
console.log(0 == "");     // true  😱
console.log(0 === "");    // false ✅
```

> **Rule for this course (and for professional code): always use `===` and `!==`. Never `==` or `!=`.**

---

## 5. Logical operators — combining questions

Real decisions often involve multiple conditions: *"discount if the customer is a student AND has a coupon."* Three operators combine booleans:

### `&&` — AND: "both must be true"

```js
const age = 20;
const hasTicket = true;

console.log(age >= 18 && hasTicket);   // true — both conditions hold
console.log(age >= 18 && false);       // false — one side fails, so the whole thing fails
```

| left | right | `left && right` |
|------|-------|------|
| true | true | **true** |
| true | false | false |
| false | true | false |
| false | false | false |

### `||` — OR: "at least one must be true"

```js
const isWeekend = false;
const isHoliday = true;

console.log(isWeekend || isHoliday);   // true — no school either way!
```

| left | right | `left \|\| right` |
|------|-------|------|
| true | true | **true** |
| true | false | **true** |
| false | true | **true** |
| false | false | false |

### `!` — NOT: flips a boolean

```js
const isRaining = false;
console.log(!isRaining);    // true — "it is NOT raining" is a true statement
console.log(!true);         // false
```

### Combining them

```js
const age = 25;
const isMember = false;
const isVip = true;

// Entry allowed if adult AND (member OR VIP)
const canEnter = age >= 18 && (isMember || isVip);
console.log(canEnter);   // true
```

Read complex conditions out loud in English — if the sentence sounds right, the logic probably is.

---

## 6. Operator questions to build intuition

Work through these in the console. **Predict first, then run:**

```js
10 % 3            // ?
"10" === 10       // ?
3 > 2 && 2 > 3    // ?
3 > 2 || 2 > 3    // ?
!(5 === 5)        // ?
"b" > "a"         // ? (strings compare alphabetically — true!)
7 % 2 === 1       // ? (is 7 odd?)
```

---

## ⚠️ Common mistakes

```js
// 1. One = in a comparison
if (score = 100) { ... }        // ❌ assigns 100! Wanted: score === 100

// 2. Using == instead of ===
"5" == 5                        // true, and that's the problem. Use ===.

// 3. Chaining comparisons like math class
// ❌ 10 < x < 20  — this "works" but does something totally different than you expect
// ✅ x > 10 && x < 20

// 4. Math on strings
"5" * "2"    // 10 — JS converts! Sometimes it "works" — which is worse, because
"5" + "2"    // "52" — this one doesn't. Never rely on it. Convert with Number() first.
```

---

## ✅ Classwork

1. A movie ticket costs 2500. Store it. Calculate the cost for a family of 5, print with a template literal.
2. Using `%`: is 2026 an even number? Is 777? Print `true`/`false` for each.
3. `const temp = 31;` — write expressions that answer: is it above 30? exactly 25? between 20 and 30 (inclusive)?
4. A cinema allows entry if age is 13+ **or** the person is with a parent. Given `const age = 10; const withParent = true;`, write one expression for `canWatch`.
5. Predict-then-run table: `5 === "5"`, `5 !== 4`, `true && false`, `false || true`, `!false`, `10 % 5`.

## 📝 Homework

1. **Grade helper:** `const examScore = 67;` Write and print boolean expressions: `passed` (score ≥ 50), `gotDistinction` (score ≥ 70), `needsRetake` (score < 40).
2. **Odd or even machine:** store any number in a variable; print `` `Is ${n} even? ${...}` `` using `%`.
3. **Discount logic:** a store gives a discount if the customer is a student AND the cart total is above 5000, OR if they have a coupon (regardless of anything else). Create the three variables, write the single boolean expression `getsDiscount`, and test it with at least 3 different combinations of values.
4. Explain in a comment, in your own words: why do we use `===` instead of `==`?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **expression** | Code that produces a value |
| **operator** | A symbol that acts on values (`+`, `===`, `&&`...) |
| **`%` modulo** | Remainder after division |
| **strict equality `===`** | Same value AND same type |
| **`&&` / `\|\|` / `!`** | AND / OR / NOT |
| **increment `++`** | Add 1 to a variable |
