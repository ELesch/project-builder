---
technology: vitest
version: "2"
versionRange: ">=1.0.0"
aiConfidence: High
context7Available: true
dependencies: []
lastUpdated: 2026-02-01
shared: true
sharedKnowledge: true
---

# Vitest Testing Knowledge

## AI Training Context

| Aspect | Status |
|--------|--------|
| **AI Trained On** | 1.x |
| **Gap Level** | Minor |
| **Confidence** | High |
| **Context7** | Available |

**Stable API:** Vitest API is Jest-compatible and stable across versions.

---

## Core Patterns (Embed in Agent)

### Test Structure

```typescript
// src/services/UserService.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { UserService } from './UserService'

describe('UserService', () => {
  let service: UserService

  beforeEach(() => {
    service = new UserService()
    vi.clearAllMocks()
  })

  describe('createUser', () => {
    it('creates user with valid data', async () => {
      const input = { email: 'test@example.com', name: 'Test' }

      const user = await service.createUser(input)

      expect(user).toMatchObject({
        email: 'test@example.com',
        name: 'Test',
      })
      expect(user.id).toBeDefined()
    })

    it('throws on duplicate email', async () => {
      const input = { email: 'existing@example.com', name: 'Test' }

      await expect(service.createUser(input)).rejects.toThrow(
        'Email already exists'
      )
    })
  })
})
```

### Mocking

```typescript
import { vi, Mock } from 'vitest'

// Mock modules
vi.mock('@/lib/db/client', () => ({
  prisma: {
    user: {
      create: vi.fn(),
      findUnique: vi.fn(),
      findMany: vi.fn(),
    },
  },
}))

// Mock functions
const mockFn = vi.fn()
mockFn.mockReturnValue('value')
mockFn.mockResolvedValue('async value')
mockFn.mockImplementation((x) => x * 2)

// Spy on methods
const spy = vi.spyOn(object, 'method')
spy.mockReturnValue('mocked')

// Verify calls
expect(mockFn).toHaveBeenCalledWith('arg')
expect(mockFn).toHaveBeenCalledTimes(1)

// Reset mocks
vi.clearAllMocks()  // Clear call history
vi.resetAllMocks()  // Reset to undefined
vi.restoreAllMocks() // Restore originals
```

### Testing Async Code

```typescript
// Async/await (preferred)
it('handles async operations', async () => {
  const result = await fetchData()
  expect(result).toBe('data')
})

// Promise assertions
it('rejects with error', async () => {
  await expect(asyncFn()).rejects.toThrow('Error message')
})

// Timers
import { vi, beforeEach, afterEach } from 'vitest'

beforeEach(() => {
  vi.useFakeTimers()
})

afterEach(() => {
  vi.useRealTimers()
})

it('handles setTimeout', async () => {
  const fn = vi.fn()
  setTimeout(fn, 1000)

  vi.advanceTimersByTime(1000)

  expect(fn).toHaveBeenCalled()
})
```

### Test Setup

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    globals: true,
    environment: 'jsdom', // or 'node' for backend
    setupFiles: ['./src/test/setup.ts'],
    include: ['**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
})
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest'
import { afterEach } from 'vitest'
import { cleanup } from '@testing-library/react'

afterEach(() => {
  cleanup()
})
```

### React Component Testing

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, it, expect } from 'vitest'
import { LoginForm } from './LoginForm'

describe('LoginForm', () => {
  it('renders login form', () => {
    render(<LoginForm />)

    expect(screen.getByLabelText(/email/i)).toBeInTheDocument()
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument()
    expect(screen.getByRole('button', { name: /login/i })).toBeInTheDocument()
  })

  it('submits form with credentials', async () => {
    const user = userEvent.setup()
    const onSubmit = vi.fn()

    render(<LoginForm onSubmit={onSubmit} />)

    await user.type(screen.getByLabelText(/email/i), 'test@example.com')
    await user.type(screen.getByLabelText(/password/i), 'password123')
    await user.click(screen.getByRole('button', { name: /login/i }))

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      })
    })
  })

  it('shows validation error', async () => {
    const user = userEvent.setup()
    render(<LoginForm />)

    await user.click(screen.getByRole('button', { name: /login/i }))

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument()
  })
})
```

