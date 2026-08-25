# Lesson 06 — Databases and SQL

> **Goal:** Replace the JSON file with a real database. Learn what databases are for, the SQL language to talk to them, how to use SQLite from Node, and the security rule that protects every database on earth: **parameterized queries**. Next lesson an ORM (Prisma) will write most of the SQL for you — but you must be able to read what it writes.

---

## 1. Why a JSON file isn't enough

Our `tasks.json` store works — until:

- **Two requests write at the same time.** Request A loads, request B loads, A saves, B saves → A's change is gone. With 1 user this never happens; with 100 it happens constantly.
- **The file grows.** Finding one task among 1,000,000 means loading and scanning all of them. Every request.
- **Data relates.** Users have tasks; tasks belong to projects. Nesting JSON gets messy fast.
- **You need guarantees.** "Deduct money from A *and* add to B — both or neither." Files can't promise that.

A **database** is a program built for exactly these problems: many simultaneous users, fast lookup via indexes, relationships, and transactions (all-or-nothing operations). Your server talks to it; it does the hard part.

### Two families

| | Relational (SQL) | Document (NoSQL) |
|---|---|---|
| Examples | **PostgreSQL**, MySQL, **SQLite** | MongoDB, Firebase |
| Data shape | Tables with rows and columns, strict schema | JSON-like documents, flexible |
| Relationships | First-class (foreign keys, joins) | Manual |
| Best when | Data is structured and related (almost always) | Rapid prototypes, very flexible shapes |

We learn **SQL** — it's the default for most backends, it's what Prisma and Next.js projects typically use, and the skills transfer to every relational database. We'll practice with **SQLite** (a full SQL database that lives in a single file, zero setup) and deploy with **PostgreSQL** (the production standard) in Lesson 9. Same SQL, different engine.

---

## 2. Thinking in tables

A relational database stores data in **tables**. Each table has typed **columns**; each record is a **row**:

```
tasks
┌────┬──────────────────┬───────┬─────────┬──────────────────────┐
│ id │ text             │ done  │ user_id │ created_at           │
├────┼──────────────────┼───────┼─────────┼──────────────────────┤
│ 1  │ Learn SQL        │ 0     │ 1       │ 2026-08-01T10:00:00Z │
│ 2  │ Build the API    │ 1     │ 1       │ 2026-08-01T10:05:00Z │
│ 3  │ Buy milk         │ 0     │ 2       │ 2026-08-02T08:00:00Z │
└────┴──────────────────┴───────┴─────────┴──────────────────────┘

users
┌────┬──────────────────┬────────────────┐
│ id │ email            │ password_hash  │
├────┼──────────────────┼────────────────┤
│ 1  │ amina@mail.com   │ $2b$10$...     │
│ 2  │ bola@mail.com    │ $2b$10$...     │
└────┴──────────────────┴────────────────┘
```

- **Primary key** — the `id` column: unique, never reused, auto-generated.
- **Foreign key** — `tasks.user_id` points at `users.id`: *this task belongs to that user*. This is how relationships work: not by nesting, but by reference. (Objects-by-reference from Part 1 L8, in table form.)
- **Schema** — the definition of tables, columns, and types. Fixed up front; changed via **migrations** (next lesson).

An array of objects ⟷ a table: each object is a row; each property is a column. Your brain already knows this shape.

---

## 3. SQL — the language

**SQL** (Structured Query Language) is how you ask a relational database for things. It reads almost like English. The core statements — CRUD again:

```sql
-- CREATE TABLE: define the schema
CREATE TABLE tasks (
  id         INTEGER PRIMARY KEY AUTOINCREMENT,
  text       TEXT NOT NULL,
  done       INTEGER NOT NULL DEFAULT 0,        -- SQLite has no boolean; 0/1
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- INSERT: create a row
INSERT INTO tasks (text) VALUES ('Learn SQL');

-- SELECT: read rows
SELECT * FROM tasks;                             -- every column, every row
SELECT id, text FROM tasks WHERE done = 0;       -- some columns, filtered
SELECT * FROM tasks WHERE text LIKE '%SQL%';     -- pattern match (% = anything)
SELECT * FROM tasks ORDER BY created_at DESC LIMIT 10;   -- newest 10
SELECT COUNT(*) FROM tasks WHERE done = 1;       -- aggregate

-- UPDATE: change rows — ALWAYS with WHERE
UPDATE tasks SET done = 1 WHERE id = 2;

-- DELETE: remove rows — ALWAYS with WHERE
DELETE FROM tasks WHERE id = 3;
```

