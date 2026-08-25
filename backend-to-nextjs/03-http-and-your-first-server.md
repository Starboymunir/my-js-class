# Lesson 03 — HTTP In Depth and Your First Server

> **Goal:** Understand HTTP properly — the protocol every web app speaks — and build a web server with nothing but Node's built-in `http` module. We do it the hard way *once*, so that when Express arrives next lesson you know exactly what it's saving you from.

---

## 1. HTTP, properly this time

In Part 1 L16 you learned the shape: request → response, methods, status codes. Now the full anatomy, because you're about to *produce* these by hand.

### A request

When your browser fetches `http://localhost:3000/tasks?done=true`, it sends text that looks like this:

```
GET /tasks?done=true HTTP/1.1          ← request line: METHOD  PATH(+query)  version
Host: localhost:3000                   ← headers: key-value metadata
Accept: application/json
Content-Type: application/json         ← (for POST/PUT) "the body is JSON"
Authorization: Bearer eyJhbGci...      ← (later) "here's who I am"
                                       ← blank line
{"text": "Learn HTTP"}                 ← body (only for POST/PUT/PATCH; GET has none)
```

Parts of a URL, since you'll be picking them apart:

```
http://localhost:3000/tasks/42?done=true&sort=asc
└─┬─┘  └───┬───┘└─┬┘└───┬───┘└────────┬─────────┘
protocol  host   port   path        query string
```

- **`localhost`** = "this computer". **Port** = which program on it (many can listen at once; 3000 is a common dev choice).
- **Path** identifies the resource. **Query string** carries optional filters/options.

### A response

```
HTTP/1.1 200 OK                        ← status line
Content-Type: application/json         ← headers
Content-Length: 27

[{"id":1,"text":"Learn HTTP"}]         ← body
```

### Methods and status codes — the full working set

| Method | Meaning | Has body? |
|--------|---------|-----------|
| `GET` | Read | No |
| `POST` | Create | Yes |
| `PUT` | Replace entirely | Yes |
| `PATCH` | Update partially | Yes |
| `DELETE` | Remove | Usually no |

| Code | Meaning | When you'll send it |
|------|---------|---------------------|
| `200 OK` | Success | GET, PUT, PATCH succeeded |
| `201 Created` | Success, something new exists | After a POST creates a record |
| `204 No Content` | Success, nothing to return | After a DELETE |
| `400 Bad Request` | Client sent invalid data | Validation failed |
| `401 Unauthorized` | Not logged in | Missing/invalid token |
| `403 Forbidden` | Logged in, but not allowed | Not the owner |
| `404 Not Found` | No such resource | Unknown id or path |
| `409 Conflict` | Clashes with existing state | Email already registered |
| `500 Internal Server Error` | *Your* code crashed | Bugs (should be rare!) |

**HTTP is stateless**: each request is a complete, independent conversation. The server doesn't remember the previous one. "Staying logged in" therefore requires sending proof with *every* request — that's Lesson 8.

---

## 2. A server with `node:http`

A server is a program that (1) listens on a port, (2) runs a function for each incoming request, (3) writes a response. Node's built-in module does exactly this:

```js
// server.js
import http from "node:http";

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);         // log every request

  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain");
  res.end("Hello from my server!");                // send body and finish
});

server.listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});
```