---

## Do/Don't Table

| Do | Don't |
|----|-------|
| Use `describe` blocks to group related tests | Write flat test files |
| Use `beforeEach` for test isolation | Share state between tests |
| Test behavior, not implementation | Test internal methods directly |
| Use `userEvent` for user interactions | Use `fireEvent` for everything |
| Mock at module boundaries | Mock every function |
| Use `waitFor` for async assertions | Use arbitrary `setTimeout` |
| Name tests with expected behavior | Use vague test names |
| Test error cases | Only test happy paths |

---

## Test Organization

```
src/
├── services/
│   ├── UserService.ts
│   └── UserService.test.ts      # Colocated test
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   └── Button.test.tsx      # Colocated test
└── test/
    ├── setup.ts                  # Global setup
    ├── utils.tsx                 # Test utilities
    └── mocks/                    # Shared mocks
        └── prisma.ts
```

---

## Common Assertions

```typescript
// Equality
expect(value).toBe(exact)
expect(value).toEqual(deepEqual)
expect(value).toStrictEqual(strictDeepEqual)

// Truthiness
expect(value).toBeTruthy()
expect(value).toBeFalsy()
expect(value).toBeNull()
expect(value).toBeDefined()
expect(value).toBeUndefined()

// Numbers
expect(num).toBeGreaterThan(3)
expect(num).toBeLessThanOrEqual(5)
expect(num).toBeCloseTo(0.3, 5) // For floats

// Strings
expect(str).toMatch(/regex/)
expect(str).toContain('substring')

// Arrays/Iterables
expect(arr).toContain(item)
expect(arr).toHaveLength(3)
expect(arr).toContainEqual({ id: 1 })

// Objects
expect(obj).toHaveProperty('key')
expect(obj).toMatchObject({ subset: 'value' })

// Exceptions
expect(() => fn()).toThrow('error message')
await expect(asyncFn()).rejects.toThrow()

// Functions (mocks)
expect(fn).toHaveBeenCalled()
expect(fn).toHaveBeenCalledWith(arg)
expect(fn).toHaveBeenCalledTimes(1)
```

---

## Dependencies

```bash
# Core
npm install -D vitest

# React testing
npm install -D @testing-library/react @testing-library/jest-dom
npm install -D @testing-library/user-event

# Coverage
npm install -D @vitest/coverage-v8
```

---

## Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## CDD Workflow

### Feature Development

```
1. CONTRACT: Define TypeScript interfaces + Zod schemas
   tsc --noEmit             # Verify contracts compile

2. IMPLEMENT: Write code + co-generate unit tests
   vitest --watch           # Run in watch mode
   - Implementation agent writes code AND tests together

3. VERIFY: Integration/E2E tests (dev-test, post-implementation)
   vitest run               # Run full suite

4. REFACTOR: Improve while green
   - Clean up code
   - All tests still pass

5. COMMIT: When contracts compile and tests pass
   git add . && git commit
```

### Bug Fix (Strict Test-First)

```
1. REPRODUCE: Write failing test that proves the bug
   vitest --watch           # Run in watch mode

2. FIX: Write minimum code to pass the test
   - Verify test passes

3. VERIFY: Confirm no regressions
   vitest run               # Full suite

4. COMMIT: When tests pass
   git add . && git commit
```

---

## Database Testing

```typescript
// Use a test database or mock Prisma
import { prisma } from '@/lib/db/client'
import { beforeEach, afterAll } from 'vitest'

beforeEach(async () => {
  // Clean database before each test
  await prisma.$transaction([
    prisma.user.deleteMany(),
    prisma.post.deleteMany(),
  ])
})

afterAll(async () => {
  await prisma.$disconnect()
})
```

---

## Snapshot Testing

```typescript
import { expect, it } from 'vitest'

it('matches snapshot', () => {
  const result = generateOutput()
  expect(result).toMatchSnapshot()
})

// Inline snapshot
it('matches inline snapshot', () => {
  expect(result).toMatchInlineSnapshot(`
    {
      "id": 1,
      "name": "Test",
    }
  `)
})
```

---

## Context7 Usage

When working with Vitest patterns, use:

```
use context7 for vitest mocking
use context7 for vitest config
```
