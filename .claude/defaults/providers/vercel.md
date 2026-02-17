# Vercel Provider Guide

Zero-config hosting platform optimized for Next.js and frontend frameworks.

## When to Use

- Default for web-app projects (especially Next.js)
- When you need automatic deployments from GitHub
- When you want edge functions and serverless

## Setup Steps

### Option 1: CLI (Preferred)

**IMPORTANT:** Check auth before any Vercel CLI command. `vercel login` is interactive -- never run it from Claude Code.

```bash
# 0. Check authentication first
npx vercel whoami
# If NOT authenticated, ask user: "Please run `npx vercel login` in a separate terminal."

# 1. Link project to Vercel (--yes skips interactive prompts)
npx vercel link --yes

# 2. Add environment variables (pipe via stdin, not interactive prompt)
echo "$DATABASE_URL_VALUE" | npx vercel env add DATABASE_URL production
echo "$DIRECT_URL_VALUE" | npx vercel env add DIRECT_URL production
echo "$NEXTAUTH_SECRET_VALUE" | npx vercel env add NEXTAUTH_SECRET production
# Repeat for each variable

# 3. Deploy (--yes skips confirmation)
npx vercel --prod --yes
```

### Option 2: Dashboard

1. Go to vercel.com/new
2. Import repository from GitHub
3. Add environment variables in dashboard
4. Deploy

## Environment Variables

Add via CLI (pipe value to avoid interactive prompt):
```bash
echo "$VALUE" | npx vercel env add VARIABLE_NAME production
```

Or via dashboard:
- Project Settings > Environment Variables
- Add for Production, Preview, Development as needed

## Verification

```bash
# Check project is linked
vercel project ls

# Open in browser
vercel --prod
# Note the deployment URL

# Open dashboard
vercel dashboard
```

## Common Variables to Set

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | Pooled database connection |
| `DIRECT_URL` | Direct database connection |
| `NEXTAUTH_SECRET` | Session encryption |
| `NEXTAUTH_URL` | Production URL (auto-set by Vercel) |
| `NEXT_PUBLIC_*` | Client-side variables |

## Auto-Deploy

Vercel auto-deploys from GitHub by default:
- Push to main = Production deploy
- Push to other branches = Preview deploy
- PRs get preview URLs automatically

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Build failed | Missing env var | Add all required variables |
| 500 errors | Database connection | Check DATABASE_URL is set |
| Module not found | Build command wrong | Check package.json scripts |
| Cold start slow | Large bundle | Enable Edge Runtime where possible |

## With Supabase

For serverless (Vercel), use pooled connection:
```bash
# Use port 6543 with pgbouncer
DATABASE_URL="...pooler.supabase.com:6543/postgres?pgbouncer=true"
```

## Build Settings

Usually auto-detected, but can configure:
- **Framework Preset**: Next.js (auto)
- **Build Command**: `npm run build` or `next build`
- **Output Directory**: `.next` (auto)
- **Install Command**: `npm install`

## Deployment Checklist

- [ ] `npx vercel whoami` confirms authentication
- [ ] `npx vercel link --yes` completed
- [ ] All environment variables added (via stdin pipe)
- [ ] `npx vercel --prod --yes` succeeds
- [ ] Production URL loads without errors
- [ ] Database queries work
- [ ] Auto-deploy from GitHub enabled
