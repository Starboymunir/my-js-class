# Lesson 08 — Objects: Structured Data

> **Goal:** Store related information together under named labels. If arrays are *lists*, objects are *profiles*. Objects are the most important data structure in JavaScript — the entire language (and every API you'll ever call, and every database record you'll ever fetch on the backend) is built on them.

---

## 1. Why objects?

Describe a student with what we know so far:

```js
const studentName = "Amina";
const studentAge = 15;
const studentCity = "Kano";
const studentIsEnrolled = true;
```

Clunky — and nothing actually *connects* these variables. With two students it becomes chaos. An array doesn't fit either: `["Amina", 15, "Kano", true]` — position 2 means... city? You'd have to memorize what each index means.

An **object** groups related values under **named labels**:

```js
const student = {
  name: "Amina",
  age: 15,
  city: "Kano",
  isEnrolled: true,
};
```

Each `label: value` pair is called a **property**. The label is the **key**, and keys are *names*, not positions. `student.age` says exactly what it is — readable by humans, which is the whole point.

- Array = ordered list, accessed by **number** → "the 3rd item"
- Object = labeled collection, accessed by **name** → "the age"

---

## 2. Reading and writing properties

### Dot notation (use this 95% of the time)

```js
console.log(student.name);         // "Amina"
console.log(student.age);          // 15
console.log(`${student.name} lives in ${student.city}.`);

student.age = 16;                  // change an existing property
student.hobby = "chess";           // add a NEW property — just assign to it!
delete student.hobby;              // remove a property (rarely needed)

console.log(student.height);       // undefined — asking for a missing property is not an error
```

(Like arrays, a `const` object's *contents* can change — you just can't point the variable at a different object.)

### Bracket notation (for special cases)

```js
console.log(student["name"]);      // same as student.name
```

Why does this exist? Because sometimes the key isn't known until the program runs, or isn't a valid name:

```js
const key = "city";
console.log(student[key]);         // "Kano" — the key comes from a VARIABLE. Dot can't do this:
console.log(student.key);          // undefined — looks for a property literally named "key"!

const scores = { "math test": 90 };   // key with a space — dot can't touch it
console.log(scores["math test"]);     // 90
```

**Rule:** dot when you know the name while writing the code; brackets when the name lives in a variable.

---

## 3. Nesting — objects and arrays inside objects

Values can be *anything* — including arrays and other objects. This is how real-world data is shaped:

```js
const student = {
  name: "Amina",
  age: 15,
  subjects: ["math", "physics", "english"],          // array inside object
  address: {                                          // object inside object
    city: "Kano",
    street: "12 Unity Road",
  },
};

console.log(student.subjects[0]);        // "math"     — walk in one step at a time
console.log(student.address.city);       // "Kano"
console.log(student.subjects.length);    // 3
```

Read chains left to right: `student.address.city` = "in student, take address; in that, take city."

### Arrays of objects ⭐ — THE shape of real data

The single most common data structure in all of web development:

```js
const products = [
  { name: "Laptop", price: 450000, inStock: true },
  { name: "Mouse", price: 8000, inStock: true },
  { name: "Monitor", price: 120000, inStock: false },
];

console.log(products[1].name);       // "Mouse" — item 1 of the array, then its name
console.log(products.length);        // 3

// Loop over them — everything from Lesson 7 applies:
for (const product of products) {
  console.log(`${product.name}: ₦${product.price}`);
}

// Combine with conditions — total value of in-stock items:
let total = 0;
for (const product of products) {
  if (product.inStock) {
    total += product.price;
  }
}
console.log(`In-stock value: ₦${total}`);   // 458000
```

When you later fetch products from a server (frontend) or read users from a database (backend), the data arrives in *exactly* this shape. Master "loop over an array of objects, filter and compute" and you've mastered daily-driver programming.

---

## 4. Methods and `this` — objects that can act

A property whose value is a *function* is called a **method**. (Surprise: you've been using methods all along — `arr.push()`, `console.log()` — `log` is a method of the `console` object!)

```js
const player = {
  name: "Amina",
  score: 0,

  addPoints(points) {                 // a method (shorthand syntax)
    this.score += points;             // `this` = "the object I belong to"
    console.log(`${this.name} now has ${this.score} points.`);
  },
};

player.addPoints(10);   // "Amina now has 10 points."
player.addPoints(5);    // "Amina now has 15 points."
```

`this` lets a method reach its own object's other properties. For now, one rule covers you: **inside a method called as `object.method()`, `this` is that object.** (`this` has trickier corners in advanced JS — not needed yet; we'll revisit if/when it bites.)

---

