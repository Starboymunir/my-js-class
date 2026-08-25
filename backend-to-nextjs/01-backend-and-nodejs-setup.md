# Lesson 01 — What Is a Backend? (And Setting Up Node.js)

> **Goal:** Understand what happens on "the other side" of `fetch`, install Node.js, and run JavaScript *outside the browser* for the first time. You already know the language — today you meet its second home.

---

## 1. Where you're standing

In Part 1, Lesson 16, you wrote this:

```js
const response = await fetch("https://jsonplaceholder.typicode.com/users");
const users = await response.json();
```

You sent a request across the internet and some computer answered with JSON. **That computer was running a backend.** Somebody wrote code that said: *"when a GET request arrives for `/users`, look up the users, turn them into JSON, send them back."*

Starting today, **you are that somebody.**

```
   ┌──────────────┐   HTTP request    ┌──────────────┐   query    ┌──────────┐
   │   Browser    │ ────────────────▶ │    Server    │ ─────────▶ │ Database │
   │  (frontend)  │ ◀──────────────── │  (backend)   │ ◀───────── │          │
   └──────────────┘   HTTP response   └──────────────┘   rows     └──────────┘
      your Part 1 code                 your Part 2 code            Lesson 6+
```

Draw this diagram from memory at the start of every session until it's boring. Nearly every backend confusion is really a "which box am I in?" confusion.

### What does a backend actually do?

| Job | Example |
|-----|---------|
| **Store data permanently** | Users, posts, orders — in a database, not in a variable that dies on refresh |
| **Enforce rules** | "Only the owner can delete this post." "Password must be 8+ characters." (Remember Part 1 L13: *never trust the client* — this is where the real checks live) |
| **Keep secrets** | API keys, database passwords — things that must never be sent to a browser |
| **Talk to other services** | Send emails, charge cards, call other APIs |
| **Serve the frontend** | Send the HTML/JS/CSS files that *are* the frontend |

### Why the browser can't do these

The browser is a sandbox on a stranger's computer. It can't be trusted (anyone can open DevTools and change your code), it can't keep secrets (all its code is visible), and it disappears when the tab closes. A backend runs on a machine *you* control, 24/7.

---

## 2. What is Node.js?

For JavaScript's first 14 years it only ran inside browsers. In 2009, Node.js took the JavaScript engine out of Chrome (V8) and wrapped it in a program that runs on a normal computer — with tools browsers never had: reading files, opening network ports, talking to databases.

**Same language. Different toolbox.**

| | Browser | Node.js |
|---|---|---|
| Syntax, `let`/`const`, functions, arrays, objects, promises, `async/await`, modules | ✅ identical | ✅ identical |
| `document`, `window`, `alert`, DOM, `localStorage` | ✅ | ❌ no web page here! |
| File system, network servers, OS access, `process` | ❌ | ✅ |
| `console.log` | prints to DevTools console | prints to the terminal |
| `fetch` | ✅ | ✅ (since Node 18) |

Everything from Part 1 Lessons 1–10 and 14–15 works unchanged. That was the plan.

*(Alternatives exist — Deno, Bun — but Node is the standard, and Next.js runs on it.)*

---

## 3. Install Node.js

1. Go to **nodejs.org** and download the **LTS** version (Long-Term Support — the stable one). Install with defaults.
2. Open a terminal. In VS Code: **Terminal → New Terminal** (or `` Ctrl+` ``). On Windows this is PowerShell — fine.
3. Verify:

```bash
node -v       # v22.x.x or newer
npm -v        # 10.x.x or newer
```

If you see version numbers, you're done. If "not recognized", restart VS Code (it needs to pick up the new PATH).

### 15 minutes of terminal survival

You'll now live in the terminal. Essentials:

```bash
pwd                  # where am I?
ls                   # list files here (dir also works in PowerShell)
cd my-folder         # go into a folder
cd ..                # go up one level
mkdir my-api         # make a folder
code .               # open the current folder in VS Code
node app.js          # run a JavaScript file with Node
Ctrl + C             # STOP a running program (you'll use this constantly for servers)
```

Tab completes names. Up-arrow recalls previous commands. That's 90% of it.

---

## 4. Your first Node program

Make a folder `node-practice`, open it in VS Code, create `hello.js`:

```js
console.log("Hello from Node.js!");
console.log("2 + 3 =", 2 + 3);

const user = { name: "Amina", role: "backend developer (in training)" };
console.log(user);
```

Run it in the terminal:

```bash
node hello.js
```

Output appears in the terminal — no browser, no HTML file, no DevTools. **That's the whole shift.** Your code is now a program on a computer, not a script inside a web page.

Now try:

```js
console.log(window);      // ReferenceError: window is not defined
```

There's no window. No document. This is not a browser. Delete that line and try instead:

```js
console.log(process.platform);        // "win32" — the OS
console.log(process.version);         // Node version
console.log(process.argv);            // the command-line arguments — try: node hello.js foo bar
```

`process` is a global object Node gives you instead of `window` — information about, and control over, the running program.

### The REPL

Type `node` alone in the terminal and you get an interactive prompt — Node's version of the browser console. Great for quick experiments. Exit with `.exit` or Ctrl+C twice.

---

## 5. npm and `package.json` — every project's ID card

**npm** (Node Package Manager) does two things:

1. Installs **packages** — code other people wrote (Express, Prisma, React… over 2 million of them).
2. Runs project **scripts**.

Every Node project starts with a `package.json` file. Create one:

```bash
npm init -y          # -y = accept defaults
```

Open it — then make the two edits you will make in **every** project in this course:

```json
{
  "name": "node-practice",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node app.js",
    "dev": "node --watch app.js"
  }
}
```

- **`"type": "module"`** — tells Node to use the modern `import`/`export` syntax you learned in Part 1 L14. Without it, Node expects the older `require()` style (see below).
- **scripts** — named commands. `npm start` runs `node app.js`; `npm run dev` runs it in **watch mode**, which restarts automatically every time you save. You'll use `npm run dev` for the rest of your career.

Try it: rename `hello.js` to `app.js`, run `npm run dev`, then edit and save the file — it re-runs by itself. Stop with Ctrl+C.

### Two module systems — you must recognize both

```js
// ESM (modern) — what we write. Needs "type": "module".
import { readFile } from "node:fs/promises";
export function helper() {}

