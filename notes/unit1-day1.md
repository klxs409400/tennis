# Unit 1 · Day 1 — Project Setup & First Server

## What you're building today

A backend server that responds to `GET /health` with `{ status: "ok" }`.
Small goal, but it forces you to understand every tool in the chain.

---

## Concepts

### Node.js

- A **runtime** — it lets JavaScript run outside the browser, on your machine/server
- Built on V8 (the same JS engine Chrome uses)
- **Single-threaded** — only one thing runs at a time in JS land
- But: I/O (file reads, network calls) is handled by `libuv` in the background, so waiting for a DB query doesn't block everything else
- Key insight: Node is great for I/O-heavy servers (APIs), not for CPU-heavy work (video encoding)

### npm & package.json

- `npm` = Node Package Manager — downloads libraries from the internet
- `package.json` = your project's manifest: name, version, scripts, dependencies
- `dependencies` — needed to **run** the app (Express, Prisma)
- `devDependencies` — needed only to **develop** it (TypeScript, ts-node-dev, type definitions)
- `node_modules/` = where downloaded packages live — **never commit this to git**

### TypeScript

- A superset of JavaScript — every `.js` file is valid `.ts`
- Adds **static types**: you declare what shape data has, and the compiler catches mistakes before you run the code
- TypeScript compiles (`
tsc`) → plain JavaScript — Node.js runs the JS, not the TS
- `tsconfig.json` controls the compiler: which JS version to target, where to output files, how strict to be

Key `tsconfig.json` options to know:

```json
{
  "compilerOptions": {
    "target": "ES2020", // which JS version to output
    "module": "commonjs", // module system (Node uses CommonJS)
    "outDir": "./dist", // where compiled JS goes
    "rootDir": "./src", // where your TS source lives
    "strict": true, // enables all strict type checks — always use this
    "esModuleInterop": true // makes imports work nicely with CommonJS packages
  }
}
```

### Express

- A minimal web framework for Node.js
- Handles: routing (which URL does what), middleware (functions that run between request and response), JSON parsing
- Alternative to writing raw `http.createServer()` — much less boilerplate

### CommonJS vs ES Modules

This **will** confuse you today. Here's the cheat sheet:

|         | CommonJS (old)                       | ES Modules (modern)             |
| ------- | ------------------------------------ | ------------------------------- |
| Import  | `const express = require('express')` | `import express from 'express'` |
| Export  | `module.exports = ...`               | `export default ...`            |
| Used by | Node.js by default                   | Browsers, modern bundlers       |

In this project: `tsconfig.json` sets `"module": "commonjs"` so Node can run the compiled output. But you write `import` syntax in TypeScript — the compiler converts it to `require` automatically.

---

## Things to watch out for

**`@types/` packages** — TypeScript doesn't know the shape of Express or Node built-ins by default. The `@types/express` and `@types/node` packages add that knowledge. Without them you'll get "Could not find declaration file" errors.

**`ts-node-dev`** — runs TypeScript directly without a manual compile step, and restarts when files change (like nodemon but for TS). Your `dev` script uses this. You only need `tsc` for production builds.

**Port conflicts** — if `npm run dev` fails with "address already in use", something is already on port 3000. Kill it with `lsof -ti:3000 | xargs kill` or just change the port.

**The dist/ folder** — when you run `tsc`, compiled JS appears in `dist/`. Don't edit files there — they get overwritten every build. Always edit files in `src/`.

---

## Today's steps (for reference)

1. Create `tennis-log/backend/` and `tennis-log/frontend/`
2. In `backend/`: `npm init -y`
3. Install dependencies
4. Create `tsconfig.json`
5. Create `src/index.ts` with a minimal Express app
6. Add `GET /health` route
7. Add `dev` script to `package.json`
8. Run and verify

---

## Interview questions to be able to answer after today

- "What is Node.js? Is it single-threaded?" → Yes, but libuv handles async I/O in the background
- "What does TypeScript give you over JavaScript?" → Compile-time type safety, autocomplete, safer refactors
- "Difference between `dependencies` and `devDependencies`?" → Runtime vs build-time only
- "What's the event loop?" → Start thinking about it — full answer comes on Day 4
