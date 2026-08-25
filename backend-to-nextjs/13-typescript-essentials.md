# Lesson 13 — TypeScript Essentials (Enough for Next.js)

> **Goal:** Learn just enough TypeScript to read and write typical Next.js/React code. TypeScript is JavaScript plus type annotations; nearly every professional Next.js project uses it, and `create-next-app` defaults to it. This is a *short* lesson by design — you'll deepen it through use.

---

## 1. What TypeScript is and why it exists

You've met these bugs:

```js
const price = input.value;       // "50" — a string
price * 2;                       // 100? "5050"? Lesson 2 of Part 1...
user.profile.email;              // Cannot read properties of undefined
tasksApi.creat(text);            // typo, discovered at runtime, in production, by a user
```

**TypeScript** (TS) adds *types* to JavaScript so a tool can catch these **before the code runs** — in your editor, as you type. It compiles down to plain JS (types are erased; the browser/Node never sees them). Everything you know still applies; you're adding labels.

```ts
let price: number = 50;
price = "50";                    // ❌ Editor error: Type 'string' is not assignable to type 'number'
```

Benefits you'll feel immediately: autocomplete that actually knows your data (`task.` shows `id`, `text`, `done`), refactoring safety, and self-documenting function signatures. Costs: some extra syntax, and occasional wrestling with the checker. Worth it.

---

## 2. Basic types

```ts
// Primitives — usually INFERRED; you rarely need to write these
let name: string = "Amina";
let age: number = 15;
let active: boolean = true;
let nothing: null = null;
let notSet: undefined = undefined;

let city = "Kano";               // inferred as string — no annotation needed
city = 42;                       // ❌ error

// Arrays
let scores: number[] = [90, 85];
let names: string[] = [];

// Any — the escape hatch that turns TS off. Avoid; prefer unknown.
let anything: any = "whatever";

// Union — one of several
let id: number | string = 7;
let status: "loading" | "ready" | "error" = "loading";     // literal union — fantastic for state machines
```

**Rule of thumb:** annotate function parameters, return types when helpful, and data shapes. Let TS infer local variables.

---

## 3. Object shapes: `type` and `interface`

```ts
type Task = {
  id: number;
  text: string;
  done: boolean;
  createdAt: string;
  projectId?: number;            // ? = optional (number | undefined)
};

const task: Task = { id: 1, text: "Learn TS", done: false, createdAt: "2026-08-01" };
task.txet;                       // ❌ Property 'txet' does not exist on type 'Task'

const tasks: Task[] = [];
```

`interface Task { ... }` does nearly the same thing. Convention in many codebases: `interface` for object shapes, `type` for unions/aliases. Either is fine — be consistent.

Composing shapes:

```ts
type User = { id: number; email: string; name: string | null };
type TaskWithUser = Task & { user: User };                 // intersection: both
type NewTask = Omit<Task, "id" | "createdAt">;              // Task minus some fields — for create payloads
type TaskPatch = Partial<Pick<Task, "text" | "done">>;      // some fields, all optional — for PATCH bodies
```

`Omit`, `Pick`, `Partial` are built-in **utility types**. You'll use these three constantly for API payloads.

---

## 4. Functions

```ts
function add(a: number, b: number): number {
  return a + b;
}

const isDone = (task: Task): boolean => task.done;

// Optional and default params
function greet(name: string, greeting = "Hello"): string {
  return `${greeting}, ${name}`;
}

// Functions as values (callback props!)
type OnToggle = (id: number) => void;         // void = returns nothing

// async functions return Promise<T>
async function fetchTasks(): Promise<Task[]> {
  const res = await fetch("/api/tasks");
  return res.json();
}
```

Return types are often inferred; write them on exported/API functions for clarity.

---

## 5. Narrowing — TS follows your `if`s

```ts
function describe(value: number | string) {
  if (typeof value === "string") {
    return value.toUpperCase();          // TS knows: string here
  }
  return value.toFixed(2);               // and number here
}

function label(task: Task | undefined) {
  if (!task) return "none";              // after this line, task is Task
  return task.text;
}

// Errors in catch are `unknown` — narrow before using
try { ... } catch (err) {
  const message = err instanceof Error ? err.message : "Unknown error";
}
```

This is why TS catches the `undefined.email` bug: it forces you to handle the possibility.

---

## 6. Generics — types with parameters (recognize them)

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}
first([1, 2, 3]);          // T = number → returns number | undefined
first(["a", "b"]);         // T = string

// You'll mostly USE generics from libraries:
useState<Task[]>([]);                     // React: "this state is an array of Task"
useState<User | null>(null);
Promise<Task[]>
Array<string>                              // same as string[]
```

Read `<T>` as "for some type T, decided at the call site." Writing your own generics is intermediate TS; reading them is required now.

---

## 7. TypeScript in React

```bash
npm create vite@latest tasks-ui-ts -- --template react-ts
```

Files become `.tsx`. The main additions: prop types and state types.

```tsx
import { useState } from "react";

type Task = { id: number; text: string; done: boolean };

