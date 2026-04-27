# Tennis Training Log — Learning Plan

**Stack:** Node.js · TypeScript · Express · PostgreSQL · Prisma · React · Vite
**Goal:** A deployed full-stack app with auth + AI integration, built from scratch.

---

## How to navigate

Each day is broken into numbered **Steps** — concrete, completable actions.
You can jump to any exact point, e.g. **Unit 2 · Day 1 · Step 3**.

- **Concepts** — read *before* coding
- **Steps** — what to build (you write the code)
- **Interview** — questions you should be able to answer after the day
- **Stretch** — only if you finish early

---

## Unit 1 — CRUD Foundation

### Unit 1 · Day 1 — Project setup & first server

**Concepts**
- What Node.js is (a JS runtime on V8) vs the browser
- What `npm` does — `package.json`, `dependencies` vs `devDependencies`
- Why TypeScript exists and what `tsconfig.json` controls
- What Express is and why it exists (vs the raw `http` module)
- CommonJS (`require`) vs ES Modules (`import`) — you'll hit this confusion

**Steps**
1. Create `tennis-log/` with two subfolders: `backend/` and `frontend/`
2. In `backend/`, run `npm init -y`
3. Install: `express typescript @types/node @types/express ts-node-dev`
4. Create `tsconfig.json` — look up what each option means before copying
5. Create `src/index.ts` and set up a minimal Express app
6. Add `GET /health` route that returns `{ status: "ok" }`
7. Add `"dev": "ts-node-dev src/index.ts"` script to `package.json`
8. Run `npm run dev` and verify the endpoint works in your browser

**Interview**
- "What is Node.js? Is it single-threaded?"
- "What does TypeScript give you over JavaScript?"
- "Difference between `dependencies` and `devDependencies`?"

**Stretch**
- Set up ESLint + Prettier
- Add a `.env` file with `dotenv` for the port number

---

### Unit 1 · Day 2 — PostgreSQL + Prisma

**Concepts**
- What a relational database is — tables, rows, primary keys, foreign keys
- SQL vs NoSQL — when to use which
- What an ORM is and why Prisma (vs raw SQL)
- What a "migration" is — versioned changes to your DB schema

**Steps**
1. Install PostgreSQL locally (Mac: Postgres.app or `brew install postgresql`)
2. Start PostgreSQL; create a database: `createdb tennis_log_dev`
3. In `backend/`, install `prisma` and `@prisma/client`
4. Run `npx prisma init` — sets up `prisma/schema.prisma` and `.env`
5. Set `DATABASE_URL` in `.env` to `postgresql://localhost/tennis_log_dev`
6. Define the `TrainingSession` model in `schema.prisma`:
   - fields: `id`, `date`, `durationMinutes`, `focusArea`, `notes`, `createdAt`, `updatedAt`
7. Run `npx prisma migrate dev --name init`
8. Open `npx prisma studio` and add one row manually to verify

**Interview**
- "What's the difference between SQL and NoSQL? When would you pick each?"
- "What's an ORM? Pros and cons?"
- "What's a migration?"
- "What's a primary key vs a foreign key?"

**Stretch**
- Open `psql` and run `SELECT * FROM "TrainingSession";` to see the raw table

---

### Unit 1 · Day 3 — First CRUD endpoints (Create + Read)

**Concepts**
- REST — resources, nouns not verbs in URLs
- HTTP methods: GET, POST, PATCH, PUT, DELETE
- Status codes: 200, 201, 400, 401, 403, 404, 500
- What middleware is in Express (functions that run before your route handler)

**Steps**
1. Add `express.json()` middleware in `src/index.ts` (needed to parse request bodies)
2. Create `src/routes/sessions.ts` and set up an Express Router
3. Mount the router in `src/index.ts` at path `/api`
4. Implement `POST /api/sessions` — read body, call `prisma.trainingSession.create`, return 201 + the created row
5. Implement `GET /api/sessions` — call `prisma.trainingSession.findMany`, return 200 + array
6. Install Postman or the Thunder Client VS Code extension
7. Test both endpoints manually — confirm correct status codes and response shapes

**Interview**
- "What is REST?"
- "Difference between PUT and PATCH?"
- "Is GET idempotent? Is POST?"
- "When would you return 400 vs 422 vs 500?"
- "What is middleware?"

