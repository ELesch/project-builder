# Project Builder Templates

This directory contains templates used during project creation.

## Directory Structure

```
templates/
├── discovery/           # Templates for gathering requirements
│   ├── project-brief.md
│   └── tech-stack.md
└── orchestrator/        # Generic orchestrator template to copy
    ├── README.md
    ├── CLAUDE.md.template
    ├── CHANGELOG.md.template
    └── .claude/
        ├── agents/
        ├── templates/
        └── status files
```

## Discovery Templates

Used by the `@project-discovery` agent to create project documentation.

## Orchestrator Template

Contains the generic orchestrator pattern files. The `@project-initializer` agent copies these files and customizes them based on the architecture document.

### Template Variables

Files in `orchestrator/` use these placeholders:

- `{{PROJECT_NAME}}` - Project name
- `{{PROJECT_DESCRIPTION}}` - One-line description
- `{{TECH_STACK}}` - Primary technologies
- `{{DATE}}` - Creation date
- `{{AGENTS}}` - List of included agents

The initializer replaces these with actual values during project creation.
