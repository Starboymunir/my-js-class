# Lesson 09 — The Complete Tasks API: Testing and Deployment

> **Goal:** Finish Phase A with a complete, tested, *deployed* API. Automated tests with Node's built-in runner, a production database (PostgreSQL on Neon), and a live URL on Render. At the end of this lesson your backend exists on the internet — a real milestone.

---

## 1. Where the project stands

After Lessons 4–8, `tasks-api` should look like this:

```
tasks-api/
├── server.js                    # listen
├── app.js                       # middleware, routers, 404, error handler
├── routes/      auth.routes.js  tasks.routes.js  projects.routes.js
├── controllers/ auth.controller.js  tasks.controller.js  ...
├── services/    auth.service.js  tasks.service.js  ...
├── middleware/  requireAuth.js  errorHandler.js  validate.js
├── utils/       HttpError.js
├── db/          prisma.js
├── prisma/      schema.prisma  migrations/  seed.js
├── tests/                       # today
├── .env  .env.example  .gitignore  package.json  README.md
```

Endpoints:

```
POST   /api/auth/register        POST   /api/auth/login        GET  /api/auth/me
GET    /api/tasks                POST   /api/tasks
GET    /api/tasks/:id            PATCH  /api/tasks/:id         DELETE /api/tasks/:id
GET    /api/projects ... (CRUD)  GET    /api/projects/:id/tasks
GET    /api/health               → { ok: true }   (add this — hosts use it to check you're alive)
```

Before going further, do a **review pass** together: every route validated? Every task query scoped by `userId`? Error handler last? Secrets in `.env`? `.gitignore` correct? `README.md` explaining how to run it? This checklist is what a senior dev would look for in a code review.

---

## 2. Automated tests

Thunder Client collections are manual. Automated tests run in seconds, every time, and catch what you broke while adding features. Node has a built-in runner; `supertest` lets us call the Express app without opening a port.

```bash
npm install --save-dev supertest
```

```json
"scripts": {
  "test": "node --test --env-file=.env.test"
}
```

Create `.env.test` pointing at a **separate database** (`DATABASE_URL="file:./test.db"`) so tests never touch real data, and run the migrations against it once: `npx dotenv -e .env.test -- prisma migrate deploy` — or simply `DATABASE_URL="file:./test.db" npx prisma migrate deploy` (PowerShell: `$env:DATABASE_URL="file:./test.db"; npx prisma migrate deploy`).

```js
// tests/tasks.test.js
import { test, before, beforeEach } from "node:test";
import assert from "node:assert/strict";
import request from "supertest";
import app from "../app.js";
import { prisma } from "../db/prisma.js";

let token;

before(async () => {
  await prisma.task.deleteMany();
  await prisma.user.deleteMany();
  await request(app).post("/api/auth/register").send({ email: "t@test.com", password: "password123" });
  const res = await request(app).post("/api/auth/login").send({ email: "t@test.com", password: "password123" });
  token = res.body.token;
});

beforeEach(async () => {
  await prisma.task.deleteMany();
});

test("GET /api/tasks requires auth", async () => {
  const res = await request(app).get("/api/tasks");
  assert.equal(res.status, 401);
});

test("POST /api/tasks creates a task", async () => {
  const res = await request(app)
    .post("/api/tasks")
    .set("Authorization", `Bearer ${token}`)
    .send({ text: "Write tests" });

  assert.equal(res.status, 201);
  assert.equal(res.body.text, "Write tests");
  assert.equal(res.body.done, false);
});

test("POST /api/tasks rejects empty text", async () => {
  const res = await request(app)
    .post("/api/tasks")
    .set("Authorization", `Bearer ${token}`)
    .send({ text: "   " });
  assert.equal(res.status, 400);
});

test("users cannot see each other's tasks", async () => {
  await request(app).post("/api/tasks").set("Authorization", `Bearer ${token}`).send({ text: "Mine" });

  await request(app).post("/api/auth/register").send({ email: "other@test.com", password: "password123" });
  const login = await request(app).post("/api/auth/login").send({ email: "other@test.com", password: "password123" });
  const res = await request(app).get("/api/tasks").set("Authorization", `Bearer ${login.body.token}`);

  assert.equal(res.status, 200);
  assert.deepEqual(res.body, []);
});
```

Run `npm test`. Each `test` is: **arrange** (set up data), **act** (make the request), **assert** (check status and body). Write the failure-path tests too — the auth and ownership tests above are the ones that catch security regressions.

You don't need 100% coverage. Aim for: every endpoint's happy path, its main validation failure, and every auth/ownership rule. That's a professional baseline.

---

## 3. Preparing for production

Code that runs on your laptop needs a few adjustments to run on a server you don't control:

### PostgreSQL instead of SQLite

SQLite is a file on disk; most hosting platforms have ephemeral disks (files vanish on redeploy). Production uses a database *server*. **Neon** offers free hosted PostgreSQL.