⚠️ `UPDATE tasks SET done = 1;` with no `WHERE` marks **every** task done. `DELETE FROM tasks;` empties the table. Every developer does this once. Say "WHERE" out loud before pressing Enter.

### Relationships: JOIN

```sql
-- tasks with their owner's email — combine two tables on the foreign key
SELECT tasks.id, tasks.text, users.email
FROM tasks
JOIN users ON tasks.user_id = users.id
WHERE users.id = 1;
```

`JOIN … ON` says "line up rows where these columns match." It's `find()` across two arrays, done by the database, fast.

### Map it to what you know

| SQL | JavaScript equivalent |
|-----|----------------------|
| `SELECT * FROM tasks WHERE done = 0` | `tasks.filter(t => !t.done)` |
| `SELECT text FROM tasks` | `tasks.map(t => t.text)` |
| `SELECT * FROM tasks WHERE id = 2` | `tasks.find(t => t.id === 2)` |
| `ORDER BY created_at DESC` | `.sort((a, b) => ...)` |
| `COUNT(*)` | `.length` |
| `JOIN` | nested `find()` / `map()` across two arrays |

The difference: SQL runs *inside the database*, which has indexes and doesn't load everything into your server's memory.

---

## 4. SQLite from Node

```bash
npm install better-sqlite3
```

*(Node 22+ also ships an experimental built-in `node:sqlite` module; `better-sqlite3` is the mature choice. If the install fails on Windows, it needs build tools — ask your teacher, or use `node:sqlite` with the `--experimental-sqlite` flag.)*

```js
// db/index.js
import Database from "better-sqlite3";

export const db = new Database("data/tasks.db");     // creates the file if missing

// run once at startup: create the schema if it doesn't exist
db.exec(`
  CREATE TABLE IF NOT EXISTS tasks (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    text       TEXT NOT NULL,
    done       INTEGER NOT NULL DEFAULT 0,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
  )
`);
```

