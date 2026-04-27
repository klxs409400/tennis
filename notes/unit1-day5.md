# Unit 1 · Day 5 — React Frontend Skeleton

## What you're building today

A React + TypeScript frontend with Vite, structured into components and pages. You'll build `<SessionList />` and `<SessionForm />` with hardcoded data — no backend connection yet. The goal is to understand the component model before adding real data.

---

## Concepts

### Why React

- **Component model** — UI is a tree of reusable functions that return JSX
- **Declarative** — you describe *what* the UI should look like given state; React figures out *how* to update the DOM
- **Virtual DOM** — React keeps an in-memory representation of the DOM and diffs it against the real DOM, only applying the minimum necessary changes

### Vite vs Create React App

| | Vite | Create React App |
|---|---|---|
| Dev server startup | ~300ms | 5–30s |
| HMR (hot reload) | Near-instant | Slow |
| Build tool | Rollup (esbuild for dev) | Webpack |
| Status | Actively maintained | Deprecated |

Always use Vite for new projects.

### JSX

JSX is syntactic sugar for `React.createElement()`. This:

```tsx
const el = <h1 className="title">Hello</h1>
```

compiles to:

```ts
const el = React.createElement('h1', { className: 'title' }, 'Hello')
```

Key rules: use `className` not `class`; every JSX expression must have one root element (or `<>` fragment); expressions go in `{}`.

### useState and one-way data flow

```tsx
const [sessions, setSessions] = useState<TrainingSession[]>([])
```

- State lives in a component; updating it via the setter re-renders the component
- **Props flow down** — parent passes data to child via props
- **Events bubble up** — child notifies parent by calling a callback prop

```tsx
// Parent
<SessionForm onSubmit={(data) => setSessions([...sessions, data])} />

// Child
<button onClick={() => onSubmit(formData)}>Save</button>
```

### Controlled components

An input is **controlled** when React state drives its value:

```tsx
const [notes, setNotes] = useState('')
<input value={notes} onChange={e => setNotes(e.target.value)} />
```

The input never has its own state — every keystroke updates React state, which re-renders the input. This is the standard pattern for forms in React.

---

## Things to watch out for

**`key` prop in lists** — when rendering arrays with `.map()`, every element needs a unique `key` prop. React uses it to track which items changed. Use the item's `id`, not the array index.

```tsx
sessions.map(s => <SessionCard key={s.id} session={s} />)
```

**TypeScript + React** — define an interface for your props. Without it you'll get type errors when passing data between components.

```tsx
interface Props {
  sessions: TrainingSession[]
}
export function SessionList({ sessions }: Props) { ... }
```

**`npm run dev` inside `frontend/`** — Vite's dev server only works when you run it from the `frontend/` directory, not the project root.

---

## Today's steps (for reference)

1. In `frontend/`, run `npm create vite@latest . -- --template react-ts`
2. Run `npm install` then `npm run dev` — confirm the default app loads
3. Create folders: `src/components/`, `src/pages/`, `src/services/`, `src/types/`
4. Define a `TrainingSession` TypeScript interface in `src/types/index.ts`
5. Build `<SessionList />` — accepts a `sessions` prop and renders each in a list (hardcoded data)
6. Build `<SessionForm />` — controlled inputs for date, duration, focusArea, notes; `console.log` on submit
7. Render both in `App.tsx` and confirm they display

---

## Interview questions to be able to answer after today

- "Why React? What's the virtual DOM?" → Declarative component model; virtual DOM diffs minimize real DOM updates for better performance
- "Controlled vs uncontrolled components?" → Controlled: React state drives input value; uncontrolled: DOM manages its own state (via ref). Controlled is preferred — predictable, testable
- "What's a key prop and why does React need it?" → Unique identifier for list items; lets React efficiently reconcile which items were added, removed, or reordered
- "What re-renders a component?" → Calling setState/useState setter, receiving new props, or parent re-rendering