## 5. Working with keys and values

Three built-in helpers turn objects into arrays so you can loop over them:

```js
const scores = { math: 90, physics: 75, english: 82 };

Object.keys(scores);      // ["math", "physics", "english"]
Object.values(scores);    // [90, 75, 82]
Object.entries(scores);   // [["math", 90], ["physics", 75], ["english", 82]]

// Loop over all properties:
for (const subject of Object.keys(scores)) {
  console.log(`${subject}: ${scores[subject]}`);     // bracket notation — key is in a variable!
}
```

### Checking a property exists

```js
console.log("math" in scores);        // true
console.log(scores.art !== undefined) // another common way
```

---

## 6. References — why `===` acts weird with objects (important!)

Primitives (numbers, strings, booleans) are copied by **value**. Objects and arrays are copied by **reference** — the variable holds a *pointer* to the object, not the object itself:

```js
// Primitives: a real copy
let a = 5;
let b = a;
b = 10;
console.log(a);   // 5 — unaffected

// Objects: both variables point at the SAME object
const obj1 = { score: 5 };
const obj2 = obj1;            // NOT a copy — a second label on the same box!
obj2.score = 10;
console.log(obj1.score);      // 10 😱

// This also explains array/object comparison:
console.log({ a: 1 } === { a: 1 });   // false — two different boxes with similar contents
console.log(obj1 === obj2);           // true — literally the same box
```

Mental model: the variable holds the object's *address*, like a house address on a sticky note. Copying the note doesn't copy the house.

To make a real (shallow) copy:

```js
const copy = { ...obj1 };          // spread syntax — full details in Lesson 14
const arrCopy = [...myArray];
```

This value-vs-reference distinction is a top-5 source of bugs for beginners *and* professionals. When "changing one thing mysteriously changed another," suspect a shared reference.

---

## ⚠️ Common mistakes

```js
// 1. Reading a property of undefined — THE most common JS error, ever:
const user = { profile: undefined };
user.profile.name;
// ❌ TypeError: Cannot read properties of undefined (reading 'name')
// Translation: user.profile was undefined, and you asked undefined for .name.
// Fix: check each link in the chain. (Lesson 14 shows the ?. shortcut.)

// 2. Dot notation with a variable key
const key = "age";
student.key       // undefined — wanted student[key]

// 3. = instead of : inside object literals
const obj = { name = "Amina" };   // ❌ SyntaxError — inside { } it's name: "Amina"

// 4. Forgetting commas between properties

// 5. Expecting {..} === {..} to compare contents (it compares identity)
```

---

## ✅ Classwork

1. Build an object `me` with your name, age, favorite food, and an array `hobbies` (3 items). Print a sentence using three of its properties, and your second hobby.
2. Add a new property `country` after creation. Change `age`. Print the final object.
3. Given `const key = "food"` — read that property from `me` using brackets. Try it with dot notation and observe the failure.
4. Build the `products` array-of-objects from section 3. Print only the names of in-stock products. Then compute the average price of ALL products.
5. **Predict then run:**
   ```js
   const x = { n: 1 };
   const y = x;
   y.n = 99;
   console.log(x.n);
   ```

## 📝 Homework

1. **Contact card:** an object `contact` with `firstName`, `lastName`, `phone`, and a method `fullName()` that RETURNS first + last combined (use `this`). Print `` `Call ${contact.fullName()} on ${contact.phone}` ``.
2. **Music library:** an array `songs` of 5 objects: `{ title, artist, minutes }`. Write functions:
   - `totalLength(songs)` → total minutes of all songs
   - `songsBy(songs, artist)` → NEW array of that artist's songs (loop + if + push)
   - `longestSong(songs)` → the whole song **object** with the biggest `minutes`
3. **Inventory report:** using the products array — write `stockReport(products)` that returns a string like `"2 of 3 products in stock"` (count with a loop).
4. **Bank account:** object with `owner`, `balance`, and methods `deposit(amount)` and `withdraw(amount)`. Withdraw must refuse (print a message, return early) if `amount > this.balance`. Test a few operations. *This is a mini-preview of backend thinking: data + rules that protect it.*
5. In comments: explain what happened in Classwork #5 in your own words.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **object** | Collection of labeled values: `{ name: "A", age: 15 }` |
| **property** | One key–value pair |
| **key** | The property's name |
| **dot / bracket notation** | `obj.name` / `obj["name"]` — brackets when the key is in a variable |
| **method** | A property whose value is a function |
| **`this`** | Inside a method: the object it was called on |
| **reference** | Variables hold a *pointer* to an object, not a copy |
| **array of objects** | The standard shape of real-world data |
