# 19 — Backend → Next.js Cheat Sheet

Lesson numbers in [brackets]. Keep this open while building.

---

## Node & npm [L1–2]
```bash
node -v && npm -v                  # versions
npm init -y                        # package.json
npm install express                # dependency
npm install -D prisma supertest    # dev dependency
npm run dev                        # runs "scripts.dev"
node --watch --env-file=.env server.js
```
```json
{ "type": "module", "scripts": { "dev": "node --watch --env-file=.env server.js", "start": "node server.js" } }
```
```js
import { readFile, writeFile } from "node:fs/promises";
const text = await readFile("f.txt", "utf8");            // top-level await OK in ESM
await writeFile("f.json", JSON.stringify(data, null, 2));
process.argv.slice(2)                                    // CLI args (strings)
process.env.PORT ?? 3000                                 // env var (string!) with default
// .gitignore: node_modules/  .env  *.db
```

## HTTP [L3]
```
GET read · POST create · PATCH update · PUT replace · DELETE remove
200 OK · 201 Created · 204 No Content · 400 Bad Request · 401 Unauthenticated
403 Forbidden · 404 Not Found · 409 Conflict · 500 Server Error
Stateless: every request stands alone → tokens/cookies carry identity.
```

## Express [L4–5]
```js
import express from "express";
const app = express();
app.use(express.json());                                 // req.body — BEFORE routes
app.use(cors({ origin: process.env.CLIENT_ORIGIN }));
app.use((req, res, next) => { console.log(req.method, req.path); next(); });   // middleware

app.get("/api/tasks/:id", (req, res) => {
  const id = Number(req.params.id);                      // params & query are STRINGS
  if (!task) return res.status(404).json({ error: "Not found" });  // return after sending!
  res.json(task);
});
app.post("/api/tasks", (req, res) => { /* validate req.body */ res.status(201).json(task); });
app.delete("/api/tasks/:id", (req, res) => res.sendStatus(204));

const router = Router(); router.get("/", list); app.use("/api/tasks", router);
app.use((req, res) => res.status(404).json({ error: "Route not found" }));   // catch-all
app.use((err, req, res, next) => {                       // error handler: 4 params, LAST
  res.status(err.status ?? 500).json({ error: err.status ? err.message : "Something went wrong" });
});
class HttpError extends Error { constructor(status, msg) { super(msg); this.status = status; } }
```
REST: nouns in URLs (`/api/tasks/42`), verbs as methods, plural collections, JSON in/out.
Layers: `routes/` → `controllers/` → `services/` (+ `middleware/`, `db/`).

## SQL [L6]
```sql
CREATE TABLE tasks (id INTEGER PRIMARY KEY AUTOINCREMENT, text TEXT NOT NULL, done INTEGER DEFAULT 0);
INSERT INTO tasks (text) VALUES ('x');
SELECT * FROM tasks WHERE done = 0 ORDER BY created_at DESC LIMIT 10;
UPDATE tasks SET done = 1 WHERE id = 2;      -- WHERE!
DELETE FROM tasks WHERE id = 3;              -- WHERE!
SELECT t.text, u.email FROM tasks t JOIN users u ON t.user_id = u.id;
```
```js
db.prepare("SELECT * FROM tasks WHERE id = ?").get(id);   // parameters — NEVER string-concat (SQL injection)
```

## Prisma [L7]
```bash
npx prisma init --datasource-provider sqlite     # or postgresql
npx prisma migrate dev --name init               # after every schema change
npx prisma migrate deploy                        # production
npx prisma studio                                # GUI
```
```prisma
model Task {
  id        Int      @id @default(autoincrement())
  text      String
  done      Boolean  @default(false)
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    Int
}
```
```js
prisma.task.findMany({ where: { userId, done: false, text: { contains: q } }, orderBy: { createdAt: "desc" }, skip, take, include: { user: true } })
prisma.task.findUnique({ where: { id } })          // null if missing
prisma.task.findFirst({ where: { id, userId } })   // ownership check
prisma.task.create({ data })  .update({ where: { id }, data })  .delete({ where: { id } })   // update/delete THROW P2025
prisma.task.count({ where })  prisma.$transaction([...])
// P2002 unique violation → 409 · P2025 not found → 404
```

## Auth [L8, L17]
```js
await bcrypt.hash(password, 10); await bcrypt.compare(password, hash);   // never store plaintext
jwt.sign({ sub: user.id }, process.env.JWT_SECRET, { expiresIn: "7d" }); jwt.verify(token, secret);
// Header: Authorization: Bearer <token> → middleware sets req.user
// Identity from the TOKEN/SESSION, never from the body. Every query: where: { ..., userId }.
// Same error for bad email & bad password. Never return the hash. 404 (not 403) for others' records.
```
Cookie sessions (Next): random id → Session row → `cookies().set("session", id, { httpOnly, secure, sameSite: "lax" })`.

## Testing & deploy [L9]
```js
import { test } from "node:test"; import assert from "node:assert/strict"; import request from "supertest";
const res = await request(app).post("/api/tasks").set("Authorization", `Bearer ${token}`).send({ text: "x" });
assert.equal(res.status, 201);
```
Render: build `npm install && npm run build` · start `npm start` · env vars in dashboard · Postgres via Neon.