**Stretch**
- Add Zod for runtime input validation on the POST body

---

### Unit 1 · Day 4 — Finish CRUD + error handling

**Concepts**
- Route parameters (`/sessions/:id`) vs query parameters (`?focus=serve`)
- `async`/`await` deeply — what a Promise actually is
- Why you need a global error handler in Express
- One-to-many relationships in Prisma

**Steps**
1. Implement `GET /api/sessions/:id` — return 404 with a message if not found
2. Implement `PATCH /api/sessions/:id` — partial update using `prisma.trainingSession.update`
3. Implement `DELETE /api/sessions/:id` — delete and return 204 (no body)
4. Add a global error-handling middleware (4 arguments: `err, req, res, next`) at the bottom of `src/index.ts`
5. Add `CoachSuggestion` model in `schema.prisma` with a `sessionId` FK to `TrainingSession`
6. Run `npx prisma migrate dev --name add-coach-suggestion`
7. Implement `POST /api/sessions/:id/coach-suggestions` — attach a suggestion to a session
8. Test all five endpoints in Postman

**Interview**
- "Explain the event loop"
- "Callbacks vs Promises vs async/await — what problem does each solve?"
- "How do you handle errors in async Express routes?"

**Stretch**
- Add pagination to `GET /api/sessions` (`?page=1&limit=10`)

---

### Unit 1 · Day 5 — React frontend skeleton

**Concepts**
- Why React — component model, declarative UI, virtual DOM
- Vite vs Create React App — why Vite wins (speed, modern tooling)
- JSX — syntactic sugar for `React.createElement`
- `useState` hook and one-way data flow (props down, events up)

**Steps**
1. In `frontend/`, run `npm create vite@latest . -- --template react-ts`
2. Run `npm install` then `npm run dev` — confirm the default app loads
3. Create folders: `src/components/`, `src/pages/`, `src/services/`, `src/types/`
4. Define a `TrainingSession` TypeScript interface in `src/types/index.ts`
5. Build `<SessionList />` — accepts a `sessions` prop and renders each item in a list (hardcoded data for now)
6. Build `<SessionForm />` — controlled inputs for date, duration, focusArea, notes; `console.log` the data on submit
7. Render both components in `App.tsx` and confirm they display

**Interview**
- "Why React? What's the virtual DOM?"
- "Controlled vs uncontrolled components?"
- "What's a key prop and why does React need it?"
- "What re-renders a component?"

**Stretch**
- Add Tailwind CSS and style the form and list

---

### Unit 1 · Day 6 — Connect frontend to backend

**Concepts**
- CORS — what it is and why browsers enforce it (same-origin policy)
- `fetch` vs `axios` — axios is friendlier (automatic JSON, interceptors, error throwing)
- The `useEffect` hook — what it's for, what the dependency array controls, common pitfalls
- Loading/error states — every async call has 3 states: loading, success, error

**Steps**
1. In `backend/`, install `cors` and `@types/cors`
2. Add `app.use(cors({ origin: 'http://localhost:5173' }))` before your routes
3. In `frontend/`, install `axios`
4. Create `src/services/api.ts` — an axios instance with `baseURL: 'http://localhost:3000'`
5. Add a `getSessions()` and `createSession()` function to `api.ts`
6. Update `<SessionList />` to fetch real data with `useEffect` on mount; add loading + error UI states
7. Update `<SessionForm />` to call `createSession()` on submit, then trigger a list refresh
8. Test the full flow: create a session in the form, see it appear in the list

**Interview**
- "What is CORS? Why does it exist?"
- "What is `useEffect` for? What goes in the dependency array?"
- "What's a stale closure?"

**Stretch**
- Add edit and delete buttons to each session card (full CRUD UI)

---

## Unit 2 — Authentication

### Unit 2 · Day 1 — Auth concepts + register/login endpoints

**Concepts**
- Authentication (who are you?) vs Authorization (what can you do?)
- Why passwords must never be stored in plain text
- Hashing vs encryption — hashing is one-way
- bcrypt and what a "salt" is
- JWTs — structure: `header.payload.signature` — signed, not encrypted

