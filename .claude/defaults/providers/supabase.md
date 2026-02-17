# Supabase Provider Guide

Database-as-a-service with PostgreSQL, authentication, and real-time subscriptions.

## When to Use

- Default for web-app and backend-api projects
- When you need PostgreSQL with managed hosting
- When you want built-in auth and real-time features

## Setup Steps

1. **Create Project**
   - Go to supabase.com/dashboard
   - Click "New Project"
   - Note the region and database password

2. **Get Connection Strings**
   - Navigate to Project Settings > Database
   - Copy from "Connection String" section:
     - **Transaction (Session)** mode URL > `DATABASE_URL` (pooled, port 6543)
     - **Session** mode URL > `DIRECT_URL` (direct, port 5432)

3. **Get API Keys**
   - Navigate to Project Settings > API
   - Copy:
     - Project URL > `NEXT_PUBLIC_SUPABASE_URL`
     - anon key > `NEXT_PUBLIC_SUPABASE_ANON_KEY`
     - service role key > `SUPABASE_SERVICE_ROLE_KEY`

## Connection String Format

```bash
# Pooled connection (for serverless/Vercel)
DATABASE_URL=postgres://postgres.{project-ref}:{url-encoded-password}@aws-0-{region}.pooler.supabase.com:6543/postgres?pgbouncer=true

# Direct connection (for Prisma migrations)
DIRECT_URL=postgres://postgres.{project-ref}:{url-encoded-password}@db.{project-ref}.supabase.co:5432/postgres
```

## Password Special Characters

**CRITICAL: URL-encode special characters in passwords**

| Character | Encoded |
|-----------|---------|
| `@` | `%40` |
| `!` | `%21` |
| `#` | `%23` |
| `$` | `%24` |
| `%` | `%25` |
| `&` | `%26` |

**Example:** `FyyCB@8JY63@!1fNc@nT` becomes `FyyCB%408JY63%40%211fNc%40nT`

## Environment Variables

```bash
# .env.local
DATABASE_URL="postgresql://postgres.{project-ref}:{password}@aws-0-{region}.pooler.supabase.com:6543/postgres?pgbouncer=true"
DIRECT_URL="postgresql://postgres.{project-ref}:{password}@db.{project-ref}.supabase.co:5432/postgres"
NEXT_PUBLIC_SUPABASE_URL="https://{project-ref}.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
SUPABASE_SERVICE_ROLE_KEY="eyJ..."
```

## Verification

```bash
# Test connection with psql
psql "$DIRECT_URL" -c "SELECT 1"

# Or with Prisma
npx prisma db pull
```

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection refused | Wrong port | Pooled=6543, Direct=5432 |
| Authentication failed | Unencoded password | URL-encode special characters |
| SSL required | Missing param | Add `?sslmode=require` if needed |
| Too many connections | Using DIRECT_URL in app | Use DATABASE_URL (pooled) for queries |

## With Prisma

- Use `DIRECT_URL` for migrations (`prisma db push`, `prisma migrate`)
- Use `DATABASE_URL` for application queries (pooled connection)
- See @.claude/defaults/agent-knowledge/databases/prisma-7.md for full setup

## Schema Isolation (Shared Instance)

For development cost savings, use one Supabase instance with separate schemas:

```sql
-- Create schema for project
CREATE SCHEMA project_name;

-- Set search_path in connection string
DATABASE_URL="...?schema=project_name"
```

See @.claude/defaults/stacks/web-app.md for schema isolation strategy.
