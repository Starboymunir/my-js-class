# Lesson 01 — What Is Programming? (And Getting Set Up)

> **Goal of this lesson:** Understand what a program is, what JavaScript is, and get your computer ready to write code. By the end you will run your very first JavaScript.

---

## 1. What is a program?

A **program** is a list of instructions that a computer follows, one at a time, in order.

That's it. That's the whole secret.

Think of a cooking recipe:

```
1. Crack two eggs into a bowl.
2. Add a pinch of salt.
3. Whisk for 30 seconds.
4. Pour into a hot pan.
```

A recipe is instructions for a *human*. A program is instructions for a *computer*. The big difference: a human can guess what you mean, but a computer **cannot guess anything**. It does exactly what you wrote — not what you meant.

If step 3 of your recipe said "whisk for 30" a human would assume seconds. A computer would stop and complain: *30 what?* This is why programming feels strict at first. The computer is not smart; it is extremely fast and extremely obedient. **Your job as a programmer is to be precise.**

### Programming = solving problems in small steps

Before writing any code, programmers break a big problem into tiny steps. This is called **thinking algorithmically**, and it's the real skill you're learning. The language (JavaScript) is just how we write the steps down.

**Try it (no computer needed):** Write instructions for making tea for someone who has *never* seen tea, a kettle, or a cup. You'll discover you have to explain things like "open the tap" and "wait until the water bubbles." That's programming thinking.

---

## 2. What is JavaScript?

Websites are built from three languages, each with one job:

| Language | Job | Analogy (a human body) |
|----------|-----|------------------------|
| **HTML** | The content and structure — headings, paragraphs, buttons | The skeleton |
| **CSS** | The appearance — colors, sizes, layout | The skin and clothes |
| **JavaScript** | The behavior — what happens when you click, type, scroll | The muscles and brain |

When you click "Add to Cart" and the cart number changes without the page reloading — that's JavaScript. When a form tells you "password too short" — JavaScript. Google Maps dragging, YouTube's player, chat apps — all JavaScript.

### The important part for your future: JavaScript is not only for browsers

JavaScript runs in two main places:

1. **In the browser** (Chrome, Firefox…) — controlling web pages. This is **frontend**.
2. **On a server** using a program called **Node.js** — reading databases, handling logins, sending data to apps. This is **backend**.

**It is the same language in both places.** The variables, functions, loops, and logic you learn in the first half of this course work *identically* in the browser and on a server. Only the surroundings differ: the browser gives you web-page tools, Node.js gives you server tools.

This is why we will spend a lot of time on fundamentals: everything in Lessons 1–10 is knowledge you will use again, unchanged, when you learn backend.

---

## 3. Setting up your tools

You need two things: a **browser** and a **code editor**.

### Step 1 — Google Chrome

You probably already have it. If not, download from google.com/chrome. (Microsoft Edge works the same way if you prefer it.)

### Step 2 — Visual Studio Code (VS Code)

This is the program where you'll write code. It's free and it's what most professional developers use.

1. Go to **code.visualstudio.com**
2. Download for Windows and install (accept the defaults).
3. Open it. Don't worry about all the buttons — we'll use very little of it at first.

### Step 3 — Install the "Live Server" extension in VS Code

1. In VS Code, click the **Extensions** icon in the left sidebar (looks like 4 squares).
2. Search for **Live Server** (by Ritwick Dey).
3. Click **Install**.

This lets us open our pages in the browser with one click later.

### Step 4 — Make a folder for this course

Create a folder somewhere easy to find, e.g. `Desktop\my-js-practice`. All your practice files will live here. In VS Code: **File → Open Folder** and pick it.

---

## 4. Your first JavaScript — the Console

The fastest place to run JavaScript is already installed on your computer: the **browser console**. It's a hidden panel in Chrome where you can type JavaScript and see the result instantly.

### Opening the console

1. Open Chrome.
2. Press **F12** (or right-click any page → **Inspect**).
3. Click the **Console** tab.

You'll see a blinking cursor next to a `>` symbol. This is a direct line to JavaScript. Type an instruction, press Enter, and it runs immediately.

### Try these (type them, don't copy-paste!)

```js
2 + 3
```
Press Enter. The console answers `5`. You just ran JavaScript — the computer evaluated your expression.

```js
10 * 4
```

