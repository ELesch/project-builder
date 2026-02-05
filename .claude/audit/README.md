# Audit Trail System

This directory contains audit logs and decision records for tracking activity during project creation and development.

## What Gets Tracked

| Component | Capture Method | Purpose |
|-----------|----------------|---------|
| **Session Log** | Automatic (hooks) | Agent delegations, completions, failures |
| **Decision Log** | Manual (`/audit-decision`) | Alternatives considered, rationale |
| **Audit Summary** | Manual (`/audit-summary`) | Retrospective analysis |
| **Orchestrator Analysis** | Manual (`/orc-analyze`) | Rule compliance, pattern detection |

## Directory Structure

```
.claude/audit/
├── README.md           # This file
├── sessions/           # Auto-generated session logs
│   └── YYYY-MM-DD.md   # Daily log file
├── decisions/          # Manual decision records
│   └── YYYY-MM-DD-{id}.md
└── analysis/           # Orchestrator compliance reports
    └── {session-id}-analysis.md
```

## Automatic Capture (via hooks)

Events automatically logged to session files:

| Event | What's Captured |
|-------|-----------------|
| `SubagentStart` | Agent name, task description |
| `SubagentStop` | Agent completion status |
| `PostToolUseFailure` | Tool name, error message |
| `SessionStart/End` | Session boundaries |

### Session Log Format

```markdown
## Session: 2026-01-31T10:00:00

### 10:15:00 - Agent Delegated: dev-backend

| Field | Value |
|-------|-------|
| Task | Implement user service endpoints |
| Status | In Progress |

### 10:30:00 - Agent Completed: dev-backend

| Field | Value |
|-------|-------|
| Duration | 15m |
| Status | Success |
```

## Skills

### /audit-decision - Record Decisions

Use when making significant decisions:

```
/audit-decision
```

Records:
- What decision was made
- Alternatives considered (with pros/cons)
- Why this choice was made
- Constraints that influenced the decision

**When to use:** Technology choices, architecture decisions, trade-off resolutions, changing previous decisions.

### /audit-summary - Analyze Patterns

Generate insights from session logs:

```
/audit-summary
```

Produces agent delegation patterns, failure frequency, and recommendations.

### /orc-analyze - Compliance Analysis

Analyze session transcripts for orchestrator pattern compliance:

```
/orc-analyze [session-id] [options]
```

Options:
- `session-id` - Specific session (default: most recent)
- `--batch` - Analyze all sessions
- `--list` - List available sessions

**Rules Evaluated:**

| Rule | Severity | Description |
|------|----------|-------------|
| Plan Mode Usage | Warning | Non-trivial tasks should enter plan mode |
| Agent Delegation | Error | Must delegate to agents (not work directly) |
| File Read Limit | Warning | ≤3 consecutive reads in main context |
| No Direct Code Write | Error | Never write code files in main context |
| Explore Agent Usage | Warning | Use Explore agent for >5 reads |
| Batch File Limit | Warning | ≤20 files per agent delegation |

**When to use:** After project creation/migration, during retrospectives, periodically with `--batch`.

### /analyze-task - Task-Level Analysis

Analyze compliance at task level (spans multiple sessions):

```
node .claude/scripts/analyze-task.mjs [options]
```

Options:
- `--slug <slug>` - Analyze by task slug
- `--session <id>` - Find task containing session
- `--list` - List all tasks
- `--timeline` - Show timeline
- `--batch` - Analyze all tasks

**Why Task-Level?** Session-level analysis can produce false violations when planning and execution happen in different sessions.

**Task Linkage Signals:**

| Signal | Reliability | Description |
|--------|-------------|-------------|
| Slug match | High | Same slug across sessions |
| Plan content | Very High | `planContent` field in execution session |
| Transcript reference | High | Execution session references planning transcript |
| Timing proximity | Medium | Sessions within 5 minutes |

## Configuration

Audit hooks in `.claude/settings.json`:

```json
{
  "hooks": {
    "SubagentStart": [{"hooks": [{"type": "command", "command": ".claude/hooks/audit-hooks.sh"}]}],
    "SubagentStop": [{"hooks": [{"type": "command", "command": ".claude/hooks/audit-hooks.sh"}]}],
    "PostToolUseFailure": [{"hooks": [{"type": "command", "command": ".claude/hooks/audit-hooks.sh"}]}]
  }
}
```

## Created Projects

All created projects include the audit trail system. The manifest.json tracks:

```json
{
  "audit": {
    "enabled": true,
    "sessionsDir": ".claude/audit/sessions",
    "decisionsDir": ".claude/audit/decisions"
  }
}
```

## Privacy Considerations

- Session logs may contain file paths and error messages
- Decision records may contain business context
- Both are gitignored by default
- Share only sanitized summaries externally

## Retention

- Session logs: Keep for project lifetime
- Decision records: Keep permanently (they document architecture)
- Files are append-only - never edit past entries
