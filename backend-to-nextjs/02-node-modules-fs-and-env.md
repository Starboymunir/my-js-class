# Lesson 02 — Node's Toolbox: Modules, Files, and Environment Variables

> **Goal:** Use Node's built-in modules to read and write files, store data as JSON on disk, and handle configuration and secrets properly. By the end you'll build a command-line to-do app with real persistence — the first version of the Tasks project that threads through this whole course.

---

## 1. Built-in modules

Node ships with a standard library — modules you import without installing. The important ones:

| Module | Gives you |
|--------|-----------|
| `node:fs` | File system: read, write, delete files and folders |
| `node:path` | Build file paths safely across operating systems |
| `node:http` | Create web servers (Lesson 3) |
| `node:os` | Info about the machine |
| `node:crypto` | Hashing, random IDs, encryption |
| `node:test` | A built-in test runner |

The `node:` prefix marks them as built-ins (older code omits it — `import fs from "fs"` also works).

```js
import os from "node:os";
import { randomUUID } from "node:crypto";

console.log(os.hostname(), os.cpus().length, "cores");
console.log(randomUUID());     // "3f2a...-..." — a unique ID. You'll use this for records.
```

---

## 2. Reading and writing files

Files are slow compared to memory, so file operations are **asynchronous** — and you already know how to handle that: `await`.

```js
import { readFile, writeFile } from "node:fs/promises";

// WRITE — creates or overwrites the file
await writeFile("notes.txt", "First line\nSecond line");

// READ — "utf8" says "give me text", not raw bytes
const text = await readFile("notes.txt", "utf8");
console.log(text);
```

Yes — top-level `await` works in ESM modules (`"type": "module"`), no wrapper function needed.

More operations:

```js
import { appendFile, mkdir, readdir, unlink, access } from "node:fs/promises";

await appendFile("notes.txt", "\nThird line");       // add to the end
await mkdir("data", { recursive: true });            // create folder (no error if it exists)
const files = await readdir(".");                    // list folder contents → array of names
await unlink("old.txt");                             // delete a file

// Does a file exist? access() throws if not — so wrap it:
async function exists(path) {
  try {
    await access(path);
    return true;
  } catch {
    return false;
  }
}
```

That `try/catch` is Part 1 Lesson 10 doing exactly what it was designed for: reality (missing file) is not a bug, so we catch it and return a sensible value.

### Paths — use `node:path`

Windows uses backslashes, Mac/Linux use forward slashes. Never glue paths with string concatenation; let `path` do it:

```js
import path from "node:path";

const filePath = path.join("data", "tasks.json");   // correct separator on every OS
```

One subtlety: relative paths like `"data/tasks.json"` are relative to **where you ran `node` from**, not where the file lives. To anchor paths to the current file's folder:

```js
const dataFile = new URL("./data/tasks.json", import.meta.url);
await readFile(dataFile, "utf8");        // works no matter what folder you launched from
```

(`import.meta.url` = this file's location. Good enough to know it exists; you'll see `__dirname` in CommonJS code for the same purpose.)

---

## 3. JSON files as a baby database

Combine files + JSON (Part 1 L14) and you have persistent storage:

```js
import { readFile, writeFile } from "node:fs/promises";

const FILE = "tasks.json";

async function loadTasks() {
  try {
    const text = await readFile(FILE, "utf8");
    return JSON.parse(text);
  } catch {
    return [];                        // first run: no file yet → start empty (the || [] idiom, async edition)
  }
}

async function saveTasks(tasks) {
  await writeFile(FILE, JSON.stringify(tasks, null, 2));   // pretty-printed for humans
}

// Use them
const tasks = await loadTasks();
tasks.push({ id: 1, text: "Learn Node", done: false });
await saveTasks(tasks);
```

Run it twice — the second run *loads* the first run's task. This is exactly `localStorage` from Part 1 L13, except the data lives in a real file on a real disk. Real databases (Lesson 6) are this idea with superpowers: fast search, many users at once, no corruption when two writes collide.

---

## 4. Environment variables and secrets ⭐

Programs need configuration that changes between machines — and secrets that must never be in code:

- Which port to listen on (3000 on your laptop, something else in production)
- Database connection string (contains a password!)
- API keys for external services

Hard-coding these is wrong twice: it leaks secrets to GitHub, and it makes the same code unable to run in two places. The universal solution: **environment variables** — key/value settings the operating system hands to a program at startup. Node exposes them as `process.env`:

```js
console.log(process.env.PATH);            // one every machine has
console.log(process.env.PORT);            // undefined unless you set it
```

### The `.env` file

Typing variables into the OS is clumsy, so the convention is a `.env` file in the project root:

```
PORT=3000
DATABASE_URL=file:./dev.db
API_KEY=sk_live_abc123secret
```

Node loads it with the `--env-file` flag (Node 20.6+):

```json
"scripts": {
  "dev": "node --watch --env-file=.env app.js"
}
```

```js
const port = process.env.PORT ?? 3000;      // ?? for a default — Part 1 L14!
const apiKey = process.env.API_KEY;
```

(Older tutorials use the `dotenv` package: `npm install dotenv` + `import "dotenv/config"`. Same result.)

### The rules — memorize them, they're non-negotiable

1. **`.env` is in `.gitignore`.** Always. Check before every first commit.
2. Commit a **`.env.example`** with the *names* but fake values, so teammates know what to set.
3. **Never** send `process.env` values to the browser. They stay on the server.
4. Every value from `process.env` is a **string** (`"3000"`) — `Number()` it if you need a number. Sound familiar?

