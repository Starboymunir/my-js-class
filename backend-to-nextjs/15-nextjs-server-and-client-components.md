# Lesson 15 — Server Components vs Client Components ⭐

> **Goal:** Master the single most important Next.js concept: some components run **on the server**, some **in the browser**, and you choose per file. Get this right and Next.js is simple; get it wrong and every error is a mystery. Also today: fetching data on the server, straight from the database, with no `useEffect`.

---

## 1. The question you must always answer: *where does this run?*

Remember the diagram — browser box, server box. In Phase B, all React ran in the browser box. In Next.js, **React components can run in either box**, and the default is the server.

| | **Server Component** (default) | **Client Component** (`"use client"`) |
|---|---|---|
| Runs | On the Node server, at request time (or build time) | In the browser (and once on the server for the initial HTML) |
| Can | `await` data, read the database, use `process.env` secrets, read files, call Prisma | Use `useState`, `useEffect`, event handlers, browser APIs (`localStorage`, `window`) |
| Cannot | Use hooks or `onClick`/`onChange` — there's no browser here | Access the DB or secrets — it's a browser |
| Ships JS to browser? | **No** — only the rendered HTML | Yes — the component's code is sent to run there |
| Think of it as | Your Express controller that returns HTML instead of JSON | Your Phase B React component |

Mental model: a server component is **a function that runs on the server and produces HTML**. It can do anything Express could — it *is* the backend. A client component is **a Phase B React component** — interactive, stateful, in the browser.

The rule of thumb: **server by default; add `"use client"` only where you need interactivity.**

---

## 2. Server components: fetch data by just awaiting ⭐

Here's the payoff of Phase A + B. In Phase B, loading tasks was: `useState` ×3, `useEffect`, `fetch`, CORS, an API route in another project, JSON, loading/error branches. In a server component:

```tsx
// src/app/tasks/page.tsx — a SERVER component (no "use client")
import { prisma } from "@/lib/prisma";

export default async function TasksPage() {
  const tasks = await prisma.task.findMany({ orderBy: { createdAt: "desc" } });   // yes. Directly.

  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>{task.text}</li>
      ))}
    </ul>
  );
}
```

That's it. The component is `async`, awaits the database (Lesson 7's Prisma, unchanged), and returns JSX. Next runs it on the server, renders HTML with the data already in it, and sends that HTML to the browser. **Zero JavaScript for this component reaches the client.** No loading state needed here — `loading.tsx` from Lesson 14 covers the wait automatically.

Where's the API? *There isn't one* — you don't need HTTP to talk to yourself. The server component **is** the controller + view. (You'll still write API routes when *other* programs need your data — Lesson 16.)

Set up Prisma in the Next project (same as Lesson 7, one twist):

```ts
// src/lib/prisma.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;   // survive hot-reload in dev
```