type TaskItemProps = {
  task: Task;
  onToggle: (id: number) => void;
  onDelete: (id: number) => void;
};

function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li>
      <input type="checkbox" checked={task.done} onChange={() => onToggle(task.id)} />
      {task.text}
      <button onClick={() => onDelete(task.id)}>✖</button>
    </li>
  );
}

function App() {
  const [tasks, setTasks] = useState<Task[]>([]);                    // generic: tells TS the array type
  const [status, setStatus] = useState<"loading" | "ready" | "error">("loading");
  const [text, setText] = useState("");                             // inferred string

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {   // event types
    setText(e.target.value);
  }
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
  }
  // ...
}
```

Event types look scary; you'll type them from memory after five uses: `React.ChangeEvent<HTMLInputElement>`, `React.FormEvent<HTMLFormElement>`, `React.MouseEvent<HTMLButtonElement>`. Or skip them by writing inline arrows (`onChange={(e) => setText(e.target.value)}` — TS infers `e`).

`children` prop: `children: React.ReactNode`.

### Typing API responses

```ts
// api.ts
export async function apiFetch<T>(path: string, options: RequestInit = {}): Promise<T> {
  const res = await fetch(`${BASE}${path}`, { ...options, headers: { "Content-Type": "application/json", ...options.headers } });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

const tasks = await apiFetch<Task[]>("/api/tasks");      // tasks is Task[] — autocomplete everywhere
```

A caveat worth understanding: `apiFetch<Task[]>` is a *promise* to TS, not a check — if the server returns something else, TS can't know. That's why Zod (Lesson 5) shows up on the frontend too: `TaskSchema.parse(data)` validates at runtime *and* infers the TS type (`z.infer<typeof TaskSchema>`). One schema, both worlds. You'll see this pattern in Next.js projects.

---

## 8. Working with the checker

- **Red squiggles are your friend.** Hover to read the message. They're usually right.
- `npx tsc --noEmit` checks the whole project (Vite/Next run this on build).
- **Don't reach for `any`** to silence errors. Use `unknown` and narrow, or fix the type.
- When a library's types confuse you, hover the function to see its signature — that's TS documentation.
- Non-null assertion `value!` ("trust me, it's not null") — use rarely and knowingly.
- `tsconfig.json` — the config; the templates set sensible defaults (`strict: true` — keep it).

---

## ⚠️ Common mistakes

```
1. Annotating everything → noise. Let inference work; annotate params, exports, and data shapes.
2. `any` everywhere → you have JavaScript with extra steps. Use unknown + narrowing.
3. Treating TS types as runtime checks → they vanish at compile time. Validate external data (Zod).
4. Fighting the checker with `as` casts → usually hides a real bug. Narrow instead.
5. Forgetting `?` on optional fields, then getting "possibly undefined" errors — the error is correct; handle it.
6. `.js` vs `.tsx`: JSX in TS files needs the .tsx extension.
```

---

## ✅ Classwork

1. Create the `react-ts` project. Define `Task`, `User`, `NewTask` (Omit), `TaskPatch` (Partial<Pick>). Create a valid and an invalid object of each; read the errors.
2. Write `add`, `isDone`, `describe` (narrowing), and `first<T>`; call each and hover to see inferred types.
3. Convert the Lesson 11 to-do app to `.tsx`: prop types for every component, `useState<Task[]>`, typed handlers. Fix every squiggle without `any`.
4. Convert `api.js` to `api.ts` with a generic `apiFetch<T>`, and type `tasksApi`'s return values.

## 📝 Homework

1. **Full conversion:** move the connected to-do app (Lesson 12, with auth) to TypeScript. Types for `User`, `LoginResponse`, the auth API, `useFetch<T>`.
2. **Zod bridge:** install Zod in the frontend; define `TaskSchema`; derive `type Task = z.infer<typeof TaskSchema>`; validate `apiFetch` responses with `z.array(TaskSchema).parse(data)`. Break the API's shape on purpose (rename a field) and see the runtime error where TS alone would have been silent.
3. **Reading practice:** open the type definitions of `useState` (Ctrl+click in VS Code) and of `fetch`'s `RequestInit`. Write three things you learned from reading them.
4. Written: types are erased at runtime — what does that imply for data from an API? When would you use `type` vs `interface`? What does `<T>` mean?

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **TypeScript** | JS + static types, checked before running, erased at compile time |
| **annotation** | `: type` after a name |
| **inference** | TS figures out the type without annotation |
| **`type` / `interface`** | Define object shapes |
| **union `A \| B`** | One of several types; literal unions for state machines |
| **optional `?`** | Field may be missing (`T \| undefined`) |
| **narrowing** | `typeof`, `instanceof`, `if (!x)` — TS refines the type in a branch |
| **generic `<T>`** | A type parameter: `useState<Task[]>`, `Promise<T>` |
| **utility types** | `Partial`, `Pick`, `Omit` |
| **`unknown` vs `any`** | Safe "don't know yet" vs "turn off checking" |
| **`z.infer`** | Derive a TS type from a Zod schema — runtime + compile-time in one |
