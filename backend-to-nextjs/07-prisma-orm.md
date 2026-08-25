# Lesson 07 — Prisma: Talking to the Database in JavaScript

> **Goal:** Use Prisma, an ORM, to define your database schema in one file, generate migrations, and query with JavaScript objects instead of SQL strings. Prisma is the most common database tool in the Next.js world — everything here carries straight into Phase C.

---

## 1. What an ORM is (and isn't)

An **ORM** (Object-Relational Mapper) translates between your JavaScript objects and database tables:

```js
// SQL (Lesson 6)
db.prepare("SELECT * FROM tasks WHERE done = ? ORDER BY created_at DESC").all(0);

// Prisma
await prisma.task.findMany({ where: { done: false }, orderBy: { createdAt: "desc" } });
```

What you gain:

- **Type-safe queries** — typos in column names are caught before running (and with TypeScript, in the editor).
- **Automatic parameterization** — SQL injection is handled for you.
- **One schema file** that generates both the database tables *and* your query API.
- **Migrations** — versioned, repeatable schema changes.
- **Database portability** — SQLite today, Postgres at deploy time, same code.

What you don't gain: an excuse to skip Lesson 6. When a Prisma query is slow or does something surprising, you'll read the SQL it generated. ORMs are a convenience layer over SQL, not a replacement for understanding it.

