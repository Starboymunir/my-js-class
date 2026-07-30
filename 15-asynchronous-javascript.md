# Lesson 15 — Asynchronous JavaScript: Waiting Without Freezing

> **Goal:** Understand how JavaScript handles operations that take time — network requests, timers, file reads — without freezing the page. Callbacks → Promises → `async/await`. This is the last big *conceptual* mountain of the course, and it is **essential** for both frontend (fetching data, next lesson) and backend (where nearly everything — databases, files, other services — is asynchronous). Go slow; take two sessions if needed.

---

## 1. The problem: some things take time

Most of your code so far is **synchronous**: each line finishes before the next begins, all effectively instant.

But real programs constantly do slow things:

- Ask a server for data (0.1s – 5s, who knows?)
- Wait for a timer
- (Backend:) query a database, read a big file

Here's the catch: **JavaScript runs on a single thread** — it can only do one thing at a time. If it simply *stopped and waited* three seconds for a server reply, the entire page would freeze: no clicks, no scrolling, no typing. Unacceptable.

The solution — **asynchronous** execution:

> *"Start the slow thing. While it works in the background, keep running the rest of my code. When it finishes, call this function I'm leaving with you."*

You've already done this without noticing! `setTimeout` (Lesson 12) is exactly that shape:

```js
console.log("1: Ordering food...");

setTimeout(() => {
  console.log("3: 🍕 Food arrived!");
}, 2000);

console.log("2: Watching TV while I wait...");
```

Output order: **1, 2, 3** — line "2" prints *before* the pizza, even though its code comes after. Prediction exercise: the single most important thing to internalize this lesson. Async code does not run where it's written; it runs **when it's ready**, and the surrounding code does NOT wait.

```js
// The classic beginner shock, in four lines:
let data = null;
setTimeout(() => { data = "loaded!"; }, 1000);
console.log(data);        // null 😱 — this line ran immediately, long before the timer fired
```

You cannot "reach forward in time" for an async result. All async patterns below are just increasingly pleasant ways to say *"...and when it's ready, do this with it."*

---

## 2. Callbacks — the original way

A **callback** is a function you hand over to be called later — you know these from events and `setTimeout`. Async code used to be built entirely from them:

```js
function downloadData(url, onDone) {
  console.log(`Downloading from ${url}...`);
  setTimeout(() => {                       // simulating a 2s network delay
    const data = { user: "Amina" };
    onDone(data);                          // "calling you back" with the result
  }, 2000);
}

downloadData("example.com/api", (result) => {
  console.log("Got:", result);
});
```

Fine for one step. But real tasks chain: log in → then fetch profile → then fetch posts → then render. With callbacks, each step nests inside the last:

```js
login(user, (session) => {
  fetchProfile(session, (profile) => {
    fetchPosts(profile, (posts) => {
      render(posts, () => {
        // 4 levels deep and drifting rightward... and where does error handling go? Everywhere. 😵
      });
    });
  });
});
```

This pyramid is infamous enough to have a name: **callback hell**. It's why Promises were invented. (You still need to *recognize* callback style — older Node.js code is full of it.)

---

## 3. Promises — a receipt for a future value

A **Promise** is an object representing *"a value that isn't here yet."* Like a receipt from a food stall: not the food, but a claim on food that's coming — unless the kitchen fails.

A promise is always in one of three states:

1. **pending** — still working
2. **fulfilled** — done! The value is ready
3. **rejected** — failed, with an error

You mostly *consume* promises returned by built-in functions (like `fetch`, next lesson). Consuming uses `.then` and `.catch`:

```js
downloadData("example.com/api")          // returns a Promise immediately (pending)
  .then((data) => {                      // .then registers: "when fulfilled, run this with the value"
    console.log("Got:", data);
  })
  .catch((error) => {                    // .catch: "if it rejects, run this with the error"
    console.log("Failed:", error.message);
  });

console.log("This still prints FIRST — promises don't pause the program!");
```

The killer feature — **chains instead of pyramids**. Each `.then` can return another promise, and the chain flows flat:

```js
login(user)
  .then((session) => fetchProfile(session))
  .then((profile) => fetchPosts(profile))
  .then((posts) => render(posts))
  .catch((error) => showError(error));     // ONE catch handles a failure at ANY step 🎉
```

Compare with callback hell above: same steps, flat and readable, centralized error handling.

*For completeness — making your own promise (rarely needed as a beginner):*

```js
function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
wait(1000).then(() => console.log("one second later"));
```

---

## 4. `async` / `await` — promises in comfortable clothes ⭐

Promises still read as "callback-ish." The final evolution makes async code *look* synchronous while behaving asynchronously. Two keywords:

- Mark a function **`async`** — it now automatically returns a promise, and gains the right to use...
- **`await`** — "pause *this function* (not the whole program!) until the promise settles, then hand me its value."

```js
async function showUser() {
  console.log("Loading...");
  const data = await downloadData("example.com/api");   // function suspends here until ready
  console.log("Got:", data);                            // then continues with the real value
}

showUser();
console.log("The rest of the program still doesn't wait!");   // prints before "Got:"
```

