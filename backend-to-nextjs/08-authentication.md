# Lesson 08 — Authentication: Who Are You?

> **Goal:** Let users register and log in, and make the API know *who* is making each request — so that tasks belong to users and only their owner can touch them. Password hashing, JSON Web Tokens, auth middleware, and ownership checks. Security concepts here are non-negotiable; learn them right the first time.

---

## 1. Two questions every protected request must answer

- **Authentication** — *who are you?* (Proving identity: login.)
- **Authorization** — *are you allowed to do this?* (Permissions: is this your task?)

HTTP is stateless (Lesson 3), so "logged in" can't live in the server's memory between requests. Instead, at login the server hands the client a **proof**, and the client sends that proof with every subsequent request. Two common designs:

| | Session cookie | JWT (token) |
|---|---|---|
| Server stores | A session record (id → user) in DB/memory | Nothing — the token itself carries the data, signed |
| Client sends | A cookie (browser does it automatically) | An `Authorization: Bearer <token>` header (your code does it) |
| Good for | Traditional web apps, Next.js (Lesson 17) | APIs consumed by separate frontends/mobile apps |
| Logout | Delete the session record | Wait for expiry (or keep a blocklist) |

We build **JWT** auth today because it fits a standalone API and teaches the mechanics clearly. In Lesson 17 we'll build cookie sessions inside Next.js. Knowing both is the goal.

---

## 2. Passwords: never store them ⭐⭐

**Rule zero:** the database must *never* contain a password. Not even yours. Databases leak (breaches happen to huge companies yearly), and users reuse passwords everywhere.

Instead, store a **hash** — the output of a one-way function. Given the hash, nobody can recover the password; given a password attempt, you can hash it and compare.

```bash
npm install bcryptjs
```

```js
import bcrypt from "bcryptjs";

const hash = await bcrypt.hash("hunter2", 10);        // "$2a$10$N9qo8uLOickgx2ZMRZoMy..."
await bcrypt.compare("hunter2", hash);                // true
await bcrypt.compare("hunter3", hash);                // false
```

Why bcrypt and not, say, SHA-256? bcrypt is **deliberately slow** (the `10` is a cost factor — ~100ms) and **salted** (the same password produces a different hash each time, defeating precomputed tables). Slow-for-attackers is the feature. Use bcrypt or its modern cousins (argon2, scrypt) — never a fast hash, never your own scheme.

---

## 3. Register and login

Schema (from Lesson 7) already has `User { email @unique, password }`. The `password` column will hold the *hash*.

```js
// services/auth.service.js
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import { prisma } from "../db/prisma.js";
import { HttpError } from "../utils/HttpError.js";

export async function register({ email, password, name }) {
  const existing = await prisma.user.findUnique({ where: { email } });
  if (existing) throw new HttpError(409, "Email already registered");

  const passwordHash = await bcrypt.hash(password, 10);
  const user = await prisma.user.create({
    data: { email, name, password: passwordHash },
    select: { id: true, email: true, name: true },        // NEVER return the hash
  });
  return user;
}

export async function login({ email, password }) {
  const user = await prisma.user.findUnique({ where: { email } });
  const valid = user && (await bcrypt.compare(password, user.password));
  if (!valid) throw new HttpError(401, "Invalid email or password");     // same message for both cases!

  const token = jwt.sign({ sub: user.id }, process.env.JWT_SECRET, { expiresIn: "7d" });
  return { token, user: { id: user.id, email: user.email, name: user.name } };
}
```

Two security details hiding in plain sight:

- **"Invalid email or password"** — one message whether the email doesn't exist or the password is wrong. Saying "no such email" lets attackers discover which emails are registered.
- **`select` without password** — the hash never leaves the server. Make this a reflex.

Validation (Lesson 5) applies: email format, password ≥ 8 chars — do it in the controller/schema before calling the service.

---

## 4. JWTs — signed proof of identity

```bash
npm install jsonwebtoken
```