## React [L10–12]
```jsx
function TaskItem({ task, onToggle }) { return <li onClick={() => onToggle(task.id)}>{task.text}</li>; }
{tasks.map((t) => <TaskItem key={t.id} task={t} onToggle={toggle} />)}      // key = stable id
{cond ? <A /> : <B />}   {count > 0 && <Badge />}   <>fragments</>   className   htmlFor   onClick={fn}

const [tasks, setTasks] = useState([]);                  // hooks: top level only
setTasks((prev) => [...prev, task]);                     // immutable updates: spread / map / filter
setTasks((prev) => prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t)));
<input value={text} onChange={(e) => setText(e.target.value)} />           // controlled
<form onSubmit={(e) => { e.preventDefault(); ... }}>
useEffect(() => { const id = setInterval(...); return () => clearInterval(id); }, []);  // [] once, [x] on change
// Data down (props), events up (callback props). Lift state to the common parent.
// CORS: fix on the SERVER. VITE_* env vars are PUBLIC.
```

## TypeScript [L13]
```ts
type Task = { id: number; text: string; done: boolean; projectId?: number };
type NewTask = Omit<Task, "id">;  type Patch = Partial<Pick<Task, "text" | "done">>;
type Status = "loading" | "ready" | "error";
function f(a: number, b = 2): string {}      const g = (t: Task): boolean => t.done;
useState<Task[]>([])   Promise<Task[]>   React.ReactNode   React.ChangeEvent<HTMLInputElement>
catch (err) { err instanceof Error ? err.message : "Unknown" }     // unknown → narrow; avoid any
type Task = z.infer<typeof TaskSchema>;      // Zod: runtime + compile-time
```

## Next.js — routing [L14]
```
app/page.tsx → /        app/tasks/[id]/page.tsx → /tasks/:id        app/api/tasks/route.ts → /api/tasks
layout.tsx (shared shell; root has <html><body>)   loading.tsx   error.tsx ("use client")   not-found.tsx
(group)/ → no URL segment
```
```tsx
export default async function Page({ params, searchParams }: { params: Promise<{ id: string }>; searchParams: Promise<{ q?: string }> }) {
  const { id } = await params; const { q } = await searchParams;
}
import Link from "next/link";  <Link href="/tasks">…</Link>
import { redirect, notFound } from "next/navigation";         // server
import { useRouter, usePathname } from "next/navigation";      // client ("use client")
export const metadata = { title: "…" };  export async function generateMetadata({ params }) {}
```

## Next.js — server vs client [L15]
```
SERVER (default): async, await prisma/fetch, process.env secrets, no hooks/events, ships no JS
CLIENT ("use client" first line): useState/useEffect/onClick/browser APIs, no DB/secrets
Server → Client: import & render ✅ ; pass serializable props ✅ ; pass functions ❌ (use actions)
Client → Server: import ❌ ; receive as children ✅
Keep "use client" as LOW in the tree as possible.  NEXT_PUBLIC_* only reaches the browser.
```
```ts
// lib/prisma.ts singleton
const g = globalThis as unknown as { prisma?: PrismaClient };
export const prisma = g.prisma ?? new PrismaClient();
if (process.env.NODE_ENV !== "production") g.prisma = prisma;
```

## Next.js — mutations [L16]
```ts
"use server";                                            // actions.ts
export async function createTask(_prev: State, formData: FormData): Promise<State> {
  const user = await requireUser();                      // auth INSIDE the action
  const parsed = Schema.safeParse({ text: formData.get("text") });
  if (!parsed.success) return { error: parsed.error.issues[0].message };
  await prisma.task.create({ data: { ...parsed.data, userId: user.id } });
  revalidatePath("/tasks");                              // or the page shows stale data
  return { success: true };
}
```
```tsx
"use client";
const [state, action, pending] = useActionState(createTask, {});
<form action={action}> … <button disabled={pending}>Add</button> {state.error && <p>{state.error}</p>} </form>
<form action={deleteTask.bind(null, task.id)}><button>✖</button></form>
```
```ts
// app/api/tasks/route.ts
export async function GET(req: Request) { const { searchParams } = new URL(req.url); return NextResponse.json(data); }
export async function POST(req: Request) { const body = await req.json().catch(() => null); return NextResponse.json(task, { status: 201 }); }
// app/api/tasks/[id]/route.ts: export async function DELETE(_: Request, { params }: { params: Promise<{ id: string }> })
```

## Next.js — auth & deploy [L17]
```ts
import "server-only";
const store = await cookies(); store.get("session")?.value; store.set(name, value, { httpOnly: true, secure: prod, sameSite: "lax", path: "/", expires });
export const getCurrentUser = cache(async () => { /* cookie → session row → user (no hash) */ });
// Page: if (!user) redirect("/login")   (UX)      Action: await requireUser()   (security)
```
Vercel: `"build": "prisma generate && prisma migrate deploy && next build"`, `"postinstall": "prisma generate"`, env `DATABASE_URL`.

---

## The 12 commandments of Part 2

1. Draw the diagram: browser → server → database. Know which box you're in.
2. `.env` is git-ignored; secrets never reach the browser; `NEXT_PUBLIC_`/`VITE_` = public.
3. Validate every input on the server. The client is not trusted — ever.
4. Parameterized queries only. Prisma does it; never concatenate SQL.
5. Hash passwords with bcrypt. Never store or return them.
6. Identity comes from the verified token/session — never from the request body.
7. Every user-data query includes `userId`. Missing it once = IDOR.
8. Return the right status: 201 create, 204 delete, 400 bad input, 401/403/404 as meant, 500 never on purpose.
9. Error handler last; log the real error, send a clean message.
10. React: new objects on every state update; hooks at the top level; `key` on lists.
11. Next: server by default, `"use client"` low in the tree, `revalidatePath` after every mutation.
12. Read the logs. Terminal, DevTools, host dashboard. The error tells you where.
