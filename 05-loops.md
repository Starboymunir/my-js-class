# Lesson 05 — Loops: Repeating Work

> **Goal:** Make the computer repeat actions without repeating code. Computers are champions of repetition — loops are how you unleash that. This lesson needs *lots* of practice; budget extra time for it.

---

## 1. Why loops?

Print "I love JavaScript" five times:

```js
console.log("I love JavaScript");
console.log("I love JavaScript");
console.log("I love JavaScript");
console.log("I love JavaScript");
console.log("I love JavaScript");
```

Painful. Now imagine 5,000 times, or "once for every product in the store," or "until the user gets the answer right." Copy-paste can't do that. Loops can:

```js
for (let i = 0; i < 5; i++) {
  console.log("I love JavaScript");
}
```

Five lines of output, and changing `5` to `5000` is a one-character edit. **Loops = "repeat this block, under these rules."**

---

## 2. The `while` loop — simplest first

`while` is just an `if` that repeats:

```js
let count = 1;

while (count <= 5) {
  console.log(`Count is ${count}`);
  count++;
}
console.log("Done!");
```

Output:
```
Count is 1
Count is 2
Count is 3
Count is 4
Count is 5
Done!
```

How the computer executes it — follow this slowly, it's the heart of the lesson:

1. Check the condition: `count <= 5`? (1 ≤ 5 → true) → run the block. Print, then `count` becomes 2.
2. **Go back up.** Check again: 2 ≤ 5 → true → run again. `count` becomes 3.
3. ... repeat ...
4. Eventually `count` is 6. Check: 6 ≤ 5 → **false** → skip the block, continue after the loop.

Each pass through the block is called an **iteration**. This loop had 5 iterations.

### ⚠️ Infinite loops

What if `count++` were missing? `count` stays 1, the condition stays true, and the loop runs **forever** — the page freezes. Every programmer creates an infinite loop eventually (close the tab, fix the code, laugh about it). A `while` loop needs three things:

1. A starting point (`let count = 1`)
2. A condition that can eventually fail (`count <= 5`)
3. **Something inside the loop that moves toward failing it** (`count++`)

Forget #3 and you loop forever.

---

## 3. The `for` loop — the workhorse

Those three ingredients are so standard that `for` packs them into one line:

```js
for (let i = 0; i < 5; i++) {
  console.log(`Iteration ${i}`);
}
```

```
for ( start        ; keep-going condition ; step   )
      let i = 0    ; i < 5                ; i++
```

- **start** — runs once, before anything: create the counter.
- **condition** — checked before *every* iteration; loop continues while true.
- **step** — runs after *every* iteration.

Output: `Iteration 0`, `1`, `2`, `3`, `4`. **Five iterations, starting from 0.**

### Why start at 0? Why `i`?

- Programmers count from 0 by convention — and it will match perfectly with arrays (next lesson), where the first item is item 0. `i < 5` with a start of 0 gives exactly 5 rounds: 0,1,2,3,4.
- `i` stands for *index/iteration*. For a simple counter it's universally understood — this is the one place a one-letter name is good style.

### The counter is a real variable — use it!

```js
// 5 times table
for (let i = 1; i <= 12; i++) {
  console.log(`5 x ${i} = ${5 * i}`);
}

// Countdown — loops can go DOWN too
for (let i = 10; i >= 1; i--) {
  console.log(i);
}
console.log("Liftoff! 🚀");

// Step by 2 — even numbers from 0 to 20
for (let i = 0; i <= 20; i += 2) {
  console.log(i);
}
```

---

## 4. The accumulator pattern ⭐

The single most important loop pattern. **Build up a result across iterations** by keeping a variable *outside* the loop and updating it *inside*:

```js
// Sum of 1..100
let total = 0;                      // the accumulator, starts empty

for (let i = 1; i <= 100; i++) {
  total += i;                       // each iteration adds its bit
}

console.log(total);                 // 5050
```

Trace the start: `total`=0 → i=1, total=1 → i=2, total=3 → i=3, total=6 → ... The loop "accumulates" the answer.

Accumulators aren't only numbers:

```js
// Building a string
let stars = "";
for (let i = 0; i < 5; i++) {
  stars += "⭐";
}
console.log(stars);   // ⭐⭐⭐⭐⭐

// Counting things that match a condition
let evenCount = 0;
for (let i = 1; i <= 20; i++) {
  if (i % 2 === 0) {
    evenCount++;
  }
}
console.log(`There are ${evenCount} even numbers between 1 and 20.`);  // 10
```

That last example shows the big unlock: **an `if` inside a loop**. Repeat + decide = most of programming. Study it until it's obvious.

---

## 5. `break` and `continue` — loop controls

