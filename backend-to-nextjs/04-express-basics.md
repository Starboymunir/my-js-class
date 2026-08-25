# Lesson 04 — Express: The Web Framework

> **Goal:** Learn Express, the standard Node.js web framework, and rebuild last lesson's server in a fraction of the code. Routes, parameters, query strings, JSON bodies, and the big idea that makes Express tick: **middleware**.

---

## 1. What Express is

Express is a package that wraps `node:http` and handles the repetitive parts for you: routing, parsing URLs and bodies, sending JSON, organizing code. It's been the default Node web framework since 2010; nearly every Node tutorial and job listing assumes it. (Next.js has its own built-in server layer, but it's built on the same concepts — request, response, routes, middleware — so learning Express first is learning Next.js's foundations.)

```bash
mkdir tasks-api && cd tasks-api
npm init -y
npm install express
```

Set `"type": "module"` and a `dev` script (`node --watch --env-file=.env server.js`) as always. Create a `.env` with `PORT=3000` and a `.gitignore`.

---

## 2. Hello, Express

```js
// server.js
import express from "express";

const app = express();                          // the application object

app.get("/", (req, res) => {                    // "when a GET request hits /, run this"
  res.send("Hello from Express!");
});

app.get("/api/status", (req, res) => {
  res.json({ ok: true, time: new Date().toISOString() });   // JSON + correct header, one call
});

const PORT = process.env.PORT ?? 3000;
app.listen(PORT, () => console.log(`Server on http://localhost:${PORT}`));
```

Compare with Lesson 3: no `createServer`, no manual `writeHead`, no `JSON.stringify`, no if/else chain on `req.url`. Each route is one declaration: **method + path + handler function**.

The handler signature `(req, res)` is the same idea as raw Node, but both objects are upgraded with conveniences:

| Express `req` | Gives you |
|---|---|
| `req.params` | Route parameters (`/tasks/:id` → `{ id: "42" }`) |
| `req.query` | Query string as an object (`?done=true` → `{ done: "true" }`) |
| `req.body` | Parsed JSON body (needs middleware — section 4) |
| `req.method`, `req.path`, `req.headers` | The basics |

| Express `res` | Does |
|---|---|
| `res.json(data)` | Send JSON with the right header |
| `res.send(text)` | Send text/HTML |
| `res.status(404)` | Set the status code (chainable: `res.status(404).json({...})`) |
| `res.sendStatus(204)` | Set status and end with no body |
| `res.redirect("/somewhere")` | Send a redirect |

---

## 3. Routes and parameters

### Route parameters — `:name`

```js
const tasks = [
  { id: 1, text: "Learn Express", done: false },
  { id: 2, text: "Build an API", done: false },
];

app.get("/api/tasks", (req, res) => {
  res.json(tasks);
});

app.get("/api/tasks/:id", (req, res) => {
  const id = Number(req.params.id);            // params are STRINGS — convert!
  const task = tasks.find((t) => t.id === id);
  if (!task) {
    return res.status(404).json({ error: "Task not found" });    // return to stop here!
  }
  res.json(task);
});
```

`:id` is a placeholder that matches whatever sits in that position of the URL. `/api/tasks/42` → `req.params.id === "42"`. Multiple params work too: `/users/:userId/posts/:postId`.

### Query strings — `req.query`

Optional filters, sorting, pagination:

```js
// GET /api/tasks?done=true
app.get("/api/tasks", (req, res) => {
  let result = tasks;
  if (req.query.done !== undefined) {
    const wantDone = req.query.done === "true";       // query values are strings too
    result = tasks.filter((t) => t.done === wantDone);
  }
  res.json(result);
});
```

Rule of thumb: **params identify a thing** (`/tasks/42`); **query modifies a list** (`/tasks?done=true&sort=newest`).

### Route order matters

Express checks routes top to bottom and uses the **first match**. Put specific routes before generic ones, and catch-alls last.

---

## 4. Middleware — the heart of Express ⭐

A **middleware** is a function that runs *between* the request arriving and your route handler. It receives `(req, res, next)` and must either end the response or call `next()` to pass control down the line.

```
request → [logger] → [json parser] → [auth check] → route handler → response
```

Think of it as an assembly line: each station can inspect the request, modify it, reject it, or wave it through.

### Built-in: parsing JSON bodies

Remember Lesson 3's 15-line `readBody`? Express has it as one line:

```js
app.use(express.json());          // for every request: if the body is JSON, parse it into req.body

app.post("/api/tasks", (req, res) => {
  const { text } = req.body;      // already an object 🎉
  if (!text || typeof text !== "string") {
    return res.status(400).json({ error: "text is required" });
  }
  const task = { id: tasks.length + 1, text, done: false };
  tasks.push(task);
  res.status(201).json(task);
});
```

`app.use(fn)` registers middleware for *all* routes. **It must come before the routes that need it** — order matters, because it's a line.

### Writing your own: a request logger

```js
function logger(req, res, next) {
  console.log(`${new Date().toISOString()} ${req.method} ${req.path}`);
  next();                          // ← without this, the request hangs forever
}

