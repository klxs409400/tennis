# Unit 2 · Day 2 — Protected Routes & Ownership

## What you're building today

JWT auth middleware that verifies tokens on every protected request, plus data ownership — every session query filters by `userId` so users can only see and modify their own data.

---

## Concepts

### Auth middleware flow

```
Request → authMiddleware → route handler
            ↓ if invalid
          401 response
```

The middleware:
1. Reads `Authorization: Bearer <token>` from the request header
2. Calls `jwt.verify(token, secret)` — throws if expired or tampered
3. Attaches `decoded.userId` to `req.user`
4. Calls `next()` so the route handler runs

```ts
export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const auth = req.headers.authorization
  if (!auth?.startsWith('Bearer ')) return res.status(401).json({ error: 'Missing token' })
  try {
    const decoded = jwt.verify(auth.slice(7), process.env.JWT_SECRET!) as { userId: string }
    req.user = { id: decoded.userId }
    next()
  } catch {
    res.status(401).json({ error: 'Invalid token' })
  }
}
```

### Extending Express's Request type

TypeScript doesn't know about `req.user` by default. Add a declaration file:

```ts
// src/types/express.d.ts
declare namespace Express {
  interface Request {
    user?: { id: string }
  }
}
```

### IDOR vulnerability

**IDOR** = Insecure Direct Object Reference. A user guesses another user's resource ID and accesses it directly.

Example vulnerability:
```ts
// WRONG — any logged-in user can read any session by ID
const session = await prisma.trainingSession.findUnique({ where: { id: req.params.id } })
```

Fix — always scope queries to the authenticated user:
```ts
// CORRECT
const session = await prisma.trainingSession.findFirst({
  where: { id: req.params.id, userId: req.user!.id }
})
if (!session) return res.status(404).json({ error: 'Not found' })
```

Return 404 (not 403) when the record belongs to another user — don't confirm the resource exists.

### Foreign keys for ownership

Adding `userId` to `TrainingSession` creates the ownership link:

```prisma
model TrainingSession {
  userId String
  user   User   @relation(fields: [userId], references: [id])
}
```

Every create must include `userId: req.user!.id`. Every read/update/delete must include `userId: req.user!.id` in the `where` clause.

---

## Things to watch out for

**Applying middleware to a whole router** — `router.use(authMiddleware)` before any route definitions protects everything on that router. Safer than adding it to each route individually — easier to miss one.

**The `!` non-null assertion** — after `authMiddleware` runs, `req.user` is guaranteed to be set. TypeScript doesn't know this, so you'll use `req.user!.id`. If you see a TypeScript error here, it means you forgot to apply the middleware.

**Migration required after schema change** — adding `userId` to `TrainingSession` requires a new migration. Prisma will warn you if there are existing rows (the new required field has no value for them). In dev, you can reset the DB: `npx prisma migrate reset`.

---

## Today's steps (for reference)

1. Add `userId String` field and `@relation` to `TrainingSession` in `schema.prisma`
2. Run `npx prisma migrate dev --name add-session-owner`
3. Create `src/middleware/auth.ts` with JWT verification logic
4. Add `src/types/express.d.ts` to extend `req.user`
5. Apply `authMiddleware` to all routes in `sessions.ts` via `router.use()`
6. Update `POST /api/sessions` to include `userId: req.user!.id`
7. Update all session queries to add `userId: req.user!.id` in the `where` clause
8. Test with two users — verify they cannot see each other's sessions

---

## Interview questions to be able to answer after today

- "What's the difference between authentication and authorization?" → Authentication verifies identity (who you are); authorization checks permissions (what you can do)
- "What's an IDOR attack and how do you prevent it?" → User accesses another user's resource by guessing its ID; prevent by always scoping DB queries to the authenticated user's ID
- "How does middleware work in Express?" → Functions in the request pipeline; each receives req/res/next; calls next() to continue or sends a response to terminate