**Steps**
1. Add `User` model to `schema.prisma`: `id`, `email` (unique), `passwordHash`, `createdAt`
2. Run `npx prisma migrate dev --name add-user`
3. Install `bcrypt @types/bcrypt jsonwebtoken @types/jsonwebtoken`
4. Create `src/routes/auth.ts` with an Express Router
5. Implement `POST /api/auth/register`:
   - validate email + password in body
   - hash password with `bcrypt.hash(password, 10)`
   - create user in DB
   - sign and return a JWT with `{ userId }` as payload
6. Implement `POST /api/auth/login`:
   - find user by email; return 401 if not found
   - compare password with `bcrypt.compare`; return 401 if wrong
   - sign and return a JWT
7. Mount auth router in `src/index.ts` at `/api/auth`
8. Test both endpoints in Postman

**Interview**
- "How do you store passwords securely?"
- "What's a JWT? Is it encrypted?"
- "Sessions vs JWTs — which would you use?"
- "What's a salt and why does it matter?"

---

### Unit 2 · Day 2 — Protected routes & ownership

**Concepts**
- Auth middleware — extract token from `Authorization` header, verify, attach user to `req`
- The IDOR vulnerability — users accessing other users' data by changing an ID in the URL
- Foreign keys for data ownership

**Steps**
1. Add `userId String` field to `TrainingSession` in `schema.prisma`; add the `@relation` to `User`
2. Run `npx prisma migrate dev --name add-session-owner`
3. Create `src/middleware/auth.ts`:
   - extract `Bearer <token>` from `Authorization` header
   - call `jwt.verify`; return 401 if missing or invalid
   - attach decoded `userId` to `req.user`
4. Apply `authMiddleware` to all routes in `sessions.ts` (use `router.use(authMiddleware)`)
5. Update `POST /api/sessions` to include `userId: req.user.id` in the Prisma create call
6. Update all session queries to add `where: { userId: req.user.id }` — prevents IDOR
7. Test: register two users, create sessions as each, verify they can't see each other's data

**Interview**
- "What's the difference between authentication and authorization?"
- "What's an IDOR attack and how do you prevent it?"
- "How does middleware work in Express?"

---

### Unit 2 · Day 3 — Frontend auth

**Concepts**
- Where to store the JWT — `localStorage` vs `httpOnly` cookies — XSS vs CSRF trade-offs
- Axios request interceptors — auto-attach token to every request
- React Context API — global state without prop drilling
- Protected routes in React Router

**Steps**
1. Install `react-router-dom`; wrap `App.tsx` in `<BrowserRouter>`
2. Create `src/context/AuthContext.tsx`:
   - stores `token` and `user` in state
   - `login(token)` — saves to state + localStorage
   - `logout()` — clears state + localStorage
   - reads token from localStorage on initial load
3. Wrap the app in `<AuthProvider>`
4. Add axios request interceptor in `api.ts` — reads token from context/localStorage and attaches `Authorization: Bearer <token>`
5. Build `/register` and `/login` pages — form submits call the auth API, then `login(token)` on success
6. Build `<ProtectedRoute>` component — if no token, redirect to `/login`
7. Wrap the sessions page with `<ProtectedRoute>`
8. Test: access sessions page without logging in → redirected; log in → sessions load

**Interview**
- "XSS vs CSRF — what's the difference, what defends against each?"
- "Where would you store a JWT in the browser?"
- "How would you implement protected routes in React?"
- "What's prop drilling and how does Context solve it?"

---

## Unit 3 — AI Integration

### Unit 3 · Day 1 — AI backend endpoint

**Concepts**
- External API integration patterns
- Environment variables and secret management — never commit API keys
- Anthropic API basics: messages format, system prompts, model selection
- Prompt engineering — how prompt shape affects output quality

**Steps**
1. Sign up for an Anthropic API key at console.anthropic.com
2. Add `ANTHROPIC_API_KEY=sk-ant-...` to `backend/.env`
3. Install `@anthropic-ai/sdk`
4. Add `AISuggestion` model to `schema.prisma` with a `sessionId` FK to `TrainingSession`
5. Run `npx prisma migrate dev --name add-ai-suggestion`
6. Create `POST /api/sessions/:id/ai-suggestion` route (protected by `authMiddleware`):
   - fetch the session + its coach suggestions from DB (verify it belongs to `req.user.id`)
   - build a structured prompt describing the session and suggestions
   - call the Anthropic API with your prompt
   - save the AI response text as an `AISuggestion` record
   - return the saved suggestion
