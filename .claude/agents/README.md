# Project Builder Agents

These agents support the project creation workflow.

## Available Agents

| Agent | File | Purpose |
|-------|------|---------|
| Discovery | `project-discovery.md` | Gather project requirements |
| Architect | `project-architect.md` | Design project structure |
| Initializer | `project-initializer.md` | Create project files |

## Agent Pattern

Each agent follows this structure:

```markdown
# Agent Name

## Role
What this agent does

## CRITICAL: YOU MUST ALWAYS
- Required behaviors

## CRITICAL: NEVER DO THESE
- Prohibited behaviors

## Inputs
What the agent receives

## Outputs
What the agent produces

## Workflow
Step-by-step process
```

## Usage

Agents are invoked by the orchestrator using the Task tool. Agents cannot spawn other agents - only the main orchestrator can delegate.

## Coordination Rules

See `@.claude/roster.md` for detailed agent coordination rules including:

- **Scope limits**: Max 15-20 files per agent run for initializer/migrator
- **Parallel execution matrix**: Which agents can run concurrently
- **Directory ownership**: Which agent owns which directories
- **Batching strategy**: How to split large operations

**Key constraints:**
1. Never run file-writing agents in parallel
2. Batch large operations into multiple agent runs
3. The orchestrator (not agents) manages batching and coordination