(Dev mode reloads modules constantly; without this, you'd open a new DB connection per reload and exhaust them. Standard boilerplate — copy it.)

### Secrets are safe in server components

```tsx
const res = await fetch("https://api.weather.com/...", { headers: { "x-api-key": process.env.WEATHER_KEY! } });
```

That key never reaches the browser, because this code never runs there. In Phase B this needed a backend proxy route. Here it's just... a component. **Only `NEXT_PUBLIC_`-prefixed env vars are exposed to the browser** — same rule as Vite's `VITE_`.

---

## 3. Client components: where interactivity lives

Anything with `useState`, `useEffect`, `onClick`, `onChange`, browser APIs → `"use client"` on the **first line** of the file:

```tsx
// src/components/Counter.tsx
"use client";

import { useState } from "react";

export function Counter() {
  const [n, setN] = useState(0);
  return <button onClick={() => setN(n + 1)}>Clicked {n} times</button>;
}
```

`"use client"` marks the **boundary**: this file and everything it imports run in the browser. Use it from a server page like any component:

```tsx
// src/app/page.tsx (server)
import { Counter } from "@/components/Counter";

export default function Home() {
  return (
    <div>
      <h1>Server-rendered heading</h1>
      <Counter />                       {/* an island of interactivity inside a server page */}
    </div>
  );
}
```

The page is server-rendered HTML; `Counter` is shipped as JS and "hydrated" (React attaches its event handlers in the browser). This is the **islands** pattern: mostly static HTML with interactive islands — fast to load, interactive where needed.

---

## 4. Composing the two — the rules ⭐

1. **Server components can import and render client components.** ✅ (above)
2. **Client components cannot import server components.** ❌ A browser can't run database code.
3. **But** client components can receive server components as **`children`** (or any prop). The server renders them first and hands the result over:

```tsx
// src/components/Collapsible.tsx — client: manages open/closed
"use client";
import { useState } from "react";

export function Collapsible({ title, children }: { title: string; children: React.ReactNode }) {
  const [open, setOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setOpen(!open)}>{title}</button>
      {open && children}
    </div>
  );
}
```

```tsx
// src/app/tasks/page.tsx — server
export default async function TasksPage() {
  const tasks = await prisma.task.findMany();
  return (
    <Collapsible title="All tasks">
      <TaskTable tasks={tasks} />          {/* server component passed as children — allowed */}
    </Collapsible>
  );
}
```

4. **Props from server to client must be serializable** — plain data: strings, numbers, booleans, arrays, plain objects, `Date`s. **Not functions**, not class instances. (You can't pass `onClick={() => prisma...}` across the boundary; that's what server actions are for — Lesson 16.)

5. **Push `"use client"` as far down the tree as possible.** Don't mark the page as client because one button needs state; extract the button into its own client file. Smaller islands = less JS = faster site.

### Decision checklist per file

- Needs `useState`/`useEffect`/hooks? → client
- Has `onClick`/`onChange`/etc.? → client
- Uses `window`, `localStorage`, `document`? → client
- Otherwise → **server** (and if it needs data, make it `async` and await)

---

## 5. Data flow: server → client

Pattern you'll use constantly — server fetches, client interacts:

```tsx
// src/app/tasks/page.tsx — server
import { prisma } from "@/lib/prisma";
import { TaskList } from "@/components/TaskList";

export default async function TasksPage() {
  const tasks = await prisma.task.findMany({ orderBy: { createdAt: "desc" } });
  return <TaskList initialTasks={tasks} />;         // data crosses the boundary as a prop
}
```

```tsx
// src/components/TaskList.tsx — client
"use client";
import { useState } from "react";
import type { Task } from "@prisma/client";        // Prisma generates TS types for your models! (L13)

export function TaskList({ initialTasks }: { initialTasks: Task[] }) {
  const [filter, setFilter] = useState<"all" | "active" | "done">("all");
  const visible = initialTasks.filter((t) => filter === "all" || (filter === "done") === t.done);

  return (
    <>
      <div>
        {(["all", "active", "done"] as const).map((f) => (
          <button key={f} onClick={() => setFilter(f)} className={f === filter ? "font-bold" : ""}>{f}</button>
        ))}
      </div>
      <ul>{visible.map((t) => <li key={t.id}>{t.text}</li>)}</ul>
    </>
  );
}
```

Server gets the data (no API, no CORS, no loading state); client owns the UI state (filter). Each box does what it's good at.

---

## 6. Static vs dynamic rendering (just enough)

Next decides *when* to run a server component:

- **Static** — at build time, once; served as cached HTML. Blazing fast. Default when the page uses no request-specific data.
- **Dynamic** — on every request. Automatic when you use `cookies()`, `headers()`, `searchParams`, or opt in with `export const dynamic = "force-dynamic"`.

A page reading tasks from the DB with no user-specific data might get statically rendered *at build time* — and then show stale data! Two fixes: make it dynamic (`export const dynamic = "force-dynamic"`), or — better — **revalidate** after mutations (`revalidatePath("/tasks")`, Lesson 16) so the cache refreshes exactly when data changes. Once the app has login (Lesson 17), reading the session cookie makes user pages dynamic automatically.

*(Caching in Next.js has evolved across versions; the docs' "Caching" page is the reference. The instinct to keep: when data looks stale, ask "is this page static, and did I revalidate?")*

---

## 7. Fetching from external APIs on the server

`fetch` works in server components too — Part 1 Lesson 16's country lookup, without the client-side state dance:

```tsx
export default async function CountryPage({ params }: { params: Promise<{ name: string }> }) {
  const { name } = await params;
  const res = await fetch(`https://restcountries.com/v3.1/name/${name}`, { next: { revalidate: 3600 } });   // cache 1h
  if (!res.ok) notFound();
  const [country] = await res.json();
  return <h1>{country.name.common} — {country.capital?.[0]}</h1>;
}
```

---

## ⚠️ Common mistakes

```
1. "useState only works in Client Components" → add "use client" to THAT file (not the page).
2. "You're importing a component that needs X. It only works in a Server Component" → you imported
   Prisma/fs/server code into a client file. Move the data fetching up to a server component, pass props.
3. Passing a function as a prop from server to client → "Functions cannot be passed directly to Client
   Components". Use a server action (Lesson 16) or move the handler into the client.
4. Marking the whole page "use client" to fix one hook → now you can't await the DB there. Extract an island.
5. Reading process.env.SECRET in a client component → undefined (and would be a leak if it worked).
   Only NEXT_PUBLIC_* reaches the browser.
6. Page shows stale data after changes → static rendering without revalidation (section 6).
7. Missing the Prisma singleton → "too many connections" in dev.
8. Forgetting `async` on a server component that awaits → syntax error on await.
```

---

## ✅ Classwork

1. Set up Prisma in `tasks-next` (copy `schema.prisma` from Phase A — User and Task; `npx prisma migrate dev`; the singleton in `src/lib/prisma.ts`; seed a few tasks). Render them in `/tasks` from a server component. View page source (Ctrl+U): the task text is **in the HTML**. Compare with the Vite app's empty `<div id="root">`.
2. Build `Counter` as a client component; render it inside the server-rendered `/tasks` page. Then try importing `prisma` inside `Counter` — read the error, understand it, undo.
3. Build `Collapsible` (client) wrapping a server-rendered task table. Confirm it works. Then try passing `onSomething={() => {}}` from the page to `Collapsible` — read that error too.
4. Build the server → client `TaskList` with filter buttons (section 5). Then a `/tasks/[id]` page that awaits `prisma.task.findUnique` and calls `notFound()` when missing.
5. Add `console.log("rendering TasksPage")` in the server page and `console.log("rendering TaskList")` in the client one. Refresh. Which log appears in the **terminal**, which in the **browser console**? (Client components also log once on the server — the initial render. Why?)

## 📝 Homework

1. **User pages:** `/users` (server: list users with task counts via Prisma `_count`) → `/users/[id]` (server: the user and their tasks with `include`). Add `generateMetadata` using the user's name.
2. **Islands audit:** take your Phase B to-do app's components and sort them into server vs client. Write the list with one-line reasons. Which ones lose `useEffect` entirely in Next.js?
3. **External API on the server:** rebuild Part 1's "random dog gallery" as a server page (`/dogs`) — no `useEffect`, no loading state in the component (use `loading.tsx`). Add a client `RefreshButton` that calls `router.refresh()` from `useRouter`.
4. **Research:** read the docs page "Rendering → Server and Client Components" (the composition patterns section). Write down the "children" pattern in your own words and one situation where you'd need it.
5. Written: complete the sentence three ways — "A server component can ___ but a client component cannot" and vice versa. Why does keeping `"use client"` low in the tree make sites faster?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **server component** | Default; runs on the server; can `await` DB/secrets; ships no JS |
| **client component** | `"use client"`; runs in the browser; hooks, events, browser APIs |
| **boundary** | The `"use client"` file — everything it imports becomes client code |
| **island** | An interactive client component inside server-rendered HTML |
| **hydration** | React attaching interactivity to server-rendered HTML in the browser |
| **serializable props** | Plain data only across the server → client boundary |
| **`children` pattern** | Client wrapper receiving server-rendered content |
| **static / dynamic rendering** | Build-time cached vs per-request |
| **`revalidatePath`** | Refresh cached pages after a change (Lesson 16) |
| **`NEXT_PUBLIC_`** | The only env vars exposed to the browser |
| **Prisma singleton** | `globalThis` trick to reuse one client in dev |
