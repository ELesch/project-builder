---
technologies: [nextjs, prisma]
versions:
  nextjs: "15"
  prisma: "7"
aiConfidence: Medium
context7Available: true
dependencies: [nextjs-15, prisma-7]
lastUpdated: 2026-02-01
integration: true
---

# Next.js 15 + Prisma 7 Integration Knowledge

## AI Training Context

| Aspect | Status |
|--------|--------|
| **Technologies** | Next.js 15, Prisma 7 |
| **Gap Level** | Major |
| **Confidence** | Medium |
| **Context7** | Available for both |

**Integration Challenges:**
- Both technologies have major version changes
- Server Components + Prisma patterns evolved
- Server Actions replace API routes for CRUD
- Connection pooling for serverless

---

## Critical Integration Patterns

### Server Component Data Fetching

```typescript
// app/users/page.tsx (Server Component)
import { prisma } from '@/lib/db/client'

// Direct database access in Server Components
export default async function UsersPage() {
  const users = await prisma.user.findMany({
    include: { posts: true },
  })

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### Server Actions for Mutations

```typescript
// app/users/actions.ts
'use server'

import { prisma } from '@/lib/db/client'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
})

export async function createUser(formData: FormData) {
  const validated = CreateUserSchema.parse({
    email: formData.get('email'),
    name: formData.get('name'),
  })

  await prisma.user.create({
    data: validated,
  })

  revalidatePath('/users')
}

export async function deleteUser(id: string) {
  await prisma.user.delete({
    where: { id },
  })

  revalidatePath('/users')
}
```

### Form Component Using Actions

```typescript
// app/users/CreateUserForm.tsx
import { createUser } from './actions'

export default function CreateUserForm() {
  return (
    <form action={createUser}>
      <input name="email" type="email" required />
      <input name="name" required />
      <button type="submit">Create User</button>
    </form>
  )
}
```

### Client Component with useActionState

```typescript
// app/users/CreateUserFormWithState.tsx
'use client'

import { useActionState } from 'react'
import { createUser } from './actions'

export function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState: any, formData: FormData) => {
      try {
        await createUser(formData)
        return { success: true, error: null }
      } catch (err) {
        return { success: false, error: 'Failed to create user' }
      }
    },
    { success: false, error: null }
  )

  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <input name="name" required />
      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">User created!</p>}
    </form>
  )
}
```

### Prisma Singleton for Edge/Serverless

```typescript
// src/lib/db/client.ts
import { PrismaClient } from '@/generated/prisma/client'
import { PrismaPg } from '@prisma/adapter-pg'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

function createPrismaClient(): PrismaClient {
  const connectionString = process.env.DATABASE_URL

  if (!connectionString) {
    throw new Error('DATABASE_URL not set')
  }

  const adapter = new PrismaPg({ connectionString })

  return new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === 'development'
      ? ['query', 'error', 'warn']
      : ['error'],
  })
}

// Singleton pattern prevents connection exhaustion
export const prisma = globalForPrisma.prisma ?? createPrismaClient()

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

---

## Environment Configuration

```bash
# .env.local

# Pooled connection for serverless (Vercel)
DATABASE_URL="postgresql://postgres.{ref}:{pass}@aws-0-{region}.pooler.supabase.com:6543/postgres?pgbouncer=true"

# Direct connection for Prisma CLI (migrations)
DIRECT_URL="postgresql://postgres.{ref}:{pass}@db.{ref}.supabase.co:5432/postgres"
```

```typescript
// prisma.config.ts
import { config } from 'dotenv'
config({ path: '.env.local' })

import { defineConfig, env } from 'prisma/config'

export default defineConfig({
  schema: 'prisma/schema.prisma',
  datasource: {
    url: env('DIRECT_URL'),  // For CLI operations
  },
})
```

---

## Do/Don't Table

