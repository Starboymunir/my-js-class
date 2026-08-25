# Lesson 17 — Authentication in Next.js and Deploying to Vercel

> **Goal:** Add real login to the Next.js Tasks app using **cookie sessions built by hand** (so you understand exactly how "logged in" works on the web), protect pages and actions, scope every query to the user — then deploy the whole full-stack app to Vercel with a Neon Postgres database. This is the finish line.

---

## 1. Sessions with cookies — the other half of Lesson 8

Phase A used JWTs in an `Authorization` header because a separate frontend had to attach them manually. In Next.js the frontend and backend share an origin, so we can use the web's native mechanism: **cookies**.

```
Login:   browser ──POST credentials──▶ server verifies password, creates a Session row,
                                       sets cookie  session=<random id>; HttpOnly; Secure
Later:   browser ──GET /tasks────────▶ browser attaches the cookie AUTOMATICALLY;
         (cookie: session=...)          server looks up the session → knows the user
Logout:  server deletes the Session row and clears the cookie
```

Why **HttpOnly** cookies beat `localStorage` tokens: JavaScript in the page cannot read an HttpOnly cookie, so an XSS attack (Part 1 Lesson 11) can't steal it. And because the server keeps a session row, logout is real — delete the row and the cookie is instantly useless (JWTs can't do that without extra machinery).

### Schema

```prisma
model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String?
  password  String                          // bcrypt hash — Lesson 8 rules apply
  tasks     Task[]
  sessions  Session[]
  createdAt DateTime  @default(now())
}

model Session {
  id        String   @id                    // random, unguessable
  userId    Int
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  expiresAt DateTime
  createdAt DateTime @default(now())
}
```

`npx prisma migrate dev --name sessions`. Install `bcryptjs` (+ `@types/bcryptjs`).

---

## 2. The auth library — `src/lib/auth.ts`

All session logic in one file, server-only:

```ts
import "server-only";                                       // build error if ever imported by a client component
import { cookies } from "next/headers";
import { randomBytes } from "node:crypto";
import bcrypt from "bcryptjs";
import { cache } from "react";
import { prisma } from "@/lib/prisma";

const COOKIE = "session";
const SESSION_DAYS = 7;

export async function hashPassword(password: string) {
  return bcrypt.hash(password, 10);
}
export async function verifyPassword(password: string, hash: string) {
  return bcrypt.compare(password, hash);
}

export async function createSession(userId: number) {
  const id = randomBytes(32).toString("hex");               // 256 bits of randomness
  const expiresAt = new Date(Date.now() + SESSION_DAYS * 24 * 60 * 60 * 1000);
  await prisma.session.create({ data: { id, userId, expiresAt } });

  const cookieStore = await cookies();                      // async in Next 15+
  cookieStore.set(COOKIE, id, {
    httpOnly: true,                                         // JS can't read it
    secure: process.env.NODE_ENV === "production",          // HTTPS only in prod
    sameSite: "lax",                                        // CSRF protection
    path: "/",
    expires: expiresAt,
  });
}

export async function destroySession() {
  const cookieStore = await cookies();
  const id = cookieStore.get(COOKIE)?.value;
  if (id) await prisma.session.deleteMany({ where: { id } });
  cookieStore.delete(COOKIE);
}

// Who is making this request? null if nobody. Cached per request so many components can call it cheaply.
export const getCurrentUser = cache(async () => {
  const cookieStore = await cookies();
  const id = cookieStore.get(COOKIE)?.value;
  if (!id) return null;

  const session = await prisma.session.findUnique({
    where: { id },
    include: { user: { select: { id: true, email: true, name: true } } },   // never select the hash
  });
  if (!session || session.expiresAt < new Date()) return null;
  return session.user;
});
```

Every rule from Lesson 8 is here: hashed passwords, unguessable ids, no hash in responses, expiry. Setting cookies is allowed only in **server actions and route handlers** — not during page rendering.