app.use(logger);
```

Now every request prints a line. Three-line middleware; big habit — most real apps have one.

### Middleware that rejects

```js
function requireApiKey(req, res, next) {
  if (req.headers["x-api-key"] !== process.env.API_KEY) {
    return res.status(401).json({ error: "Invalid API key" });   // end here; next() never called
  }
  next();
}

app.get("/api/secret", requireApiKey, (req, res) => {     // route-specific: passed before the handler
  res.json({ secret: "the cake is real" });
});
```

Test in Thunder Client: without the header → 401; with header `x-api-key: <your value>` → 200. This exact shape — *check something, reject or pass* — becomes real authentication in Lesson 8.

### Middleware that adds to `req`

```js
app.use((req, res, next) => {
  req.requestTime = Date.now();
  next();
});
// later, any handler can read req.requestTime
```

Auth middleware will do this too: verify the token, then set `req.user` for handlers to use.

---

## 5. Serving static files

Your frontend (HTML/CSS/JS) can be served by the same Express app:

```js
app.use(express.static("public"));    // any file in /public is served at its path
```

Put `index.html` and `app.js` in `public/`, and `http://localhost:3000/` serves the page, whose `fetch("/api/tasks")` hits the same server. One app, both boxes of the diagram. (The `/api` prefix on routes exists precisely to keep API paths separate from page paths.)

---

## 6. The full server so far

```js
import express from "express";

const app = express();
app.use(express.json());
app.use(express.static("public"));
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});

const tasks = [];
let nextId = 1;

app.get("/api/tasks", (req, res) => res.json(tasks));

app.get("/api/tasks/:id", (req, res) => {
  const task = tasks.find((t) => t.id === Number(req.params.id));
  if (!task) return res.status(404).json({ error: "Task not found" });
  res.json(task);
});

app.post("/api/tasks", (req, res) => {
  const { text } = req.body;
  if (!text?.trim()) return res.status(400).json({ error: "text is required" });
  const task = { id: nextId++, text: text.trim(), done: false };
  tasks.push(task);
  res.status(201).json(task);
});

app.listen(process.env.PORT ?? 3000, () => console.log("Ready"));
```

Twenty-five readable lines for what took sixty fragile ones in Lesson 3. Next lesson we finish the CRUD set properly and organize this into files.

---

## ⚠️ Common mistakes

```js
// 1. Forgetting app.use(express.json()) → req.body is undefined → "Cannot read properties of undefined"

// 2. Putting express.json() AFTER the routes → those routes don't get it. Middleware order = line order.

// 3. Forgetting next() in middleware → request hangs, client waits forever

// 4. Comparing req.params.id === 42 → params are strings. Number() first.

// 5. Sending twice ("Cannot set headers after they are sent") → missing return before res.status(404)...

// 6. Thunder Client body not set to JSON → server sees an empty body. Check the Body tab: JSON, not Text.

// 7. Route defined as "/api/tasks/" vs client calling "/api/tasks" — Express is lenient here, but check spelling!
```

---

## ✅ Classwork

1. Set up `tasks-api`, install Express, build the hello server. Confirm `/` and `/api/status` in the browser.
2. Add the tasks routes (GET list, GET one, POST). Test all with Thunder Client, including 404 and 400 cases.
3. Write the logger middleware. Then add a middleware that counts total requests (a variable outside the function) and exposes the count at `GET /api/stats`.
4. Implement `?done=true|false` filtering and `?search=word` (case-insensitive `includes`). Combine both in one request.
5. Add `requireApiKey` on one route only. Test with and without the header.

## 📝 Homework

1. **Finish CRUD (draft):** `PATCH /api/tasks/:id` (update `text` and/or `done` from the body — only fields provided) and `DELETE /api/tasks/:id` (204). Full Thunder Client test pass: every route, every failure case.
2. **Static frontend:** put a copy of your Part 1 to-do page in `public/` and rewire it to use your API (`fetch` GET on load, POST on add, PATCH on toggle, DELETE on ✖). The page's `render()` now draws data that lives on the server. Notice which parts of the Part 1 code survived unchanged.
3. **Middleware practice:** write `slowDown` middleware that waits 500ms (`await new Promise(r => setTimeout(r, 500))` — middleware can be async) before `next()`. Apply it to one route and feel the delay. Then write `blockBots` that rejects with 403 when the `user-agent` header contains `"curl"`.
4. Written: explain middleware to someone using a non-programming analogy (airport security? a factory line?). Where in the line must `express.json()` go, and why?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **Express** | The standard Node web framework |
| **`app.get/post/patch/delete(path, handler)`** | Declare a route |
| **`req.params`** | Values from `:placeholders` in the path (strings!) |
| **`req.query`** | Query string values (strings!) |
| **`req.body`** | Parsed JSON body (requires `express.json()`) |
| **`res.status(n).json(obj)`** | Send a JSON response with a status code |
| **middleware** | `(req, res, next)` function in the request pipeline |
| **`next()`** | Pass to the next middleware/route |
| **`app.use()`** | Register middleware (order = execution order) |
| **`express.static`** | Serve files from a folder |
