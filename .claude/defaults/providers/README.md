# Provider Guides

Service-specific setup guides for common providers used in projects.

## Database Providers

| Provider | File | Use Case |
|----------|------|----------|
| **Supabase** | @supabase.md | Default for web-app, backend-api |
| Firebase | @firebase.md | Mobile-first, real-time, Google ecosystem |
| MongoDB Atlas | External docs | Document database |
| AWS RDS | External docs | Enterprise PostgreSQL/MySQL |
| PlanetScale | External docs | Serverless MySQL |

## Hosting Providers

| Provider | File | Use Case |
|----------|------|----------|
| **Vercel** | @vercel.md | Default for Next.js web-app |
| Railway | @railway.md | Backend APIs with database |
| Azure Container Apps | @azure-container-apps.md | .NET Aspire, microservices |
| Netlify | External docs | Static sites, JAMstack |
| Fly.io | External docs | Edge deployment |

## Version Control

| Provider | File | Use Case |
|----------|------|----------|
| **GitHub** | @github.md | Default for all projects |
| GitLab | See @github.md notes | Self-hosted option |
| Bitbucket | See @github.md notes | Atlassian ecosystem |

## Error Tracking

| Provider | File | Use Case |
|----------|------|----------|
| **Sentry** | @sentry.md | Default for all projects |
| Datadog | External docs | Full observability |
| Rollbar | External docs | Error focus |

## Usage

These guides are referenced by `@project-initializer` during project setup:

```markdown
# In project-initializer.md
See @.claude/defaults/providers/supabase.md for Supabase setup steps.
```

Each guide includes:
- Setup steps
- Environment variables
- Connection string formats
- Verification commands
- Common issues and solutions

## Adding New Providers

1. Create `{provider}.md` in this directory
2. Include: When to Use, Setup Steps, Env Vars, Verification, Common Issues
3. Reference from project-initializer.md