---

## 3. Register, login, logout — as server actions

```ts
// src/app/(auth)/actions.ts
"use server";

import { z } from "zod";
import { redirect } from "next/navigation";
import { prisma } from "@/lib/prisma";
import { hashPassword, verifyPassword, createSession, destroySession } from "@/lib/auth";

const Credentials = z.object({
  email: z.string().trim().toLowerCase().email("Enter a valid email"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

export type AuthState = { error?: string };

export async function register(_prev: AuthState, formData: FormData): Promise<AuthState> {
  const parsed = Credentials.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: parsed.error.issues[0].message };
  const { email, password } = parsed.data;

  if (await prisma.user.findUnique({ where: { email } })) return { error: "Email already registered" };

  const user = await prisma.user.create({ data: { email, password: await hashPassword(password) } });
  await createSession(user.id);
  redirect("/tasks");
}

export async function login(_prev: AuthState, formData: FormData): Promise<AuthState> {
  const parsed = Credentials.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: "Invalid email or password" };
  const { email, password } = parsed.data;

  const user = await prisma.user.findUnique({ where: { email } });
  const ok = user && (await verifyPassword(password, user.password));
  if (!ok) return { error: "Invalid email or password" };            // one message, both cases (L8)

  await createSession(user.id);
  redirect("/tasks");
}

export async function logout() {
  await destroySession();
  redirect("/login");
}
```

`(auth)` is a **route group** (Lesson 14 homework): it groups `/login` and `/register` without appearing in the URL. The login page is a client form with `useActionState` exactly like Lesson 16's `AddTaskForm`:

```tsx
// src/app/(auth)/login/page.tsx
"use client";
import { useActionState } from "react";
import Link from "next/link";
import { login, type AuthState } from "../actions";

export default function LoginPage() {
  const [state, action, pending] = useActionState(login, {} as AuthState);
  return (
    <form action={action} className="flex flex-col gap-2 max-w-sm">
      <h1 className="text-2xl">Log in</h1>
      <input name="email" type="email" placeholder="Email" className="border p-2" required />
      <input name="password" type="password" placeholder="Password" className="border p-2" required />
      <button disabled={pending}>{pending ? "Logging in…" : "Log in"}</button>
      {state.error && <p className="text-red-600">{state.error}</p>}
      <p>No account? <Link href="/register">Register</Link></p>
    </form>
  );
}
```

