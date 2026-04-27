# Unit 1 · Day 3 — First CRUD Endpoints (Create + Read)

## What you're building today

Two working API endpoints: `POST /api/sessions` to create a training session and `GET /api/sessions` to list them. You'll also wire up Prisma to Express and test both endpoints in Postman.

---

## Concepts

### REST

- REST = Representational State Transfer — an architectural style for APIs
- Key idea: **URLs identify resources (nouns), HTTP methods describe the action (verb)**
- Good: `POST /api/sessions` — bad: `POST /api/createSession`
- Resources are plural nouns: `/sessions`, `/users`, `/suggestions`

### HTTP methods

| Method | Purpose | Idempotent? |
|--------|---------|-------------|
| GET | Read | Yes |
| POST | Create | No |
| PATCH | Partial update | Yes |
| PUT | Full replace | Yes |
| DELETE | Delete | Yes |

Idempotent = calling it multiple times has the same effect as calling it once.

### Status codes to know

| Code | Meaning | When to use |
|------|---------|-------------|
| 200 | OK | Successful GET, PATCH, DELETE |
| 201 | Created | Successful POST that created a resource |
| 400 | Bad Request | Client sent invalid data |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but not allowed |
| 404 | Not Found | Resource doesn't exist |
| 500 | Internal Server Error | Unexpected server failure |

### Middleware in Express

Middleware = a function that runs **between** the incoming request and your route handler.

```ts
app.use(express.json()) // parses JSON body → req.body
app.use(cors())         // adds CORS headers
app.use(authMiddleware) // verifies JWT → req.user
```

Each middleware calls `next()` to pass control to the next function. If it doesn't call `next()`, the request stops there.

### Express Router

Keeps route files organized. Instead of `app.get(...)` everywhere in `index.ts`, you create a Router in its own file:

```ts
// src/routes/sessions.ts
import { Router } from 'express'
const router = Router()
router.get('/', ...)
router.post('/', ...)
export default router

// src/index.ts
import sessionsRouter from './routes/sessions'
app.use('/api/sessions', sessionsRouter)
```

---

## Things to watch out for

**`express.json()` must come before your routes** — if you don't add this middleware, `req.body` will be `undefined` on POST requests. Add it early in `index.ts`.

**`async` route handlers need try/catch** — if a Prisma call throws (DB down, constraint violation), an unhandled promise rejection will crash your server or hang the request. Wrap in try/catch or use a global error handler.

**201 vs 200 on create** — always return 201 for a successful POST that creates a resource. It signals to clients that a new resource was made.

**`router.get('/')` vs `app.get('/api/sessions')`** — when you mount a router with `app.use('/api/sessions', router)`, the router sees paths *relative to the mount point*. So `router.get('/')` matches `GET /api/sessions`.

---

## Today's steps (for reference)

1. Add `express.json()` middleware in `src/index.ts`
2. Create `src/routes/sessions.ts` with an Express Router
3. Mount the router in `src/index.ts` at `/api`
4. Implement `POST /api/sessions` — read body, call `prisma.trainingSession.create`, return 201 + created row
5. Implement `GET /api/sessions` — call `prisma.trainingSession.findMany`, return 200 + array
6. Install Postman or Thunder Client (VS Code extension)
7. Test both endpoints — confirm correct status codes and response shapes

---

## Interview questions to be able to answer after today

- "What is REST?" → An architectural style where URLs identify resources and HTTP methods describe the action; stateless, uniform interface
- "Difference between PUT and PATCH?" → PUT replaces the entire resource; PATCH applies a partial update
- "Is GET idempotent? Is POST?" → GET yes (reading never changes state); POST no (each call creates a new resource)
- "When would you return 400 vs 422 vs 500?" → 400 for malformed requests; 422 for valid syntax but semantic errors (e.g. failed validation); 500 for unexpected server errors
- "What is middleware?" → A function in the request/response pipeline that runs before your route handler; calls next() to continue
