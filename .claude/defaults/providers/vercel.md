# Vercel Provider Guide

Zero-config hosting platform optimized for Next.js and frontend frameworks.

## When to Use

- Default for web-app projects (especially Next.js)
- When you need automatic deployments from GitHub
- When you want edge functions and serverless

## Setup Steps

### Option 1: CLI (Preferred)

```bash
# 1. Link project to Vercel (creates project if needed)
npx vercel link

# 2. Add environment variables
vercel env add DATABASE_URL production
vercel env add DIRECT_URL production
vercel env add NEXTAUTH_SECRET production
# Repeat for each variable

# 3. Deploy
npx vercel --prod
```

### Option 2: Dashboard

1. Go to vercel.com/new
2. Import repository from GitHub
3. Add environment variables in dashboard
4. Deploy

## Environment Variables

Add via CLI:
```bash
vercel env add VARIABLE_NAME production
# Then paste the value when prompted
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

- [ ] `npx vercel link` completed
- [ ] All environment variables added
- [ ] `npx vercel --prod` succeeds
- [ ] Production URL loads without errors
- [ ] Database queries work
- [ ] Auto-deploy from GitHub enabled
