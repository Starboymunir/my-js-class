# Lesson 16 — Fetch and APIs: Talking to Servers

> **Goal:** The final skill of the course — getting real data from real servers into your page. This connects everything: async/await, JSON, promises, DOM rendering, error handling. It's also your first genuine look across the bridge to backend: today you *consume* APIs; in your backend journey you'll *build* them.

---

## 1. What is an API?

When your frontend needs data — products, weather, user profiles — it asks a **server**: another computer on the internet whose job is answering such requests. The server exposes an **API** (Application Programming Interface): a set of URLs that return **data** (JSON!) instead of web pages.

```
Your JS  ──── HTTP request:  "GET /users/1" ────▶  Server
Your JS  ◀─── HTTP response: '{"id":1,"name":"Amina"}' ── (backend code + database live here)
```

Key vocabulary of the **HTTP** protocol (the web's language):

- **Request** — what you send: a URL + a **method**:
  - `GET` — "give me data" (the default; today's main tool)
  - `POST` — "here's new data, store it" (submitting forms, creating things)
  - `PUT`/`PATCH` — "update existing data" · `DELETE` — "remove data"
- **Response** — what comes back: a **status code** + a **body** (usually JSON):
  - `200` OK · `404` Not Found · `400` Bad Request · `500` Server Error
  - Memorize the neighborhoods: **2xx = success, 4xx = your request was wrong, 5xx = the server broke.**

An API URL like `https://api.example.com/users/1` is called an **endpoint**. API documentation = the list of endpoints and what they return.

> **The backend connection, explicitly:** when you learn Node.js, your job will be writing the *other side* of this conversation — code that receives `GET /users/1`, checks it (validation! Lesson 13's warning about never trusting the client!), queries a database, and sends back JSON with the right status code. Everything today has a mirror image waiting for you.

---

## 2. `fetch` — making requests

The browser's built-in function for HTTP requests. It's async (of course — networks are slow), returning a promise:

```js
async function loadUser() {
  const response = await fetch("https://jsonplaceholder.typicode.com/users/1");
  const user = await response.json();       // parse the JSON body (also async!)
  console.log(user.name);
}
loadUser();
```

Two awaits, and both matter:

1. `await fetch(url)` → resolves to a **Response object** once headers arrive: status code, ok-flag, and a body you haven't read yet.
2. `await response.json()` → reads the full body and `JSON.parse`s it into a real object.

*(That URL is real — **JSONPlaceholder** is a free fake API made for practicing. Open it in a browser tab and see raw JSON with your own eyes.)*

### Errors — the part beginners skip and regret

Two *different* failure modes, handled two different ways:

```js
async function loadUser(id) {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);

    // Failure mode A: server responded, but with an error status (404, 500...)
    // fetch does NOT throw for these! You must check yourself:
    if (!response.ok) {                       // ok === status in 200-299
      throw new Error(`Server said: ${response.status}`);
    }

    const user = await response.json();
    return user;

  } catch (error) {
    // Failure mode B lands here too: network dead, DNS failure, no internet — fetch itself rejects.
    console.log("Could not load user:", error.message);
    return null;
  }
}
```

This shape — `fetch → check response.ok → parse → try/catch around it all` — is **the** standard fetch recipe. Type it until your fingers know it.

---

## 3. Fetch → render: the complete frontend loop ⭐

Data from the internet, onto the page, with the three UI states (loading / success / error) you rehearsed last lesson:

```html
<h1>Our Team</h1>
<button id="load-btn">Load team</button>
<p id="status"></p>
<ul id="user-list"></ul>
```

```js
const statusEl = document.querySelector("#status");
const listEl = document.querySelector("#user-list");

async function loadTeam() {
  statusEl.textContent = "Loading… ⏳";
  listEl.innerHTML = "";

  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/users");
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    const users = await response.json();          // an array of user objects!

    for (const user of users) {                   // Lesson 8: array of objects
      const li = document.createElement("li");    // Lesson 11: render pattern
      li.textContent = `${user.name} — ${user.email}`;
      listEl.appendChild(li);
    }
    statusEl.textContent = `Loaded ${users.length} team members ✅`;

  } catch (error) {
    statusEl.textContent = `Something went wrong: ${error.message} ❌`;
  }
}

document.querySelector("#load-btn").addEventListener("click", loadTeam);
```

Pause and appreciate: **every single lesson of this course is in this file.** Variables, functions, loops, arrays, objects, template literals, DOM, events, async/await, try/catch, HTTP. This is real, professional-shape frontend code. This is what you now know how to do.

### Exploring an API's data shape

Before rendering unfamiliar data, *look at it first*:

```js
const data = await response.json();
console.log(data);                        // expand it in the console — what fields exist?
console.table(data);                      // gorgeous for arrays of objects
```

Real developer workflow: fetch → log → inspect → then write the render code. Never guess at property names.

---

## 4. Sending data — `POST` (a working preview)

`fetch` sends data too — the reverse direction. Compare with GET: now *we* stringify, and we announce the format:

```js
async function createPost(title, body) {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",                            // not the default GET
    headers: { "Content-Type": "application/json" },   // "dear server: I'm sending JSON"
    body: JSON.stringify({ title, body }),     // package our object for shipping
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return await response.json();                // servers usually echo back the created item
}

const created = await createPost("Hello", "My first API post!");
console.log(created);                          // { title: "Hello", body: "...", id: 101 }
```

(JSONPlaceholder *pretends* to save it — perfect for practice.) Form data flowing through POST to a server is exactly how signups, comments, and orders work everywhere. And handling this incoming POST on the server side? That's backend. You're standing at the border, waving across. 🌉

---

## 5. Free practice APIs

All free, no signup, great for projects:

| API | URL | Gives you |
|-----|-----|-----------|
| JSONPlaceholder | jsonplaceholder.typicode.com | Fake users/posts/todos (+ fake POST) |
| Open-Meteo | open-meteo.com | Real weather, no key needed |
| REST Countries | restcountries.com | Country data: flags, capitals, population |
| PokéAPI | pokeapi.co | Pokémon (huge, fun to explore) |
| Advice Slip | api.adviceslip.com/advice | Random advice one-liners |
| Dog CEO | dog.ceo/dog-api | Random dog pictures 🐶 |

*(Some APIs require an API key — a personal access code in the URL or headers. The ones above don't, which keeps practice friction-free. If a project someday needs a keyed API, read its docs — you can do that now.)*

---

## ⚠️ Common mistakes

```js
// 1. Forgetting that fetch does NOT throw on 404/500 → always check response.ok

// 2. Forgetting the second await:
const data = response.json();        // ❌ Promise, not data
const data = await response.json();  // ✅

// 3. Rendering before inspecting → "undefined" everywhere because the field is
//    actually data.results[0].title, not data.title. console.log FIRST.

// 4. No loading state → user clicks, sees nothing for 2s, clicks 5 more times.

// 5. CORS errors ("blocked by CORS policy") — the server refusing browser access.
//    Not your bug! Some APIs simply don't allow browser calls. Use the friendly APIs above.
//    (When YOU build backends, you'll configure CORS yourself — foreshadowing!)
```

---

## ✅ Classwork

1. Open `https://jsonplaceholder.typicode.com/users/3` in a browser tab. Read the raw JSON. Then fetch it from the console and log the user's name, email, and — careful, it's nested — company name and city.
2. Build the team loader from section 3 together, typing it all. Then break it two ways and observe both failure modes: (a) misspell the domain (network error), (b) change `/users` to `/userz` (404 → your `response.ok` check).
3. Fetch `https://jsonplaceholder.typicode.com/todos` (200 items!). Using Lesson 9 pipelines: how many are completed? Render only the first 10 uncompleted ones. (`slice`!)
4. Advice button: fetch `https://api.adviceslip.com/advice` on click, display the advice. Inspect the JSON first — the text is nested inside. (Notice it's the full loop: click → fetch → parse → render.)

## 📝 Homework

1. **Country lookup:** input + button. User types a country name → fetch `https://restcountries.com/v3.1/name/{name}` → display flag (it's an image URL — create an `<img>`!), capital, population (`toLocaleString()` for the commas), and region. Handle: empty input (validate first!), country not found (404), no internet. All three UI states required.
2. **Random dog gallery:** button fetches `https://dog.ceo/api/breeds/image/random/6` and renders six `<img>`s. Extra: a dropdown of 3 favorite breeds using the by-breed endpoint (read the docs at dog.ceo — documentation practice!).
3. **Post composer:** a form (title + body, validated) that POSTs to JSONPlaceholder and renders the returned object (including its new `id`) into a "Sent messages" list. Disable the submit button while sending, re-enable after (hint: `button.disabled = true` — and remember `finally` exists… or just re-enable in both try and catch. MDN: "try...catch finally").
4. **Written (comments):** in your own words — what happens, step by step, between clicking "Load team" and seeing names on screen? Include: HTTP request, server, status code, JSON, parsing, DOM. If you can write this paragraph clearly, you understand the web.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **API / endpoint** | A server's data-serving URLs |
| **HTTP** | The request/response protocol of the web |
| **GET / POST / PUT / DELETE** | Read / create / update / remove |
| **status code** | 2xx success · 4xx your fault · 5xx server's fault |
| **`fetch(url)`** | Make an HTTP request; resolves to a Response |
| **`response.ok` / `response.status`** | Did it succeed? Check it — fetch won't throw on 404! |
| **`response.json()`** | Parse the body (async — await it) |
| **headers / `Content-Type`** | Request metadata: "I'm sending JSON" |
| **CORS** | Server-side permission for browser access |
| **loading / success / error states** | The three faces of every data-driven screen |
