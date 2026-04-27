# Unit 2 · Day 3 — Frontend Auth

## What you're building today

Login and register pages, a React Context for auth state, an axios interceptor that attaches the JWT to every request, and protected routes that redirect unauthenticated users to `/login`.

---

## Concepts

### Where to store the JWT

| Storage | XSS risk | CSRF risk | Notes |
|---------|----------|-----------|-------|
| `localStorage` | High (JS readable) | None | Simple; vulnerable if XSS exists |
| `httpOnly` cookie | None (JS can't read) | High | Needs CSRF tokens; more complex |
| Memory (React state) | Low | None | Lost on page refresh |

This project uses `localStorage` — simpler for a learning project. In production, `httpOnly` cookies are more secure if you also add CSRF protection.

### Axios request interceptors

```ts
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})
```

Set up once in `api.ts`. Every subsequent `api.get(...)` or `api.post(...)` call automatically includes the token — you don't repeat this logic in every component.

### React Context API

Context provides global state without threading props through every component (prop drilling).

```tsx
// src/context/AuthContext.tsx
const AuthContext = createContext<AuthContextType | null>(null)

export function AuthProvider({ children }: { children: ReactNode }) {
  const [token, setToken] = useState<string | null>(localStorage.getItem('token'))

  function login(newToken: string) {
    setToken(newToken)
    localStorage.setItem('token', newToken)
  }

  function logout() {
    setToken(null)
    localStorage.removeItem('token')
  }

  return <AuthContext.Provider value={{ token, login, logout }}>{children}</AuthContext.Provider>
}

export const useAuth = () => useContext(AuthContext)!
```

### React Router

```tsx
// main.tsx or App.tsx
<BrowserRouter>
  <Routes>
    <Route path="/login" element={<LoginPage />} />
    <Route path="/register" element={<RegisterPage />} />
    <Route path="/sessions" element={
      <ProtectedRoute><SessionsPage /></ProtectedRoute>
    } />
  </Routes>
</BrowserRouter>
```

### Protected route component

```tsx
function ProtectedRoute({ children }: { children: ReactNode }) {
  const { token } = useAuth()
  if (!token) return <Navigate to="/login" replace />
  return <>{children}</>
}
```

---

## Things to watch out for

**XSS vs CSRF** — XSS (Cross-Site Scripting): injected JS reads your localStorage token. CSRF (Cross-Site Request Forgery): a malicious site tricks your browser into making a request using your cookie. `localStorage` is vulnerable to XSS; `httpOnly` cookies are vulnerable to CSRF. Neither is perfect — know the trade-off.

**Reading token on initial load** — `localStorage.getItem('token')` as the `useState` initial value means the user stays logged in across page refreshes. Without this, every refresh logs them out.

**Redirect after login** — after a successful login, use React Router's `useNavigate` to send the user to `/sessions`. Don't just set the token and leave them on the login page.

**401 response interceptor** — optionally add a response interceptor to automatically log out when the server returns 401 (expired token):

```ts
api.interceptors.response.use(
  res => res,
  err => {
    if (err.response?.status === 401) logout()
    return Promise.reject(err)
  }
)
```

---

## Today's steps (for reference)

1. Install `react-router-dom`; wrap `App.tsx` in `<BrowserRouter>`
2. Create `src/context/AuthContext.tsx` with `login`, `logout`, and token state
3. Wrap the app in `<AuthProvider>`
4. Add axios request interceptor in `api.ts` to attach the token
5. Build `/register` and `/login` pages with forms that call the auth API
6. Build `<ProtectedRoute>` that redirects to `/login` if no token
7. Wrap the sessions page in `<ProtectedRoute>`
8. Test: access sessions without logging in → redirected; log in → sessions load

---

## Interview questions to be able to answer after today

- "XSS vs CSRF — what's the difference, what defends against each?" → XSS: injected JS steals data (defend with CSP, sanitize input); CSRF: forged cross-origin requests (defend with CSRF tokens or SameSite cookies)
- "Where would you store a JWT in the browser?" → httpOnly cookie (more secure, no JS access) vs localStorage (simpler, XSS risk) — know the trade-off
- "How would you implement protected routes in React?" → Wrapper component that reads auth state and renders Navigate to /login if unauthenticated
- "What's prop drilling and how does Context solve it?" → Passing props through many layers just to reach a deep child; Context provides a global value any descendant can read without intermediaries
