# Unit 1 · Day 2 — PostgreSQL + Prisma

## What you're building today

A live PostgreSQL database connected to your Express server via Prisma ORM. By the end, you'll have a `TrainingSession` table and be able to inspect rows visually in Prisma Studio.

---

## Concepts

### Relational databases

- Data lives in **tables** (like spreadsheets) — rows are records, columns are fields
- Every row has a **primary key** — a unique identifier (you'll use `cuid()` strings)
- Tables relate to each other via **foreign keys** — a column that points to another table's primary key
- Example: `TrainingSession.userId` is a FK pointing to `User.id`

### SQL vs NoSQL

| | SQL (PostgreSQL) | NoSQL (MongoDB) |
|---|---|---|
| Structure | Fixed schema, tables | Flexible, documents |
| Relations | First-class (JOINs) | Manual or embedded |
| Best for | Structured relational data | Flexible/hierarchical data |
| Transactions | Strong guarantees (ACID) | Varies |

This project uses PostgreSQL — the data is relational (users own sessions, sessions have suggestions).

### What an ORM is

- ORM = Object-Relational Mapper — translates between your code objects and database rows
- Without ORM: you write raw SQL strings (`SELECT * FROM "TrainingSession" WHERE id = $1`)
- With Prisma: you write `prisma.trainingSession.findUnique({ where: { id } })` — TypeScript, autocomplete, type-safe

### Prisma specifically

- `schema.prisma` — defines your data models; Prisma reads this to generate the client and migrations
- `@prisma/client` — the generated TypeScript client you import in your routes
- `prisma migrate dev` — creates a timestamped SQL migration file and runs it against the DB
- `prisma studio` — a web UI to inspect and edit your database

### What a migration is

A migration is a versioned, ordered SQL file that describes a change to your schema. Prisma generates these automatically from your `schema.prisma` changes. Never edit migration files by hand — if you need to change the schema, edit `schema.prisma` and run `migrate dev` again.

---

## Things to watch out for

**`DATABASE_URL` format** — PostgreSQL connection strings look like `postgresql://user:password@host:5432/dbname`. If you're running locally with no password, it's often `postgresql://localhost/tennis_log_dev`. A wrong URL gives a cryptic connection error.

**`npx prisma generate`** — after changing `schema.prisma`, Prisma needs to regenerate the client. `migrate dev` does this automatically, but if you change the schema without migrating, your TypeScript types won't match.

**`node_modules/.prisma`** — the generated client lives here. Never commit it. It gets regenerated on `npm install` + `prisma generate`.

**Prisma Studio port** — Studio runs on `http://localhost:5555` by default. It won't work if the DB isn't running.

---

## Today's steps (for reference)

1. Install PostgreSQL locally (Mac: Postgres.app or `brew install postgresql`)
2. Start PostgreSQL; create a database: `createdb tennis_log_dev`
3. In `backend/`, install `prisma` and `@prisma/client`
4. Run `npx prisma init` — creates `prisma/schema.prisma` and `backend/.env`
5. Set `DATABASE_URL` in `.env` to `postgresql://localhost/tennis_log_dev`
6. Define the `TrainingSession` model in `schema.prisma`
7. Run `npx prisma migrate dev --name init`
8. Open `npx prisma studio` and add one row manually to verify

---

## Interview questions to be able to answer after today

- "What's the difference between SQL and NoSQL? When would you pick each?" → SQL for structured relational data with strong consistency; NoSQL for flexible schemas or document-style data
- "What's an ORM? Pros and cons?" → Abstracts SQL into code; pro: type-safe, less boilerplate; con: hides what SQL actually runs, can produce inefficient queries
- "What's a migration?" → A versioned SQL file that applies a schema change; lets you evolve the DB schema safely over time
- "What's a primary key vs a foreign key?" → PK uniquely identifies a row in its own table; FK references a PK in another table to model a relationship
