# Part 2 — Backend to Next.js: Teaching Guide (For the Teacher)

This folder continues the course after the frontend curriculum (`../01` – `../18`). It takes the student from "I can build an interactive page that fetches an API" to **"I can build and deploy a full-stack application with Next.js"**.

**Prerequisites:** the student must be solid on Part 1 — especially functions, arrays of objects, `async/await`, `fetch`, JSON, modules (`import`/`export`), and the state → render pattern. If any of these are shaky, review before starting: everything here builds on them directly.

---

## The road map — and why this order

```
Part 1 (done)         Phase A: Backend          Phase B: React        Phase C: Next.js
Browser JS      →     Node.js + Express    →    Components/State  →   Full-stack framework
fetch("...")          the thing fetch talks to  UI from data, v2     React + Node, unified
```

Next.js is React **plus** a Node.js server in one framework. A student who hasn't written a server by hand doesn't understand what Next.js does for them; a student who hasn't written React doesn't understand its component model. So we build both foundations first — deliberately, by hand — then show Next.js as the thing that *unifies* what they already know. This is slower than "jump straight to Next.js" and far more durable.

| Phase | Lessons | Goal |
|-------|---------|------|
| **A: Backend** | 01 – 09 | Node.js, Express, REST APIs, SQL, Prisma, auth, deployment. The student *builds the other side* of Part 1's Lesson 16. |
| **B: React** | 10 – 13 | Component thinking, hooks, data fetching, plus a short TypeScript primer (Next.js projects are TypeScript by default in the industry). |
| **C: Next.js** | 14 – 17 | App Router, server/client components, route handlers, server actions, Prisma, auth, Vercel. |
| **Projects** | 18 – 19 | Capstones and a cheat sheet. |

### Lesson list

1. `01-backend-and-nodejs-setup.md` — What a backend is, installing Node, npm, package.json, running scripts
2. `02-node-modules-fs-and-env.md` — Built-in modules, the file system, JSON storage, environment variables, npm packages
3. `03-http-and-your-first-server.md` — HTTP in depth; a server with the raw `node:http` module (so Express makes sense)
4. `04-express-basics.md` — Routes, params, query, JSON bodies, middleware
5. `05-rest-api-crud-and-errors.md` — REST design, full CRUD, validation, error handling, project structure
6. `06-databases-and-sql.md` — Why databases, SQL fundamentals, SQLite in Node, SQL injection
7. `07-prisma-orm.md` — Schema, migrations, Prisma Client, relations
8. `08-authentication.md` — Password hashing, JWTs, protected routes, ownership
9. `09-api-project-and-deployment.md` — The complete Tasks API, testing, deploying to Render + Postgres
10. `10-react-fundamentals.md` — Vite, JSX, components, props, lists
11. `11-react-state-and-forms.md` — `useState`, events, controlled forms, lifting state
12. `12-react-effects-and-data.md` — `useEffect`, fetching from YOUR API, custom hooks
13. `13-typescript-essentials.md` — Types in one lesson (enough for Next.js)
14. `14-nextjs-routing-and-layouts.md` — App Router, pages, layouts, dynamic routes, loading/error
15. `15-nextjs-server-and-client-components.md` — The core Next.js mental model; data fetching on the server
16. `16-nextjs-data-mutations-and-prisma.md` — Route handlers, server actions, Prisma in Next, full CRUD
17. `17-nextjs-auth-and-deployment.md` — Cookie sessions by hand, protecting pages, Auth.js, Vercel + Neon
18. `18-PROJECTS.md` — Mini-projects and capstones
19. `19-CHEATSHEET.md` — Quick reference

---

## Suggested pace (~2 sessions/week)

| Weeks | Content |
|-------|---------|
| 1 | 01 + 02 |
| 2 | 03 + 04 |
| 3 | 05 (the API lesson — big; may spill into week 4) |
| 4 | 06 |
| 5 | 07 |
| 6 | 08 (auth is conceptually dense — two sessions) |
| 7 | 09 — finish, test, DEPLOY the API. A live URL is a huge milestone. |
| 8 | 10 + 11 |
| 9 | 12 + React mini-project (connect to the deployed API!) |
| 10 | 13 (TypeScript) |
| 11 | 14 |
| 12 | 15 |
| 13 | 16 |
| 14 | 17 — deploy on Vercel |
| 15–17 | Capstone |

As always: pace by understanding, not the calendar.

---

## Teaching tips specific to this part

1. **Keep drawing the diagram.** Browser → HTTP request → server → database → response → browser. Draw it every single session until the student draws it unprompted. Most backend confusion is "where does this code run?" confusion.
2. **"Where does this run?"** is the question to ask constantly, especially in Next.js where server and client code live in the same folder. Make the student answer before writing every file.
3. **Terminal comfort.** The student will now live in a terminal (`cd`, `ls`/`dir`, `node`, `npm`, `npx`). Spend 15 minutes on it in Lesson 1 and don't apologize for it — it's a core professional skill.
4. **Test APIs with a REST client from day one.** Install the **Thunder Client** VS Code extension (or Postman). Seeing raw requests and responses demystifies everything.
5. **Read errors in the terminal** the same way they learned to read errors in the console. Stack traces now include file paths in `node_modules` — teach them to find the topmost line in *their own* file.
6. **Versions drift.** These notes were written in 2026 (Node 22/24 LTS, Express 5, Prisma 6, React 19, Next.js 15+ App Router). Frameworks change; before each lesson, glance at the official docs (nodejs.org, expressjs.com, prisma.io, react.dev, nextjs.org) for renamed commands. Teach the student that checking docs is normal, not a sign the notes are "wrong".
7. **One project threads the whole part:** a **Tasks app** (the to-do list from Part 1, grown up). It becomes: a JSON-file CLI (L2) → an Express API (L5) → backed by SQLite (L6) → Prisma (L7) → with users and auth (L8) → deployed (L9) → with a React frontend (L12) → rebuilt as a Next.js full-stack app (L16–17). Building the *same* thing in each new layer makes the differences crystal clear.
8. **TypeScript:** Lesson 13 is short on purpose. The student should *read* TS confidently and *write* simple annotations. Deep TS is a later course. Next.js projects in this part use TypeScript with light annotations — mirror what the industry does.
9. **Security is not optional.** Hash passwords, parameterize queries, validate on the server, keep secrets in `.env`. Insist on these from the first time they appear; unlearning bad habits is expensive.

---

## Tools to install (Lesson 1 walks through it)

- **Node.js** LTS — nodejs.org (includes npm)
- **VS Code** + extensions: **Thunder Client** (API testing), **Prisma** (schema highlighting), **ES7+ React snippets** (optional)
- **Git** — git-scm.com (the student should start committing their own work now; `git init`, `git add .`, `git commit -m`, `git push` are enough)
- Accounts (free): **GitHub**, **Render** (deploy Express), **Neon** (Postgres), **Vercel** (deploy Next.js)

---

## Exit goals

By the end, the student can independently:

- Build a REST API with Express + Prisma, with validation, auth, and proper errors, and deploy it.
- Build a React app that manages state and talks to an API.
- Build and deploy a full-stack Next.js app with server components, server actions, a database, and login.
- Read Next.js documentation and apply a feature they haven't seen before.
- Explain, with a diagram, exactly what happens between a click in the browser and a row changing in the database.
