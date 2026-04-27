# Unit 3 · Day 1 — AI Backend Endpoint

## What you're building today

A protected `POST /api/sessions/:id/ai-suggestion` endpoint that fetches a session, builds a prompt, calls the Anthropic API, saves the response, and returns it. After today, your app generates real AI coaching feedback.

---

## Concepts

### External API integration pattern

```
Client → Your API → Third-party API (Anthropic)
                  ← response
         Save to DB
       ← return to client
```

Your backend acts as a proxy — it keeps the API key secret, validates input, saves results, and can add rate limiting or caching. Never call third-party APIs directly from the frontend when they require secret keys.

### Environment variables and secret management

- API keys go in `.env` — never commit this file
- Access in code: `process.env.ANTHROPIC_API_KEY`
- In production (Render/Railway): set them in the dashboard, not in a deployed file
- Rotate keys if they're ever exposed (pushed to git, logged, etc.)

`.gitignore` must include:
```
.env
node_modules/
dist/
```

### Anthropic API basics

```ts
import Anthropic from '@anthropic-ai/sdk'

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })

const message = await client.messages.create({
  model: 'claude-opus-4-7',
  max_tokens: 1024,
  messages: [
    { role: 'user', content: 'Your prompt here' }
  ]
})

const text = message.content[0].type === 'text' ? message.content[0].text : ''
```

### Prompt engineering basics

The prompt shape directly affects output quality. A structured prompt with context produces better coaching than a vague one.

Good prompt structure for this use case:
```
You are a tennis coach reviewing a training session.

Session details:
- Date: {date}
- Duration: {duration} minutes
- Focus area: {focusArea}
- Notes: {notes}

Coach suggestions already given:
{coachSuggestions}

Provide specific, actionable feedback for improvement. Be concise (3-5 bullet points).
```

More context + clear output format = more useful response.

### Saving AI output to the database

Don't just return the AI response — save it as an `AISuggestion` record. This means:
- Users can view past AI feedback without re-calling the API (expensive)
- You have an audit trail
- You can add caching logic later

---

## Things to watch out for

**Verify session ownership before calling the API** — always check `session.userId === req.user!.id` before building the prompt. An attacker could trigger expensive AI calls on other users' sessions.

**`max_tokens` controls response length and cost** — Anthropic charges per token (input + output). Set a reasonable `max_tokens` limit. For coaching feedback, 512–1024 is plenty.

**API key in `.env` only** — double-check `.gitignore` includes `.env`. Run `git status` to confirm `.env` is not staged before every commit.

**Handle API failures gracefully** — the Anthropic API can time out or return errors. Wrap the call in try/catch and return a 502 (Bad Gateway) with a user-friendly message rather than a 500 with raw API error details.

---

## Today's steps (for reference)

1. Sign up for an Anthropic API key at console.anthropic.com
2. Add `ANTHROPIC_API_KEY=sk-ant-...` to `backend/.env`
3. Install `@anthropic-ai/sdk`
4. Add `AISuggestion` model to `schema.prisma` with a `sessionId` FK
5. Run `npx prisma migrate dev --name add-ai-suggestion`
6. Create `POST /api/sessions/:id/ai-suggestion` (protected):
   - fetch session + coach suggestions; verify ownership
   - build a structured prompt
   - call the Anthropic API
   - save the response as an `AISuggestion` record
   - return the saved suggestion
7. Test with Postman — tune the prompt until the feedback is useful

---

## Interview questions to be able to answer after today

- "How do you manage secrets in production?" → Environment variables set in the hosting dashboard; never in code or committed files; rotate immediately if exposed
- "How would you handle rate limits from a third-party API?" → Exponential backoff with retries; queue requests; cache responses to avoid repeat calls for identical inputs
- "How would you handle a slow third-party API that times out?" → Set a timeout on the request; return a 202 Accepted and process async (background job + webhook or polling); show loading state in the UI