*(Alternatives you'll hear of: Drizzle, Kysely, TypeORM. Prisma is the most beginner-friendly and the most common in Next.js tutorials. The concepts transfer.)*

---

## 2. Setup

Inside `tasks-api`:

```bash
npm install prisma --save-dev        # the CLI (dev tool)
npm install @prisma/client           # the runtime library your code imports
npx prisma init --datasource-provider sqlite
```

*(`npx` runs a package's command-line tool. `--save-dev` marks a dev-only dependency.)*

This creates `prisma/schema.prisma` and adds `DATABASE_URL="file:./dev.db"` to `.env`. Install the **Prisma** VS Code extension for syntax highlighting.

> **Version note:** these commands match Prisma 6. Prisma 7 (late 2025) changed some setup details — a `prisma.config.ts` file, a different `generator` block, and a driver adapter for SQLite. If `npx prisma init` produces a different layout than shown here, follow the generated comments and the "Quickstart" on prisma.io — the *querying* API below is the same.

---

## 3. The schema — one file to rule them all

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  password  String
  createdAt DateTime @default(now())
  tasks     Task[]                        // relation: a user HAS MANY tasks
}

model Task {
  id        Int      @id @default(autoincrement())
  text      String
  done      Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    Int                           // the foreign key column
}
```

Reading it:

- `model` = table. Fields = columns with types (`Int`, `String`, `Boolean`, `DateTime`). `?` = optional (nullable).
- `@id @default(autoincrement())` = primary key. `@unique` = no duplicates. `@default(now())` and `@updatedAt` = timestamps handled for you.
- The **relation**: `Task.userId` is the foreign key (a real column). `Task.user` and `User.tasks` are *virtual* — Prisma's way of letting you walk the relationship in code. `onDelete: Cascade` = deleting a user deletes their tasks.

Compare with Lesson 6's `CREATE TABLE`: same information, and Prisma writes the SQL. Prisma also maps `Boolean` to SQLite's 0/1 and back — the `toTask` conversion is gone.

---

## 4. Migrations — versioned schema changes

```bash
npx prisma migrate dev --name init
```

This does three things:

1. Creates `prisma/migrations/2026..._init/migration.sql` — the actual SQL (open it — it's Lesson 6's `CREATE TABLE`!).
2. Applies it to your database.
3. Regenerates the Prisma Client so `prisma.task` knows the new shape.

Every time you change the schema, run `migrate dev --name something-descriptive` again. The migrations folder becomes a **history** of your database's structure, committed to git, replayable on any machine — including the production server (`npx prisma migrate deploy`, Lesson 9). This solves Lesson 6's "delete the .db when the schema changes" hack.

Explore your data visually:

```bash
npx prisma studio         # opens a browser GUI for your tables
```

---

## 5. Prisma Client — querying

```js
// db/prisma.js — one client for the whole app
import { PrismaClient } from "@prisma/client";
export const prisma = new PrismaClient();
```

The client is *generated from your schema*: `prisma.user` and `prisma.task` exist because the models do. Every method returns a promise → `await`.

### Create

```js
const user = await prisma.user.create({
  data: { email: "amina@mail.com", name: "Amina", password: "hashed-later" },
});

const task = await prisma.task.create({
  data: { text: "Learn Prisma", userId: user.id },
});
```

### Read

```js
await prisma.task.findMany();                                   // all
await prisma.task.findMany({ where: { done: false } });         // filtered
await prisma.task.findMany({
  where: { userId: 1, text: { contains: "prisma" } },           // LIKE '%prisma%'
  orderBy: { createdAt: "desc" },
  skip: 10, take: 10,                                           // pagination
});
await prisma.task.findUnique({ where: { id: 3 } });             // one by unique field (or null)
await prisma.task.findFirst({ where: { done: true } });         // first match (or null)
await prisma.task.count({ where: { done: true } });

// Relations — the JOIN, without writing it:
await prisma.task.findMany({ include: { user: true } });        // each task with its user object
await prisma.user.findUnique({ where: { id: 1 }, include: { tasks: true } });   // user + all their tasks
await prisma.task.findMany({ select: { id: true, text: true } });               // only some columns
```

### Update and delete

```js
await prisma.task.update({ where: { id: 3 }, data: { done: true } });
await prisma.task.delete({ where: { id: 3 } });
await prisma.task.updateMany({ where: { userId: 1, done: true }, data: { done: false } });
await prisma.task.deleteMany({ where: { done: true } });
```

⚠️ `update` and `delete` **throw** if the row doesn't exist (error code `P2025`) — they don't return null. Handle it (section 6).

Every one of these is a parameterized SQL statement under the hood. Want to see? Enable logging: `new PrismaClient({ log: ["query"] })` — and read Lesson 6 in your terminal.

---

## 6. The service layer, Prisma edition

```js
// services/tasks.service.js
import { prisma } from "../db/prisma.js";
import { HttpError } from "../utils/HttpError.js";

export function listTasks({ done, search } = {}) {
  return prisma.task.findMany({
    where: {
      ...(done !== undefined && { done }),                       // include the filter only if given
      ...(search && { text: { contains: search } }),
    },
    orderBy: { createdAt: "desc" },
  });
}

export async function getTask(id) {
  const task = await prisma.task.findUnique({ where: { id } });
  if (!task) throw new HttpError(404, "Task not found");
  return task;
}

export function createTask(data) {
  return prisma.task.create({ data });
}

export async function updateTask(id, data) {
  await getTask(id);                                             // 404 if missing
  return prisma.task.update({ where: { id }, data });
}

export async function deleteTask(id) {
  await getTask(id);
  await prisma.task.delete({ where: { id } });
}
```

The `...(condition && { field })` trick: spread of `false` adds nothing, spread of an object adds its fields — a compact way to build a `where` from optional filters (Part 1 L14 spread, put to work).

Controllers stay as they were in Lesson 5/6 — they call the service and send JSON. Storage changed for the third time; the HTTP layer didn't move. That's what layers buy you.

### Prisma errors → HTTP errors

Add to your error handler:

```js
import { Prisma } from "@prisma/client";

export function errorHandler(err, req, res, next) {
  if (err instanceof Prisma.PrismaClientKnownRequestError) {
    if (err.code === "P2002") return res.status(409).json({ error: "Already exists" });   // unique violation
    if (err.code === "P2025") return res.status(404).json({ error: "Not found" });
  }
  // ... the existing handling
}
```

`P2002` (unique constraint) is what you'll hit when two users register the same email — Lesson 8.

---

## 7. Seeding — test data on demand

```js
// prisma/seed.js
import { prisma } from "../db/prisma.js";

await prisma.task.deleteMany();
await prisma.user.deleteMany();

const amina = await prisma.user.create({
  data: {
    email: "amina@mail.com", name: "Amina", password: "x",
    tasks: { create: [{ text: "Learn Prisma" }, { text: "Seed the DB", done: true }] },   // nested create!
  },
});
console.log("Seeded user", amina.id);
await prisma.$disconnect();
```

Add `"seed": "node prisma/seed.js"` to scripts and run `npm run seed` whenever you want a clean, known dataset. Automating this now pays off every day of the project.

---

## ⚠️ Common mistakes

```
1. Changing schema.prisma and not running migrate dev → "Unknown field" / table doesn't exist.
2. Creating `new PrismaClient()` in every file → connection exhaustion. One instance, exported (and see Lesson 16 for the Next.js hot-reload twist).
3. Forgetting await → you get a Promise, not data (Part 1 L15 all over again).
4. findUnique on a non-unique field → error. Use findFirst.
5. Expecting update/delete to return null for missing rows → they THROW (P2025). Check first or catch.
6. Passing req.params.id (a string) as `id` → Prisma expects Int. Number() it — validation should do this.
7. Committing dev.db → add *.db to .gitignore. Commit prisma/migrations (that IS the schema history).
```

---

## ✅ Classwork

1. Install Prisma, write the schema, run the first migration. Open `migration.sql` and compare it to your Lesson 6 `CREATE TABLE`. Open Prisma Studio.
2. In a scratch file (`scratch.js`), practice: create 2 users, 5 tasks across them, list tasks with users included, count done tasks, update one, delete one, list a user with their tasks. Turn on `log: ["query"]` and read the SQL.
3. Rewrite `tasks.service.js` on Prisma. Run the full Thunder Client collection — it should pass unchanged. (If controllers needed edits, ask why — that's a layering leak.)
4. Add Prisma error mapping to the error handler; trigger a P2025 by deleting an id twice.

## 📝 Homework

1. **Projects model:** add `Project { id, name, color, tasks Task[] }` and an optional `projectId` on Task. Migrate with a descriptive name. Implement `/api/projects` CRUD via Prisma and `GET /api/projects/:id/tasks` with `include`.
2. **Filtering & pagination via Prisma:** `GET /api/tasks?done=false&search=x&page=2&limit=5` returning `{ data, page, limit, total }` — `findMany` with `skip`/`take` plus a `count` with the same `where` (hint: build the `where` object once, reuse it).
3. **Seed script** with 3 users and 15 tasks using nested creates; add it to `package.json`.
4. **Research:** read Prisma's docs on `select` vs `include`, and on `@@index`. Write in comments: when would you add an index to `Task.userId`?
5. Written: three things an ORM does for you, and one reason you still need SQL.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **ORM** | Maps objects ⟷ tables; writes SQL for you |
| **Prisma** | The ORM used in this course (and most Next.js projects) |
| **`schema.prisma`** | Single source of truth for models and relations |
| **model** | A table definition |
| **relation** | `@relation(fields, references)` — the foreign key, plus virtual navigation fields |
| **migration** | A versioned SQL change generated from schema edits (`migrate dev`) |
| **Prisma Client** | The generated query API: `prisma.task.findMany(...)` |
| **`include` / `select`** | Join related data / choose columns |
| **`P2002` / `P2025`** | Unique violation / record not found |
| **seed** | Script that fills the DB with known test data |