7. Test with Postman — tune the prompt until the feedback is useful

**Interview**
- "How do you manage secrets in production?"
- "How would you handle rate limits from a third-party API?"
- "How would you handle a slow third-party API that times out?"

---

### Unit 3 · Day 2 — AI frontend + polish

**Concepts**
- Handling slow API calls in the UI — Claude can take 5–15 seconds
- Optimistic updates vs waiting for server confirmation
- Empty states, error states, success states — every screen needs all three

**Steps**
1. Add a "Get AI Feedback" button to each session card in `<SessionList />`
2. On button click: call `POST /api/sessions/:id/ai-suggestion`
3. While waiting: show a loading spinner on the button (disable it to prevent double-clicks)
4. On success: display the AI suggestion text below the session card
5. On error: show a user-friendly error message with a retry option
6. Add an empty state to `<SessionList />` for when there are no sessions yet
7. Check that all loading/error/empty states look reasonable

**Interview**
- "How would you UX a 10-second API call?"
- "What's an optimistic update?"
- "Debouncing vs throttling?"

---

## Unit 4 — Deployment

### Unit 4 · Day 1 — Deploy backend + database

**Concepts**
- The 12-factor app — especially "config in environment variables"
- Hosting platforms: Render, Railway, Fly.io (Render is recommended)
- Managed Postgres: Render, Neon, Supabase
- The build pipeline: `npm install` → `tsc` → `node dist/index.js`

**Steps**
1. Add `"build": "tsc"` and `"start": "node dist/index.js"` to backend `package.json`
2. Test locally: `npm run build` then `npm start` — confirm it works
3. Add a `.gitignore` in `backend/`: exclude `node_modules/`, `dist/`, `.env`
4. Push all code to GitHub
5. Create a Render account; create a new Web Service pointing at your GitHub repo
6. Set build command to `npm install && npm run build`; start command to `npm start`
7. Create a Postgres instance (Render Postgres or Neon) and copy the connection string
8. Set environment variables in Render dashboard: `DATABASE_URL`, `JWT_SECRET`, `ANTHROPIC_API_KEY`, `PORT`
9. Deploy; once live, run `npx prisma migrate deploy` to apply migrations to production DB
10. Test `GET /health` on the live URL

**Interview**
- "What are the 12-factor app principles?"
- "What's the difference between build time and runtime?"
- "How would you handle DB migrations in production?"

---

### Unit 4 · Day 2 — Deploy frontend + final polish

**Concepts**
- Static site hosting (Vercel, Netlify) — what a CDN is
- SPA vs SSR — your app is an SPA, know the trade-offs
- Production CORS — lock it down to your exact frontend domain

**Steps**
1. In `frontend/`, add `.env.production` with `VITE_API_URL=https://your-render-url.onrender.com`
2. Update `api.ts` to use `import.meta.env.VITE_API_URL` as the axios `baseURL`
3. Run `npm run build` locally — confirm no TypeScript errors and `dist/` is generated
4. Add a `.gitignore` in `frontend/`: exclude `node_modules/`, `dist/`
5. Push to GitHub
6. Create a Vercel account; import the frontend repo; set `VITE_API_URL` in Vercel environment variables
7. Deploy; copy your Vercel domain (e.g. `https://tennis-log.vercel.app`)
8. Back in backend: update CORS to `origin: 'https://tennis-log.vercel.app'` — remove localhost
9. Redeploy backend on Render
10. Full end-to-end test on the live URLs: register → log sessions → get AI feedback

**Interview**
- "SPA vs SSR vs SSG — when would you use each?"
- "What's a CDN and why does it speed up your site?"
- "What's the build output of a Vite project?"

---

## After Day 13 — Bonus topics

Pick what interests you. These come up in senior interviews.

- **Testing** — Vitest (unit), Supertest (API), Playwright (end-to-end)
- **CI/CD** — GitHub Actions: run tests on every PR, auto-deploy on merge
- **Docker** — containerize the backend; understand images vs containers
- **Observability** — structured logging, Sentry for error tracking
- **Caching** — Redis for caching AI suggestions (expensive API calls)
- **Background jobs** — BullMQ to process AI suggestions in a queue