```js
// break: exit the loop immediately
for (let i = 1; i <= 100; i++) {
  if (i * i > 50) {
    console.log(`First number whose square exceeds 50: ${i}`);
    break;                       // stop looping entirely
  }
}

// continue: skip the REST of this iteration, jump to the next one
for (let i = 1; i <= 10; i++) {
  if (i % 3 === 0) {
    continue;                    // skip multiples of 3
  }
  console.log(i);                // 1 2 4 5 7 8 10
}
```

Use sparingly — most loops don't need them — but recognize them on sight.

---

## 6. Nested loops — a loop inside a loop

The whole inner loop runs completely for **each single iteration** of the outer loop:

```js
for (let row = 1; row <= 3; row++) {
  for (let col = 1; col <= 3; col++) {
    console.log(`row ${row}, col ${col}`);
  }
}
// row1col1, row1col2, row1col3, row2col1, ... 9 lines total (3 × 3)
```

Classic use — drawing a grid/triangle:

```js
let output = "";
for (let i = 1; i <= 5; i++) {
  let line = "";
  for (let j = 1; j <= i; j++) {
    line += "*";
  }
  output += line + "\n";        // \n = new line character
}
console.log(output);
// *
// **
// ***
// ****
// *****
```

Nested loops melt beginner brains at first — that's normal. Trace one on paper: write down `i`, `j`, and the output at every step. Once traced by hand, it clicks.

---

## 7. `for` vs `while` — which to use?

- **Know how many times?** → `for`. ("do this 10 times", "once per item")
- **Repeat until something happens?** → `while`. ("until the user guesses right", "while there is data left")

```js
// while shines when the count is unknown:
let waterLevel = 100;
let sips = 0;
while (waterLevel > 0) {
  waterLevel -= 23;
  sips++;
}
console.log(`Finished the bottle in ${sips} sips.`);
```

(There's also `do...while`, which checks the condition *after* running once — rare; just know it exists.)

---

## ⚠️ Common mistakes

```js
// 1. Off-by-one errors — the eternal classic
for (let i = 0; i <= 5; i++) { }   // runs 6 times (0..5)! Wanted 5? Use i < 5.
// Always ask: "what's the first value of i? the last? how many is that?"

// 2. Infinite loop — condition never becomes false
while (x > 0) { console.log(x); }  // forgot x-- → forever

// 3. Wrong direction
for (let i = 10; i >= 0; i++) { }  // i GROWS, never < 0 → infinite. Wanted i--.

// 4. Accumulator declared INSIDE the loop
for (let i = 1; i <= 5; i++) {
  let total = 0;                   // ❌ reset to 0 every iteration!
  total += i;
}
// The accumulator must live OUTSIDE the loop.

// 5. Semicolon after the loop header
for (let i = 0; i < 5; i++);       // ❌ that ; is the (empty!) loop body
{ console.log(i); }
```

---

## ✅ Classwork

1. Print the numbers 1–20. Then modify: only odd numbers. Then: each number and whether it's odd or even (`if` inside the loop).
2. **Trace on paper** (no computer!): what does this print?
   ```js
   let t = 0;
   for (let i = 1; i <= 4; i++) {
     t += i * 2;
   }
   console.log(t);
   ```
   Write the value of `i` and `t` at every step, then verify in the console.
3. Times table: ask which number (just set `const n = 7;`), print its full 1–12 table.
4. Sum all multiples of 3 between 1 and 50 (accumulator + `if` + `%`).
5. Build the star triangle from section 6, then flip it upside down.

## 📝 Homework

1. **FizzBuzz** — the most famous beginner exercise (used in real job interviews!). Print numbers 1–30, BUT: multiples of 3 print `"Fizz"` instead, multiples of 5 print `"Buzz"`, multiples of BOTH print `"FizzBuzz"`. *Hint: check the "both" case first — why?*
2. **Savings calculator:** you save 500 per week. Use a loop to find how many weeks until you pass 20,000, and print a progress line each week (`Week 3: 1500 saved`). (Which loop type fits — `for` or `while`? Why?)
3. **Factorial:** `5! = 5×4×3×2×1 = 120`. Compute the factorial of `n = 8` with a loop. *Hint: the accumulator must start at 1, not 0 — why?*
4. **Countdown by tens:** 100, 90, 80 ... 0, then "Done!".
5. **Challenge — multiplication grid:** nested loops printing the full 1–10 times table, one line per row, like `1 2 3 4 5 6 7 8 9 10`, `2 4 6 8 ...`, etc. (Build each row as a string, then log it.)

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **loop** | Code that repeats while a condition holds |
| **iteration** | One pass through the loop body |
| **counter** | The variable tracking progress (usually `i`) |
| **accumulator** | A variable outside the loop that collects a result |
| **infinite loop** | A loop whose condition never fails — freezes the program |
| **off-by-one** | Looping one time too many/few — check `<` vs `<=` |
| **`break` / `continue`** | Exit the loop / skip to the next iteration |
