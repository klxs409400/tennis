# Unit 1 · Day 4 — Finish CRUD + Error Handling

## What you're building today

The remaining three CRUD endpoints (GET by ID, PATCH, DELETE), a global error handler, the `CoachSuggestion` model, and a POST endpoint to attach suggestions to sessions. After today you have a fully functional REST API.

---

## Concepts

### Route parameters vs query parameters

```
GET /api/sessions/abc123        ← route parameter (:id)
GET /api/sessions?focus=serve   ← query parameter (?key=value)
```

- Route params (`req.params.id`) identify a specific resource
- Query params (`req.query.focus`) filter, sort, or paginate a collection

### Promises and async/await deeply

A **Promise** is an object representing a value that will be available in the future. It has three states: pending → fulfilled or rejected.

```ts
// Promise chain style
prisma.trainingSession.findUnique({ where: { id } })
  .then(session => res.json(session))
  .catch(err => next(err))

// async/await style (same thing, cleaner)
const session = await prisma.trainingSession.findUnique({ where: { id } })
res.json(session)
```

`async/await` is syntactic sugar over Promises. Under the hood it's the same — `await` just pauses execution of the current function until the Promise resolves.

### The event loop

Node.js is single-threaded but handles concurrency via the **event loop**:
1. JS runs on one thread
2. I/O operations (DB queries, file reads) are offloaded to `libuv` (C++ thread pool)
3. When the I/O finishes, the callback is queued on the event loop
4. The event loop picks it up and runs it on the JS thread

This is why `await prisma.findMany()` doesn't block other requests — the DB query runs in the background, and Node handles other requests while waiting.

### Global error handler in Express

A special middleware with **4 arguments**: `(err, req, res, next)`. Express recognizes this signature and routes unhandled errors here.

```ts
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack)
  res.status(500).json({ error: err.message })
})
```

Must be registered **after all routes** in `index.ts`. In your route handlers, call `next(err)` to trigger it.

### One-to-many relationships in Prisma

```prisma
model TrainingSession {
  coachSuggestions CoachSuggestion[]
}

model CoachSuggestion {
  sessionId String
  session   TrainingSession @relation(fields: [sessionId], references: [id])
}
```

`CoachSuggestion` has a FK `sessionId` pointing to `TrainingSession.id`. One session can have many suggestions.

---

## Things to watch out for

**Return 404 on missing records** — `prisma.findUnique` returns `null` if the record doesn't exist. Check for null and return 404 before trying to use the result.

**DELETE returns 204 (no body)** — `204 No Content` means success with no response body. Don't call `res.json(...)` after a delete — just `res.status(204).send()`.

**`next(err)` in async routes** — in an `async` route handler, errors from `await` won't reach the global error handler unless you catch them and pass to `next`. Wrap in try/catch: `catch (err) { next(err) }`.

**Migration naming** — use descriptive names: `--name add-coach-suggestion`, not `--name update`. The name becomes part of the migration filename and matters for reading history later.

---

## Today's steps (for reference)

1. Implement `GET /api/sessions/:id` — return 404 with a message if not found
2. Implement `PATCH /api/sessions/:id` — partial update using `prisma.trainingSession.update`
3. Implement `DELETE /api/sessions/:id` — delete and return 204
4. Add a global error-handling middleware (4 arguments) at the bottom of `src/index.ts`
5. Add `CoachSuggestion` model in `schema.prisma` with a `sessionId` FK
6. Run `npx prisma migrate dev --name add-coach-suggestion`
7. Implement `POST /api/sessions/:id/coach-suggestions`
8. Test all five endpoints in Postman

---

## Interview questions to be able to answer after today

- "Explain the event loop" → Single JS thread; I/O offloaded to libuv; callbacks queued and processed when the thread is free — enables concurrency without threads
- "Callbacks vs Promises vs async/await — what problem does each solve?" → Callbacks: original async pattern, leads to callback hell; Promises: chainable, better error handling; async/await: syntactic sugar over Promises, reads like synchronous code
- "How do you handle errors in async Express routes?" → Wrap in try/catch and call next(err); register a 4-argument error middleware after all routes
