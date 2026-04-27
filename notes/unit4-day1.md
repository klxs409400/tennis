# Unit 4 · Day 1 — Deploy Backend + Database

## What you're building today

Your Express backend running live on Render with a managed PostgreSQL database. After today, `GET https://your-app.onrender.com/health` returns `{ status: "ok" }` from a real server on the internet.

---

## Concepts

### The 12-factor app (relevant factors)

A methodology for building deployable, scalable apps:

1. **Config in environment variables** — never hardcode secrets or environment-specific values in code
2. **Disposability** — the app starts fast and shuts down gracefully
3. **Dev/prod parity** — keep dev and production as similar as possible
4. **Logs as event streams** — write to stdout, let the platform collect logs

Factor 1 is the most important for today: `DATABASE_URL`, `JWT_SECRET`, `ANTHROPIC_API_KEY`, and `PORT` all come from environment variables — not from a committed `.env` file.

### Hosting platforms

| Platform | Free tier | Postgres | Notes |
|----------|-----------|----------|-------|
| Render | Yes (spins down) | Yes | Easiest setup |
| Railway | Trial credits | Yes | Fast deploys |
| Fly.io | Yes | Via extension | More control |

Recommended: **Render** — simple GitHub integration, managed Postgres, no credit card needed for free tier.

### The build pipeline

What Render runs to deploy your backend:

```
npm install          → installs dependencies
npm run build        → tsc → compiles TS to dist/
npm start            → node dist/index.js
```

You must have `"build": "tsc"` and `"start": "node dist/index.js"` in `package.json` scripts.

### Managed Postgres

Options:
- **Render Postgres** — provisioned in the same dashboard, free tier available
- **Neon** — serverless Postgres, generous free tier, fast cold starts
- **Supabase** — Postgres + auth + storage, more features than needed here

All give you a `DATABASE_URL` connection string. You paste it into Render's environment variable settings — never in code.

### Production migrations

In dev: `npx prisma migrate dev` (creates + applies + generates client).
In production: `npx prisma migrate deploy` — applies pending migrations only, no prompts, safe for CI/CD.

Run this once after the first deploy: via Render's shell or as part of the start command.

---

## Things to watch out for

**Free tier Render apps spin down** — after 15 minutes of inactivity, the app sleeps and takes ~30 seconds to wake on the next request. Acceptable for a portfolio project; upgrade for anything real.

**`dist/` in `.gitignore`** — you don't commit compiled JS. Render runs `npm run build` on each deploy, so it compiles fresh. If `dist/` is committed, you risk deploying stale compiled files.

**`PORT` environment variable** — Render assigns a dynamic port via the `PORT` environment variable. Your `index.ts` must use `process.env.PORT || 3000`, not a hardcoded `3000`.

**CORS in production** — currently your backend allows `http://localhost:5173`. After deploying the frontend, update the CORS `origin` to your Vercel domain. Leaving `localhost` in production CORS is harmless but sloppy.

---

## Today's steps (for reference)

1. Verify `"build": "tsc"` and `"start": "node dist/index.js"` are in backend `package.json`
2. Update `src/index.ts` to use `process.env.PORT || 3000`
3. Add `.gitignore` in `backend/`: exclude `node_modules/`, `dist/`, `.env`
4. Push all code to GitHub
5. Create a Render account; create a new Web Service from your GitHub repo
6. Set build command: `npm install && npm run build`; start command: `npm start`
7. Create a Postgres instance and copy the connection string
8. Set all environment variables in Render: `DATABASE_URL`, `JWT_SECRET`, `ANTHROPIC_API_KEY`, `PORT`
9. Deploy; then run `npx prisma migrate deploy` to apply migrations
10. Test `GET /health` on the live URL

---

## Interview questions to be able to answer after today

- "What are the 12-factor app principles?" → Config in env vars, stateless processes, logs as streams, dev/prod parity — methodology for deployable, scalable apps
- "What's the difference between build time and runtime?" → Build time: tsc compiles TS → JS, dependencies installed; runtime: Node executes the compiled JS to serve requests
- "How would you handle DB migrations in production?" → `prisma migrate deploy` (not dev) — applies pending migrations without prompting; run as part of deploy pipeline or one-time after deploy
