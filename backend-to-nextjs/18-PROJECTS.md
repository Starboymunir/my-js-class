# 18 — Projects: Backend, React, and Next.js

Part 1's rule still holds: **projects make it stick.** This part is heavier on building than Part 1 — every lesson already grows the Tasks app — so the extra projects here are about *transfer*: applying the same skills to a different domain, which is where real understanding shows.

---

## How to run projects in this part (teacher notes)

1. **Plan on paper first, always:** (a) the data model — tables, columns, relations, drawn as boxes and arrows; (b) the endpoints or pages; (c) which parts are server vs client (Phase C). Twenty minutes of planning saves hours.
2. **Student drives.** You navigate. The moment you're tempted to type, ask a question instead.
3. **Ship something small and deployed** rather than something big and local. A live URL the student can send to a friend is worth more than three unfinished features.
4. **Git from the start**, commits after every working step. Read the diff together sometimes — reviewing your own change before committing is a professional habit.
5. **Security checklist before "done":** validated input · hashed passwords · queries scoped to the user · secrets in env · no hashes/secrets in responses · errors don't leak internals.
6. **Debug from logs.** Terminal for the server, DevTools for the browser, host dashboard for production. Ask "which box did this error come from?" every time.

---

## Mini-projects (one or two sessions each)

### After Lesson 2 — 📓 Notes CLI
Lesson 2 homework, extended: notes as JSON with tags; `search <word>`; `tag <slug> <tag>`; `list --tag work`. Practice: argv parsing, fs, filtering.

### After Lesson 5 — 📚 Library API (Express, JSON store)
Resources: `books` (`title, author, year, available`) and `members`. Endpoints: full CRUD on both, plus `POST /api/books/:id/borrow` and `/return` with rules (can't borrow an unavailable book → 409). Validation, error handler, Thunder Client collection, README. *Exercises REST design and rule enforcement — the first "business logic" project.*

### After Lesson 7 — 🏦 Bank Ledger (Express + Prisma)
`Account` and `Transaction` models. `POST /api/accounts/:id/transfer` moves money between accounts **inside a Prisma transaction** (`prisma.$transaction`) — all or nothing; insufficient funds → 400. `GET /api/accounts/:id/statement` lists transactions with running balance. *Exercises relations, transactions, and computing over rows.*

### After Lesson 9 — 🔗 URL Shortener (deployed)
`POST /api/links { url }` → `{ code, shortUrl }`; `GET /:code` redirects (302) and increments a click counter; `GET /api/links/:code/stats`. Auth optional: logged-in users see their own links. Deploy to Render. *Small, complete, satisfying, and used by real people if you share it.*

### After Lesson 12 — 🎬 Movie Watchlist (React + your API)
Search movies via a public API on the client (OMDb needs a free key — or use TVMaze, no key), and save favorites to *your* Express API (a `favorites` resource, per user). React: search with debounce, results grid, "Save" button with optimistic UI, a "My list" page. *Exercises the two-origin setup one last time — CORS, tokens, API client — right before Next.js removes the seams.*

### After Lesson 15 — 📰 Blog with Markdown (Next.js, read-only)
`Post` model seeded with markdown content. `/blog` (server list), `/blog/[slug]` (server render, convert markdown with a library like `marked` — research), `generateMetadata`, `loading.tsx`, a client `ShareButton` island. Deploy to Vercel. *Exercises server components + routing without any mutations yet.*

---

## Capstone projects

Choose one. Build over 3–4 weeks. Each uses: Next.js App Router, Prisma + Postgres, server actions, auth, ownership, validation, deployment.

### 🗂️ Capstone A — Personal Kanban Board
Boards → Columns → Cards (three related models). Pages: `/boards` (list + create), `/boards/[id]` (columns with cards). Server actions: create/rename/delete each level; move a card between columns (an `order` field; update in a transaction). Client islands: inline editing, drag-and-drop (optional; research `@dnd-kit`). Auth: boards belong to users; ownership on every action. Stretch: share a board read-only via a public link (`isPublic` + a `/share/[token]` page).
*Closest to a real SaaS product. Data modeling is the main challenge — draw it first.*

### 🧾 Capstone B — Expense Tracker with Reports
`Expense { amount, category, date, note }` per user; categories as an enum or a table. Pages: `/expenses` (filter by month/category via `searchParams`, server-rendered), `/expenses/new` (action with Zod), `/reports` (server component computing totals by category and by month — Prisma `groupBy`; render with a simple chart library in a client island). CSV export via a route handler (`Content-Type: text/csv`). Budgets with warnings.
*Exercises aggregation, search params, route handlers for non-HTML responses.*

### 📖 Capstone C — Recipe Community
Public and private recipes. `Recipe { title, ingredients (JSON or related table), steps, imageUrl, isPublic }`, `Comment`, `Favorite`. Public pages are static/revalidated (`revalidatePath` on publish); `/my-recipes` is dynamic. Actions: create/edit/publish/comment/favorite with ownership rules (only the author edits; anyone logged-in comments on public recipes). Image upload via a hosted service (research: UploadThing or Vercel Blob). Search via a route handler + client debounce.
*Exercises the static/dynamic distinction, multiple ownership rules, and third-party integration.*

### Capstone deliverables (for all three)

- Deployed URL + GitHub repo with a README (what, how to run, data model diagram, screenshots)
- A 5-minute walkthrough the student gives *you*: the data model, one request traced from click to database and back, and the security checklist applied
- At least a few automated tests (route handlers or pure functions) — the habit matters more than coverage

---

## After this course — what's next

1. **Ship one more thing you invented.** The capstones were guided; the next project should be an idea the student chose and modeled alone. That's the true graduation.
2. **Read code.** Open a well-known open-source Next.js app (e.g. from Vercel's examples or a "T3 stack" template) and trace one feature end to end. Reading unfamiliar code is the skill that compounds.
3. **Deepen the fundamentals under the frameworks:** HTTP caching headers, database indexing and query plans, the event loop, how TLS works. Curiosity about the layer below is what separates seniors from juniors.
4. **Pick specializations by interest:**
   - Frontend depth: accessibility, animation, design systems, testing with Playwright
   - Backend depth: background jobs/queues, WebSockets/real-time, caching (Redis), observability
   - Platform: Docker, CI/CD with GitHub Actions, infrastructure basics
5. **Get feedback from strangers.** Post a project, ask for code review, contribute a small fix to an open-source project. The community is the last teacher.

The student now knows one language across the whole stack, has deployed real applications, and — most importantly — can read documentation and error messages without fear. That's a developer.
