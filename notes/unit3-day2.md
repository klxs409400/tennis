# Unit 3 · Day 2 — AI Frontend + Polish

## What you're building today

A "Get AI Feedback" button on each session card that calls the AI endpoint, shows a loading state while waiting, displays the result, and handles errors gracefully. You'll also add empty states throughout the app.

---

## Concepts

### Handling slow API calls in the UI

Claude can take 5–15 seconds to respond. Users need feedback that something is happening:

1. **Disable the button** immediately on click — prevents duplicate requests
2. **Show a spinner or loading text** on the button
3. **Keep the rest of the UI interactive** — don't block the whole page
4. **Show elapsed time** (optional) for very slow calls

```tsx
const [loading, setLoading] = useState(false)

async function handleGetFeedback() {
  setLoading(true)
  try {
    const suggestion = await getAISuggestion(session.id)
    setSuggestion(suggestion)
  } catch (err) {
    setError('Failed to get feedback. Try again.')
  } finally {
    setLoading(false)
  }
}

<button onClick={handleGetFeedback} disabled={loading}>
  {loading ? 'Getting feedback...' : 'Get AI Feedback'}
</button>
```

### Optimistic updates vs waiting

- **Optimistic update** — update the UI immediately, assume success, roll back on failure. Great for low-stakes actions (like/unlike, toggle). Bad for AI feedback (you don't know the content yet).
- **Waiting for server** — show loading, update on success. Correct for AI feedback.

Use optimistic updates when the success rate is near 100% and the action is cheap to reverse. For AI calls: always wait.

### The three states every async screen needs

Every component that loads async data must handle:

| State | What to show |
|-------|-------------|
| Loading | Spinner, skeleton, disabled button |
| Error | Error message + retry option |
| Empty | Helpful empty state (not just a blank list) |
| Success | The actual content |

An empty state is not the same as a loading state. If a user has no sessions yet, show "No sessions yet — add your first one!" not a blank list.

---

## Things to watch out for

**`finally` always runs** — use `finally` to reset `loading` to `false`, not just in the success path. If you only set `setLoading(false)` in the success branch, a network error leaves the button permanently disabled.

**Per-item loading state** — if you have multiple sessions, the loading state should be per-session, not global. Otherwise clicking "Get Feedback" on session A disables buttons on sessions B and C too.

```tsx
const [loadingId, setLoadingId] = useState<string | null>(null)
// ...
setLoadingId(session.id)
// ...
setLoadingId(null)
// in render:
disabled={loadingId === session.id}
```

**Displaying AI text** — the AI response may contain newlines and bullet points. Use `white-space: pre-wrap` in CSS or split on `\n` to render it properly instead of one collapsed line.

---

## Today's steps (for reference)

1. Add a "Get AI Feedback" button to each session card in `<SessionList />`
2. On click: call `POST /api/sessions/:id/ai-suggestion`
3. While waiting: show loading text on the button and disable it
4. On success: display the AI suggestion text below the session card
5. On error: show a user-friendly error message with a retry option
6. Add an empty state to `<SessionList />` for when there are no sessions
7. Review all loading/error/empty states across the app — make sure none are missing

---

## Interview questions to be able to answer after today

- "How would you UX a 10-second API call?" → Immediate feedback (loading state, disabled button); keep rest of UI interactive; consider streaming or showing partial results; set a timeout with a friendly error if it takes too long
- "What's an optimistic update?" → Updating the UI before the server confirms success; fast UX, roll back on failure; appropriate for high-confidence, reversible actions
- "Debouncing vs throttling?" → Debounce: wait until the user stops typing, then fire once (search input); throttle: fire at most once per interval regardless of how often triggered (scroll handler, resize)
