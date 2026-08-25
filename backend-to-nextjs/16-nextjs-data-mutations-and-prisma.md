# Lesson 16 — Mutations: Server Actions, Route Handlers, and Full CRUD

> **Goal:** Change data from Next.js. Two tools: **server actions** (call a server function straight from a form or button — the Next.js way to create/update/delete) and **route handlers** (Express-style API endpoints for when other programs need JSON). Rebuild the Tasks app end-to-end inside Next.js, with validation, error display, and pending states.

---

## 1. Two ways to mutate — when to use which

| | Server Actions | Route Handlers (`route.ts`) |
|---|---|---|
| What | An `async` function marked `"use server"` that you call from a component; Next handles the HTTP for you | A function exporting `GET`/`POST`/… that receives a `Request` and returns a `Response` — an Express route, Next-flavored |
| Call from | `<form action={fn}>`, `onClick={() => fn(id)}` | `fetch("/api/tasks")` — from your app, a mobile app, another server, a webhook |
| Returns | Any serializable value to the caller | JSON/text/anything over HTTP |
| Use for | **Your own UI's** create/update/delete | Public/external APIs, webhooks, non-React clients |

**Default to server actions for your UI.** Write route handlers when someone *outside your React tree* needs an endpoint.

---

## 2. Server actions ⭐

A server action is a function that runs on the server but can be *called* from the client — Next turns the call into a POST request behind the scenes. Define them in a file marked `"use server"`:

```ts
// src/app/tasks/actions.ts
"use server";

import { prisma } from "@/lib/prisma";
import { revalidatePath } from "next/cache";

export async function createTask(formData: FormData) {
  const text = String(formData.get("text") ?? "").trim();
  if (!text) return;                                   // (better validation below)

  await prisma.task.create({ data: { text, userId: 1 } });    // userId hard-coded until Lesson 17
  revalidatePath("/tasks");                            // "the /tasks page's data changed — re-render it"
}

export async function toggleTask(id: number, done: boolean) {
  await prisma.task.update({ where: { id }, data: { done } });
  revalidatePath("/tasks");
}

export async function deleteTask(id: number) {
  await prisma.task.delete({ where: { id } });
  revalidatePath("/tasks");
}
```

Use them directly from a form — **no fetch, no API route, no JSON**:

```tsx
// src/app/tasks/page.tsx — server component
import { prisma } from "@/lib/prisma";
import { createTask, toggleTask, deleteTask } from "./actions";

export default async function TasksPage() {
  const tasks = await prisma.task.findMany({ orderBy: { createdAt: "desc" } });

  return (
    <div>
      <form action={createTask} className="flex gap-2">
        <input name="text" placeholder="What needs doing?" className="border p-2" />
        <button type="submit">Add</button>
      </form>

      <ul>
        {tasks.map((task) => (
          <li key={task.id} className="flex gap-2">
            <form action={toggleTask.bind(null, task.id, !task.done)}>
              <button type="submit">{task.done ? "☑" : "☐"}</button>
            </form>
            <span className={task.done ? "line-through" : ""}>{task.text}</span>
            <form action={deleteTask.bind(null, task.id)}>
              <button type="submit">✖</button>
            </form>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Read what just happened:

1. The user submits the form. Next sends the form data to the server and runs `createTask`.
2. The action writes to the DB with Prisma (Lesson 7).
3. `revalidatePath("/tasks")` tells Next the page's cached output is stale; it re-renders the server component with fresh data and sends the update to the browser.
4. The list updates. **No `useState`, no `useEffect`, no fetch, no JSON, and it works even with JavaScript disabled** (it's a real HTML form POST).

`.bind(null, arg)` pre-fills arguments for actions used in forms — a way to pass the id without a hidden input. (`<input type="hidden" name="id" value={task.id} />` and `formData.get("id")` works too.)

Compare with Phase A + B: Express route + controller + service + CORS + `api.js` + `useState` + handler + optimistic update. The *logic* (Prisma call, validation) is identical; the plumbing is gone. This is what Next.js exists for.

> **Security reminder:** server actions are **public HTTP endpoints** — anyone can call them with any arguments, just like Express routes. Validate every input and check auth/ownership inside the action (Lesson 17). "Never trust the client" applies with full force.

---

## 3. Validation, errors, and pending state

Real forms need: validation messages, a disabled button while submitting, and a cleared field on success. React 19's `useActionState` hook (client side) pairs with an action that **returns** a result:

```ts
// src/app/tasks/actions.ts
"use server";
import { z } from "zod";                                    // Lesson 5's schema library, back again

const TaskInput = z.object({
  text: z.string().trim().min(1, "Task cannot be empty").max(200, "Keep it under 200 characters"),
});

export type ActionState = { error?: string; success?: boolean };

