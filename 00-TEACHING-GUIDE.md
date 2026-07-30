# JavaScript Course — Teaching Guide (For the Teacher)

This folder contains a complete JavaScript course for a student with **zero programming experience**, taking them from "what is code?" to being ready for ordinary frontend work — and building fundamentals strong enough to carry into backend (Node.js) later.

---

## How the course is organized

| Phase | Lessons | Goal |
|-------|---------|------|
| **Phase 1: Core Programming** | 01 – 10 | Learn to *think* like a programmer. Everything here applies to backend too. **Do not rush this phase.** |
| **Phase 2: The Browser (Frontend)** | 11 – 13 | Use JavaScript to control web pages: DOM, events, forms. |
| **Phase 3: Modern & Async JS** | 14 – 16 | ES6+ features, promises, async/await, fetching data from APIs. |
| **Projects & Reference** | 17 – 18 | Guided projects and a cheat sheet. |

### Lesson list

1. `01-what-is-programming.md` — What programming is, how JavaScript fits in, setting up the tools
2. `02-variables-and-data-types.md` — Variables, let/const, the 7 data types
3. `03-operators.md` — Math, comparison, logical operators
4. `04-conditionals.md` — if/else, switch, truthy/falsy
5. `05-loops.md` — for, while, loop patterns
6. `06-functions.md` — Declaring, parameters, return, scope, arrow functions
7. `07-arrays.md` — Lists of data, core array methods
8. `08-objects.md` — Key-value data, methods, nesting
9. `09-built-in-methods.md` — String, Number, Math, Date, and the big array methods (map/filter/reduce)
10. `10-errors-and-debugging.md` — Reading errors, console debugging, try/catch
11. `11-the-dom.md` — Selecting and changing page elements
12. `12-events.md` — Clicks, keyboard, event listeners
13. `13-forms.md` — Reading inputs, validation
14. `14-modern-javascript.md` — Destructuring, spread, template literals, modules, JSON
15. `15-asynchronous-javascript.md` — Callbacks, promises, async/await
16. `16-fetch-and-apis.md` — Talking to servers, building an API-powered app
17. `17-PROJECTS.md` — Mini-projects and 3 capstone projects
18. `18-CHEATSHEET.md` — One-page quick reference for the student

---

## Suggested pace

Assuming ~2 sessions per week, 1.5–2 hours each:

| Weeks | Content |
|-------|---------|
| 1 | Lesson 1 + 2 |
| 2 | Lesson 3 + 4 |
| 3 | Lesson 5 (loops need extra practice — take your time) |
| 4 | Lesson 6 (functions — the most important lesson in the course) |
| 5 | Lesson 7 |
| 6 | Lesson 8 + 9 (start) |
| 7 | Lesson 9 (finish) + 10 |
| 8 | **Review week + Phase 1 mini-projects** (number guessing game, etc.) |
| 9 | Lesson 11 |
| 10 | Lesson 12 |
| 11 | Lesson 13 + first DOM project (to-do list) |
| 12 | Lesson 14 |
| 13 | Lesson 15 |
| 14 | Lesson 16 |
| 15–16 | Capstone project(s) |

This is a guide, not a law. **Move at the speed of understanding, not the speed of the calendar.** If the student is struggling with loops, stay on loops.

---

## Teaching tips for a complete beginner

1. **Type everything live.** Never just show finished code. Type it in front of the student, make mistakes on purpose, and show how you read the error message. Debugging is a skill they learn by watching you.

2. **Make the student type too.** Reading code feels like understanding; it isn't. The student should type every example themselves. "I'll just watch" is how people fail to learn programming.

3. **Predict before running.** Before running any code, ask: *"What do you think this will print?"* This single habit builds the mental model of how code executes — which is the real skill.

4. **One new idea at a time.** If an example teaches loops, don't also introduce a fancy string method in it. Keep examples boring except for the one new thing.

5. **Vocabulary matters.** Insist on correct words early: *variable, value, function, call, argument, parameter, return, property, method*. Backend documentation assumes this vocabulary.

6. **Homework every lesson.** Each lesson file ends with Classwork (do together) and Homework (student does alone). Review homework at the start of the next session — this reveals what actually stuck.

7. **Repetition is not failure.** Beginners need to see the same concept 4–5 times in different contexts before it sticks. Re-explaining is the job.

8. **Console first, browser later.** Phase 1 lives entirely in the console. Resist the urge to jump to "cool" DOM stuff early — weak fundamentals are the #1 reason self-taught developers plateau.

9. **When the student is stuck, don't grab the keyboard.** Ask questions instead: "What is this line doing? What is the value of `x` right now? What did the error say?"

10. **Connect to backend early.** Occasionally mention: "This exact code works in Node.js on a server too." It motivates and frames JS as one language, two environments.

---

## What "ready for ordinary frontend" means (course exit goals)

By the end, the student should be able to, without help:

- Build an interactive page: read user input, respond to clicks, update the page.
- Build a to-do list app with add/delete/complete, saved in `localStorage`.
- Fetch data from a public API and display it, with loading and error states.
- Read an unfamiliar error message and locate the bug.
- Read documentation (MDN) and use a method they've never seen before.

And crucially for their backend future: they should deeply understand **variables, functions, arrays, objects, and async** — because Node.js is the same language with a different set of built-in tools.

---

## Tools you need (both of you)

- **Google Chrome** (or Edge) — for the console and DevTools
- **Visual Studio Code** — the editor (free, code.visualstudio.com)
- **Node.js** (later, optional in this course but useful) — nodejs.org
- VS Code extension: **Live Server** (launches HTML pages with auto-reload)

Setup instructions for the student are in Lesson 01.

---

## Recommended free references (share with the student)

- **MDN Web Docs** (developer.mozilla.org) — the official-quality reference. Teach the student to search "mdn array push" instead of random blogs.
- **javascript.info** — excellent free deep-dive tutorial, good for re-reading topics.
- **freeCodeCamp** — extra practice exercises if the student wants more reps.

Discourage tutorial-hopping. This course + MDN is enough.
