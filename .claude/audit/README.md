# Audit Trail System

This directory contains audit logs and decision records for tracking activity during project creation and development.

## Purpose

The audit trail captures **process** rather than just **outcomes**:

- **What happened** - Agent delegations, tool usage, failures
- **Why decisions were made** - Alternatives considered, rationale
- **When things occurred** - Timestamps and phase transitions
- **What went wrong** - Failure details for debugging

## Directory Structure

```
.claude/audit/
├── README.md           # This file
├── sessions/           # Auto-generated session logs (by date)
│   └── YYYY-MM-DD.md   # Daily session log
└── decisions/          # Manual decision records
    └── YYYY-MM-DD-{id}.md
```

## Session Logs (Automatic)

Session logs are automatically created by hooks when:

- Agents are delegated to (`SubagentStart`)
- Agents complete (`SubagentStop`)
- Tools fail (`PostToolUseFailure`)
- Sessions start/end

### Session Log Format

```markdown
## Session: 2026-01-31T10:00:00

### 10:15:00 - Agent Delegated: dev-backend

| Field | Value |
|-------|-------|
| Task | Implement user service endpoints |
| Scope | src/modules/user/ |
| Status | In Progress |

### 10:30:00 - Agent Completed: dev-backend

| Field | Value |
|-------|-------|
| Duration | 15m |
| Status | Success |
| Files Modified | 8 |

### 10:45:00 - Tool Failed: Bash

| Field | Value |
|-------|-------|
| Command | npm run build |
| Error | TypeScript error TS2345 |
| Context | dev-frontend agent |
```

## Decision Records (Manual via /audit-decision)

Use the `/audit-decision` skill to record significant decisions:

```
/audit-decision
```

Creates a structured record of:
- What decision was made
- Alternatives considered (with pros/cons)
- Why the choice was made
- Context and constraints

### When to Record Decisions

- Technology choices (framework, database, patterns)
- Architecture decisions (module boundaries, API design)
- Trade-off decisions (performance vs. simplicity)
- Reversals (changing previous decisions)

## Audit Summary (via /audit-summary)

Use the `/audit-summary` skill to analyze session logs:

```
/audit-summary
```

Generates:
- Agent delegation patterns
- Failure frequency and types
- Time spent in different phases
- Recommendations for improvement

## Usage in Project Builder

The Project Builder uses audit trails to:

1. **Debug failures** - Trace what happened before an error
2. **Improve agents** - Identify patterns in agent outputs
3. **Track progress** - See phase transitions during creation
4. **Learn patterns** - Understand what works and what doesn't

## Privacy Considerations

- Session logs may contain file paths and error messages
- Decision records may contain business context
- Both are gitignored by default (see `.gitignore`)
- Share only sanitized summaries externally

## Files Are Append-Only

Session log files are append-only. Never edit past entries. This ensures an accurate historical record.

## Retention

- Session logs: Keep for project lifetime
- Decision records: Keep permanently (they document architecture)
- Old logs can be archived but not deleted during active development
