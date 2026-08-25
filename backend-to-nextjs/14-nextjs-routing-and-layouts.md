# Lesson 14 — Next.js: The App Router, Pages, and Layouts

> **Goal:** Begin Phase C — the destination. Understand what Next.js adds to React, create a project, and master file-based routing: pages, layouts, dynamic routes, navigation, loading and error states, and metadata. Everything from Phases A and B is about to click together.

---

## 1. What Next.js is

React (Phase B) gives you components. It does **not** give you: routing between pages, a server, data fetching conventions, SEO-friendly HTML, image optimization, or a deployment story. Every React app needs those, so teams kept assembling them by hand. **Next.js** is React with all of that built in — and, crucially, **it runs on Node.js**, so your components can talk to the database directly.

```
Phase A: Express API  ─┐
                       ├──▶  Next.js = React frontend + Node backend, one project, one deploy
Phase B: React UI     ─┘
```

What that buys you:

| Problem | Vite + Express (what you built) | Next.js |
|---|---|---|
| Two projects, two deploys, CORS | ✅ you lived it | One project, same origin, no CORS |
| Routing | Would need react-router | Folders = routes |
| First page load | Empty HTML, then JS fetches data (slow, bad SEO) | HTML rendered **on the server** with data already in it |
| Data fetching | `useEffect` + loading states | `await` directly in components (server components) |
| Forms/mutations | fetch → API route → JSON | Server actions: call a server function from a form |
| Deploy | Render + Vercel/Netlify, env vars in two places | Vercel, one click |

You were made to build the two-project version first so you'd know *exactly* what Next.js abstracts. When something breaks, you'll know which layer it's in.

> **Version note:** this lesson uses the **App Router** (the `app/` directory), standard since Next.js 13 and the only one to learn today. Older tutorials show the `pages/` directory ("Pages Router") — different APIs; skip them. Written against Next.js 15/16; check nextjs.org for renamed details.

---

## 2. Create the project

```bash
npx create-next-app@latest tasks-next
```

Answer the prompts: **TypeScript: Yes** · ESLint: Yes · **Tailwind CSS: Yes** (utility classes, optional but the default) · **`src/` directory: Yes** · **App Router: Yes** · import alias: default (`@/*`).

```bash
cd tasks-next
npm run dev          # http://localhost:3000
```

The structure that matters:

```
tasks-next/
├── src/app/
│   ├── layout.tsx       ← root layout: <html>, <body>, shared UI
│   ├── page.tsx         ← the "/" page
│   ├── globals.css
│   └── favicon.ico
├── public/              ← static files (served at /)
├── next.config.ts
├── package.json         ← scripts: dev, build, start
└── tsconfig.json
```

Replace `src/app/page.tsx`:

```tsx
export default function HomePage() {
  return <h1 className="text-3xl font-bold">Tasks</h1>;
}
```

A page is a React component, default-exported from a file named `page.tsx`. (`className="text-3xl font-bold"` is Tailwind — CSS utility classes. Use plain CSS if you prefer; not the point of this course.)

---

## 3. File-based routing ⭐

**Folders define URLs. `page.tsx` makes a folder a page.**

```
src/app/page.tsx                 →  /
src/app/about/page.tsx           →  /about
src/app/tasks/page.tsx           →  /tasks
src/app/tasks/new/page.tsx       →  /tasks/new
src/app/tasks/[id]/page.tsx      →  /tasks/1, /tasks/42, ...   (dynamic segment)
src/app/api/tasks/route.ts       →  /api/tasks                  (API endpoint — Lesson 16)
```

No route config, no `app.get(...)`. Express's `router.get("/tasks/:id", ...)` became a folder named `[id]`.

### Dynamic routes and `params`

```tsx
// src/app/tasks/[id]/page.tsx
type Props = { params: Promise<{ id: string }> };

export default async function TaskPage({ params }: Props) {
  const { id } = await params;                  // params is a Promise in Next 15+ — await it
  return <h1>Task #{id}</h1>;
}
```

