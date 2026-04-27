# Unit 4 · Day 2 — Deploy Frontend + Final Polish

## What you're building today

Your React frontend deployed on Vercel, pointed at the live Render backend, with CORS locked down to the production domain. After today, the full app is live and accessible from any browser.

---

## Concepts

### Static site hosting

Your Vite build output (`dist/`) is just HTML, CSS, and JS files — no server needed to run them. A **CDN** (Content Delivery Network) serves these files from edge nodes close to the user worldwide.

- **Vercel** — best for frontend; GitHub integration, instant deploys, free tier
- **Netlify** — similar to Vercel; also great
- Both detect Vite projects automatically and run `npm run build`

### SPA vs SSR vs SSG

| | SPA | SSR | SSG |
|---|---|---|---|
| Rendering | Client (browser) | Server per request | Build time |
| Examples | This app | Next.js pages | Gatsby, Astro |
| SEO | Poor (JS required) | Good | Excellent |
| Speed | Slow first load | Fast first load | Fastest |
| Best for | Authenticated apps | Public content | Blogs, docs |

Your app is a **SPA** — fine for an authenticated app where SEO doesn't matter.

### Production environment variables in Vite

Vite exposes env vars prefixed with `VITE_` to the browser bundle:

```ts
// src/services/api.ts
const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL
})
```

`.env.production` (not committed):
```
VITE_API_URL=https://your-app.onrender.com
```

In Vercel: set `VITE_API_URL` in the environment variables dashboard — Vercel injects it at build time.

### Locking down CORS for production

In development: `origin: 'http://localhost:5173'`
In production: `origin: 'https://tennis-log.vercel.app'`

Use an environment variable to switch:

```ts
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:5173'
}))
```

Set `FRONTEND_URL=https://tennis-log.vercel.app` in Render's environment variables.

### What a CDN is

A CDN caches your static files at servers (edge nodes) distributed globally. When a user in Sydney loads your app, they get files from a Sydney edge node — not your origin server in Virginia. This cuts load time from ~200ms to ~20ms for static assets.

---

## Things to watch out for

**SPA routing on Vercel** — when a user navigates directly to `https://yourapp.vercel.app/sessions`, Vercel looks for a `sessions.html` file (which doesn't exist) and returns 404. Fix with a `vercel.json` rewrite:

```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/" }]
}
```

**Vite env vars at build time** — `import.meta.env.VITE_API_URL` is replaced with its value during the build, not at runtime. If you change the value in Vercel, you must redeploy for it to take effect.

**CORS mismatch** — after deploying, if the browser shows CORS errors, check: (1) the `FRONTEND_URL` env var is set on Render with your exact Vercel domain (no trailing slash); (2) Render redeployed after you set the variable.

**Render cold starts during demo** — if you're demoing and the backend has gone to sleep, hit the health endpoint a minute before to wake it up.

---

## Today's steps (for reference)

1. Add `VITE_API_URL` to `frontend/.env.production` pointing at your Render URL
2. Update `api.ts` to use `import.meta.env.VITE_API_URL` as the axios `baseURL`
3. Run `npm run build` locally — confirm no TypeScript errors
4. Add `vercel.json` with SPA rewrites
5. Push to GitHub
6. Create a Vercel account; import the frontend repo; set `VITE_API_URL` in Vercel env vars
7. Deploy; copy your Vercel domain
8. Update `FRONTEND_URL` on Render to your Vercel domain; redeploy backend
9. Full end-to-end test on live URLs: register → log sessions → get AI feedback

---

## Interview questions to be able to answer after today

- "SPA vs SSR vs SSG — when would you use each?" → SPA for authenticated apps (SEO irrelevant); SSR for public content needing fast first load and SEO; SSG for static content like docs and blogs
- "What's a CDN and why does it speed up your site?" → Distributed cache of static files at edge nodes worldwide; serves files from the nearest node, dramatically reducing latency
- "What's the build output of a Vite project?" → Static HTML/CSS/JS files in `dist/` with content-hashed filenames for cache busting; no server required to serve them