A **JSON Web Token** is three base64 parts joined by dots: `header.payload.signature`.

```
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOjEsImlhdCI6MTcyMjUwMDAwMCwiZXhwIjoxNzIzMTA0ODAwfQ . SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
        header                              payload                                      signature
```

- **Payload** — the claims: `sub` (subject = user id), `iat` (issued at), `exp` (expiry). **Readable by anyone** — it's only base64, not encrypted. Never put secrets in it.
- **Signature** — computed from header + payload + your `JWT_SECRET`. If anyone changes the payload (say, `sub: 1` → `sub: 2`), the signature no longer matches and `verify` rejects it. **The secret is what makes it trustworthy.**

```js
const token = jwt.sign({ sub: 42 }, process.env.JWT_SECRET, { expiresIn: "7d" });
const payload = jwt.verify(token, process.env.JWT_SECRET);    // { sub: 42, iat, exp } — or THROWS if invalid/expired
```

`JWT_SECRET` goes in `.env` — long and random. Generate one: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`. Leak it and anyone can forge tokens for any user. Rotate it if that ever happens.

Paste a token into **jwt.io** to see its parts decoded — it's a great "aha".

---

## 5. Auth middleware — turning a token into `req.user`

```js
// middleware/requireAuth.js
import jwt from "jsonwebtoken";
import { HttpError } from "../utils/HttpError.js";

export function requireAuth(req, res, next) {
  const header = req.headers.authorization ?? "";
  const [scheme, token] = header.split(" ");
  if (scheme !== "Bearer" || !token) throw new HttpError(401, "Missing token");

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = { id: payload.sub };                  // now every handler downstream knows who's asking
    next();
  } catch {
    throw new HttpError(401, "Invalid or expired token");
  }
}
```

Lesson 4's `requireApiKey` grown up: inspect a header, reject or annotate `req`, pass along.

Mount it:

```js
// routes/auth.routes.js
router.post("/register", authController.register);
router.post("/login", authController.login);
router.get("/me", requireAuth, authController.me);        // who am I? (returns req.user's record)

// routes/tasks.routes.js — every task route requires login:
router.use(requireAuth);
router.get("/", tasks.list);
// ...
```

The client flow, from Thunder Client (and later from React):

1. `POST /api/auth/register` → 201.
2. `POST /api/auth/login` → `{ token, user }`. Copy the token.
3. `GET /api/tasks` with header `Authorization: Bearer <token>` → 200. Without → 401. With a modified token → 401.

---

## 6. Authorization — ownership ⭐

Logged in isn't enough. Amina must not read or delete Bola's tasks. Every task query must be **scoped to the current user**:

```js
// services/tasks.service.js — every function takes userId
export function listTasks(userId, filters) {
  return prisma.task.findMany({ where: { userId, ...buildFilters(filters) }, orderBy: { createdAt: "desc" } });
}

export async function getTask(userId, id) {
  const task = await prisma.task.findFirst({ where: { id, userId } });     // both must match
  if (!task) throw new HttpError(404, "Task not found");
  return task;
}

export function createTask(userId, data) {
  return prisma.task.create({ data: { ...data, userId } });               // owner comes from the token, NEVER from the body
}

export async function updateTask(userId, id, data) {
  await getTask(userId, id);                                              // 404 if not yours
  return prisma.task.update({ where: { id }, data });
}
```

```js
// controllers — pass req.user.id everywhere
export async function list(req, res) {
  res.json(await listTasks(req.user.id, req.query));
}
```

Two design choices worth noticing:

- **404, not 403, for someone else's task.** Saying "forbidden" confirms the task exists. "Not found" reveals nothing. (Use 403 when the resource is legitimately visible but the action isn't allowed — e.g., an admin-only operation.)
- **`userId` always comes from the token.** If `createTask` trusted `req.body.userId`, anyone could create tasks as anyone. The body is the client's opinion; the token is verified fact.

This is *the* authorization pattern for user-owned data: put the user's id in every `where`. Forgetting it once is an IDOR vulnerability (Insecure Direct Object Reference) — one of the most common real-world API bugs.

---

## 7. Hardening basics

Three packages every Express API should have — install and understand them:

```bash
npm install cors helmet express-rate-limit
```

```js
import cors from "cors";
import helmet from "helmet";
import rateLimit from "express-rate-limit";