```js
"Hello"
```
Text in quotes is called a **string** (more on this next lesson).

```js
alert("Hello, world!")
```
A popup appears! `alert` is a **command** (properly called a *function*) that the browser provides. You *called* it and gave it the text to display.

```js
console.log("I am learning JavaScript")
```
`console.log` prints something to the console. This is the command you will use more than any other — it's how programmers look inside their programs to see what's happening. Remember it well.

### What just happened?

Each line you typed was an **instruction**. JavaScript read it, did it, and showed the result. A real program is just many of these instructions saved in a file, run in order.

---

## 5. Running JavaScript from a file

Typing in the console is great for experiments, but real programs live in files. Let's make your first one.

### Step 1 — Create two files in your practice folder

In VS Code, create a file called `index.html` and paste this in. (This is HTML — you don't need to understand it deeply yet; it's just the page that will *carry* our JavaScript.)

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My First Program</title>
  </head>
  <body>
    <h1>Open the console to see my program!</h1>

    <script src="app.js"></script>
  </body>
</html>
```

The important line is `<script src="app.js"></script>` — it tells the browser: *"load and run the JavaScript in the file app.js."*

Then create a second file called `app.js` with:

```js
console.log("My first program is running!");
console.log(2 + 3);
console.log("JavaScript is fun");
```

### Step 2 — Run it

Right-click `index.html` in VS Code → **Open with Live Server**. Chrome opens. Press **F12** → **Console**. You should see your three lines printed.

Congratulations — you've written and run a program. 🎉

### Note the semicolons

Lines in `app.js` end with `;` — like a full stop at the end of a sentence. JavaScript can often survive without them, but **we will always write them** in this course. It's a good habit and avoids rare, confusing bugs.

---

## 6. Comments — notes inside code

Sometimes you want to write a note in your code that the computer should *ignore*. These are **comments**:

```js
// This is a comment. JavaScript skips this line completely.
console.log("This line runs"); // a comment can also sit after code

/*
  This is a multi-line comment.
  Everything between these markers is ignored.
*/
```

We'll use comments constantly in these notes to explain what lines do. You should use them to leave notes for your future self.

---

## 7. How to learn programming (read this twice)

1. **Type every example yourself.** Copy-pasting teaches your clipboard, not your brain. Typing (including the typos you'll make and fix) is where learning happens.
2. **Predict, then run.** Before pressing Enter, say out loud what you expect to happen. If you were wrong, figure out why — that gap is exactly the thing to learn.
3. **Errors are normal.** Professional programmers see error messages all day, every day. An error is not failure; it's the computer telling you *precisely* what confused it. We'll learn to read them.
4. **Struggling is the workout.** Feeling confused for 10 minutes and then getting it — that's a rep at the gym. If everything feels easy, you're probably only reading and not doing.
5. **Small, daily practice beats long, rare sessions.** 30 minutes daily > 4 hours once a week.

---

## ✅ Classwork (do together in the session)

1. Open the console and calculate: `7 * 8`, `100 / 4`, `2 + 2 * 3` (why is the last one 8 and not 12? — order of operations, like math class).
2. Use `alert()` to greet yourself by name.
3. Use `console.log()` to print your name, your age, and your favorite food — three separate lines.
4. Create the `index.html` + `app.js` files, run with Live Server, and confirm you see console output.
5. Break it on purpose: change `console.log` to `console.lg` and run it. Read the red error message together. What is it telling you?

## 📝 Homework

1. In `app.js`, write a program that prints (using `console.log`):
   - Your full name
   - The year you were born
   - The result of multiplying your age by 365 (roughly how many days you've been alive!)
2. Write one comment above each line explaining what it does.
3. Experiment: what happens if you put a number in quotes, like `console.log("2" + "3")`? Write down what you see — we'll explain it next lesson. (Spoiler: it's weird, and the reason why teaches you something important.)

## 🔑 Key words from this lesson

| Word | Meaning |
|------|---------|
| **program** | A list of instructions the computer follows in order |
| **JavaScript** | The programming language of the web (browser *and* server) |
| **frontend** | Code that runs in the user's browser |
| **backend** | Code that runs on a server (you'll learn this later — same language!) |
| **console** | A panel in the browser where you can run JS and see printed output |
| **`console.log()`** | Prints a value to the console |
| **comment** | A note in code that the computer ignores (`//`) |