Like `req.params.id`, it's a **string**. Visit `/tasks/7` — there it is. (Pages can be `async` — Lesson 15 explains why that's a superpower.)

### Search params

```tsx
// /tasks?filter=done
type Props = { searchParams: Promise<{ filter?: string }> };

export default async function TasksPage({ searchParams }: Props) {
  const { filter = "all" } = await searchParams;
  return <p>Showing: {filter}</p>;
}
```

`req.query`, reborn.

---

## 4. Layouts — shared UI that doesn't re-render ⭐

A `layout.tsx` wraps every page in its folder *and all subfolders*. The root layout is required and holds `<html>` and `<body>`:

```tsx
// src/app/layout.tsx
import type { Metadata } from "next";
import Link from "next/link";
import "./globals.css";

export const metadata: Metadata = {
  title: "Tasks",
  description: "A full-stack tasks app built with Next.js",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <header className="flex gap-4 p-4 border-b">
          <Link href="/">Home</Link>
          <Link href="/tasks">Tasks</Link>
          <Link href="/about">About</Link>
        </header>
        <main className="p-4">{children}</main>
      </body>
    </html>
  );
}
```

`children` is the current page (or a nested layout). Navigating between pages swaps `children` while the header stays mounted — no flicker, state in the layout preserved.

Nested layouts compose:

```
src/app/layout.tsx                → wraps everything
src/app/tasks/layout.tsx          → wraps /tasks and /tasks/*  (e.g., a sidebar of projects)
src/app/tasks/[id]/page.tsx       → rendered inside BOTH
```

### Metadata

`export const metadata` in a layout or page sets `<title>`, description, Open Graph tags — the SEO bits. Dynamic version for `[id]` pages:

```tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { id } = await params;
  return { title: `Task #${id} · Tasks` };
}
```

---

## 5. Navigation

### `<Link>` — client-side navigation

```tsx
import Link from "next/link";
<Link href="/tasks">All tasks</Link>
<Link href={`/tasks/${task.id}`}>{task.text}</Link>
```

Renders an `<a>`, but clicking it doesn't reload the page — Next fetches just the new page's content and swaps it in (and prefetches links in view, so it feels instant). Always use `Link` for internal links; plain `<a>` for external.

### Programmatic navigation — in client components

```tsx
"use client";
import { useRouter } from "next/navigation";      // NOT "next/router" (that's the old Pages Router)

function SaveButton() {
  const router = useRouter();
  return <button onClick={() => router.push("/tasks")}>Done</button>;
}
```

`"use client"` at the top of the file — that marker is the subject of the next lesson. Short version: hooks and event handlers need it.

### Redirects and 404s — in server code

```tsx
import { redirect, notFound } from "next/navigation";

