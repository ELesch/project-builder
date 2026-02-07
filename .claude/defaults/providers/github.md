# GitHub Provider Guide

Version control and repository hosting with CI/CD integration.

## When to Use

- Default for all project types
- When you need collaboration features
- When you want automated deployments (with Vercel, etc.)

## Setup Steps

### New Repository (CLI - Preferred)

```bash
# Initialize git and create repo in one command
cd {project-name}
git init
git add .
git commit -m "Initial commit: Project scaffolding with orchestrator framework"

# Create repo and push (--private or --public)
gh repo create {project-name} --private --source=. --remote=origin --push
```

### New Repository (Manual)

1. Create at github.com/new
2. Link local project:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/{username}/{project-name}.git
   git push -u origin main
   ```

### Existing Repository

```bash
# Add orchestrator framework and push
git add .
git commit -m "Add orchestrator framework"
git push
```

## Verification

```bash
# Check authentication
gh auth status

# View repo in browser
gh repo view --web

# List recent repos
gh repo list --limit 5
```

## Authentication

```bash
# Login if needed
gh auth login

# Check status
gh auth status

# Switch accounts
gh auth logout
gh auth login
```

## Common Operations

```bash
# Clone a repo
git clone https://github.com/{user}/{repo}.git

# Or with GitHub CLI
gh repo clone {user}/{repo}

# Create from template
gh repo create {name} --template {template-repo}
```

## Integration with Vercel

1. Create GitHub repo first
2. Import to Vercel from GitHub
3. Auto-deploy enabled automatically

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Permission denied | Not authenticated | `gh auth login` |
| Repo not found | Wrong URL | Verify repo name/user |
| Push rejected | Protected branch | Create PR instead |
| SSH error | Missing key | Use HTTPS or add SSH key |

## GitLab Alternative

If using GitLab instead:
```bash
# Create project at gitlab.com/projects/new
# Or self-hosted GitLab URL

# Clone
git clone https://gitlab.com/{user}/{repo}.git

# Note: GitLab CI uses .gitlab-ci.yml
```

## Bitbucket Alternative

If using Bitbucket:
```bash
# Create at bitbucket.org/repo/create

# Clone
git clone https://bitbucket.org/{user}/{repo}.git

# Note: Bitbucket Pipelines uses bitbucket-pipelines.yml
```
