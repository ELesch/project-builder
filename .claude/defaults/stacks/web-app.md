# Web Application Stack

Full-stack web applications with user interface. This is the standard tech stack for web-based applications. Use this unless there's a specific reason to deviate.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Next.js | 14.x | Major if 15+ |
| React | 18.x | Moderate if 19+ |
| Prisma | 7.x | Moderate (new config patterns) |
| Tailwind CSS | 3.x | Major if 4+ |
| NextAuth.js | 4.x | Major if 5+ (Auth.js) |

### Prisma 7.x Configuration Notes

Prisma 7 introduces significant configuration changes:

1. **Configuration file**: Use `prisma.config.ts` with `defineConfig()` instead of environment variables in schema
2. **Client generation**: Generator outputs to custom path (e.g., `src/generated/prisma`)
3. **Database adapter**: Use `@prisma/adapter-pg` with `PrismaPg` for PostgreSQL
4. **Environment loading**: Handle in `prisma.config.ts` with dotenv, not `dotenv-cli`

See the Prisma 7 templates for correct configuration patterns.

**Recommendation**: For maximum code reliability, use AI-confident versions unless latest features are required. Document gotchas in `tech/stack.md` if using newer versions.

## Core Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Next.js (App Router) | Full-stack React framework |
| **Language** | TypeScript (strict mode) | Type safety across codebase |
| **Database** | Supabase (PostgreSQL) | Managed database with real-time |
| **ORM** | Prisma | Type-safe database queries |
| **Styling** | Tailwind CSS | Utility-first CSS |
| **Components** | shadcn/ui | Accessible, customizable components |
| **Validation** | Zod | Runtime validation + TypeScript |
| **Auth** | NextAuth.js | Flexible authentication |
| **Deployment** | Vercel | Zero-config Next.js hosting |
| **Source Control** | GitHub | Repository hosting |

## Testing Stack

| Tool | Purpose |
|------|---------|
| Jest | Unit testing |
| React Testing Library | Component testing |
| Playwright | End-to-end testing |

## Logging Stack

| Tool | Purpose |
|------|---------|
| Pino | Structured JSON logging |
| Sentry | Error tracking and monitoring |

## When to Deviate

Only use alternatives when there's a specific requirement:

| Requirement | Alternative |
|-------------|-------------|
| Mobile app needed | React Native or Expo |
| Existing AWS infrastructure | AWS Amplify instead of Vercel |
| Self-hosted requirement | Docker + any cloud provider |
| Real-time heavy (gaming, collab) | Consider Convex or custom WebSocket |
| ML/AI backend heavy | Python FastAPI backend + Next.js frontend |

## Database Options

| Service | Best For | Notes |
|---------|----------|-------|
| Supabase | Default choice | PostgreSQL, real-time, auth built-in |
| Firebase | Mobile-first apps | Real-time, Google ecosystem |
| PlanetScale | Serverless MySQL | Auto-scaling, branching |
| AWS RDS | Enterprise | Full control, existing AWS |
| MongoDB Atlas | Document-based | Flexible schema |

## Development Database Strategy

> **Cost Optimization**: Using separate databases per project during development can significantly increase costs. Consider schema-based isolation instead.

### Shared Database with Separate Schemas (Recommended for Development)

Instead of creating a new database instance for each project, use a single development database with separate schemas:

| Approach | Cost | Isolation | Best For |
|----------|------|-----------|----------|
| **Shared DB + schemas** | Low | Schema-level | Development, staging |
| Separate databases | High | Full | Production, compliance |

**Benefits:**
- Significant cost savings (one database instance vs. many)
- Same isolation patterns you'll use in production
- Easier to manage across multiple projects
- Works with Supabase, Neon, PlanetScale, and most PostgreSQL providers

**Implementation (Supabase/PostgreSQL):**

```sql
-- Create a schema for each project
CREATE SCHEMA project_taskflow;
CREATE SCHEMA project_dashboard;

-- Set search_path in your app's database URL or Prisma schema
-- postgresql://user:pass@host:5432/db?schema=project_taskflow
```

**Prisma Configuration:**
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL") // includes ?schema=project_name
}
```

**Tradeoffs:**
- Requires discipline around migrations (per-schema)
- Shared resource limits (connections, storage)
- Not suitable for production workloads with strict isolation needs

### When to Use Separate Databases

| Scenario | Recommendation |
|----------|---------------|
| Development/prototyping | Shared + schemas |
| Staging (team shared) | Shared + schemas |
| Production | Separate database |
| Compliance requirements | Separate database |
| Performance isolation needed | Separate database |

### Provider-Specific Notes

| Provider | Schema Support | Notes |
|----------|---------------|-------|
| Supabase | Full | Use SQL editor to create schemas |
| Neon | Full | Branch per project is also cost-effective |
| PlanetScale | Limited | Use database branching instead |
| AWS RDS | Full | Standard PostgreSQL schemas |
| Firebase | N/A | Document-based, use collections |

## Hosting Options

| Service | Best For | Notes |
|---------|----------|-------|
| Vercel | Default for Next.js | Zero-config, edge functions |
| Netlify | Static sites, JAMstack | Good free tier |
| AWS Amplify | AWS ecosystem | Full AWS integration |
| Railway | Simple deployments | PostgreSQL included |
| DigitalOcean App Platform | Budget-friendly | Good for small teams |

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll use our standard modern web stack - it's reliable, well-supported, and deploys easily. The specific technologies are documented in the project for any developers who work on it."

## For Technical Users

Ask only if they want to deviate:

> "Our default stack is Next.js + TypeScript + Supabase + Vercel. Any preferences for something different?"