When you deploy (Lessons 9 and 17), the hosting service has a settings page where you enter these same variables — that's how the same code runs with different config in production.

---

## 5. Project: the Tasks CLI 📋 (v1 of our running project)

A command-line to-do app with JSON persistence. Build it together; it uses everything so far.

```
node tasks.js add "Learn Node"
node tasks.js list
node tasks.js done 2
node tasks.js delete 1
```

```js
// tasks.js
import { readFile, writeFile } from "node:fs/promises";

const FILE = "tasks.json";

async function load() {
  try {
    return JSON.parse(await readFile(FILE, "utf8"));
  } catch {
    return [];
  }
}

async function save(tasks) {
  await writeFile(FILE, JSON.stringify(tasks, null, 2));
}

function nextId(tasks) {
  return tasks.length === 0 ? 1 : Math.max(...tasks.map((t) => t.id)) + 1;
}

// ---- commands: each one loads → changes → saves (the state → render rhythm, server edition)
async function add(text) {
  if (!text) return console.log("Usage: add <text>");
  const tasks = await load();
  const task = { id: nextId(tasks), text, done: false, createdAt: new Date().toISOString() };
  tasks.push(task);
  await save(tasks);
  console.log(`Added #${task.id}: ${task.text}`);
}

async function list() {
  const tasks = await load();
  if (tasks.length === 0) return console.log("No tasks yet.");
  for (const t of tasks) {
    console.log(`${t.done ? "[x]" : "[ ]"} #${t.id} ${t.text}`);
  }
}

async function markDone(id) {
  const tasks = await load();
  const task = tasks.find((t) => t.id === id);
  if (!task) return console.log(`No task with id ${id}`);
  task.done = true;
  await save(tasks);
  console.log(`Completed #${id}`);
}

async function remove(id) {
  const tasks = await load();
  const remaining = tasks.filter((t) => t.id !== id);
  if (remaining.length === tasks.length) return console.log(`No task with id ${id}`);
  await save(remaining);
  console.log(`Deleted #${id}`);
}

// ---- dispatch on the command-line arguments
const [command, ...rest] = process.argv.slice(2);     // destructuring + rest (Part 1 L14)

switch (command) {
  case "add":    await add(rest.join(" ")); break;
  case "list":   await list(); break;
  case "done":   await markDone(Number(rest[0])); break;
  case "delete": await remove(Number(rest[0])); break;
  default:       console.log("Commands: add <text> | list | done <id> | delete <id>");
}
```

Look at the four command functions: **create, read, update, delete.** Remember those four words. In Lesson 5 they become HTTP endpoints; in Lesson 6 they become SQL statements; in Lesson 16 they become Next.js server actions. Same four verbs, all the way down. This is called **CRUD**, and most backend work is CRUD wearing different clothes.

---

## ⚠️ Common mistakes

```js
// 1. Forgetting "utf8" → you get a Buffer (raw bytes), not a string
const data = await readFile("x.txt");          // <Buffer 48 65 ...>

// 2. Forgetting await → reads/writes race, results are empty or stale

// 3. JSON.parse on an empty/missing file → SyntaxError. Wrap in try/catch, default to [].

// 4. process.env.PORT is "3000" (string) — comparisons/math need Number()

// 5. .env committed to GitHub. Rotate the secret immediately; git history remembers forever.

// 6. Path relative to the wrong folder → ENOENT (no such file). Check with pwd; anchor with import.meta.url.
```

---

## ✅ Classwork

1. Write a program that creates `log.txt`, appends a timestamped line each run (`new Date().toISOString()`), then prints the whole file. Run it 3 times.
2. Write `countWords(filename)` that reads a text file and returns the word count. Handle a missing file with a friendly message (try/catch).
3. Build the Tasks CLI together, command by command, testing each in the terminal as you go. Open `tasks.json` in VS Code and watch it change.
4. Add a `.env` with `TASKS_FILE=data/tasks.json` and make the CLI use it (with a default of `tasks.json` if unset). Create `.env.example` and confirm `.env` is git-ignored (`git status` must not show it).

## 📝 Homework

1. **Extend the Tasks CLI:** `clear-done` (remove all completed), `edit <id> <new text>`, and `stats` (X of Y done, using `filter`). Refuse `add` with an empty text and `done` with a non-numeric id (validation!).
2. **Notes app:** `node notes.js new "title" "body"` stores notes as individual files in a `notes/` folder (filename = a slug of the title: lowercase, spaces → dashes). `node notes.js list` lists titles; `node notes.js read <slug>` prints one. Use `path.join`, `mkdir` with `recursive`, and `readdir`.
3. **Config check:** write `config.js` that exports an object built from `process.env` (`port` as a Number with default 3000, `appName` with a default) and **throws** a clear error at startup if `API_KEY` is missing. Import it from `app.js`. Failing fast on missing config is a professional habit.
4. Written: why must `.env` never be committed? What is CRUD, and where have you already done it in Part 1 (hint: to-do list)?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **built-in module** | Node's standard library: `node:fs`, `node:path`, `node:http`… |
| **`fs/promises`** | File operations that return promises → `await` them |
| **`utf8`** | "Give me text" when reading a file |
| **`path.join`** | OS-safe path building |
| **persistence** | Data that survives the program stopping (files, databases) |
| **environment variable** | Config handed to the program from outside; `process.env.NAME` |
| **`.env` / `.env.example`** | Local secrets (git-ignored) / shareable template |
| **CRUD** | Create, Read, Update, Delete — the four eternal operations |
