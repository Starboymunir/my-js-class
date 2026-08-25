# Lesson 05 — REST API Design, Full CRUD, Validation, and Error Handling

> **Goal:** Build a *professional-shape* API: consistent REST design, complete CRUD, proper validation, centralized error handling, and a folder structure that scales. This is the lesson where "some Express routes" becomes "an API someone could actually use".

---

## 1. REST — the conventions everyone agrees on

**REST** is a set of conventions for designing APIs so that they're predictable. You've been following it already without the name. The core ideas:

1. **URLs name resources (nouns), not actions.** `/api/tasks`, `/api/users/7` — not `/api/getTasks` or `/api/deleteUser`.
2. **The HTTP method is the verb.** `GET /tasks` reads; `POST /tasks` creates; `DELETE /tasks/7` removes.
3. **Plural nouns for collections; an id for one item.**
4. **Status codes carry meaning** (Lesson 3's table).
5. **JSON in, JSON out.**

The complete pattern for any resource:

| Action | Method + URL | Body in | Success response |
|--------|--------------|---------|------------------|
| List | `GET /api/tasks` | — | `200` + array |
| Read one | `GET /api/tasks/:id` | — | `200` + object (or `404`) |
| Create | `POST /api/tasks` | new data | `201` + created object |
| Update | `PATCH /api/tasks/:id` | changed fields | `200` + updated object (or `404`) |
| Replace | `PUT /api/tasks/:id` | full object | `200` + object |
| Delete | `DELETE /api/tasks/:id` | — | `204` (or `404`) |

Nested resources read naturally: `GET /api/users/7/tasks` — "tasks belonging to user 7".

Why bother with conventions? Because a frontend developer (or future you) can *guess* your API without reading docs. That's the whole value.

---

## 2. Project structure — splitting the file

One `server.js` doesn't scale. The standard layout:

```
tasks-api/
├── server.js               ← starts the app (listen)
├── app.js                  ← builds the app: middleware + mounts routers
├── routes/
│   └── tasks.routes.js     ← URL → controller mapping for tasks
├── controllers/
│   └── tasks.controller.js ← the handler functions (request in, response out)
├── services/               ← (later) business logic + data access
├── middleware/
│   ├── errorHandler.js
│   └── validate.js
├── data/                   ← JSON store for now; database later
├── .env  .env.example  .gitignore  package.json
```

Each layer has one job: routes *map*, controllers *handle HTTP*, services *do the work*. The layering may feel like overkill for 5 routes — it isn't, because you will add users, auth, and more resources, and each will slot into the same shape.

### Routers — `express.Router()`

A **router** is a mini-app for one resource. Define routes on it, then *mount* it at a prefix:

```js
// routes/tasks.routes.js
import { Router } from "express";
import * as tasks from "../controllers/tasks.controller.js";

const router = Router();

router.get("/", tasks.list);
router.get("/:id", tasks.getOne);
router.post("/", tasks.create);
router.patch("/:id", tasks.update);
router.delete("/:id", tasks.remove);

export default router;
```

```js
// app.js
import express from "express";
import tasksRouter from "./routes/tasks.routes.js";

const app = express();
app.use(express.json());
app.use("/api/tasks", tasksRouter);       // mount: router's "/" becomes "/api/tasks"

export default app;
```

```js
// server.js
import app from "./app.js";
app.listen(process.env.PORT ?? 3000, () => console.log("Ready"));
```

(Separating `app.js` from `server.js` lets tests import the app without starting a real server — Lesson 9.)

---

## 3. Controllers — the CRUD handlers

```js
// controllers/tasks.controller.js
import { load, save } from "../data/store.js";      // the JSON load/save from Lesson 2

export async function list(req, res) {
  const tasks = await load();
  res.json(tasks);
}

export async function getOne(req, res) {
  const tasks = await load();
  const task = tasks.find((t) => t.id === Number(req.params.id));
  if (!task) return res.status(404).json({ error: "Task not found" });
  res.json(task);
}

export async function create(req, res) {
  const { text } = req.body;
  if (typeof text !== "string" || text.trim() === "") {
    return res.status(400).json({ error: "text is required" });
  }
  const tasks = await load();
  const task = {
    id: tasks.length ? Math.max(...tasks.map((t) => t.id)) + 1 : 1,
    text: text.trim(),
    done: false,
    createdAt: new Date().toISOString(),
  };
  tasks.push(task);
  await save(tasks);
  res.status(201).json(task);
}

export async function update(req, res) {
  const tasks = await load();
  const task = tasks.find((t) => t.id === Number(req.params.id));
  if (!task) return res.status(404).json({ error: "Task not found" });

  const { text, done } = req.body;
  if (text !== undefined) {
    if (typeof text !== "string" || !text.trim()) return res.status(400).json({ error: "text must be a non-empty string" });
    task.text = text.trim();
  }
  if (done !== undefined) {
    if (typeof done !== "boolean") return res.status(400).json({ error: "done must be true or false" });
    task.done = done;
  }
  await save(tasks);
  res.json(task);
}

export async function remove(req, res) {
  const tasks = await load();
  const index = tasks.findIndex((t) => t.id === Number(req.params.id));
  if (index === -1) return res.status(404).json({ error: "Task not found" });
  tasks.splice(index, 1);
  await save(tasks);
  res.sendStatus(204);
}
```

Read `update` carefully: **PATCH means "only touch what was sent"** — hence the `!== undefined` checks. Every field is type-checked because `req.body` is whatever the client felt like sending. *Never trust the client* — you're now the one being not-trusted-by.

---

## 4. Validation — do it once, do it well

Repeating `typeof` checks in every controller gets old. Two better approaches:

### A validation function per resource

```js
// middleware/validate.js
export function validateTask({ text, done }, { partial = false } = {}) {
  const errors = [];
  if (!partial || text !== undefined) {
    if (typeof text !== "string" || text.trim() === "") errors.push("text must be a non-empty string");
    else if (text.length > 200) errors.push("text must be 200 characters or fewer");
  }
  if (done !== undefined && typeof done !== "boolean") errors.push("done must be a boolean");
  return errors;
}
```

```js
// in the controller
const errors = validateTask(req.body);
if (errors.length) return res.status(400).json({ errors });
```

Returning an **array of all problems** (not just the first) is kinder to frontend users — they can fix everything at once.

### Schema libraries (what the industry uses)

Writing validators by hand teaches the idea; real projects use a schema library. **Zod** is the current favorite (and it's everywhere in Next.js):

```bash
npm install zod
```

```js
import { z } from "zod";

const TaskSchema = z.object({
  text: z.string().trim().min(1, "text is required").max(200),
  done: z.boolean().optional(),
});

const result = TaskSchema.safeParse(req.body);
if (!result.success) {
  return res.status(400).json({ errors: result.error.issues.map((i) => i.message) });
}
const data = result.data;      // cleaned + guaranteed shape
```

Optional today; you'll meet it again in Lesson 16. Same idea, declarative.

---

## 5. Centralized error handling ⭐

Two things go wrong in every real API: (a) routes nobody defined, and (b) code that throws. Handle both in one place, at the **end** of the middleware line.

### The 404 catch-all

```js
// app.js — AFTER all routers
app.use((req, res) => {
  res.status(404).json({ error: `Route ${req.method} ${req.path} not found` });
});
```

### The error handler — four parameters

Express recognizes a middleware with **four** parameters `(err, req, res, next)` as the error handler. Anything thrown (or passed to `next(err)`) in any route skips straight to it:

```js
// middleware/errorHandler.js
export function errorHandler(err, req, res, next) {
  console.error(err);                                  // log the real thing for you...
  const status = err.status ?? 500;
  res.status(status).json({
    error: status === 500 ? "Something went wrong" : err.message,   // ...but don't leak internals to clients
  });
}
```

```js
// app.js — the LAST app.use
import { errorHandler } from "./middleware/errorHandler.js";
app.use(errorHandler);
```

*(Express 5 forwards errors from `async` handlers automatically. In Express 4 tutorials you'll see `try/catch` + `next(err)` in every route or a wrapper like `express-async-handler` — that's what this replaces.)*

### Throwing meaningful errors

With a handler in place, controllers get cleaner — **throw instead of manually responding**:

```js
// utils/HttpError.js
export class HttpError extends Error {
  constructor(status, message) {
    super(message);
    this.status = status;
  }
}
```

```js
import { HttpError } from "../utils/HttpError.js";

export async function getOne(req, res) {
  const tasks = await load();
  const task = tasks.find((t) => t.id === Number(req.params.id));
  if (!task) throw new HttpError(404, "Task not found");     // → errorHandler → 404 JSON
  res.json(task);
}
```

Every error now flows to one place with one format. Part 1 Lesson 10's *"validate, throw, catch at the boundary"* — the boundary is now the error middleware.

---

## 6. Testing the whole API

Thunder Client has **Collections** — save one request per endpoint (happy path + failure cases) and re-run them all after every change. This is your regression suite until Lesson 9 adds automated tests.

Checklist for a "done" endpoint:

- ✅ Success case returns the right status *and* shape
- ✅ Missing resource → 404 with a JSON error
- ✅ Bad input → 400 listing what's wrong
- ✅ Unknown route → 404 from the catch-all
- ✅ A deliberately thrown error → 500 with no stack trace leaked

---

## ⚠️ Common mistakes

```js
// 1. Error handler registered BEFORE routes → it never sees their errors. It goes LAST.

// 2. Error handler with 3 params → Express treats it as normal middleware. It needs exactly (err, req, res, next).

// 3. Leaking err.stack / raw database errors to clients → security hole + ugly. Log it; send a clean message.

// 4. PATCH that overwrites fields the client didn't send (done becomes undefined!) → check !== undefined.

// 5. GET /api/tasks/abc → Number("abc") is NaN → find() fails → 404. Fine — but consider a 400 "id must be a number".

// 6. Verbs in URLs: POST /api/tasks/create, GET /api/deleteTask/3. Nouns + methods, always.
```

---

## ✅ Classwork

1. Restructure the Lesson 4 server into `server.js` / `app.js` / `routes/` / `controllers/`. Confirm everything still works — refactoring without breaking is a skill.
2. Finish all five CRUD controllers with the JSON store. Save a Thunder Client collection with 10+ requests covering successes and failures.
3. Add the 404 catch-all and the error handler. Throw `new Error("boom")` inside one route to see the 500 path; then convert the 404s to `throw new HttpError(404, ...)`.
4. Write `validateTask` and use it in `create` (full) and `update` (partial). Send `{ "text": 123, "done": "yes" }` and confirm both errors come back together.

## 📝 Homework

1. **Second resource:** add `/api/projects` with full CRUD (`{ id, name, color }`), its own router + controller + validation, stored in `data/projects.json`. Then give tasks an optional `projectId`, and implement `GET /api/projects/:id/tasks`.
2. **Pagination:** `GET /api/tasks?page=2&limit=5` returns `{ data: [...], page, limit, total }`. Default page 1, limit 10, max limit 50 (validate!). Use `slice`.
3. **Zod (research):** install Zod and replace `validateTask` with a schema. Read the Zod docs for `.optional()`, `.max()`, and `.safeParse`.
4. **Written:** for each of the six REST rows in section 1, write the exact Thunder Client request (method, URL, body) and the expected status. Then explain why `POST /api/tasks/5/complete` is un-RESTful and what the RESTful version is.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **REST** | Conventions: nouns in URLs, verbs as methods, status codes, JSON |
| **resource** | A "thing" your API manages (tasks, users) |
| **router** | `express.Router()` — a mountable group of routes |
| **controller** | The handler function: reads the request, sends the response |
| **service** | Business logic + data access, separated from HTTP |
| **validation** | Checking + cleaning input; return ALL errors |
| **Zod** | Schema validation library (used heavily in Next.js) |
| **error-handling middleware** | `(err, req, res, next)` — must be last |
| **`HttpError`** | An Error with a status code, thrown by controllers |