export async function createTask(_prev: ActionState, formData: FormData): Promise<ActionState> {
  const parsed = TaskInput.safeParse({ text: formData.get("text") });
  if (!parsed.success) return { error: parsed.error.issues[0].message };

  try {
    await prisma.task.create({ data: { text: parsed.data.text, userId: 1 } });
  } catch {
    return { error: "Could not save. Try again." };         // never leak internals
  }
  revalidatePath("/tasks");
  return { success: true };
}
```

```tsx
// src/components/AddTaskForm.tsx — client (hooks!)
"use client";

import { useActionState, useRef, useEffect } from "react";
import { createTask, type ActionState } from "@/app/tasks/actions";

export function AddTaskForm() {
  const [state, formAction, pending] = useActionState(createTask, {} as ActionState);
  const formRef = useRef<HTMLFormElement>(null);

  useEffect(() => {
    if (state.success) formRef.current?.reset();            // clear the input after success
  }, [state]);

  return (
    <form ref={formRef} action={formAction} className="flex flex-col gap-2">
      <div className="flex gap-2">
        <input name="text" placeholder="What needs doing?" className="border p-2" disabled={pending} />
        <button type="submit" disabled={pending}>{pending ? "Adding…" : "Add"}</button>
      </div>
      {state.error && <p className="text-red-600">{state.error}</p>}
    </form>
  );
}
```

`useActionState(action, initialState)` returns `[latest returned state, an action to give the form, pending flag]`. The action receives the previous state as its first argument (hence `_prev`). Errors are just returned data — no try/catch in the component, no HTTP status codes; the shape is whatever you decide.

The server page stays a server component and simply renders `<AddTaskForm />` — a client island whose *logic* lives on the server. Part 1 Lesson 13's form validation: same rules, same messages, now enforced server-side by default.

### Redirect after an action

```ts
import { redirect } from "next/navigation";
export async function createProject(formData: FormData) {
  const project = await prisma.project.create({ data: { name: String(formData.get("name")) } });
  revalidatePath("/projects");
  redirect(`/projects/${project.id}`);                      // call it last — it throws to perform the redirect
}
```

---

## 4. Route handlers — Express routes in Next.js

When you need a real JSON endpoint — a mobile app, a partner, a webhook from Stripe, or just to practice — create `route.ts` in an `api/` folder:

```ts
// src/app/api/tasks/route.ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";
import { z } from "zod";

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);              // req.query, the web-standard way
  const done = searchParams.get("done");
  const tasks = await prisma.task.findMany({
    where: done === null ? {} : { done: done === "true" },
    orderBy: { createdAt: "desc" },
  });
  return NextResponse.json(tasks);                             // res.json(tasks)
}

const TaskInput = z.object({ text: z.string().trim().min(1).max(200) });

export async function POST(request: Request) {
  const body = await request.json().catch(() => null);        // express.json(), manually
  const parsed = TaskInput.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json({ errors: parsed.error.issues.map((i) => i.message) }, { status: 400 });
  }
  const task = await prisma.task.create({ data: { text: parsed.data.text, userId: 1 } });
  return NextResponse.json(task, { status: 201 });
}
```

```ts
// src/app/api/tasks/[id]/route.ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";

type Ctx = { params: Promise<{ id: string }> };

export async function GET(_req: Request, { params }: Ctx) {
  const id = Number((await params).id);
  const task = await prisma.task.findUnique({ where: { id } });
  if (!task) return NextResponse.json({ error: "Task not found" }, { status: 404 });
  return NextResponse.json(task);
}

export async function PATCH(request: Request, { params }: Ctx) {
  const id = Number((await params).id);
  const body = await request.json();
  const task = await prisma.task.update({ where: { id }, data: { done: body.done, text: body.text } });
  return NextResponse.json(task);
}

export async function DELETE(_req: Request, { params }: Ctx) {
  const id = Number((await params).id);
  await prisma.task.delete({ where: { id } });
  return new NextResponse(null, { status: 204 });
}
```

Every line maps to Lesson 5: export-per-method instead of `router.get`; `Request`/`Response` (web standards) instead of `req`/`res`; `NextResponse.json(data, { status })` instead of `res.status().json()`. Test with Thunder Client exactly as in Phase A: `GET http://localhost:3000/api/tasks`. Your Phase A tests would nearly run unchanged against this.

Error handling: there's no global error middleware here — wrap risky code in try/catch per handler, or write a small `withErrorHandling(fn)` wrapper. Prisma's `P2025` still needs mapping to 404.

---

## 5. Calling route handlers from client components (when you must)

Sometimes a client component needs data on demand — a search-as-you-type, an infinite scroll. Then the Phase B pattern returns, minus CORS:

```tsx
"use client";
import { useState, useEffect } from "react";

export function TaskSearch() {
  const [q, setQ] = useState("");
  const [results, setResults] = useState<{ id: number; text: string }[]>([]);

  useEffect(() => {
    if (!q) return setResults([]);
    const ctrl = new AbortController();
    const t = setTimeout(() => {
      fetch(`/api/tasks?search=${encodeURIComponent(q)}`, { signal: ctrl.signal })
        .then((r) => r.json())
        .then(setResults)
        .catch(() => {});
    }, 300);                                                 // debounce: wait for typing to pause
    return () => { clearTimeout(t); ctrl.abort(); };        // cleanup cancels stale requests
  }, [q]);

  return (
    <>
      <input value={q} onChange={(e) => setQ(e.target.value)} placeholder="Search…" />
      <ul>{results.map((t) => <li key={t.id}>{t.text}</li>)}</ul>
    </>
  );
}
```

