# Railway Provider Guide

Simple deployment platform for backend APIs with built-in databases.

## When to Use

- Backend API projects
- When you need quick deploys with database included
- When Vercel isn't suitable (long-running processes, etc.)

## Setup Steps

### Option 1: CLI (Preferred)

**IMPORTANT:** `railway login` and `railway init` are interactive. Never run them without flags from Claude Code.

```bash
# Install CLI if needed
npm install -g @railway/cli

# Check authentication first
railway whoami
# If NOT authenticated, ask user: "Please run `railway login` in a separate terminal."

# Create new project (--name avoids interactive prompt)
railway init --name {project-name}

# Or link existing project
railway link

# Add environment variables
railway variables set DATABASE_URL="value"
railway variables set JWT_SECRET="value"

# Deploy
railway up
```

### Option 2: Dashboard

1. Go to railway.app
2. Create new project > Deploy from GitHub repo
3. Add environment variables in Variables tab
4. Deploy

## Adding Database

**IMPORTANT:** `railway add` opens an interactive menu. Use the dashboard instead.

**Dashboard method (recommended):**
1. Go to your project at railway.app
2. Click "Add Service" > "Database" > "PostgreSQL"
3. Connection string auto-populates in variables

```bash
# After adding via dashboard, get connection string
railway variables
# DATABASE_URL will be auto-set
```

## Environment Variables

```bash
# Set single variable
railway variables set KEY="value"

# Set multiple
railway variables set KEY1="value1" KEY2="value2"

# View all
railway variables
```

## Verification

```bash
# Check deployment status
railway status

# View logs
railway logs

# Open deployment
railway open
```

## Connection String Format

Railway provides connection strings automatically:
```bash
# PostgreSQL (auto-generated)
DATABASE_URL=postgres://user:pass@hostname:port/railway
```

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Build failed | Missing start script | Add "start" to package.json |
| Port error | Hardcoded port | Use `process.env.PORT` |
| Database timeout | Wrong host | Use Railway-provided URL |
| Deploy stuck | Large dependencies | Check build logs |

## Health Check Endpoint

Railway expects a health endpoint:

```typescript
// Add to your API
app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});
```

## Deployment Checklist

- [ ] `railway whoami` confirms authentication
- [ ] `railway init --name {project-name}` or `railway link` done
- [ ] Environment variables set
- [ ] `railway up` succeeds
- [ ] Project URL accessible (https://{name}.up.railway.app)
- [ ] Database connected (if applicable)