Run `node server.js`. The terminal prints the message and… keeps running. **Servers run forever by design** — they're waiting. Open `http://localhost:3000` in your browser: there's your text. Watch the terminal log the request. (Chrome sends a second request for `/favicon.ico` — that's normal.) Stop it with Ctrl+C.

What just happened, in the diagram: the browser (left box) sent an HTTP request to port 3000 on your own machine; your callback ran (middle box); `res.end` sent the response back. **You wrote the other side.**

Recognize the shape? `createServer((req, res) => ...)` is `addEventListener("click", (event) => ...)` — *register a function; the runtime calls it when something happens.* Same async event model, different event.

---

## 3. Routing by hand

Different paths should do different things. With raw `http`, you inspect `req.url` and `req.method` yourself:

```js
import http from "node:http";

const tasks = [
  { id: 1, text: "Learn HTTP", done: false },
  { id: 2, text: "Build a server", done: false },
];

function sendJson(res, status, data) {
  res.writeHead(status, { "Content-Type": "application/json" });
  res.end(JSON.stringify(data));
}

const server = http.createServer((req, res) => {
  const { method, url } = req;

  if (method === "GET" && url === "/") {
    return sendJson(res, 200, { message: "Tasks API v0" });
  }

  if (method === "GET" && url === "/tasks") {
    return sendJson(res, 200, tasks);
  }

  // GET /tasks/1 — need to pull the id out of the path
  if (method === "GET" && url.startsWith("/tasks/")) {
    const id = Number(url.split("/")[2]);
    const task = tasks.find((t) => t.id === id);
    if (!task) return sendJson(res, 404, { error: "Task not found" });
    return sendJson(res, 200, task);
  }

  sendJson(res, 404, { error: "Route not found" });
});

server.listen(3000, () => console.log("http://localhost:3000"));
```

Test in the browser: `/`, `/tasks`, `/tasks/1`, `/tasks/99`, `/nonsense`. Look at the status codes in DevTools → Network tab. Also test with `fetch` from a browser console, or with **Thunder Client** in VS Code (install it now — it's the tool for the rest of the course).

The `sendJson` helper is a real programmer instinct: three lines repeated three times → make a function.

---

## 4. Reading a request body (the painful part)

POST requests carry data. With raw Node, the body arrives in **chunks** over time (it's a network stream), and you must collect and parse it yourself:

```js
function readBody(req) {
  return new Promise((resolve, reject) => {
    let raw = "";
    req.on("data", (chunk) => { raw += chunk; });     // collect pieces as they arrive
    req.on("end", () => {                              // all pieces received
      try {
        resolve(raw ? JSON.parse(raw) : {});
      } catch {
        reject(new Error("Invalid JSON"));
      }
    });
    req.on("error", reject);
  });
}

// inside createServer's callback (make it async):
if (method === "POST" && url === "/tasks") {
  try {
    const body = await readBody(req);
    if (!body.text || typeof body.text !== "string") {
      return sendJson(res, 400, { error: "text is required" });
    }
    const task = { id: tasks.length + 1, text: body.text, done: false };
    tasks.push(task);
    return sendJson(res, 201, task);
  } catch (err) {
    return sendJson(res, 400, { error: err.message });
  }
}
```

Test with Thunder Client: **POST** `http://localhost:3000/tasks`, body (JSON) `{ "text": "Test from Thunder" }`. Then GET `/tasks` — it's there. (Until you restart the server — it's only in memory. Lesson 6 fixes that.)

Notice `req.on("data", ...)` — events again, on the request object. And a hand-built Promise wrapping an event-based API — the pattern from Part 1 L15 in the wild.

---

## 5. Why this is enough, and why we'll stop doing it

Look back at what you've written: URL parsing by hand, method checks by hand, `split("/")[2]` to get an id, a body reader from scratch, JSON headers every time, error handling scattered everywhere. It works — and every real server does exactly these things underneath. But it's repetitive and fragile, and we've only got 4 routes.

Next lesson, **Express** makes this:

```js
app.get("/tasks/:id", (req, res) => {
  const task = tasks.find((t) => t.id === Number(req.params.id));
  if (!task) return res.status(404).json({ error: "Task not found" });
  res.json(task);
});
```

You'll appreciate every line of it, because you know what it replaces. That was the point of today.

---

## ⚠️ Common mistakes

```
1. "EADDRINUSE: address already in use :::3000"
   → a previous server is still running. Find that terminal, Ctrl+C. (Or change the port.)

2. Browser "hangs" forever → you never called res.end(). Every request must be ended.

3. Sending JSON without the Content-Type header → clients treat it as plain text.

4. "Cannot set headers after they are sent" → you responded twice; you forgot a `return` after sending.
   Every "send and stop" must be `return sendJson(...)`.

5. Testing a POST by typing the URL in the browser → the address bar only does GET. Use Thunder Client/fetch.

6. Comparing url === "/tasks/" vs "/tasks" — the trailing slash matters in raw http.
```

---

## ✅ Classwork

1. Build the hello server. Visit it. Watch the log. Change the text, restart (`--watch` helps!), reload.
2. Add routes: `GET /time` returns `{ "now": "<ISO timestamp>" }`; `GET /greet?name=Amina` returns `{ "message": "Hello, Amina!" }` (parse the query — try `new URL(req.url, "http://localhost")` and `.searchParams.get("name")`).
3. Build the tasks routes from section 3 + the POST from section 4. Test every route in Thunder Client, checking the status code each time. Send invalid JSON on purpose and confirm the 400.
4. Open DevTools → Network on your `/tasks` page and inspect the request and response headers. Find the status code, Content-Type, and your own header if you add one (`res.setHeader("X-Powered-By", "my-hands")`).

## 📝 Homework

1. **Complete the raw API:** add `PATCH /tasks/:id` (toggle `done`, or set fields from the body) and `DELETE /tasks/:id` (204 on success, 404 if missing). Test all five endpoints.
2. **Persist it:** replace the in-memory array with the `load()`/`save()` JSON-file functions from Lesson 2. Now tasks survive a server restart.
3. **Static file serving (research):** make `GET /` read `index.html` from disk with `readFile` and send it with `Content-Type: text/html`. Put your Part 1 to-do page there, and change its JS to `fetch("/tasks")` from YOUR server instead of localStorage. The browser box and the server box are now both yours. 🎉
4. Written: what does "stateless" mean, and why does it make login harder? Write the full request text (like section 1) that Thunder Client sends for your POST /tasks.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **HTTP** | The request/response text protocol of the web |
| **request line / headers / body** | The three parts of a request |
| **port** | A numbered door on a machine; one program per port |
| **`localhost`** | This machine |
| **stateless** | Each request stands alone; the server remembers nothing between them |
| **`http.createServer`** | Register a function to run per request |
| **`req` / `res`** | Incoming request / outgoing response objects |
| **routing** | Choosing what to do based on method + path |
| **stream / chunk** | Data arriving in pieces over time |
| **201 / 204 / 400 / 404** | Created / No content / Bad request / Not found |
