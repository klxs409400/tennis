# Unit 1 · Day 6 — Connect Frontend to Backend

## What you're building today

Wire the React frontend to the Express backend. After today, creating a session in the form persists it to PostgreSQL and the list fetches live data. This completes Unit 1.

---

## Concepts

### CORS

- **Same-origin policy** — browsers block JS from making requests to a different origin (protocol + domain + port)
- `http://localhost:5173` (Vite) and `http://localhost:3000` (Express) are different origins
- **CORS** = Cross-Origin Resource Sharing — the server opts in by sending response headers that tell the browser "this origin is allowed"
- `Access-Control-Allow-Origin: http://localhost:5173` is the key header

In Express:
```ts
import cors from 'cors'
app.use(cors({ origin: 'http://localhost:5173' }))
```

### fetch vs axios

| | fetch | axios |
|---|---|---|
| Built-in | Yes | No (npm install) |
| JSON parsing | Manual (`res.json()`) | Automatic |
| Error on 4xx/5xx | No (must check `res.ok`) | Yes (throws) |
| Interceptors | No | Yes |
| TypeScript | Generic | Generic |

Axios is worth the extra dependency for any non-trivial app — interceptors let you attach auth headers globally.

### useEffect

```tsx
useEffect(() => {
  // runs after render
  fetchSessions().then(setSessions)
}, []) // [] = run once on mount
```

- `[]` dependency array = run once when the component mounts
- `[userId]` = re-run whenever `userId` changes
- No array = run after every render (almost never what you want)
- Cleanup: return a function to cancel subscriptions or timers

### Loading / error / success states

Every async call has three states. Model all three in your component:

```tsx
const [sessions, setSessions] = useState<TrainingSession[]>([])
const [loading, setLoading] = useState(true)
const [error, setError] = useState<string | null>(null)
```

If you only model the success state, users see a blank screen while loading and have no way to know something went wrong.

### Axios interceptors

```ts
// services/api.ts
const api = axios.create({ baseURL: 'http://localhost:3000' })

api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})
```

The interceptor runs before every request — you set it up once and every `api.get(...)` / `api.post(...)` call automatically includes the token.

---

## Things to watch out for

**CORS must be added before routes** — `app.use(cors(...))` needs to be registered before your route middleware in `index.ts`. Otherwise the CORS headers won't be on the response.

**Stale closures in useEffect** — if you reference state inside `useEffect` without including it in the dependency array, you'll read the value from when the effect was first created. Add it to the deps array or use a functional state update.

**Triggering a list refresh after create** — the simplest pattern is a `refreshKey` state: increment it after a successful create, include it in `useEffect`'s dependency array. This re-fetches the list.

**`axios` error shape** — when the server returns 4xx/5xx, axios throws. The error is `AxiosError` — the response body is at `err.response?.data`, not `err.message`.

---

## Today's steps (for reference)

1. In `backend/`, install `cors` and `@types/cors`
2. Add `app.use(cors({ origin: 'http://localhost:5173' }))` before your routes
3. In `frontend/`, install `axios`
4. Create `src/services/api.ts` — axios instance with `baseURL: 'http://localhost:3000'`
5. Add `getSessions()` and `createSession()` functions to `api.ts`
6. Update `<SessionList />` to fetch real data with `useEffect`; add loading + error states
7. Update `<SessionForm />` to call `createSession()` on submit, then trigger a list refresh
8. Test the full flow: create a session → see it appear in the list

---

## Interview questions to be able to answer after today

- "What is CORS? Why does it exist?" → Browser security policy that blocks cross-origin requests; the server opts in via response headers
- "What is `useEffect` for? What goes in the dependency array?" → Runs side effects after render; the array controls when it re-runs — empty means once on mount, listed values mean re-run when they change
- "What's a stale closure?" → A closure that captures an old value of a variable because the function wasn't re-created when the value changed — common bug in useEffect with missing dependencies