app.use(helmet());                                   // sensible security headers
app.use(cors({ origin: process.env.CLIENT_ORIGIN }));   // allow YOUR frontend's origin (Lesson 12 needs this!)
app.use("/api/auth", rateLimit({ windowMs: 15 * 60 * 1000, limit: 20 }));   // slow down brute-force login attempts
```

Plus the rules you already know: validate everything, parameterize queries (Prisma does), keep secrets in `.env`, never return hashes, log errors but don't leak them.

---

## ⚠️ Common mistakes

```
1. Storing plaintext passwords, or "encrypting" them (reversible). HASH with bcrypt.
2. Returning the user object with the password hash included → always select/omit.
3. Different errors for "no such user" vs "wrong password" → email enumeration.
4. JWT_SECRET short, guessable, or committed → forgeable tokens.
5. Putting sensitive data in the JWT payload → it's readable by anyone with the token.
6. Trusting userId from req.body → identity must come from the verified token.
7. Forgetting userId in a where → users can read/modify each other's data (IDOR).
8. "Bearer" spelled/cased wrong in Thunder Client → 401 confusion. It's `Authorization: Bearer <token>`.
```

---

## ✅ Classwork

1. Hash a password in a scratch file; compare right and wrong guesses; hash the same password twice and observe two different hashes (salt!).
2. Build register + login. Register twice with the same email → 409. Login with a wrong password → 401 with the generic message.
3. Sign a JWT, paste it into jwt.io, read the payload. Change one character of the token and `verify` it → error. Set `expiresIn: "5s"`, wait, verify → expired error.
4. Add `requireAuth` to the task routes. Full Thunder Client pass with and without the header. Create two users; confirm each sees only their own tasks and gets 404 for the other's ids.

## 📝 Homework

1. **`/api/auth/me`** and **`PATCH /api/auth/me`** (update name; changing password requires the current password — verify it first, then hash the new one).
2. **Roles (research):** add `role String @default("user")` to User. Write `requireRole("admin")` middleware (a function that *returns* a middleware — closures!). Add `GET /api/admin/users` listing all users (without hashes) for admins only → 403 for regular users.
3. **Refresh thinking (written):** a 7-day token that's stolen is valid for 7 days. Read about short-lived access tokens + refresh tokens, and describe the flow in a paragraph. (No need to implement.)
4. **Frontend hookup:** in your `public/` to-do page, add a login form; store the token in `localStorage`; send it in the `Authorization` header on every fetch (a `apiFetch(url, options)` helper that adds the header is the clean way). Show "logged in as …" and a logout button (just delete the token).
5. Written: authentication vs authorization; what a hash is and why bcrypt is slow on purpose; why the JWT payload isn't secret.

## 🔑 Key words

| Word | Meaning |
|------|---------|
| **authentication** | Proving who you are |
| **authorization** | Checking what you may do |
| **hash** | One-way transformation of a password; bcrypt = slow + salted |
| **salt** | Random data mixed into each hash so identical passwords differ |
| **JWT** | Signed token: header.payload.signature; payload readable, signature verifiable |
| **`JWT_SECRET`** | The key that makes tokens trustworthy — guard it |
| **Bearer token** | `Authorization: Bearer <token>` header |
| **`req.user`** | Set by auth middleware from the verified token |
| **ownership check** | `where: { id, userId }` — scope every query to the user |
| **IDOR** | Bug: accessing others' records by id because ownership wasn't checked |
| **CORS** | Server-side permission for a browser origin to call the API |
