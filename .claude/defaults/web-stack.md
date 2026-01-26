# Default Web Application Stack

This is the standard tech stack for web-based applications. Use this unless there's a specific reason to deviate.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Next.js | 14.x | Major if 15+ |
| React | 18.x | Moderate if 19+ |
| Prisma | 5.x | Major if 6+ |
| Tailwind CSS | 3.x | Major if 4+ |
| NextAuth.js | 4.x | Major if 5+ (Auth.js) |

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

## When to Deviate

Only use alternatives when there's a specific requirement:

| Requirement | Alternative |
|-------------|-------------|
| Mobile app needed | React Native or Expo |
| Existing AWS infrastructure | AWS Amplify instead of Vercel |
| Self-hosted requirement | Docker + any cloud provider |
| Real-time heavy (gaming, collab) | Consider Convex or custom WebSocket |
| ML/AI backend heavy | Python FastAPI backend + Next.js frontend |

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll use our standard modern web stack - it's reliable, well-supported, and deploys easily. The specific technologies are documented in the project for any developers who work on it."

## For Technical Users

Ask only if they want to deviate:

> "Our default stack is Next.js + TypeScript + Supabase + Vercel. Any preferences for something different?"
