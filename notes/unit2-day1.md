# Unit 2 · Day 1 — Auth Concepts + Register/Login Endpoints

## What you're building today

Register and login endpoints that hash passwords with bcrypt and return JWTs. By the end, you can POST to `/api/auth/register`, get a token back, and use it to prove identity on future requests.

---

## Concepts

### Authentication vs Authorization

- **Authentication** — who are you? (identity) — "are you logged in?"
- **Authorization** — what can you do? (permissions) — "are you allowed to delete this?"
- Auth always happens before authz. Today is authentication only.

### Why you never store passwords in plain text

If your database is compromised, attackers get every user's password — and since people reuse passwords, they now have access to their email, bank, etc. You store a **hash** instead.

### Hashing vs encryption

| | Hashing | Encryption |
|---|---|---|
| Reversible? | No (one-way) | Yes (with key) |
| Purpose | Verify without storing | Store and retrieve |
| Examples | bcrypt, SHA-256 | AES, RSA |

For passwords: always hash (one-way). You never need to recover the original — you just re-hash what the user types and compare.

### bcrypt and salts

**bcrypt** is a slow hashing algorithm designed specifically for passwords:
- "Slow by design" — takes ~100ms per hash, making brute-force attacks expensive
- Adds a **salt** — a random string prepended to the password before hashing

A salt prevents **rainbow table attacks** (pre-computed hash → password lookups). Even if two users have the same password, their hashes are different because their salts differ.

```ts
const hash = await bcrypt.hash(password, 10) // 10 = cost factor (rounds)
const match = await bcrypt.compare(password, hash) // true or false
```

### JWTs

A JWT (JSON Web Token) is a compact, self-contained token with three parts separated by dots:

```
header.payload.signature
```

- **Header** — algorithm used (e.g. HS256)
- **Payload** — claims (data), e.g. `{ userId: 'abc', iat: 1234, exp: 5678 }`
- **Signature** — HMAC of header + payload, signed with `JWT_SECRET`

Key point: **JWTs are signed, not encrypted**. Anyone can base64-decode the payload and read it. The signature only proves it wasn't tampered with. Never put sensitive data (passwords, credit cards) in a JWT payload.

```ts
const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET!, { expiresIn: '7d' })
const decoded = jwt.verify(token, process.env.JWT_SECRET!) // throws if invalid
```

### Sessions vs JWTs

| | Sessions | JWTs |
|---|---|---|
| State | Server stores session in DB/Redis | Stateless — all info in token |
| Revocation | Easy (delete from DB) | Hard (must use a blocklist) |
| Scalability | Requires shared session store | Trivially scalable |
| Best for | Apps needing instant revocation | APIs, microservices |

---

## Things to watch out for

**`JWT_SECRET` must be secret and long** — if it leaks, anyone can forge tokens. Generate a random 32+ character string and put it only in `.env` (never commit `.env`).

**401 vs 403** — return 401 ("Unauthorized") when the user is not authenticated (missing/invalid token or wrong credentials). Return 403 ("Forbidden") when they are authenticated but not allowed to do the specific thing.

**bcrypt is async** — `bcrypt.hash` and `bcrypt.compare` return Promises. Always `await` them.

**Don't reveal whether email exists** — returning "email not found" vs "wrong password" lets attackers enumerate valid emails. Return a generic "Invalid credentials" for both cases.

---

## Today's steps (for reference)

1. Add `User` model to `schema.prisma`: `id`, `email` (unique), `passwordHash`, `createdAt`
2. Run `npx prisma migrate dev --name add-user`
3. Install `bcrypt @types/bcrypt jsonwebtoken @types/jsonwebtoken`
4. Create `src/routes/auth.ts` with an Express Router
5. Implement `POST /api/auth/register` — validate, hash password, create user, return JWT
6. Implement `POST /api/auth/login` — find user, compare password, return JWT
7. Mount auth router in `src/index.ts` at `/api/auth`
8. Test both endpoints in Postman

---

## Interview questions to be able to answer after today

- "How do you store passwords securely?" → Hash with a slow algorithm (bcrypt); never store plaintext; salts prevent rainbow table attacks
- "What's a JWT? Is it encrypted?" → Signed token with header/payload/signature; not encrypted — payload is readable; signature proves it wasn't tampered with
- "Sessions vs JWTs — which would you use?" → JWTs for stateless APIs and microservices; sessions when you need instant revocation
- "What's a salt and why does it matter?" → Random data added to a password before hashing; ensures identical passwords produce different hashes, defeating pre-computed lookup tables
