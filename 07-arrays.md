# Lesson 07 — Arrays: Lists of Data

> **Goal:** Store many values in one variable and work with them as a group. Almost all real data comes in lists — products, messages, users, scores — and arrays are how JavaScript holds lists.

---

## 1. Why arrays?

Storing three test scores without arrays:

```js
const score1 = 80;
const score2 = 92;
const score3 = 75;
```

Now imagine 300 students. Or "however many items the user adds to the cart" — you don't even *know* how many variables you'd need! The fix: one variable that holds a whole ordered list.

```js
const scores = [80, 92, 75];
```

An **array** is an ordered collection of values in a single container.

```js
const fruits = ["apple", "banana", "mango"];
const mixed = [42, "hello", true, null];      // types can mix (usually keep them uniform though)
const empty = [];                             // an empty list, ready to be filled
```

---

## 2. Reading items — indexes

Each item has a numbered position, its **index** — **and counting starts at 0**:

```js
const fruits = ["apple", "banana", "mango"];
//    index:      0         1         2

console.log(fruits[0]);   // "apple"   ← the FIRST item is index 0!
console.log(fruits[1]);   // "banana"
console.log(fruits[2]);   // "mango"
console.log(fruits[3]);   // undefined — no such position (not an error, just nothing there)
```

Zero-based counting feels odd for a week and then becomes second nature. Mantra: **index = distance from the start.** The first item is 0 steps from the start.

### `length` — how many items

```js
console.log(fruits.length);              // 3
console.log(fruits[fruits.length - 1]);  // "mango" — the LAST item (length 3, but last index is 2!)
```

That `length - 1` for the last item is a classic. Off-by-one alert: `fruits[fruits.length]` is `undefined`.

### Changing an item

```js
fruits[1] = "orange";
console.log(fruits);   // ["apple", "orange", "mango"]
```

