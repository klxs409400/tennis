# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tennis Training Log — a full-stack web app where users can log tennis training sessions, view session history, and get AI-generated coaching feedback via the Anthropic API. The 13-day build plan lives in `name.md`.

## Stack

- **Backend:** Node.js + TypeScript + Express + Prisma ORM + PostgreSQL
- **Frontend:** React + TypeScript + Vite + axios
- **Auth:** JWT (signed with a secret, stored in localStorage) + bcrypt password hashing
- **AI:** Anthropic API (Claude) — generates coaching feedback per training session

## Project structure (target)

```
tennis-log/
├── backend/
│   ├── src/
│   │   ├── index.ts              # Express server entry point
│   │   ├── middleware/auth.ts    # JWT verification middleware
│   │   └── routes/
│   │       ├── auth.ts           # POST /api/auth/register, /login
│   │       └── sessions.ts       # CRUD + AI suggestion endpoints
│   ├── prisma/
│   │   └── schema.prisma
│   ├── .env                      # DB URL, JWT_SECRET, ANTHROPIC_API_KEY (never commit)
│   ├── package.json
│   └── tsconfig.json
└── frontend/
    ├── src/
    │   ├── main.tsx
    │   ├── App.tsx
    │   ├── components/
    │   ├── pages/
    │   ├── services/api.ts       # axios wrapper with base URL + JWT interceptor
    │   └── types/
    ├── package.json
    └── vite.config.ts
```

## Commands

### Backend
```bash
cd backend
npm run dev          # ts-node-dev / tsx with auto-restart
npm run build        # tsc → dist/
npm start            # node dist/index.js
npx prisma migrate dev --name <name>   # create + run migration
npx prisma studio    # GUI to inspect DB
```

### Frontend
```bash
cd frontend
npm run dev          # Vite dev server (http://localhost:5173)
npm run build        # production build → dist/
npm run preview      # serve production build locally
```

## Key architecture decisions

**Auth flow:** Register/login endpoints return a JWT. The frontend stores it in localStorage and attaches it via an axios interceptor (`Authorization: Bearer <token>`) to every request. The `authMiddleware` on the backend verifies the JWT, extracts `userId`, and attaches it to `req.user`.

**Data ownership:** All session queries filter by `userId` to prevent IDOR — users can only see and modify their own data. The `TrainingSession` model has a `userId` foreign key.

**AI endpoint:** `POST /api/sessions/:id/ai-suggestion` fetches the session + its coach suggestions, sends a structured prompt to Claude, saves the result as an `AISuggestion` record, and returns it. API key is in `.env` only.

**CORS:** Backend must allow the frontend origin explicitly. In production, lock it down to the Vercel domain only.

**Error handling:** A global Express error handler catches thrown errors and returns structured JSON with appropriate status codes.

## Data models

```prisma
model User {
  id           String            @id @default(cuid())
  email        String            @unique
  passwordHash String
  sessions     TrainingSession[]
  createdAt    DateTime          @default(now())
}

model TrainingSession {
  id              String            @id @default(cuid())
  userId          String
  user            User              @relation(fields: [userId], references: [id])
  date            DateTime
  durationMinutes Int
  focusArea       String
  notes           String
  coachSuggestions CoachSuggestion[]
  aiSuggestions   AISuggestion[]
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
}

model CoachSuggestion {
  id        String          @id @default(cuid())
  sessionId String
  session   TrainingSession @relation(fields: [sessionId], references: [id])
  content   String
  createdAt DateTime        @default(now())
}

model AISuggestion {
  id        String          @id @default(cuid())
  sessionId String
  session   TrainingSession @relation(fields: [sessionId], references: [id])
  content   String
  createdAt DateTime        @default(now())
}
```

## Environment variables (backend `.env`)

```
DATABASE_URL="postgresql://..."
JWT_SECRET="..."
ANTHROPIC_API_KEY="sk-ant-..."
PORT=3000
```
