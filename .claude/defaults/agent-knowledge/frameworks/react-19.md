---
technology: react
version: "19"
versionRange: ">=19.0.0 <20.0.0"
aiConfidence: Low
context7Available: true
dependencies: []
supersedes: react-18
lastUpdated: 2026-02-01
---

# React 19 Agent Knowledge

## AI Training Context

| Aspect | Status |
|--------|--------|
| **AI Trained On** | 18.x |
| **Gap Level** | Major |
| **Confidence** | Low |
| **Context7** | Available |

**Training Gap Analysis:**
- AI has strong 18.x patterns
- 19.x introduces: `use()` hook, Actions, improved ref handling
- React Compiler (automatic memoization) changes optimization patterns
- Document metadata support built-in
- `ref` is now a regular prop (no `forwardRef` needed)

---

## Critical Patterns (Embed in Agent)

### The use() Hook

New hook for reading resources during render:

```typescript
// Reading a promise
import { use } from 'react'

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise) // Suspends until resolved
  return <div>{user.name}</div>
}

// Reading context (replaces useContext)
function ThemeButton() {
  const theme = use(ThemeContext) // Same as useContext
  return <button className={theme}>Click</button>
}
```

**Key differences from `useContext`:**
- `use()` can be called conditionally
- `use()` works with promises (suspends)
- `use()` can be called in loops

### Actions (Form Handling)

Actions are functions that handle form submissions:

```typescript
// Using action prop
function LoginForm() {
  async function login(formData: FormData) {
    'use server' // Server Action
    const email = formData.get('email')
    // ...authenticate
  }

  return (
    <form action={login}>
      <input name="email" type="email" />
      <button type="submit">Login</button>
    </form>
  )
}

// Using useActionState (replaces useFormState)
import { useActionState } from 'react'

function SubmitForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState, formData) => {
      const result = await submitData(formData)
      return result
    },
    { error: null }
  )

  return (
    <form action={formAction}>
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
      {state.error && <p>{state.error}</p>}
    </form>
  )
}
```

### useOptimistic

For optimistic UI updates:

```typescript
import { useOptimistic } from 'react'

function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  )

  async function addTodo(formData: FormData) {
    const newTodo = { id: Date.now(), text: formData.get('text') }
    addOptimisticTodo(newTodo) // Immediately show optimistic state
    await saveTodo(newTodo)    // Then persist
  }

  return (
    <>
      <ul>{optimisticTodos.map(t => <li key={t.id}>{t.text}</li>)}</ul>
      <form action={addTodo}>
        <input name="text" />
        <button>Add</button>
      </form>
    </>
  )
}
```

### ref as a Prop (No forwardRef)

In React 19, `ref` is a regular prop:

```typescript
// React 19 - Correct
function Input({ ref, ...props }: { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} />
}

// React 18 - OUTDATED, still works but not needed
const Input = forwardRef<HTMLInputElement>((props, ref) => {
  return <input ref={ref} {...props} />
})
```

### Document Metadata

Built-in support for `<title>` and `<meta>`:

```typescript
function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <title>{post.title}</title>
      <meta name="description" content={post.summary} />
      <h1>{post.title}</h1>
      {/* ... */}
    </article>
  )
}
```

### useFormStatus

For form submission state:

```typescript
import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus()

  return (
    <button disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  )
}
```

---

## Do/Don't Table

| Do | Don't |
|----|-------|
| Use `use()` for reading promises in components | Use async/await in component body |
| Use `useActionState` for form state | Use `useFormState` (renamed) |
| Pass `ref` as a regular prop | Use `forwardRef` wrapper (still works but unnecessary) |
| Use `useOptimistic` for optimistic updates | Manually manage optimistic state |
| Use `<title>` and `<meta>` in components | Use react-helmet or next/head exclusively |
| Use `useFormStatus()` from react-dom | Create custom pending state |
| Let React Compiler handle memoization | Add `useMemo`/`useCallback` everywhere |
| Use `use()` conditionally when needed | Follow hooks rules strictly for `use()` |

---

## Verification Tasks

Before deploying code using these patterns, verify:

- [ ] `use()` hook suspends correctly with Suspense boundary
- [ ] Actions work with form submissions
- [ ] `useActionState` provides correct pending state
- [ ] `ref` works as regular prop without `forwardRef`
- [ ] `useOptimistic` updates UI immediately
- [ ] Document metadata renders in `<head>`
- [ ] React Compiler (if enabled) doesn't break existing code

---

## Common Errors and Fixes

### Error: use() called outside Suspense boundary

**Problem:** `use()` with promise needs Suspense
**Fix:** Wrap component in `<Suspense fallback={...}>`

### Error: useFormState is not a function

**Problem:** Renamed in React 19
**Fix:** Import `useActionState` from 'react' instead

### Error: Hooks can only be called at top level

**Problem:** Old hooks rules don't apply to `use()`
**Note:** `use()` CAN be called conditionally, unlike other hooks

### Error: Form action not firing

**Problem:** Action function issues
**Fix:** Ensure action is async and properly formed

---

## Context7 Usage

When working with React 19 patterns, use:

```
use context7 for react use hook
use context7 for react actions
use context7 for react 19 forms
```

Context7 is especially useful for:
- Suspense patterns with `use()`
- Form handling with Actions
- Concurrent rendering patterns
- Server Component integration

---

## Migration from React 18

### Breaking Changes

1. **useFormState → useActionState**: Renamed hook
2. **forwardRef**: Still works but optional
3. **ref callback cleanup**: Now runs on unmount
4. **Error handling**: Better error messages
5. **Suspense**: SSR streaming improvements

### Gradual Adoption

React 19 features are opt-in. Existing React 18 code works:

```typescript
// React 18 code still works
const [count, setCount] = useState(0)
const ref = useRef(null)
const context = useContext(MyContext)

// Gradually adopt React 19 patterns
const value = use(promise) // New
const theme = use(ThemeContext) // New alternative to useContext
```

---

## React Compiler Notes

If React Compiler is enabled:

- **Don't add manual memoization** - Compiler handles it
- **Remove unnecessary `useMemo`/`useCallback`** - May hurt performance
- **Keep components pure** - Compiler expects pure functions
- **Avoid side effects in render** - Will break compilation

```typescript
// With React Compiler - Don't do this
const memoizedValue = useMemo(() => compute(a, b), [a, b])
const memoizedCallback = useCallback(() => handle(x), [x])

// With React Compiler - Just write plain code
const value = compute(a, b) // Compiler memoizes automatically
const callback = () => handle(x) // Compiler optimizes
```

---

## Integration Notes

### With Next.js 15

- Server Components use React 19 features
- Client Components need `'use client'`
- Server Actions are React 19 Actions
- See `frameworks/nextjs-15.md`

### With Form Libraries

- React 19 Actions may replace form libraries
- `useActionState` handles most form state
- Consider migrating from react-hook-form for simple forms