Add a rate limit on login in production (Lesson 8's reasoning) — Vercel and libraries like `@upstash/ratelimit` make this easy; note it as a follow-up.

---

## 4. Protecting pages and actions ⭐

### Pages — check, then redirect

```tsx
// src/app/tasks/page.tsx
import { redirect } from "next/navigation";
import { getCurrentUser } from "@/lib/auth";
import { prisma } from "@/lib/prisma";

export default async function TasksPage() {
  const user = await getCurrentUser();
  if (!user) redirect("/login");

  const tasks = await prisma.task.findMany({ where: { userId: user.id }, orderBy: { createdAt: "desc" } });
  //                                          ^^^^^^^^^^^^^^^^^ scoped to the user — every query, always
  return (/* ... */);
}
```

Reading the cookie makes this page **dynamic** (rendered per request) automatically — the staleness concern from Lesson 15 disappears for user-specific pages.

### Actions — the same check, because actions are public endpoints

```ts
// src/app/tasks/actions.ts
"use server";
import { getCurrentUser } from "@/lib/auth";

async function requireUser() {
  const user = await getCurrentUser();
  if (!user) throw new Error("Not authenticated");        // or redirect("/login")
  return user;
}

export async function createTask(_prev: ActionState, formData: FormData): Promise<ActionState> {
  const user = await requireUser();
  // ...validate...
  await prisma.task.create({ data: { text, userId: user.id } });   // owner from the SESSION, never the form
  revalidatePath("/tasks");
  return { success: true };
}

export async function deleteTask(id: number) {
  const user = await requireUser();
  await prisma.task.deleteMany({ where: { id, userId: user.id } });   // deleteMany: no throw if not yours — just 0 rows
  revalidatePath("/tasks");
}
```

Lesson 8's ownership rule, verbatim: **user id from the verified session; every `where` includes `userId`.** A page redirect is *UX*; the check inside the action is *security*. Do both.

### The layout — show who's logged in

```tsx
// src/app/layout.tsx (server)
import { getCurrentUser } from "@/lib/auth";
import { logout } from "@/app/(auth)/actions";

export default async function RootLayout({ children }) {
  const user = await getCurrentUser();                       // cached — cheap even if pages call it too
  return (
    <html lang="en"><body>
      <header className="flex gap-4 p-4 border-b">
        <Link href="/">Home</Link>
        {user ? (
          <>
            <Link href="/tasks">Tasks</Link>
            <span className="ml-auto">{user.email}</span>
            <form action={logout}><button>Log out</button></form>
          </>
        ) : (
          <Link href="/login" className="ml-auto">Log in</Link>
        )}
      </header>
      <main className="p-4">{children}</main>
    </body></html>
  );
}
```

### Optional: middleware / proxy for blanket redirects

A `src/middleware.ts` (renamed `proxy.ts` in Next.js 16) can redirect unauthenticated users away from `/tasks/*` before any page renders, by checking whether the session cookie *exists*. It's an optimization and a nicety — it must **not** replace the real checks above (it can't cheaply verify the session against the DB). Read the docs page for the version you're on.

---

## 5. Auth.js, Clerk, and friends — when to use a library

You've built sessions by hand so you understand them. In production, teams often reach for a library that adds OAuth ("Sign in with Google/GitHub"), email magic links, password reset, and battle-tested edge cases:

- **Auth.js** (formerly NextAuth) — open source, integrates with Prisma via an adapter, the classic choice.
- **Clerk**, **Kinde**, **Supabase Auth** — hosted services with prebuilt UI; quickest to ship.
- **Lucia** was a popular guide for exactly the hand-rolled approach you just did — its docs are a great read.

Choosing one is a project decision, not a course requirement. What you built is legitimate, secure when the rules are followed, and makes every library's docs readable — because you know what they're doing underneath.

---

## 6. Deploying to Vercel ⭐

Vercel (the company behind Next.js) deploys Next apps with zero config.

### 1. Database: Neon Postgres

Same as Lesson 9: create a Neon project (or reuse it with a new database/branch). Set `provider = "postgresql"` in `schema.prisma`; if you developed on SQLite, delete `prisma/migrations/` and re-run `npx prisma migrate dev --name init` against Neon. Develop against Postgres from here on.

### 2. Build scripts

```json
"scripts": {
  "dev": "next dev",
  "build": "prisma generate && prisma migrate deploy && next build",
  "start": "next start",
  "postinstall": "prisma generate"
}
```

`prisma generate` must run in the build (Vercel installs fresh); `migrate deploy` applies your committed migrations to production before the build.

### 3. Push and import

- `.gitignore` must contain `.env*` (the template does). Commit `prisma/migrations/`. Push to GitHub.
- **vercel.com** → Add New Project → import the repo. Framework: Next.js (auto-detected).
- **Environment Variables:** `DATABASE_URL` (Neon, production branch). Add any others you use. Nothing `NEXT_PUBLIC_` unless it's truly public.
- Deploy. Watch the build log: install → postinstall generate → migrate deploy → next build → done.

### 4. Verify

Open the URL. Register a user. Add tasks. Log out, log in. Open a second browser (or incognito) — separate session, separate tasks. Check the Neon dashboard: rows are there. **A full-stack app you built from `console.log("Hello")` is live on the internet.** 🎉

Every push to `main` redeploys; every pull request gets a preview URL.