if (!user) redirect("/login");        // throws internally — no return needed
if (!task) notFound();                // renders the nearest not-found.tsx with a 404 status
```

---

## 6. Loading, error, and not-found files

Special files next to `page.tsx` handle the three UI states automatically:

```
src/app/tasks/
├── page.tsx           ← the page (can be async and slow)
├── loading.tsx        ← shown INSTANTLY while page.tsx is loading
├── error.tsx          ← shown if page.tsx throws
└── not-found.tsx      ← shown when notFound() is called
```

```tsx
// loading.tsx
export default function Loading() {
  return <p className="animate-pulse">Loading tasks…</p>;
}
```

```tsx
// error.tsx — must be a client component
"use client";

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <p>Something went wrong: {error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

```tsx
// not-found.tsx
export default function NotFound() {
  return <p>That task doesn't exist.</p>;
}
```

Part 1 Lesson 16's loading/success/error states, Part 2 Lesson 12's `status` variable — now they're *files*, wired up by convention. Try it: `await new Promise(r => setTimeout(r, 2000))` inside an async page and watch `loading.tsx` appear.

---

## 7. Styling and assets — the short version

- **Global CSS**: `globals.css`, imported in the root layout.
- **Tailwind**: utility classes in `className` (`p-4`, `flex`, `text-red-500`). Fast to write, no naming.
- **CSS Modules**: `TaskItem.module.css` → `import styles from "./TaskItem.module.css"` → `className={styles.item}`. Scoped, no collisions.
- **Images**: `import Image from "next/image"` — `<Image src="/logo.png" width={120} height={40} alt="Logo" />` — optimized, lazy, sized. Files in `public/` are served at `/`.
- **Fonts**: `next/font` — the template shows it in `layout.tsx`.

---

## ⚠️ Common mistakes

```
1. Importing from "next/router" → Pages Router. App Router uses "next/navigation".
2. Forgetting `await params` → "params is a Promise" errors (Next 15+).
3. A folder without page.tsx → no route (404). Folders alone are just structure.
4. Using hooks (useState, useRouter) in a file without "use client" → error. (Lesson 15!)
5. error.tsx without "use client" → build error. It must be a client component.
6. <a href="/tasks"> for internal links → full reload; use <Link>.
7. Putting <html>/<body> in a nested layout → only the ROOT layout has them.
8. Component files inside app/ named page.tsx by accident → they become routes. Keep components in src/components/.
```

---

## ✅ Classwork

1. Create the project. Build pages: `/`, `/about`, `/tasks`, `/tasks/new`, `/tasks/[id]` (showing the id). Navigate between them with `Link`s in the root layout. Open DevTools → Network while clicking links: notice no full reloads.
2. Add a `tasks/layout.tsx` with a sidebar (a static list of project names) — confirm it wraps `/tasks` and `/tasks/5` but not `/about`.
3. Add `loading.tsx` to `/tasks` and a 2-second artificial delay in the page. Then throw an error from the page and add `error.tsx` with a reset button. Then call `notFound()` for ids > 100 and add `not-found.tsx`.
4. Set `metadata` in the layout and `generateMetadata` in `[id]`. Check the browser tab titles.
5. Read `?filter=` from `searchParams` on `/tasks` and render three `Link`s that set it (`/tasks?filter=done`, etc.), highlighting the active one.

## 📝 Homework

1. **Portfolio site:** `/` (hero), `/projects` (grid from a static array), `/projects/[slug]` (details; `notFound()` for unknown slugs), `/contact`. Shared header/footer in the root layout, `Image` for project thumbnails, metadata on every page. Deploy it (Lesson 17 shows Vercel — or try it early: push to GitHub, import on vercel.com; static pages need no env vars).
2. **Nested layouts:** a `/dashboard` section with its own layout (sidebar + `/dashboard`, `/dashboard/settings`, `/dashboard/stats`). The sidebar highlights the active link using `usePathname()` from `next/navigation` (a client component — add `"use client"`; we explain why next lesson).
3. **Research:** read the Next.js docs page "Routing → Route Groups" `(folder)` and "Parallel Routes" briefly. Write two sentences on what route groups are for. Use one to separate `(marketing)` and `(app)` layouts in your portfolio.
4. Written: list five things Next.js provides that your Vite + Express setup didn't. Map Express concepts to Next.js files: `router.get("/tasks/:id")` → ?, `req.query` → ?, `res.status(404)` → ?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **Next.js** | React + Node server + routing + tooling in one framework |
| **App Router** | The `app/` directory routing system (the one to learn) |
| **`page.tsx`** | Makes a folder a route; default-exports the page component |
| **`layout.tsx`** | Shared wrapper for a folder's pages; root layout holds `<html>/<body>` |
| **`[id]`** | Dynamic segment; read via `await params` |
| **`searchParams`** | Query string (awaited) |
| **`<Link>`** | Client-side navigation without reloads |
| **`next/navigation`** | `useRouter`, `usePathname`, `redirect`, `notFound` |
| **`loading.tsx` / `error.tsx` / `not-found.tsx`** | Automatic UI states by convention |
| **`metadata`** | Title/description/SEO per page |