`better-sqlite3` is synchronous (SQLite is local and fast, so it's fine) — no `await`. Postgres drivers next lesson are async.

### Parameterized queries — the rule that prevents SQL injection ⭐⭐

Suppose you build a query by gluing strings:

```js
// ❌ NEVER DO THIS
const rows = db.prepare(`SELECT * FROM tasks WHERE text = '${req.query.search}'`).all();
```

A user sends `search = anything' OR '1'='1` and the query becomes `WHERE text = 'anything' OR '1'='1'` — returns everything. Send `'; DROP TABLE tasks; --` and your table is gone. This is **SQL injection**, the most famous attack on the web, and it's entirely caused by string concatenation.

The fix — **always** pass values as parameters; the driver keeps them separate from the SQL:

```js
// ✅ placeholders: ? (or :name)
const rows = db.prepare("SELECT * FROM tasks WHERE text LIKE ?").all(`%${search}%`);
const task = db.prepare("SELECT * FROM tasks WHERE id = ?").get(id);
```

The value is *data*, never *code*, no matter what it contains. ORMs do this automatically — one big reason to use them — but you must know why.

### The service layer with SQL

```js
// services/tasks.service.js
import { db } from "../db/index.js";

const toTask = (row) => row && { ...row, done: row.done === 1 };   // 0/1 → boolean for JSON

export function listTasks() {
  return db.prepare("SELECT * FROM tasks ORDER BY created_at DESC").all().map(toTask);
}

export function getTask(id) {
  return toTask(db.prepare("SELECT * FROM tasks WHERE id = ?").get(id));
}

export function createTask(text) {
  const result = db.prepare("INSERT INTO tasks (text) VALUES (?)").run(text);
  return getTask(result.lastInsertRowid);
}

export function updateTask(id, { text, done }) {
  const existing = getTask(id);
  if (!existing) return null;
  db.prepare("UPDATE tasks SET text = ?, done = ? WHERE id = ?")
    .run(text ?? existing.text, done === undefined ? (existing.done ? 1 : 0) : (done ? 1 : 0), id);
  return getTask(id);
}

export function deleteTask(id) {
  return db.prepare("DELETE FROM tasks WHERE id = ?").run(id).changes > 0;   // true if a row was deleted
}
```

Controllers now call these instead of `load()`/`save()` — and become tiny:

```js
export async function getOne(req, res) {
  const task = getTask(Number(req.params.id));
  if (!task) throw new HttpError(404, "Task not found");
  res.json(task);
}
```

**This is why we built the layers in Lesson 5:** the storage changed completely and the routes/controllers barely noticed. Next lesson it changes again (Prisma), and again the controllers won't care.

---

## 5. Looking inside the database

Install the VS Code extension **SQLite Viewer** (or the **DB Browser for SQLite** app) to open `tasks.db` and see the tables as spreadsheets. Run your API, add a task, refresh the viewer — there's the row. Seeing the data makes SQL real.

Try queries directly too: many viewers have a query tab. Type `SELECT COUNT(*) FROM tasks WHERE done = 1;` and see the number.

---

## ⚠️ Common mistakes

```
1. UPDATE / DELETE without WHERE           → whole table changed. Say WHERE out loud.
2. String-building SQL with user input     → SQL injection. Parameters, always.
3. Forgetting SQLite booleans are 0/1      → JSON shows done: 1, frontend compares === true and fails. Convert.
4. .get() vs .all()                        → get = one row (or undefined); all = array. Mixing them up → "rows.map is not a function".
5. Committing the .db file                 → add data/*.db to .gitignore (it's data, not code).
6. Schema change after table exists        → CREATE TABLE IF NOT EXISTS won't alter it. Delete the .db in dev, or use migrations (next lesson).
```

---

## ✅ Classwork

1. In a SQLite viewer's query tab, create the `tasks` table by hand and insert 5 rows with SQL. Then run: all undone tasks; tasks containing "a"; the count of done tasks; the newest 2.
2. Write the `UPDATE` to mark task 3 done and the `DELETE` for task 5 — with WHERE. Then, on a *throwaway* table, run `DELETE FROM throwaway;` and watch it all vanish. Lesson learned safely.
3. Wire `better-sqlite3` into the API: `db/index.js`, `services/tasks.service.js`, controllers calling the service. Full Thunder Client pass. Open the `.db` file and confirm rows.
4. **Injection lab:** temporarily write ONE route with string-concatenated SQL. From Thunder Client, send `?search=x' OR '1'='1`. Observe. Then fix it with `?` and send the same payload — it now searches for that literal weird string. Delete the bad version.

## 📝 Homework

1. **Users table:** create `users (id, email UNIQUE NOT NULL, name)` and add `user_id INTEGER REFERENCES users(id)` to tasks. Seed two users. Write `GET /api/users/:id/tasks` using a parameterized query, and a `JOIN` query that returns tasks with their owner's email at `GET /api/tasks?withOwner=true`.
2. **Search + filter + sort in SQL:** `GET /api/tasks?search=milk&done=false&sort=oldest` — build the query with `WHERE ... LIKE ? AND done = ?` and a whitelisted `ORDER BY` (never put the sort string directly into SQL — map `"oldest"` → `"created_at ASC"` via an object lookup).
3. **Transactions (research):** read the `better-sqlite3` docs on `db.transaction()`. Write `POST /api/tasks/bulk` that inserts an array of tasks inside a transaction — if any one is invalid, none are saved.
4. Written: explain SQL injection to a non-programmer, and why parameterized queries stop it. Draw the tasks/users tables and the foreign key arrow.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **relational database** | Tables, rows, columns, relationships (Postgres, SQLite, MySQL) |
| **SQL** | The query language: SELECT, INSERT, UPDATE, DELETE, JOIN |
| **schema** | The defined structure of tables and columns |
| **primary key** | Unique id for a row |
| **foreign key** | A column pointing at another table's primary key |
| **JOIN** | Combine rows from two tables on a matching column |
| **SQLite** | A full SQL database in one file — perfect for learning and dev |
| **PostgreSQL** | The production-standard SQL database |
| **SQL injection** | Attack via user input glued into SQL |
| **parameterized query** | `?` placeholders — values can never become code |
| **transaction** | All-or-nothing group of operations |