// CommonJS (older) — what you'll see in countless tutorials and older packages.
const { readFile } = require("fs/promises");
module.exports = { helper };
```

Same idea, different spelling. When a tutorial uses `require`, translate in your head. If you ever see the error *"Cannot use import statement outside a module"* — you forgot `"type": "module"`.

---

## 6. Installing a package (a taste)

```bash
npm install chalk
```

Watch what happens:

- `package.json` gains a `"dependencies": { "chalk": "^5.x" }` entry — the list of what this project needs.
- A `node_modules/` folder appears — the actual downloaded code (can be huge; **never** edit it, never commit it to git).
- `package-lock.json` appears — exact versions, so everyone gets identical installs.

Use it:

```js
import chalk from "chalk";
console.log(chalk.green("Success!"), chalk.red.bold("Error!"));
```

Because `node_modules` is never committed, anyone (including future-you on a new laptop) restores it with one command: `npm install` reads `package.json` and downloads everything. That's why `package.json` is the project's ID card.

### `.gitignore` — do this now, in every project

Create a file named `.gitignore`:

```
node_modules/
.env
```

`node_modules` is huge and reproducible; `.env` (Lesson 2) will hold secrets. Neither belongs on GitHub. Forgetting this is the most common beginner git mistake — and leaking `.env` is the most *expensive* one.

---

## ⚠️ Common mistakes

```
1. "node is not recognized"          → Node not installed / terminal opened before install. Restart VS Code.
2. "Cannot find module './app.js'"   → you're in the wrong folder. pwd / ls to check.
3. "Cannot use import statement..."  → add "type": "module" to package.json
4. "window is not defined"           → you're in Node. There is no browser here.
5. Committing node_modules           → add .gitignore BEFORE the first commit
6. Program "hangs"                    → servers run forever by design. Ctrl+C stops them.
```

---

## ✅ Classwork

1. Install Node, verify versions, and run `hello.js`. Then run it with extra arguments (`node hello.js 5 10`) and print `process.argv`. Where do your arguments appear in the array?
2. Set up `package.json` with `"type": "module"` and `dev`/`start` scripts. Run in watch mode; edit the file three times and watch it re-run.
3. Terminal drill: from your Desktop, using only the terminal, create a folder `terminal-test`, go into it, create a file with `echo "hi" > note.txt`, list the folder, go back up, and delete the folder (`rm -r terminal-test` / PowerShell `Remove-Item -Recurse terminal-test`).
4. Install `chalk` and print a colored banner. Open `node_modules` and look — how many folders did one package bring? (Why? Packages depend on packages.)
5. Redraw the browser → server → database diagram from memory, and label where Part 1's `fetch` code lives and where today's code lives.

## 📝 Homework

1. **Argument calculator:** `node calc.js 5 + 3` prints `8`. Read `process.argv`, convert the numbers (they're strings — same lesson as `input.value`!), support `+ - * /`, and print a friendly error for bad input. Use functions from Part 1.
2. **Reuse:** copy your Part 1 "report card" or "average" functions into a Node file and run them with Node. Note in a comment: what needed to change? (Answer should be: nothing, except removing any `document`/`alert` usage.)
3. **Git:** initialize a git repo in `node-practice` with a proper `.gitignore`, make your first commit, and push it to a new GitHub repo. From now on, every project gets committed as you go.
4. Written (comments): three things a backend must do that a browser cannot, and why.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **backend / server** | Code on a machine you control that answers requests, stores data, enforces rules |
| **Node.js** | JavaScript runtime outside the browser |
| **runtime** | The program that executes your code (browser engine, Node) |
| **terminal** | Text interface for running programs — your new home |
| **npm** | Package manager: installs packages, runs scripts |
| **`package.json`** | Project manifest: name, dependencies, scripts, `"type": "module"` |
| **`node_modules`** | Installed packages. Never edit, never commit |
| **ESM vs CommonJS** | `import/export` vs `require/module.exports` |
| **`process`** | Node's global: args, env, platform, exit |