### When it fails

- **Prisma errors on build** → `postinstall` missing, or `DATABASE_URL` not set for the build step (Vercel exposes env vars to builds — check the environment checkboxes).
- **"Dynamic server usage"** warnings → a page uses `cookies()` during static generation; that's expected for auth pages (they become dynamic). If it's an error, ensure you're calling `cookies()` in a server component/action, not at module top level.
- **Secure cookie not set on preview URLs** → previews are HTTPS; fine. Locally, `secure` is off because of the `NODE_ENV` check.
- **Works in dev, 500 in prod** → Vercel → Deployments → Functions → logs. Read the top line of your own stack frame. Same skill as Part 1 Lesson 10.

---

## ⚠️ Common mistakes

```
1. Checking auth only in the page, not in the action → anyone can call the action directly.
2. userId from form data → identity comes from the session. Always.
3. Forgetting userId in a where → IDOR (Lesson 8). Every task query: { ..., userId: user.id }.
4. Setting cookies during render → "Cookies can only be modified in a Server Action or Route Handler".
5. Importing lib/auth.ts into a client component → "server-only" build error (good! that's the guard working).
6. Different error messages for unknown email vs wrong password → enumeration.
7. Not committing prisma/migrations → migrate deploy has nothing to apply → tables missing in prod.
8. Committing .env → rotate DATABASE_URL immediately.
```

---

## ✅ Classwork

1. Add the `Session` model, migrate, write `lib/auth.ts`. In DevTools → Application → Cookies, watch the `session` cookie appear on login (note HttpOnly ✓) and vanish on logout. Try `document.cookie` in the console — it's not there.
2. Build register/login/logout actions and pages. Register twice with one email → error. Wrong password → the generic message.
3. Protect `/tasks` and every task action with `requireUser`; scope all queries. Create two users in two browsers; confirm isolation. Then, from Thunder Client, try to call the delete action's underlying POST for another user's id — observe it does nothing.
4. Update the layout header. Deploy to Vercel with Neon. Register on the live site from your phone.

## 📝 Homework

1. **Profile page:** `/profile` (protected) with a form to update `name` and a change-password form (verify current password first). Log out everywhere: a button that deletes *all* the user's sessions.
2. **Session hygiene:** a scheduled cleanup for expired sessions (a route handler `/api/cron/cleanup` that deletes `expiresAt < now`, protected by a secret header; read Vercel's "Cron Jobs" docs to schedule it). Sliding expiry: extend `expiresAt` on each request if less than half remains.
3. **Middleware/proxy:** add a cookie-existence redirect for `/tasks` and `/profile`. Confirm the action-level checks still exist (delete the middleware temporarily and prove the app is still secure).
4. **Research:** read the Auth.js "Getting started" page and its Prisma adapter docs. Write a paragraph: what would change in your app if you swapped to Auth.js with GitHub login? What stays the same?
5. Written: why HttpOnly cookies over localStorage tokens? Why is a page redirect not a security measure? Draw the full login → request → logout flow with the browser, server, and Session table.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **session cookie** | Random id in an HttpOnly cookie, matched to a Session row |
| **HttpOnly / Secure / SameSite** | Cookie flags: no JS access / HTTPS only / CSRF protection |
| **`cookies()`** | Read/set cookies in server code (async; set only in actions/handlers) |
| **`server-only`** | Import that prevents a file from being used client-side |
| **`getCurrentUser()`** | Cookie → session → user (or null); wrapped in `cache()` |
| **`requireUser()`** | Auth check inside every action/handler — the real security |
| **route group `(auth)`** | Organize routes without affecting URLs |
| **Auth.js / Clerk** | Libraries for OAuth, magic links, hosted auth |
| **Vercel** | Zero-config Next.js hosting; deploys from GitHub |
| **`postinstall` / `migrate deploy`** | Generate Prisma Client and apply migrations in the build |