Relative URL, same origin — no CORS, no `VITE_API_URL`. Everything else is Lesson 12.

---

## 6. The full picture: where each piece lives

```
src/
├── app/
│   ├── layout.tsx                 server — shell
│   ├── page.tsx                   server — home
│   ├── tasks/
│   │   ├── page.tsx               server — awaits Prisma, renders list + <AddTaskForm/>
│   │   ├── actions.ts             "use server" — createTask/toggleTask/deleteTask (+ Zod)
│   │   ├── loading.tsx            skeleton
│   │   └── [id]/page.tsx          server — one task
│   └── api/tasks/route.ts         route handlers — JSON API for external clients
├── components/
│   ├── AddTaskForm.tsx            client — useActionState, pending, errors
│   ├── TaskItem.tsx               client or server — toggle/delete forms bound to actions
│   └── TaskSearch.tsx             client — fetches /api/tasks
├── lib/
│   ├── prisma.ts                  singleton
│   └── validation.ts              shared Zod schemas (used by actions AND route handlers)
└── prisma/schema.prisma
```

Compare to Phase A's `routes/ controllers/ services/` + Phase B's `components/ hooks/ api.js` + two deploys. Same responsibilities, one codebase, and the student can point at every file and say where it runs.

---

## ⚠️ Common mistakes

```
1. Forgetting revalidatePath after a mutation → data changes in the DB but the page shows old data.
2. Defining an action inline in a client component → actions must live in a "use server" file
   (or be defined inside a SERVER component with "use server" at the top of the function body).
3. Passing non-serializable args to an action (a Date is fine; a class instance isn't).
4. Skipping validation in an action "because the form has maxlength" → the action is a public endpoint.
5. Calling redirect() inside try/catch → it works by throwing; the catch swallows it. Call it outside.
6. request.json() on a body that isn't JSON → throws. .catch(() => null) then validate.
7. Returning a Prisma error message to the client → leaks schema details. Generic message + server log.
8. Using route handlers + fetch for your own forms out of habit → server actions are simpler; use them.
```

---

## ✅ Classwork

1. Write `createTask`/`toggleTask`/`deleteTask` actions and the pure-form version of the page (section 2). Add a task, toggle, delete. Then disable JavaScript in DevTools and do it again — it still works. Discuss why.
2. Add Zod validation + `useActionState` + pending state + auto-clear (section 3). Submit empty; submit 300 chars; observe messages. Add a 1-second `await new Promise(...)` in the action to see the pending state.
3. Build `/api/tasks` (GET with `?done=`, POST) and `/api/tasks/[id]` (GET, PATCH, DELETE). Test the whole set in Thunder Client with status codes. Map P2025 to 404.
4. Build `TaskSearch` calling `/api/tasks?search=` (add `search` support to the GET handler with Prisma `contains`).
5. Print the file tree of your project and annotate each file: server / client / action / route handler.

## 📝 Homework

1. **Projects in Next:** `Project` model (migrate), `/projects` page (server list + create form via action + redirect to the new project), `/projects/[id]` showing its tasks, and a `<select name="projectId">` on the task form. Ownership comes next lesson; for now `userId: 1`.
2. **Edit in place:** a client `EditableTaskText` component — double-click → input → Enter calls an `updateTaskText(id, text)` action via `useTransition` (`const [pending, start] = useTransition(); start(() => updateTaskText(...))`). Read the docs for `useTransition` with server actions.
3. **Shared validation:** move the Zod schemas to `src/lib/validation.ts` and use them in both actions and route handlers. Break the schema on purpose and confirm both paths reject.
4. **Optimistic UI (research):** read the docs for React's `useOptimistic` and apply it to the toggle so the checkbox flips instantly, before the server responds.
5. Written: server action vs route handler — when each? Why is a server action still a "public endpoint"? What does `revalidatePath` do and what happens without it?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **server action** | `"use server"` async function callable from components; Next does the HTTP |
| **`<form action={fn}>`** | Submit a form to a server action — works without JS |
| **`.bind(null, arg)`** | Pre-fill an action's arguments for forms |
| **`revalidatePath`** | Mark a page's cache stale so it re-renders with fresh data |
| **`useActionState`** | `[state, action, pending]` — returned errors + pending flag for forms |
| **`redirect()`** | Navigate from server code (throws — don't wrap in try/catch) |
| **route handler** | `route.ts` exporting `GET`/`POST`/… — JSON endpoints |
| **`Request` / `NextResponse`** | Web-standard request; response helper with `.json(data, { status })` |
| **debounce / AbortController** | Wait for typing to pause; cancel stale fetches |
| **`useOptimistic` / `useTransition`** | Instant UI feedback around actions |