The chain example, rewritten — this is how you'll actually write async code:

```js
async function loadPage(user) {
  try {
    const session = await login(user);
    const profile = await fetchProfile(session);
    const posts = await fetchPosts(profile);
    render(posts);
  } catch (error) {                        // Lesson 10's try/catch — works on await!
    showError(error);
  }
}
```

Read it top to bottom like normal code. Errors handled with the `try/catch` you already know. This is the style used in modern frontend AND all over Node.js backend code — routes, database calls, everything. **`async/await` is the main thing you'll write; `.then` chains you'll mostly read.**

Rules to remember:

- `await` only works **inside an `async` function** (using it at the top level of a plain script is an error — wrap your code in an async function and call it).
- `await` un-wraps a promise into its value. No promise? It just passes the value through.
- An `async` function's `return` value arrives to callers as a *promise* — so callers `await` it too. Async is contagious upward, and that's fine.

### Mental model summary

| Style | Looks like | You'll use it... |
|---|---|---|
| Callback | `doThing(input, (result) => {...})` | Events, timers; reading old code |
| Promise `.then` | `doThing(input).then((r) => ...)` | Reading code, quick one-offs |
| `async/await` | `const r = await doThing(input);` | **Your default** for anything async |

All three are the same machinery underneath: *register code to run when the slow thing finishes.*

---

## ⚠️ Common mistakes

```js
// 1. Using an async result outside the async flow (the #1 conceptual bug):
let user = null;
fetchUser().then((u) => { user = u; });
console.log(user);                 // null — ran before the fetch finished!
// Everything that NEEDS the data must live in the .then / after the await.

// 2. Forgetting await — you get a Promise object, not the value:
async function f() {
  const data = fetchUser();        // ❌ data is Promise {<pending>}
  console.log(data.name);          // undefined
  const data2 = await fetchUser(); // ✅
}

// 3. await outside an async function → SyntaxError

// 4. No .catch / no try-catch → "Uncaught (in promise)" errors that vanish silently. Always handle.

// 5. Awaiting things one-by-one that could run together:
const a = await fetchA();          // 2s
const b = await fetchB();          // +2s = 4s total, but they didn't depend on each other!
const [a2, b2] = await Promise.all([fetchA(), fetchB()]);   // 2s total — parallel! (nice-to-know)
```

---

## ✅ Classwork

1. **Order prediction drills** — for each snippet, write the output order on paper BEFORE running:
   ```js
   // (a)
   console.log("A");
   setTimeout(() => console.log("B"), 0);     // yes, zero!
   console.log("C");
   // (b)
   console.log("start");
   wait(1000).then(() => console.log("middle"));
   console.log("end");
   ```
   (a) prints A, C, B — even at 0ms, callbacks wait for the current code to finish. Discuss why.
2. Build the `wait(ms)` helper from section 3. Use it with `.then` to print "3"... "2"... "1"... "Go!" one second apart (chain it!). Then rewrite with `async/await` — feel the difference.
3. Write `fakeFetchUser(id)` that returns a promise resolving after 1.5s with `{ id, name: "User" + id }` — but *rejecting* with an error if `id < 1`. Consume it twice (good id, bad id) using async/await + try/catch, printing friendly messages.
4. Trace mistake #1 (the `null` bug) live: run it, see the null, then fix it by moving the log inside the async flow.

## 📝 Homework

1. **Traffic light simulator:** using `wait()` + async/await, cycle a real colored `<div>` (DOM!): green 3s → yellow 1s → red 2s → repeat forever (`while (true)` is fine inside async here!). Buttons to start it. *Bonus: a stop button (a boolean flag the loop checks).*
2. **Loading state practice:** `fakeFetchProducts()` resolves after 2s with an array of 3 product objects — but has a 30% chance to reject (`Math.random() < 0.3`) with an error. Page: a "Load products" button, a `<p id="status">`, an empty `<ul>`. On click: status "Loading…", then either render the list (Lesson 11 skills) and status "Done ✅", or status shows the error in red. **Loading / success / error — the three states every real app screen has.** This is a dress rehearsal for next lesson.
3. **Sequential vs parallel:** with two fake fetches of 2s each, measure (`console.time("x")` / `console.timeEnd("x")` — look them up on MDN!) the difference between awaiting them one-by-one and using `Promise.all`. Write the timings in a comment.
4. **Written (comments):** explain to an imaginary classmate: why doesn't JavaScript just pause and wait? What does `await` actually pause? Why does async matter even more on a backend server that handles hundreds of users at once?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **synchronous** | Each line finishes before the next runs |
| **asynchronous** | Start now, finish later; other code runs meanwhile |
| **single-threaded** | JS does one thing at a time — why blocking is forbidden |
| **callback** | A function handed over to be called when something finishes |
| **callback hell** | Nested-pyramid callbacks — the problem promises solve |
| **Promise** | An object representing a future value (pending → fulfilled / rejected) |
| **`.then` / `.catch`** | Register success / failure handlers on a promise |
| **`async` / `await`** | Write async code that reads top-to-bottom; pairs with try/catch |
| **`Promise.all`** | Run several promises in parallel, await them together |
