# Generic Orchestrator Template

This directory contains template files that are copied and customized when creating a new project.

## Files

| File | Purpose |
|------|---------|
| `CLAUDE.md.template` | Main instructions file |
| `CHANGELOG.md.template` | Release history |
| `README.md.template` | Project overview |
| `.claude/` | Status and agent files |

## Template Variables

All files use these placeholders that get replaced during initialization:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | Project name | "Task Manager API" |
| `{{PROJECT_SLUG}}` | URL-safe name | "task-manager-api" |
| `{{PROJECT_DESCRIPTION}}` | One-line description | "A REST API for managing tasks" |
| `{{TECH_STACK}}` | Primary technologies | "TypeScript, Express, PostgreSQL" |
| `{{DATE}}` | Creation date | "2024-01-15" |
| `{{AGENTS}}` | Included agents | "architect, backend, api, test" |
| `{{LANGUAGE}}` | Primary language | "TypeScript" |
| `{{FRAMEWORK}}` | Main framework | "Express" |

## Customization Points

The initializer customizes these sections based on project requirements:

### CLAUDE.md
- Tech stack conventions
- File size limits
- Module patterns
- Testing approach
- Agent roster

### Agents
- Which agents to include
- Tech-specific agent content
- Project patterns

### Templates
- Plan template for project type
- Results template

## Usage

The `@project-initializer` agent:
1. Reads these templates
2. Replaces all `{{VARIABLE}}` placeholders
3. Removes or adds sections based on architecture
4. Writes to the new project directory
