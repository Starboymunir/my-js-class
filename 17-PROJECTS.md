# 17 — Projects: Where Learning Becomes Skill

Lessons teach concepts; **projects make them stick**. A student who has built five small things beats a student who has watched fifty tutorials. This file has three tiers:

- **Mini-projects** — one session each, slotted between lessons (recommended points noted)
- **Capstone projects** — multi-week builds for the end of the course
- **Rules of engagement** — how to run projects so they actually teach

---

## How to run projects (teacher notes)

1. **Student drives, teacher navigates.** The student types everything. You may point, ask, and hint — grabbing the keyboard should feel like a rare event.
2. **Start every project with a plan on paper:** What's the state (data)? What does the user see? What can the user do? Only then open the editor. This is the professional habit that separates builders from tutorial-followers.
3. **Build ugly first.** Working > pretty. Styling comes last (or never — this is a JS course).
4. **When stuck, the ladder is:** re-read the error → console.log the suspects → re-read the relevant lesson → ask teacher for a *hint* → ask for the answer. Each rung teaches more than the one below it.
5. **Finish.** A finished small project beats an abandoned big one. Cut features, not completion.
6. Keep every project in its own folder: `projects/01-number-game/` etc. This becomes the student's **portfolio** — and their first taste of git/GitHub later, if you choose to introduce it.

---

## Mini-projects

### After Lesson 5 (loops) — 🎯 Number Guessing Game (console)
Computer picks 1–100 (`Math.floor(Math.random() * 100) + 1` — give them this line, it's from a future lesson). Player guesses via `prompt()`, program replies "higher"/"lower"/"correct in N tries!" with a `while` loop.
*Exercises: loops, conditionals, Number() conversion, accumulator (try counter).*

### After Lesson 6 (functions) — 🧮 Console Calculator
`prompt` for two numbers and an operation; functions for each operation; a `calculate(a, b, op)` dispatcher; handles divide-by-zero and invalid ops with early returns.
*Exercises: functions, return, parameters, switch/else-if.*

### After Lesson 8 (objects) — 🃏 Rock Paper Scissors (console)
Best of 5 vs computer. State object: `{ playerScore, computerScore, round }`. Function that decides a round's winner; loop until someone reaches 3.
*Exercises: objects, functions, loops, game logic. Surprisingly tricky — good desirable difficulty.*

### After Lesson 9 (methods) — 📊 Class Report Card (console)
Given an array of `{ name, scores: [...] }` students: compute each student's average (function reuse!), the class average, top student, pass list (avg ≥ 50), and print a formatted report with `map`/`filter`/template literals.
*Exercises: array-of-objects mastery, map/filter, composition. This one is a big rung on the ladder.*

### After Lesson 12 (events) — 🎨 Theme Picker Page
Buttons that restyle the page (classList), a live character-counting input, a click-counter easter egg, show/hide sections.
*Exercises: first fully interactive page; DOM + events fluency.*

### After Lesson 13 (forms) — 📋 To-Do List
Built inside Lesson 13 itself. Extending it is homework there; consider a second session polishing it — it's the single highest-value beginner project in existence.

### After Lesson 15 (async) — 🚦 Traffic Light + Typing Speed Test
Traffic light is Lesson 15 homework. Typing test: show a sentence, time the user retyping it (`Date.now()` differences), compute words-per-minute, track best score in localStorage.
*Exercises: timers, state, events, localStorage.*

---

## Capstone projects (choose 1–2, in order of ambition)

### 🌦️ Capstone A — Weather Dashboard
**API:** Open-Meteo (free, no key).
User enters a city → geocode it (Open-Meteo's geocoding endpoint) → fetch current weather + 5-day forecast → render nicely. Loading/error states. Recent-searches list in localStorage (click to re-search). Celsius/Fahrenheit toggle (state + re-render!).
*Everything in the course, plus reading real API docs. The geocoding two-step (city → coordinates → weather) is genuine multi-fetch logic.*

### 🛒 Capstone B — Mini Store
Product data: a local array of ~10 objects (or fetch from `https://fakestoreapi.com/products` for real API practice). Product grid rendered from data; search box filtering live (`input` event + `filter`); category buttons; a cart (add/remove/quantity, total price) that persists in localStorage; a fake "checkout" form with full validation and an order-summary screen.
*The closest to real commercial frontend work: state shape (`products`, `cart`, `filters`), rendering pipelines, forms. Big — cut scope freely.*

### 🧠 Capstone C — Quiz App
Questions from `https://opentdb.com/api.php?amount=10` (Open Trivia DB — free). One question at a time, answer buttons, instant right/wrong feedback, score tracking, progress bar ("7/10"), final results screen with a play-again button, high scores in localStorage. *Careful: the API returns text with HTML entities — decoding them is a nice real-world wrinkle (research task!).*
*State machines (question index, score, phase: playing/finished), async, rendering. Very satisfying to demo to family — motivation matters.*

---

## After the course — the path onward

When the student finishes, in rough order of value:

1. **More vanilla projects.** Two or three more self-invented projects (a habit tracker, a recipe box, a flashcard app). Self-invented = they design the state shape themselves — the real test.
2. **Git + GitHub.** Version control, pushing the portfolio online. Deploy projects free on GitHub Pages/Netlify — a live URL is rocket fuel for motivation.
3. **Deeper CSS** if frontend is the goal (flexbox, grid, responsive design).
4. **React** — only now. Every React concept (components ≈ render functions, props ≈ parameters, state ≈ state, JSX ≈ createElement) will map onto something they built by hand. Students who skip the vanilla phase struggle in React forever; your student won't.
5. **Backend: Node.js + Express.** The same JavaScript + Lesson 16 from the server's side: routes are endpoints, handlers are async functions, validation is Lesson 13's warning made real, JSON everywhere. Then a database (start with SQLite or MongoDB). The fundamentals from Phase 1 carry 100% — that was the plan all along.