| Do | Don't |
|----|-------|
| Call Prisma directly in Server Components | Create API routes for data fetching |
| Use Server Actions for mutations | Use API routes for form submissions |
| Use singleton pattern for PrismaClient | Create new PrismaClient per request |
| Use pooled connection for app | Use direct connection in production |
| Include relations with `include` | Fetch relations in separate queries |
| Call `revalidatePath` after mutations | Rely on automatic cache invalidation |
| Validate input with Zod | Trust form data directly |
| Handle errors in Server Actions | Let errors propagate to client |

---

## Common Patterns

### Dynamic Route with Prisma

```typescript
// app/users/[id]/page.tsx
import { prisma } from '@/lib/db/client'
import { notFound } from 'next/navigation'

export default async function UserPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params  // Next.js 15: await params

  const user = await prisma.user.findUnique({
    where: { id },
    include: { posts: true },
  })

  if (!user) notFound()

  return <UserProfile user={user} />
}
```

### Optimistic Updates

```typescript
// components/DeleteButton.tsx
'use client'

import { useOptimistic, useTransition } from 'react'
import { deleteUser } from '@/app/users/actions'

export function DeleteButton({ userId }: { userId: string }) {
  const [isPending, startTransition] = useTransition()

  return (
    <button
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await deleteUser(userId)
        })
      }}
    >
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  )
}
```

### Search with Server Component

```typescript
// app/users/page.tsx
import { prisma } from '@/lib/db/client'

export default async function UsersPage({
  searchParams,
}: {
  searchParams: Promise<{ q?: string }>
}) {
  const { q } = await searchParams  // Next.js 15: await searchParams

  const users = await prisma.user.findMany({
    where: q ? {
      OR: [
        { name: { contains: q, mode: 'insensitive' } },
        { email: { contains: q, mode: 'insensitive' } },
      ],
    } : undefined,
  })

  return <UserList users={users} />
}
```

### Pagination

```typescript
// app/users/page.tsx
const ITEMS_PER_PAGE = 10

export default async function UsersPage({
  searchParams,
}: {
  searchParams: Promise<{ page?: string }>
}) {
  const { page = '1' } = await searchParams
  const currentPage = parseInt(page)

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      take: ITEMS_PER_PAGE,
      skip: (currentPage - 1) * ITEMS_PER_PAGE,
      orderBy: { createdAt: 'desc' },
    }),
    prisma.user.count(),
  ])

  const totalPages = Math.ceil(total / ITEMS_PER_PAGE)

  return (
    <>
      <UserList users={users} />
      <Pagination currentPage={currentPage} totalPages={totalPages} />
    </>
  )
}
```

---

## Verification Tasks

- [ ] Prisma queries work in Server Components
- [ ] Server Actions mutate database correctly
- [ ] `revalidatePath` refreshes cached data
- [ ] Singleton prevents connection exhaustion
- [ ] No API routes created for CRUD operations
- [ ] Pooled connection used in production
- [ ] Direct connection used for migrations
- [ ] `await params` and `await searchParams` used correctly

---

## Context7 Usage

```
use context7 for nextjs server actions
use context7 for prisma client
use context7 for next.js 15 data fetching
```

---

## File Structure

```
app/
├── users/
│   ├── page.tsx              # Server Component - list users
│   ├── actions.ts            # Server Actions - CRUD
│   ├── CreateUserForm.tsx    # Form component
│   ├── [id]/
│   │   └── page.tsx          # Server Component - user detail
│   └── loading.tsx           # Loading UI
├── api/                       # Only for webhooks, external APIs
│   └── webhook/route.ts
└── layout.tsx

src/
├── lib/
│   └── db/
│       └── client.ts         # Prisma singleton
└── generated/
    └── prisma/               # Generated Prisma client
```

---

## Common Errors

### Error: PrismaClient not initialized

**Cause:** Multiple PrismaClient instances
**Fix:** Use singleton pattern in `client.ts`

### Error: Connection timeout in serverless

**Cause:** Too many connections
**Fix:** Use pooled connection (pgbouncer)

### Error: Stale data after mutation

**Cause:** Missing cache invalidation
**Fix:** Call `revalidatePath()` in Server Action

### Error: params is a Promise

**Cause:** Next.js 15 async params
**Fix:** `const { id } = await params`