**Wait — `fruits` is a `const`! Why is this allowed?** Important subtlety: `const` locks the *variable* (you can't point `fruits` at a different array), but the array's *contents* can change. The box is welded to the shelf; you can still rearrange things inside the box.

```js
fruits = ["new", "array"];   // ❌ TypeError — can't reassign the variable
fruits[0] = "pear";          // ✅ fine — modifying contents
fruits.push("kiwi");         // ✅ fine
```

---

## 3. Adding and removing items

Arrays come with built-in **methods** — functions attached to the array, called with a dot:

```js
const list = ["a", "b", "c"];

// END of the array
list.push("d");        // add to end        → ["a","b","c","d"]
list.pop();            // remove from end   → ["a","b","c"]   (and returns the removed "d")

// START of the array
list.unshift("z");     // add to start      → ["z","a","b","c"]
list.shift();          // remove from start → ["a","b","c"]   (returns "z")
```

Memory aid: **push/pop** work at the end (like a stack of plates); **shift/unshift** shove everything over at the start. `push` is by far the most used — especially with the accumulator pattern:

```js
// Collect all even numbers from 1–20 into an array
const evens = [];                 // array accumulator!
for (let i = 1; i <= 20; i++) {
  if (i % 2 === 0) {
    evens.push(i);
  }
}
console.log(evens);   // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

Recognize this? It's Lesson 5's accumulator pattern with an array instead of a number. *Start empty, loop, push what qualifies.* You will write this shape a thousand times.

---

## 4. Looping over arrays ⭐

Arrays and loops are inseparable. "Do something with EACH item" is the bread and butter of programming.

### The classic `for` loop

```js
const fruits = ["apple", "banana", "mango"];

for (let i = 0; i < fruits.length; i++) {
  console.log(`${i}: ${fruits[i]}`);
}
// 0: apple
// 1: banana
// 2: mango
```

Note the pattern: start at 0, condition `i < array.length` (not `<=`! — off-by-one), use `array[i]` inside. Memorize this shape.

### `for...of` — when you don't need the index

```js
for (const fruit of fruits) {
  console.log(fruit);
}
```

Reads like English: "for each fruit of fruits." Cleaner when you only need the values. Use the classic `for` when you need the index or fancy stepping.

### Real example — computing with a loop

```js
const prices = [1200, 350, 4999, 89, 2400];

// Total
let total = 0;
for (const price of prices) {
  total += price;
}
console.log(`Total: ${total}`);       // 9038

// Biggest
let max = prices[0];                  // start with the first item, NOT 0 (why? what if all were negative?)
for (const price of prices) {
  if (price > max) {
    max = price;
  }
}
console.log(`Most expensive: ${max}`);  // 4999
```

Sum, max, min, count, average — these five loop patterns cover a huge share of everyday programming (and of interviews). Practice each until you can write them without notes.

---

## 5. Useful array methods (a starter kit)

```js
const nums = [10, 20, 30, 40];

nums.includes(20);        // true — is this value in the array?
nums.indexOf(30);         // 2 — at which index? (-1 if not found)

const part = nums.slice(1, 3);   // [20, 30] — COPY a portion (start index, up-to-but-not-including end)
console.log(nums);               // unchanged! slice copies, never modifies

nums.reverse();           // [40, 30, 20, 10] — reverses IN PLACE (modifies the original)

const letters = ["c", "a", "b"];
letters.sort();           // ["a", "b", "c"]

const joined = ["red", "green", "blue"].join(", ");
console.log(joined);      // "red, green, blue" — array → string

const csv = "one,two,three";
const parts = csv.split(",");    // "string method" — string → array: ["one","two","three"]
```

⚠️ One famous trap for later: `.sort()` on numbers sorts them *as text* by default (`[1, 10, 2]`!). We'll fix that in Lesson 9. And note which methods **modify** the array (push, pop, reverse, sort) versus **copy** (slice, join) — mixing those up causes sneaky bugs.

The mega-methods `map`, `filter`, and `reduce` also live on arrays — they deserve their own section and get it in **Lesson 9**, after you're fluent with the basics.

---

## 6. Arrays + functions together

```js
function getAverage(numbers) {
  if (numbers.length === 0) return 0;     // guard: avoid dividing by zero
  let sum = 0;
  for (const n of numbers) {
    sum += n;
  }
  return sum / numbers.length;
}

const testScores = [80, 92, 75, 88];
console.log(getAverage(testScores));      // 83.75
console.log(getAverage([100, 50]));       // 75 — reusable for ANY list!
```

This is the payoff of Lessons 5–7 combined: a named, reusable, tested piece of logic. Backend code looks exactly like this — a function receiving a list of database records and returning a computed answer.

---

## ⚠️ Common mistakes

```js
const arr = ["a", "b", "c"];

// 1. First item is [1]
arr[1]                    // "b"! The first is arr[0].

// 2. Last item via length
arr[arr.length]           // undefined! Last is arr[arr.length - 1].

// 3. <= in loop condition
for (let i = 0; i <= arr.length; i++)   // one extra iteration → undefined at the end

// 4. Expecting slice to modify / forgetting sort DOES modify

// 5. Comparing arrays with ===
[1,2] === [1,2]           // false! Arrays are compared by reference (are they the SAME array?),
                          // not by contents. More on this in the objects lesson.
```

---

## ✅ Classwork

1. Make an array of 5 favorite foods. Print: the first, the last (via `length`), and the whole array. Replace the middle one. Add one to the end, remove one from the start.
2. Loop over the foods printing `"1. jollof rice"`, `"2. suya"`, ... (numbered from 1, even though indexes start at 0 — how?).
3. `const temps = [28, 31, 25, 33, 29];` — using loops: print the highest, the lowest, and the average.
4. Build `[100, 90, 80, ... 10]` using a loop and `push` (don't type it manually!).
5. **Predict then run:**
   ```js
   const x = [1, 2, 3];
   x.push(x.length);
   x.push(x.length);
   console.log(x);
   ```

## 📝 Homework

1. **Shopping cart:** create `const cart = [];`. Push 4 prices into it. Write `cartTotal(prices)` returning the sum, and `mostExpensive(prices)` returning the max. Print a receipt line: `` `4 items, total 8038, priciest 4999` ``.
2. **Word filter:** given `const words = ["hey", "elephant", "hi", "programming", "yes", "banana"];` — build a NEW array containing only words longer than 4 letters (loop + `if` + `push`). Print both arrays.
3. **Countdown array:** write `makeCountdown(n)` that returns `[n, n-1, ..., 1, "Go!"]`. Test with `makeCountdown(5)`.
4. **Search:** write `findIndex(arr, target)` that loops and returns the index where `target` first appears, or `-1` if absent. (Yes, `indexOf` exists — build it yourself once to understand what it does under the hood!)
5. **Challenge — reverse:** write `reverseArray(arr)` returning a NEW reversed array, without using `.reverse()`. Hint: loop backwards and push.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **array** | Ordered list of values: `[1, 2, 3]` |
| **index** | An item's position, **starting at 0** |
| **`length`** | Number of items (last index = length − 1) |
| **method** | A function attached to a value, called with a dot: `arr.push(x)` |
| **`push` / `pop`** | Add / remove at the end |
| **`shift` / `unshift`** | Remove / add at the start |
| **`for...of`** | Loop over values directly |
| **mutate** | Modify the original array (push, sort…) vs copy (slice) |