1. Sign up at **neon.tech**, create a project, copy the connection string (`postgresql://user:pass@host/db?sslmode=require`).
2. In `schema.prisma`: `provider = "postgresql"`.
3. Migrations generated for SQLite aren't valid for Postgres. In dev, **delete `prisma/migrations/`**, set `DATABASE_URL` in `.env` to the Neon string, and run `npx prisma migrate dev --name init` to regenerate against Postgres. From here on, develop against Postgres too (Neon lets you create a separate *branch* for dev — do that, so dev data and prod data are separate).
4. Run the app locally against Neon. Full test pass. (Postgres has real booleans, and Prisma handles the rest — your code shouldn't change.)

### Production settings

```js
// server.js
const PORT = process.env.PORT ?? 3000;      // hosts assign a port via env — never hard-code
app.listen(PORT, () => console.log(`Listening on ${PORT}`));
```

```json
"scripts": {
  "start": "node server.js",
  "dev": "node --watch --env-file=.env server.js",
  "build": "prisma generate && prisma migrate deploy",
  "test": "node --test --env-file=.env.test"
}
```

- `start` must run **without** `--env-file` — hosts inject env vars directly.
- `prisma migrate deploy` applies committed migrations without prompting — the production counterpart of `migrate dev`.
- `cors({ origin: process.env.CLIENT_ORIGIN })` — set to your frontend's URL in production (Lesson 12 / 17).
- Error handler: log with `console.error` (hosts capture stdout), never leak stacks.

### Commit everything (except secrets)

`git status` — `.env` must not appear. `prisma/migrations/` must. Push to GitHub.

---

## 4. Deploying to Render

**Render** deploys straight from a GitHub repo, free tier included (it sleeps when idle — the first request after a while is slow; that's normal for free hosting).

1. **render.com** → New → **Web Service** → connect your GitHub repo.
2. Settings:
   - Runtime: Node
   - Build command: `npm install && npm run build`
   - Start command: `npm start`
3. **Environment variables** (the `.env` equivalent — type them in):
   - `DATABASE_URL` = your Neon *production* connection string
   - `JWT_SECRET` = a fresh long random value (different from dev!)
   - `CLIENT_ORIGIN` = your frontend's URL (or `*` temporarily while testing)
   - `NODE_ENV` = `production`
4. Deploy. Watch the logs: install → prisma generate → migrations → "Listening on 10000".
5. Visit `https://your-app.onrender.com/api/health` → `{ "ok": true }`. 🎉

Then the real test: Thunder Client against the live URL — register, login, create tasks. Your API is on the internet, backed by a real database, with authentication. Every push to `main` redeploys automatically.

### When deployment fails (it will, at least once)

- **Build fails on `prisma generate`** → make sure `prisma` is in `devDependencies` *and* Render installs dev deps (it does by default), or move it to `dependencies`.
- **"Can't reach database server"** → wrong `DATABASE_URL`, or missing `?sslmode=require`.
- **App starts then crashes** → read the logs top to bottom. Nine times out of ten it's a missing env var — your `config.js` from Lesson 2 homework that throws on missing config earns its keep here.
- **Works locally, 500 in prod** → check the Render logs for the real error; `NODE_ENV` differences; a migration that wasn't committed.

Reading deployment logs is Lesson 10 of Part 1 with a new font.

---

## 5. Documenting the API

A `README.md` with: what it is, how to run locally (`npm install`, `.env` from `.env.example`, `npx prisma migrate dev`, `npm run seed`, `npm run dev`), how to test, and an endpoint table with example requests/responses. Writing docs forces you to see the API from a user's side — and that user is you in three months.

*(Nice-to-know: OpenAPI/Swagger is the industry standard for machine-readable API docs. Not needed now.)*

---

## ✅ Classwork

1. Review-pass checklist together (section 1). Fix what's found.
2. Set up the test script and `.env.test`. Write the four tests above; run them; break the ownership rule on purpose (remove `userId` from one `where`) and watch the test catch it. Restore it.
3. Create the Neon project, switch the provider, regenerate migrations, run locally against Postgres. Tests pass.
4. Deploy to Render together. Hit `/api/health` on the live URL. Register and log in against production from Thunder Client.

## 📝 Homework

1. **Tests:** add tests for `PATCH` (update text, toggle done, 404 for other user's id), `DELETE` (204 then 404), login failures (wrong password → 401, unknown email → 401 with the same message), and register duplicate → 409.
2. **README** with the endpoint table and setup steps. A classmate should be able to clone and run it from the README alone.
3. **Frontend on the live API:** point your `public/` to-do page at the Render URL (set `CLIENT_ORIGIN` accordingly, or serve `public/` from the same app). Use it from your phone. Show someone.
4. **Reflection (written):** list every layer a request passes through from your phone to the Neon database and back. Name the lesson each layer came from.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **`node --test`** | Node's built-in test runner |
| **supertest** | Call an Express app in tests without a network port |
| **arrange / act / assert** | The three parts of every test |
| **`.env.test`** | Separate config so tests use a separate database |
| **PostgreSQL / Neon** | Production database / free hosted Postgres |
| **`prisma migrate deploy`** | Apply committed migrations in production |
| **Render** | Hosting that deploys from GitHub |
| **environment variables (host)** | Where production secrets live — never in the repo |
| **health check** | `GET /api/health` → `{ ok: true }` for uptime monitoring |
| **README** | How to run, test, and use the project |
