# Claude Code Baseline

> This document captures what the Project Builder knows about Claude Code.
> Used by `/cpm_update` to detect when updates are needed.

**Baseline Date**: 2026-02-02
**Claude Code Version**: Latest (as of February 2026)
**Project Builder Version**: 2.18.0

---

## Core Capabilities

### Built-in Tools

| Tool | Purpose | Permission Required |
|------|---------|---------------------|
| Bash | Execute shell commands | Yes |
| Read | Access file contents | No* |
| Write | Create/overwrite files | No* |
| Edit | Make targeted code edits | Yes |
| Glob | Find files matching patterns | No |
| Grep | Search file contents | No |
| WebFetch | Fetch URL content | Yes |
| WebSearch | Perform web searches | Yes |
| NotebookEdit | Modify Jupyter notebooks | Yes |
| AskUserQuestion | Prompt user for input | No |
| Task | Spawn subagents | Yes |
| Skill | Invoke skills | Yes |
| TaskCreate | Create task list items | No |
| TaskUpdate | Update task status | No |
| TaskList | List all tasks | No |
| TaskGet | Get task details | No |
| TaskOutput | Get background task output | No |
| TaskStop | Stop background task | No |

*Can be restricted via deny rules

### New Capabilities (January 2026)

| Capability | Description |
|------------|-------------|
| Tasks | Persistent task lists across sessions (upgraded from Todos) |
| Skills Hot Reload | Skills reload without session restart |
| MCP Tool Search | Lazy loading of MCP tool definitions |
| Wildcard Permissions | Extended permission patterns (e.g., `Bash(npm *)`) |
| Prompt-Based Hooks | LLM-evaluated hooks with `type: "prompt"` |

### Built-in Subagents

| Agent | Model | Purpose | Tools |
|-------|-------|---------|-------|
| Explore | Haiku | Fast codebase exploration | Read-only (Glob, Grep, Read, WebFetch) |
| Plan | Inherited | Research for Plan Mode | Read-only |
| general-purpose | Inherited | Complex multi-step tasks | All tools |
| Bash | Inherited | Terminal commands | Bash |
| statusline-setup | Sonnet | Configure status line | Read, Edit |
| claude-code-guide | Haiku | Answer Claude Code questions | Read-only + WebFetch, WebSearch |

### Subagent Limitations

- Subagents **cannot spawn other subagents**
- Subagents inherit model unless overridden
- Subagents can have restricted tool access
- Subagents run in isolated context
- **Custom agents created during a session aren't available until session restart**

### Effective Subagent Prompting

Subagents start with **minimal context** - they don't receive:
- Parent conversation history
- Full Claude Code system prompt
- CLAUDE.md content (unless explicitly referenced)

**You must provide context explicitly** via `@path` references or inline content.

#### The Effective Delegation Formula

```
Effective Delegation =
  Clear Scope
  + Right Agent Type
  + Specific File References (@path)
  + Expected Output Format
  + Verification Criteria
```

#### Agent Selection by Task

| Task Type | Agent | Why |
|-----------|-------|-----|
| Research/investigate | `Explore` | Fast (Haiku), read-only, won't modify anything |
| Multi-step implementation | `general-purpose` | Full tool access, can read/write/test |
| Run commands | `Bash` | Isolated terminal context |
| Plan mode research | `Plan` | Read-only, prevents recursion |

#### Customizing Built-in Agents via Prompt

When no domain-specific agent exists, customize a built-in agent by providing what a custom agent would embed:

**Ineffective prompt:**
```
Review the code for issues
```

**Effective prompt:**
```
Use general-purpose agent to implement the caching layer.

CONTEXT:
- This project uses Redis with ioredis client
- Cache patterns are in @src/lib/cache.ts
- TTL conventions: user data 5min, config 1hour
- Always use the logger from @src/lib/logger.ts

TASK:
- Add cache invalidation to user profile updates
- Follow existing patterns in @src/services/user.ts

CONSTRAINTS:
- Do not modify the cache client configuration
- Run tests after changes

OUTPUT:
- List of modified files
- Test results summary
```

The prompt becomes a **temporary agent definition** with:
- **CONTEXT** - Domain knowledge, conventions, related files
- **TASK** - Specific work to accomplish
- **CONSTRAINTS** - Boundaries and patterns to follow
- **OUTPUT** - What to return to the orchestrator

#### When to Use Each Approach

| Situation | Approach |
|-----------|----------|
| Domain agent exists | Use domain agent (e.g., `dev-nextjs-15`) |
| No domain agent, will reuse | Note for future: create domain agent |
| No domain agent, one-time task | Customize built-in agent via prompt |
| Quick exploration | `Explore` with minimal prompt |

---

## Configuration System

### Settings Precedence (highest to lowest)

1. **Managed** - System-level, IT deployed
2. **CLI flags** - Current session only
3. **Local** - `.claude/*.local.*` (gitignored)
4. **Project** - `.claude/settings.json`
5. **User** - `~/.claude/settings.json`

### Key Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.claude/settings.json` | User | Personal defaults |
| `.claude/settings.json` | Project | Team settings |
| `.claude/settings.local.json` | Local | Personal project overrides |
| `CLAUDE.md` | Project | Project context/instructions |
| `CLAUDE.local.md` | Local | Personal project notes |
| `~/.claude.json` | User | MCP servers (personal) |
| `.mcp.json` | Project | MCP servers (team) |

### Permission Modes

| Mode | Behavior |
|------|----------|
| default | Standard permission prompts |
| acceptEdits | Auto-accept file modifications |
| dontAsk | Auto-deny prompts (allowed tools work) |
| bypassPermissions | Skip all checks (dangerous) |
| plan | Read-only exploration |

### New Settings Fields (January 2026)

| Field | Type | Purpose |
|-------|------|---------|
| `alwaysThinkingEnabled` | boolean | Extended thinking toggle |
| `plansDirectory` | string | Plan file location (default: ~/.claude/plans) |
| `showTurnDuration` | boolean | Show turn timing (default: true) |
| `language` | string | Response language (e.g., "japanese") |
| `autoUpdatesChannel` | string | "stable" or "latest" |
| `spinnerTipsEnabled` | boolean | Show tips during operations |
| `terminalProgressBarEnabled` | boolean | Progress bar display |
| `sandbox` | object | OS-level sandboxing configuration |
| `sandbox.enabled` | boolean | Enable sandboxing |
| `sandbox.autoAllowBashIfSandboxed` | boolean | Auto-approve bash in sandbox |
| `forceLoginMethod` | string | "claudeai" or "console" |
| `forceLoginOrgUUID` | string | Constrain to organization |
| `enableAllProjectMcpServers` | boolean | MCP project scope |
| `enabledMcpjsonServers` | array | MCP allowlist |
| `disabledMcpjsonServers` | array | MCP denylist |
| `allowManagedHooksOnly` | boolean | Block user/project hooks |

---

## Extensibility

### Skills (Custom Slash Commands)

**Location**: `.claude/skills/<skill-name>/SKILL.md`

**Format**:
```yaml
---
name: skill-name
description: When to use this skill
disable-model-invocation: true  # User-only
allowed-tools: Read, Grep
context: fork                   # Run in subagent
agent: Explore                  # Which agent type
---

Skill instructions here...
```

**Key Frontmatter Fields**:
- `name` - Display name
- `description` - When Claude should use it
- `disable-model-invocation` - true = manual only
- `user-invocable` - false = Claude only
- `allowed-tools` - Restrict available tools
- `model` - Override model
- `context` - "fork" for isolated execution *(executes in subagent)*
- `agent` - Agent type for forked context
- `hooks` - Lifecycle hooks (PreToolUse, PostToolUse, Stop)
- `once` - Run hook only once per session *(NEW)*

### Commands (Legacy)

**Location**: `.claude/commands/<command-name>.md`

**Format**: Pure markdown (no frontmatter)

Commands are simpler than skills - just markdown loaded as context.

### Hooks

**Events** (13 total):
- SessionStart - New/resumed session
- UserPromptSubmit - Before Claude processes prompt
- PreToolUse - Before tool execution (can block)
- PermissionRequest - When permission dialog shown
- PostToolUse - After tool succeeds
- **PostToolUseFailure** - After tool fails *(NEW)*
- **SubagentStart** - When spawning subagent *(NEW)*
- SubagentStop - When subagent completes
- Stop - When Claude finishes responding
- PreCompact - Before compaction
- SessionEnd - Session ends
- Notification - When Claude sends notifications
- **Setup** - With --init, --init-only, or --maintenance flags *(NEW)*

**Hook Types**:
```json
// Command-based hook (existing)
{
  "type": "command",
  "command": "validation-script.sh"
}

// Prompt-based hook (NEW - LLM-evaluated)
{
  "type": "prompt",
  "prompt": "Evaluate if this action should proceed: $ARGUMENTS",
  "timeout": 30
}
```

**Configuration** (in settings.json):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "validation-script.sh"
          }
        ]
      }
    ],
    "Setup": [
      {
        "matcher": "init",
        "hooks": [
          {
            "type": "command",
            "command": "npm install"
          }
        ]
      }
    ]
  }
}
```

**New Hook Features**:
- `updatedInput` - Modify tool parameters in PreToolUse/PermissionRequest
- `hookSpecificOutput` - Structured response format
- Exit code 2 - Documented behavior per event type
- `once: true` - Run hook only once per session (skills only)

### MCP (Model Context Protocol)

**Purpose**: Connect to external tools/services

**Installation**:
```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport stdio <name> -- <command>
```

**Configuration Files**:
- Project: `.mcp.json`
- User: `~/.claude.json`
- Managed: `managed-mcp.json`

### Plugins

**Location**: Installable packages bundling skills, agents, hooks, MCP

**Structure**:
```
plugin/
├── plugin.json          # Manifest
├── .mcp.json           # MCP servers
├── skills/             # Skills
├── agents/             # Subagents
└── hooks/              # Hook scripts
```

---

## Custom Agents

**Location**: `.claude/agents/<agent-name>.md`

**Format**:
```yaml
---
name: agent-name
description: When to use this agent
allowed-tools: Read, Grep, Bash
model: haiku
---

Agent system prompt here...
```

**Key Fields**:
- `name` - Agent identifier
- `description` - When Claude delegates to it
- `allowed-tools` - Available tools
- `model` - Model override (sonnet/opus/haiku)
- `hooks` - Agent-specific hooks (PreToolUse, PostToolUse, Stop)
- `permissionMode` - Override permission mode in subagent *(NEW)*
- `skills` - Preload skills into subagent context *(NEW)*

**Subagent Execution Modes**:
- **Foreground**: Blocking, interactive prompts pass through to user
- **Background**: Concurrent, auto-deny non-approved actions

---

## Session Management

### Session Commands

```bash
claude                    # New session
claude -c                 # Continue last
claude -r "<name>"        # Resume specific
claude --session-id "id"  # Use specific ID
```

### Context Management

- Context window limited by model
- Auto-compaction at ~95% capacity
- PreCompact hooks run before compaction
- Subagents help manage context

---

## IDE Integrations

### Supported IDEs

- VS Code (extension)
- JetBrains IDEs (plugin)
- Chrome (browser extension)

### Features

- Inline diffs
- File mentions with @
- Session resumption
- Git integration

---

## Deployment Options

- **CLI** - Local terminal
- **Web** - claude.ai async execution
- **Desktop App** - Full release (upgraded from preview)
- **Slack** - Integration
- **GitHub Actions** - CI/CD
- **GitLab CI** - CI/CD
- **Cloud Providers** - Bedrock, Vertex AI, Foundry
- **MCP Server Mode** - Claude Code as MCP server *(NEW)*
- **LLM Gateway** - Custom gateways via LiteLLM *(NEW)*
- **Dev Containers** - Streamlined container setup *(NEW)*
- **OS Sandboxing** - Isolated execution environments *(NEW)*

---

## Key Patterns for Orchestrator Framework

### CLAUDE.md Best Practices

- Keep under ~500 lines
- Use `@path` imports for references
- Layer by specificity (project → subdirectory)
- Move verbose content to skills

### Agent Delegation Pattern

```
Orchestrator (main Claude)
    ↓ delegates to
Custom Agents (.claude/agents/)
    ↓ which use
Built-in Tools (Bash, Read, Edit, etc.)
```

Agents cannot spawn agents - only the orchestrator delegates.

### Permission Patterns

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(npm *)",           // Wildcard patterns (NEW)
      "Bash(git *:*)",         // Complex wildcards (NEW)
      "Read",
      "Task(my-agent)"         // Subagent permissions (NEW)
    ],
    "deny": ["Bash(rm -rf:*)"]
  }
}
```

### Skill Patterns

For user-invoked workflows:
```yaml
---
disable-model-invocation: true
allowed-tools: Read, Grep, Edit
---
```

For Claude-invoked reference:
```yaml
---
user-invocable: false
---
```

---

## Version Tracking

This baseline should be updated when:

1. New tools are added to Claude Code
2. New subagent types become available
3. Configuration schema changes
4. New extensibility features are added
5. Breaking changes occur in skills/hooks/MCP

Use `/cpm_update` to check for and apply updates.

---

## Changelog

### 2.18.0 (2026-02-02)

**Effective Subagent Prompting - Built-in Agent Customization**

Adds comprehensive documentation on how to effectively prompt built-in agents when no domain-specific agent exists.

**Problem Solved:**

When no domain agent exists for a technology, orchestrators would either:
- Try to create a new agent file (which isn't available until session restart)
- Use generic agent names without proper context
- Produce suboptimal results from vague prompts

**Key Additions:**

- **Subagent Context Gap**: Documents that subagents don't receive parent conversation history, CLAUDE.md content, or full system prompt automatically

- **Effective Delegation Formula**: Clear structure for prompts:
  ```
  Effective Delegation =
    Clear Scope + Right Agent Type + File References (@path) +
    Expected Output Format + Verification Criteria
  ```

- **Built-in Agent Selection Guide**: When to use Explore vs general-purpose vs Bash

- **Customizing via Prompt**: How to provide CONTEXT, TASK, CONSTRAINTS, and OUTPUT sections to simulate a custom agent

**Files Changed:**

| File | Change |
|------|--------|
| `claude-code-baseline.md` | Added "Effective Subagent Prompting" section |
| `CLAUDE.md.template` | Added "Customizing Built-in Agents" section with examples |
| `roster.md.template` | Added "Fallback: No Domain Agent Exists" section |

**Key Constraint Documented:**

> Custom agents created during a session aren't available until session restart. Don't create new agent files mid-task—instead, customize a built-in agent via the prompt.

**Example Effective Prompt:**

```
Use general-purpose agent to add Redis caching.

CONTEXT:
- Redis client in @src/lib/redis.ts
- Cache patterns in @src/services/cache-utils.ts
- TTL: user data 5min, config 1hour

TASK:
- Add cache to getUserById, getUserByEmail
- Invalidate on update/delete

CONSTRAINTS:
- Don't modify redis.ts client
- Prefix keys with "user:"

OUTPUT:
- Modified files list
- Test results
```

### 2.17.0 (2026-02-02)

**Task-Level Orchestrator Analyzer - Cross-Session Compliance**

Adds task-level analysis that groups related sessions together for more accurate compliance evaluation.

**Problem Solved:**

Session-level analysis produces false positives when:
- Planning happens in session A, execution in session B
- User accepts plan (context clears for execution session)
- Multi-session workflows are used intentionally

**Example:**
```
# Session-level (false positive)
node analyze-session.mjs c497b649
# "No plan mode" VIOLATION

# Task-level (correct)
node analyze-task.mjs --session c497b649
# Task links to planning session, plan mode passes
```

**New Files:**

| File | Purpose |
|------|---------|
| `task-linker.mjs` | Groups sessions into tasks using linking signals |
| `task-rules.mjs` | Task-level compliance rules (evaluates full lifecycle) |
| `analyze-task.mjs` | CLI for task-level analysis |

**Task Linkage Signals:**

| Signal | Reliability | Use |
|--------|-------------|-----|
| Slug match | High | Same `slug` field across sessions |
| Plan content | Very High | `planContent` field in execution session |
| Transcript reference | High | Execution references planning transcript path |
| Timing proximity | Medium | Sessions within 5 minutes |

**Task Rules (vs Session Rules):**

| Rule | Task-Level Behavior |
|------|---------------------|
| Plan Mode Used | Pass if ANY session in task used plan mode |
| Plan Approved | Pass if ExitPlanMode called OR execution has planContent |
| Agent Delegation | Aggregated across all sessions |
| No Direct Code Writes | Aggregated across all sessions |
| Task Completion | Evaluates full task lifecycle status |

**CLI Usage:**

```bash
# Analyze by slug
node analyze-task.mjs --slug goofy-twirling-orbit

# Find task containing session
node analyze-task.mjs --session c497b649

# List all tasks
node analyze-task.mjs --list

# Show timeline
node analyze-task.mjs --timeline

# Batch analysis
node analyze-task.mjs --batch
```

**Output:** Reports saved to `.claude/audit/analysis/task-{slug}.md`

**Documentation Updates:**

- CLAUDE.md - Added `/analyze-task` section to Audit Trail System
- roster.md - Added task-level analysis skill and documentation

### 2.16.0 (2026-02-02)

**Version-Aware Session Analysis - Orchestrator Version in Transcripts**

Adds orchestrator version to CLAUDE.md template and implements version-aware compliance analysis.

**Problem Solved:**

Session analysis was applying v2.15.0 rules to sessions from older orchestrator versions, producing false positives. Without version info, we couldn't determine which rules to apply.

**Key Changes:**

- **CLAUDE.md.template** - Added version header:
  ```
  **Orchestrator Framework Version:** {{ORCHESTRATOR_VERSION}} (Template: {{TEMPLATE_VERSION}})
  ```
- **TEMPLATE_VERSION file** - New single source of truth at `.claude/templates/orchestrator/TEMPLATE_VERSION`
- **manifest.json.template** - Now uses `{{TEMPLATE_VERSION}}` placeholder
- **transcript-parser.mjs** - Version extraction from project manifest:
  - Added `getProjectVersion()` function to read from `.claude/manifest.json`
  - CLAUDE.md content is NOT stored in transcripts (injected at API level)
  - Parser now accepts optional `projectPath` parameter for version lookup
  - Added documentation of transcript JSONL format
  - Added handling for `system` type entries (compact_boundary, etc.)
- **orchestrator-rules.mjs** - Version-aware rule evaluation:
  - v2.15.0+: Plan mode is ERROR if missing (mandatory)
  - Pre-v2.15.0: Plan mode is WARNING if missing (recommended)
  - Unknown version: Falls back to pre-v2.15.0 behavior (safe default)
- **analyze-session.mjs** - Passes projectPath to parseSession for version lookup

**Transcript Format Investigation:**

| Entry Type | Purpose | Key Fields |
|------------|---------|------------|
| `user` | User messages | `message.content` |
| `assistant` | Claude responses | `message.model`, `message.content` (includes thinking) |
| `system` | System events | `subtype` (compact_boundary, etc.) |
| `progress` | Hook/agent progress | `hookEvent`, `data.type` |
| `file-history-snapshot` | File state tracking | `snapshot`, `trackedFileBackups` |

**Critical Finding:** System prompts (CLAUDE.md content) are injected at API level, NOT stored in transcripts. Version must be obtained from project manifest.

**Template Version:** 1.11.0

**Backward Compatibility:**

Sessions without version info are treated as pre-v2.15.0, ensuring old sessions aren't incorrectly flagged.

### 2.15.0 (2026-02-02)

**Mandatory Plan Mode - Domain Agent Selection for Every Prompt**

Removes the "extremely simple" exception from plan mode. Plan mode is now **mandatory for every user prompt** to ensure proper domain agent selection.

**Why This Change:**

AI training data has a knowledge gap with current package versions. Domain agents embed version-specific patterns that the orchestrator may lack. Skipping plan mode risks using stale patterns.

**Key Changes:**

- **Removed "extremely simple" exception** - No more skipping plan mode
- **Plan mode is MANDATORY for every prompt** - No exceptions
- **Expanded declaration format** to include:
  - Task type (implement / research / debug / review)
  - Technologies involved
  - Agent selection (check roster.md for guidance)
- **Removed hardcoded agent names from templates** - Replaced with:
  - `{{QUICK_SELECTION_EXAMPLES}}` - Populated at project creation
  - `{{TASK_TYPE_EXAMPLES}}` - Populated at project creation
  - Generic `{technology}` placeholders in examples
- **Updated project-agent-generator** to populate template variables

**New Declaration Format:**
```
ORCHESTRATOR APPROACH:
- Task: [one-line summary]
- Task type: [implement / research / debug / review]
- Technologies: [list technologies involved]
- Agents needed: [select from .claude/agents/ - check roster.md]
- Sequence: [sequential / parallel / single agent]
- My role: [coordinate, delegate, review - NOT implement]
```

**Rationale:**

1. Domain agents have embedded version-specific patterns
2. Orchestrator may have stale knowledge for newer packages
3. Proper agent selection requires conscious identification
4. "Extremely simple" was subjective and led to bypassing
5. Hardcoded agent names won't exist in all projects (different tech stacks)

**Files Changed:**

- `CLAUDE.md` - Updated Plan Mode section, removed hardcoded examples
- `CLAUDE.md.template` - Updated for created projects, uses template variables
- `roster.md.template` - Uses template variables for examples
- `project-agent-generator.md` - Generates template variable content

### 2.14.0 (2026-02-01)

**Role-Specific Domain Agents with Shared Knowledge**

Refactors the domain agent system so that version-specific knowledge is stored **once** in shared knowledge files, and multiple role-specific agents @-reference that shared knowledge.

**Architecture Change:**

- **Before:** Knowledge embedded into `dev-*` agents only
- **After:** Knowledge in `knowledge/` folder, referenced by `dev-`, `explore-`, `debug-`, `audit-` agents

**New Directory Structure (Created Projects):**

```
.claude/agents/
├── knowledge/                   # Shared knowledge (NEW location)
│   ├── nextjs-15.md            # Next.js 15 patterns
│   ├── prisma-7.md             # Prisma 7 patterns
│   ├── tailwind-4.md           # Tailwind v4 patterns
│   ├── logging-pino.md         # Cross-cutting (existing)
│   └── testing-vitest.md       # Cross-cutting (existing)
├── dev-nextjs-15.md            # Implementation (modified)
├── explore-nextjs-15.md        # Research (NEW)
├── debug-nextjs-15.md          # Debugging (NEW)
├── audit-nextjs-15.md          # Review (NEW)
└── ... (same for other technologies)
```

**The Four Agent Roles:**

| Role | Prefix | Purpose | Tools | Write Scope |
|------|--------|---------|-------|-------------|
| Implementation | `dev-` | Build features | Read, Grep, Edit, Write, Bash | Code (15 max) |
| Research | `explore-` | Investigate, understand | Read, Grep, Glob, WebFetch, WebSearch | Reports only |
| Debugging | `debug-` | Diagnose issues | Read, Grep, Glob, Bash | Reports only |
| Review | `audit-` | Code review | Read, Grep, Glob | Reports only |

**Role Generation by Confidence:**

| Confidence | Roles Generated | Rationale |
|------------|-----------------|-----------|
| High | `dev-` only | AI confident - implementation sufficient |
| Medium | All 4 roles | Moderate confidence - need investigation tools |
| Low | All 4 roles | Low confidence - full support suite needed |
| Unknown | All 4 roles | Unknown - maximum flexibility |

**New Role Agent Templates:**

- **`explore-TEMPLATE.md.template`** - Base template for exploration agents
- **`debug-TEMPLATE.md.template`** - Base template for debugging agents
- **`audit-tech-TEMPLATE.md.template`** - Base template for tech-specific audit agents

**Index.json Updates (v2.0.0):**

- Added `roleTemplates` section with tool definitions for each role
- Added `roleGenerationRules` mapping confidence levels to roles
- Changed `agentName` to `knowledgeFile` in template entries
- Updated `agentGeneration.namingPatterns` for all 4 roles

**Project-Agent-Generator Updates:**

- Deploys knowledge files to `.claude/agents/knowledge/`
- Generates 1-4 roles per technology based on confidence
- Uses @-references instead of embedding knowledge
- Creates `byRole` registry in manifest

**Template Updates:**

- **`manifest.json.template`** - Added `knowledgeFiles`, `byRole` registry (v1.10.0)
- **`CLAUDE.md.template`** - Added "Four Agent Roles" section with selection guide
- **`roster.md.template`** - Added task-type routing, role-based selection flow

**Benefits:**

- Single source of truth for technology knowledge
- Role-appropriate tool restrictions
- Safe exploration without accidental modifications
- Focused debugging with targeted access
- Pattern compliance review before deployment
- Smaller agent files (~50-100 lines vs 200+ embedded)

**Agent Selection Flow:**

```
1. What task type?
   ├── Building → dev-{tech}
   ├── Research → explore-{tech}
   ├── Debug → debug-{tech}
   └── Review → audit-{tech}

2. What technology?
   └── Check domain agents table
```

**Example Usage:**

```
Task: "The contact form isn't saving to database"

1. debug-prisma-7 → Diagnoses: "Constraint violation on email"
2. dev-prisma-7 → Implements fix
3. audit-prisma-7 → Verifies fix follows patterns
```

### 2.13.0 (2026-02-01)

**Domain-Specific Agents - Version-Aware Code Generation**

Creates technology-specific agents with embedded version knowledge instead of generic agents referencing shared stack.md.

**New Agent:**

- **`@project-agent-generator`** - Assembles domain agents from pre-built knowledge templates

**New Directory Structure:**

```
.claude/defaults/agent-knowledge/
├── README.md                           # Documentation
├── index.json                          # Template registry
├── frameworks/
│   ├── nextjs-15.md                   # Next.js 15 patterns
│   └── react-19.md                    # React 19 patterns
├── databases/
│   └── prisma-7.md                    # Prisma 7 patterns
├── styling/
│   └── tailwind-4.md                  # Tailwind v4 patterns
├── cross-cutting/
│   ├── logging-pino.md                # Pino logging patterns
│   └── testing-vitest.md              # Vitest patterns
└── integrations/
    └── nextjs-prisma.md               # Integration patterns
```

**Generated Domain Agents:**

| Agent | Technology | Supersedes |
|-------|------------|------------|
| `dev-nextjs-15` | Next.js 15 | `dev-frontend` |
| `dev-prisma-7` | Prisma 7 | `dev-backend` |
| `dev-tailwind-v4` | Tailwind CSS v4 | `dev-frontend` |
| `dev-react-19` | React 19 | `dev-frontend` |

**Shared Knowledge Files:**

- `.claude/agents/knowledge/logging-pino.md`
- `.claude/agents/knowledge/testing-vitest.md`

**Agent Updates:**

- **`project-tech-validator.md`** - Added Step 7: Select Knowledge Templates
- **`project-initializer.md`** - Added delegation to `@project-agent-generator`
- **`project-migrator.md`** - Added delegation to `@project-agent-generator`

**Template Updates:**

- **`manifest.json.template`** - Added `domainAgents` section (v1.9.0)
- **`CLAUDE.md.template`** - Added Domain Agents section
- **`roster.md.template`** - Added Domain Agents section with routing

**Knowledge Template Format:**

```yaml
---
technology: nextjs
version: "15"
versionRange: ">=15.0.0 <16.0.0"
aiConfidence: Medium
context7Available: true
dependencies: [react-19]
supersedes: nextjs-14
---

# Critical Patterns
[Version-specific Do/Don't tables, code examples]
```

**Benefits:**

- Version-specific patterns embedded in agents
- Context7 instructions for low-confidence technologies
- Intelligent delegation based on technology, not just domain
- Knowledge layering: templates + validator research + Context7
- Supersession: domain agents replace generic agents for their technology

**New Workflow:**

```
Discovery → Architect → Tech-Validator → AGENT-GENERATOR → Initializer
                              ↓
                     Selects templates
                     from index.json
                              ↓
                     Creates domain agents
                     with embedded patterns
```

### 2.12.0 (2026-02-01)

**UI Design & Responsive Layout Improvements**

Adds professional UI design and responsive layout expertise to the orchestrator framework to produce desktop and mobile-ready interfaces.

**New Agents (2):**

| Agent | Role | Purpose |
|-------|------|---------|
| `dev-ui-designer` | Research | Visual design, design systems, responsive strategy, component specs |
| `dev-auditor-responsive` | Review | Verify responsive implementation, mobile testing, breakpoint compliance |

**New Checklist:**

- **`responsive-review.md.template`** - Comprehensive responsive design checklist covering mobile-first CSS, touch targets, viewport handling, navigation, typography, images, forms, tables, and performance

**New Template:**

- **`design-system.md.template`** - Design system template with color tokens, typography scale, spacing scale, breakpoints, border radius, shadows, animation tokens, and component variants

**Agent Enhancements:**

- **`dev-frontend.md.template`:**
  - Added Design Reference section linking to ui-design.md
  - Added responsive implementation patterns (mobile-first CSS, breakpoint usage)
  - Added mobile-specific patterns (touch targets, navigation, safe areas)
  - Added component responsiveness checklist
  - Added critical rules for responsive development

- **`dev-designer.md.template`:**
  - Added Step 7: Responsive Behavior Specifications
  - Added responsive questions to ask users
  - Added responsive handoff table for mobile/tablet/desktop
  - Updated output to recommend UI Design phase

**Documentation Updates:**

- **`CLAUDE.md.template`:**
  - Added UI Design Phase section
  - Updated implementation flow diagram to include UI DESIGN phase
  - Added when to use/skip UI Design guidance

- **`roster.md.template`:**
  - Added `dev-ui-designer` to Design & Architecture Agents
  - Added `dev-auditor-responsive` to Auditor Agents
  - Updated SDLC Phase Mapping with UI DESIGN phase
  - Added "Using dev-ui-designer" section

- **`agents/README.md.template`:**
  - Added UI Designer to Design & Architecture table
  - Added Responsive to Auditor Agents table

**Template Version:** 1.8.0

**New Workflow:**

```
APP DESIGN → UI DESIGN → DESIGN → TDD DEVELOP → INTEGRATE → REVIEW → AUDIT → DEPLOY
dev-designer  dev-ui-designer  dev-architect  dev-test    dev-test   dev-reviewer  dev-auditor-*  dev-deploy
```

**Benefits:**

- Mobile-first design approach enforced
- Consistent design tokens across projects
- Responsive layouts specified before implementation
- Touch target requirements documented
- Responsive audit catches issues before deployment
- Professional UI output for both desktop and mobile

### 2.11.0 (2026-02-01)

**Orchestrator Pattern Improvements - Verification, Recovery, and Checkpoints**

Addresses deficiencies identified in compliance analysis to improve orchestrator reliability and pattern adherence.

**New Skills (3):**

| Skill | Purpose |
|-------|---------|
| `/verify-agent` | **Mandatory** verification after every agent delegation - checks files, tests, quality, scope |
| `/recover` | Structured error recovery when agents fail or tests break |
| `/parallel-check` | Pre-flight safety check before running agents in parallel |

**New Runbook:**

- **`agent-failure.md.template`** - Runbook for handling test failures, build breaks, scope violations, and unknown errors

**New Templates:**

- **`handoff-full.md.template`** - Complete handoff document for agent delegations
- **`handoff-mini.md.template`** - Minimal handoff for quick tasks

**CLAUDE.md.template Updates:**

- **Delegation Prerequisites (MANDATORY)** section - Design gate, research gate, handoff requirements
- **First Response Protocol** - Task classification on first user request
- **Mid-Phase Checkpoints** - Mandatory checkpoints every 3 agents, after tests, before phase transitions

**roster.md.template Updates:**

- Added `/verify-agent`, `/recover`, `/parallel-check` to Skills table
- Added detailed usage sections for each new skill

**plan.md.template Updates:**

- Added Mermaid phase flow diagram
- Added Completion Tracking table with status and verification columns

**Template Version:** 1.7.0

**Benefits:**

- Catches problems immediately after agent completion (not at end of session)
- Structured recovery prevents accumulating failures
- Explicit handoffs ensure agents have clear scope
- Checkpoints prevent orchestrator role drift
- Pre-flight checks prevent unsafe parallel execution

**Target:** Improve compliance scores from ~62% to 85%+

### 2.10.0 (2026-02-01)

**Orchestrator Performance Analyzer - Automated Compliance Analysis**

Adds `/analyze-orchestrator` skill that parses Claude Code session transcripts to evaluate orchestrator pattern compliance and detect anti-patterns.

**New Files:**

- **`.claude/scripts/transcript-parser.mjs`** - JSONL parsing and metric extraction from Claude Code session transcripts
- **`.claude/scripts/orchestrator-rules.mjs`** - Rule definitions and evaluation logic for orchestrator compliance
- **`.claude/scripts/analyze-session.mjs`** - CLI tool for running analysis
- **`.claude/skills/analyze-orchestrator/SKILL.md`** - Skill definition and documentation

**Rules Evaluated:**

| Rule | Severity | Detection |
|------|----------|-----------|
| Plan Mode Usage | Warning | EnterPlanMode for non-trivial tasks |
| Agent Delegation | Error | Task tool calls present |
| File Read Limit | Warning | ≤3 consecutive reads in main context |
| No Direct Code Write | Error | No Write/Edit to code files in main |
| Explore Agent Usage | Warning | Explore agent for >5 reads |
| Batch File Limit | Warning | ≤20 files per agent delegation |

**Usage:**

```bash
# Analyze most recent session
node .claude/scripts/analyze-session.mjs

# Analyze specific session
node .claude/scripts/analyze-session.mjs abc123

# List available sessions
node .claude/scripts/analyze-session.mjs --list

# Batch analysis of all sessions
node .claude/scripts/analyze-session.mjs --batch
```

**Output:**

- Single session: `.claude/audit/analysis/{session-id}-analysis.md`
- Batch: `.claude/audit/analysis/batch-analysis-{date}.md`

**Documentation Updates:**

- **CLAUDE.md** - Added `/analyze-orchestrator` to Audit Trail section, updated Key Directories
- **.claude/roster.md** - Added Skills section with all available skills
- **.gitignore** - Added `.claude/audit/analysis/` for generated reports

**Benefits:**

- Automated detection of orchestrator anti-patterns
- Quantified compliance scoring (0-100%)
- Trend analysis across sessions with `--batch`
- Actionable recommendations for improvement
- Integrates with existing audit trail system

### 2.9.0 (2026-01-31)

**Audit Trail System - Process Tracking for Framework Improvement**

Adds comprehensive audit trail to track activity during project creation and development.

**New Files (Project Builder):**

- **`.claude/audit/README.md`** - Audit system documentation
- **`.claude/audit/sessions/`** - Auto-generated session logs directory
- **`.claude/audit/decisions/`** - Manual decision records directory
- **`.claude/hooks/audit-hooks.sh`** - Hook script for automatic capture
- **`.claude/settings.json`** - Hook registrations for audit events
- **`.claude/skills/audit-decision/SKILL.md`** - Decision recording skill
- **`.claude/skills/audit-summary/SKILL.md`** - Retrospective analysis skill

**New Template Files (for created projects):**

- **`.claude/templates/orchestrator/.claude/audit/README.md.template`**
- **`.claude/templates/orchestrator/.claude/hooks/audit-hooks.sh.template`**
- **`.claude/templates/orchestrator/.claude/skills/audit-decision/SKILL.md.template`**
- **`.claude/templates/orchestrator/.claude/skills/audit-summary/SKILL.md.template`**

**Template Updates:**

- **`settings.json.template`** - Added audit hook registrations:
  - `SubagentStart` - Captures agent delegations
  - `SubagentStop` - Captures agent completions
  - `PostToolUseFailure` - Captures tool failures
  - `SessionStart` / `SessionEnd` - Session boundaries

- **`manifest.json.template`** - Added audit section, bumped templateVersion to 1.6.0

- **`roster.md.template`** - Added `/audit-decision` and `/audit-summary` to Skills table

**Project Builder Changes:**

- **`project-initializer.md`** - Deploy audit system to created projects:
  - Creates `.claude/audit/` directory structure
  - Creates hooks and skills
  - Added to verification checklist

- **`CLAUDE.md`** - Added "Audit Trail System" section documenting:
  - What gets tracked
  - Directory structure
  - Skills usage
  - Configuration

**What Gets Tracked:**

| Component | Capture Method | Purpose |
|-----------|----------------|---------|
| Session Log | Automatic (hooks) | Agent delegations, completions, failures |
| Decision Log | Manual (`/audit-decision`) | Alternatives considered, rationale |
| Audit Summary | Manual (`/audit-summary`) | Retrospective analysis |

**Session Log Entry Format:**

```markdown
### 10:15:00 - Agent Delegated: dev-backend

| Field | Value |
|-------|-------|
| Timestamp | 2026-01-31T10:15:00 |
| Agent | dev-backend |
| Task | Implement user endpoints |
| Status | In Progress |
```

**Decision Record Structure:**

- Decision summary
- Alternatives considered (with pros/cons)
- Rationale for choice
- Constraints that influenced decision
- Expected consequences

**Benefits:**

- Debug why project creation went wrong
- Identify patterns in failures
- Improve agent prompts based on outcomes
- Enable retrospective analysis
- Track significant architectural decisions

### 2.8.0 (2026-01-31)

**App Design Phase + Mandatory Connection Verification**

This release adds an App Design Phase to created projects and makes database connection verification mandatory before project handoff.

**Problem Addressed:**
- Projects were handed off as scaffolding that didn't actually work
- Users had to set up databases after handoff, leading to incomplete projects
- No guidance on designing app features, pages, and UI before implementation
- Resulted in incomplete features and inconsistent UIs

**New Orchestrator Template Files:**

- **`dev-designer.md.template`** - New agent for App Design Phase
  - Guides users through feature discovery
  - Creates page inventories and user flows
  - Defines component hierarchies
  - Specifies data requirements per component
  - Produces implementation blueprint

- **`app-design/SKILL.md.template`** - New skill `/app-design`
  - User-invoked skill for App Design Phase
  - Guides through structured design questions
  - Creates `.claude/design/app-design.md`

**Template Updates:**

- **`CLAUDE.md.template`** - Added App Design Phase section
  - Documents when and how to run `/app-design`
  - Explains implementation flow: PROJECT → APP DESIGN → IMPLEMENTATION

- **`roster.md.template`** - Updated with:
  - `dev-designer` agent in Design & Architecture section
  - `/app-design` skill in Skills table
  - "Using /app-design" documentation section
  - Updated SDLC Phase Mapping with APP DESIGN phase

- **`agents/README.md.template`** - Added `dev-designer` to agent table

- **`manifest.json.template`** - Bumped templateVersion to 1.5.0

**Project Builder Changes:**

- **`project-discovery.md`** - Database setup now defaults to "during-init"
  - Discourages deferring service provisioning
  - Clear messaging that deferring leads to incomplete projects

- **`project-initializer.md`** - Mandatory verification before handoff
  - Added connection verification to CRITICAL MUST ALWAYS section
  - Made health check mandatory (not optional)
  - Updated output report with verification status
  - Clear handoff with step-by-step instructions to start Claude Code in project

- **`CLAUDE.md`** (Project Builder) - Updated workflow:
  - New Project Flow includes verification step
  - Added "Before handoff" quality gate
  - Updated example session to show new workflow

**New Workflow:**

```
Discovery → Architecture → Tech Validation → Initialize → VERIFY → HANDOFF
                                                  ↓
                                            ✅ Deps install
                                            ✅ Build passes
                                            ✅ DB connects
                                            ✅ Dev runs
                                                  ↓
                                          HANDOFF MESSAGE:
                                          "Open terminal → cd project → claude → /app-design"
                                                  ↓
                                          [User starts Claude Code in project]
                                                  ↓
                                          /app-design → Implementation
```

**Key Design Decisions:**

1. **Set up ALL services the project needs** - Database, hosting, auth, storage, etc.
2. **Verify everything that's configured** - All services must work before handoff
3. **Smart about what's needed** - Don't force DB for stateless apps, but DO set up what IS needed
4. **Ready for App Design** - Handoff = infrastructure complete, ready to design features
5. **Clear handoff status** - Report shows all services configured and verified
6. **Explicit next step** - Handoff tells user exactly how to start: open terminal → cd to project → start Claude Code → run /app-design

**Goal:** When the project is handed off, ALL infrastructure is working. The user runs `/app-design` to design features, then implementation agents build them. No more service setup needed.

**Services to consider during discovery:**

| Service | When Needed |
|---------|-------------|
| Database | Stateful apps (user data, content) |
| Hosting | All deployed apps |
| Auth | Apps with user accounts |
| Storage | File uploads, images |
| Email | Notifications, verification |
| Payments | E-commerce, subscriptions |

### 2.7.0 (2026-01-31)

**User Preferences Memory - Faster Project Creation**

This release adds a user preferences system that remembers choices from previous projects to speed up future project creation.

**New Files:**

- **`.gitignore`** - Ignores local files (`*.local.*`, `.env`, etc.)
- **`.claude/user-preferences.local.md`** - Stores user preferences (gitignored)

**What Gets Saved:**

| Category | Examples |
|----------|----------|
| User profile | Technical level, communication preferences |
| Credentials | Supabase connection strings, API keys |
| Database strategy | Shared instance with schema isolation |
| Version preferences | Prefers latest (Next.js 15, Tailwind v4, Prisma 7) |
| Project history | Schemas created, projects built |

**Key Design Decision: Always Confirm**

Preferences are never silently applied. The flow is:

1. Check if preferences file exists
2. Present summary: "I found your saved preferences from previous projects..."
3. Ask: "Would you like to use the same choices for this project?"
4. Respect answer: Yes / No / Mostly, but change X
5. Update preferences after project creation

**Agent Updates:**

- **`project-discovery.md`** - Added preferences check with confirmation flow
- **`project-initializer.md`** - Uses confirmed preferences for `.env.local` creation
- **`CLAUDE.md`** - Added User Preferences section with confirmation template

**Benefits:**

```
Before: 10+ questions about tech level, database, versions, credentials
After:  Confirm preferences → Just project name and specific features
```

**Example Confirmation:**

```
I found your saved preferences from previous projects:

Tech Stack: Next.js 15, Tailwind v4, Prisma 7
Database: Supabase (shared instance, schema isolation)
Deployment: Vercel + GitHub
Existing schemas: public, snakey

Would you like to use these same choices for this project?
```

### 2.6.0 (2026-01-31)

**Context7 MCP Integration - Live Documentation for LLMs**

This release adds Context7 MCP server integration to all created projects, providing live documentation lookup for fast-moving libraries.

**New Template Files:**

- **`.mcp.json.template`** - Context7 MCP server configuration

**Template Updates:**

- **`manifest.json.template`** - Added `mcp` section for MCP server tracking (v1.4.0)
- **`roster.md.template`** - Added "MCP Servers (External Tools)" section
- **`CLAUDE.md.template`** - Added "Live Documentation (MCP)" section
- **`stack.md.template`** - Added Context7 availability column

**Agent Updates:**

- **`project-tech-validator`** - Now checks Context7 library availability during validation
- **`project-initializer`** - Includes Context7 status in generated stack.md

**Why this matters:**

- LLMs generate broken code when working with libraries newer than training data
- Context7 fetches up-to-date, version-specific documentation in real-time
- Eliminates hallucinated APIs and outdated patterns
- Reduces debugging time for fast-moving frameworks (Next.js, Tailwind, Prisma)

**Usage in created projects:**
```
use context7 for Next.js app router
```

### 2.5.0 (2026-01-31)

**Auditor Agents - Comprehensive Review Perspectives**

This release adds specialized auditor agents to the orchestrator framework for focused code and project reviews.

**New Auditor Agents (8 total):**

| Agent | Focus Area |
|-------|------------|
| `dev-auditor-performance` | N+1 queries, bundle size, renders, caching, memory leaks |
| `dev-auditor-accessibility` | WCAG 2.1 AA, keyboard nav, ARIA, screen readers |
| `dev-auditor-architecture` | Layer violations, coupling, circular dependencies, dead code |
| `dev-auditor-testing` | Coverage gaps, test quality, flaky tests, edge cases |
| `dev-auditor-api` | REST conventions, HTTP methods, status codes, consistency |
| `dev-auditor-docs` | README completeness, API docs, code comments, accuracy |
| `dev-auditor-dependencies` | CVEs, license compatibility, outdated packages, bloat |
| `dev-auditor-errors` | Error boundaries, user messages, logging, graceful degradation |

**New Review Checklists (4 total):**

- `accessibility-review.md.template` - WCAG 2.1 Level A and AA checklist
- `performance-review.md.template` - Core Web Vitals, database, frontend, API performance
- `architecture-review.md.template` - Layers, modules, SOLID principles, scalability
- `api-review.md.template` - REST conventions, HTTP methods, status codes, security

**Key Design Decisions:**

- All auditors use **Review Agent** role classification (read-only, reports only)
- Auditors can run in parallel with each other and with implementation agents
- Each auditor has severity levels (CRITICAL, HIGH, MEDIUM, LOW)
- Structured report format for consistent output
- Integration with SDLC phase mapping (new AUDIT phase)

**Template Updates:**

- `roster.md.template` - Added Auditor Agents section, updated SDLC mapping, parallel execution matrix
- `agents/README.md.template` - Added Auditor Agents table
- `manifest.json.template` - Bumped templateVersion to 1.3.0

**Usage Patterns:**

Pre-deployment review:
```
dev-auditor-dependencies → dev-auditor-security → dev-auditor-performance
```

Accessibility sprint:
```
dev-auditor-accessibility → [fix with dev-frontend] → dev-auditor-accessibility (verify)
```

### 2.4.0 (2026-01-31)

**Always Plan Mode + Orchestrator Self-Reminder**

This release inverts plan mode behavior and adds mechanisms to prevent orchestrator role drift during long sessions.

**Problem Addressed:**
- Plan mode was skipped for "trivial" tasks, but the definition was subjective
- Orchestrators would drift into doing work directly during long sessions
- No recurring mechanism to remind orchestrators of their role
- Context degradation caused orchestrators to "forget" delegation rules

**Key Changes to Project Builder:**

- **Replaced "Trivial vs Non-Trivial" with "Plan Mode: Always Enter (Except Extremely Simple)"**
  - Default behavior: ALWAYS enter plan mode
  - "Extremely simple" requires ALL criteria: single file, exact instruction, zero ambiguity, no delegation
  - When in doubt, enter plan mode

- **Added "Orchestrator Management Declaration" section**
  - MANDATORY declaration when entering plan mode
  - Format: Task, Agents needed, Sequence, My role
  - Forces conscious decision about delegation vs direct work

- **Added "Long Task Self-Reminder Protocol" section**
  - Self-reminder triggers (3+ agents, >3 files, any file write)
  - Self-reminder checklist table
  - Recovery pattern for when direct work has begun
  - Key principle reinforcement

- **Updated Orchestrator Responsibilities** (now 7 items):
  1. Greet and clarify
  2. Enter plan mode (NEW)
  3. Declare management approach (NEW)
  4. Delegate to agents
  5. Self-check periodically (NEW)
  6. Review handoffs
  7. Summarize results

**New Skill:**

- **`/orchestrator-checkpoint`** - Self-reminder skill for orchestrator role
  - Identity check: PLAN, DELEGATE, REVIEW, COORDINATE, COMMUNICATE
  - Constraint verification table
  - Current task audit questions
  - Resume guidance

**Template Updates:**

- **CLAUDE.md.template** - Added all new sections (Plan Mode, Declaration, Self-Reminder)
- **roster.md.template** - Added `/orchestrator-checkpoint` to Skills table
- **orchestrator-checkpoint/SKILL.md.template** - New skill template for created projects
- **handoff-full.md** - Added orchestrator checkpoint header

**Impact:**
- Plan mode is now the default, not the exception
- Orchestrators must explicitly declare their management approach
- Built-in reminders prevent role drift during long sessions
- All created projects inherit these behaviors

### 2.3.0 (2026-01-28)

**Credentials Security - Defense in Depth**

This release adds comprehensive credentials and secrets protection to all created projects, preventing accidental commits of sensitive data.

**New Template Files:**

- **`.gitignore.template`** - Comprehensive gitignore protecting credentials
  - Blocks `.env` files (except `.env.example`)
  - Ignores credential files (`.pem`, `.key`, `.p12`, `.pfx`)
  - Ignores `secrets/` directory

- **`.env.example.template`** - Environment variable documentation template
  - Placeholder pattern for required variables
  - Uses `{{PROJECT_NAME}}` placeholders

- **`.claude/hooks/check-secrets.sh.template`** - Claude Code PreToolUse hook
  - Blocks `git add .env` commands
  - Blocks commits if `.env` is staged
  - Scans for hardcoded secrets using regex patterns
  - Provides helpful error messages

- **`.claude/settings.json.template`** - Claude Code settings with security hooks
  - Configures PreToolUse hook for Bash commands
  - Adds deny permissions for reading `.env` files directly

- **`.claude/skills/commit/SKILL.md.template`** - Safe commit skill
  - Mandatory pre-commit secret checks
  - Documents proper commit workflow
  - Instructions for handling detected secrets

**Template Updates:**

- **CLAUDE.md.template** - Added "Credentials & Secrets Management" section
  - 5 critical rules for handling secrets
  - Correct environment variable patterns
  - File tracking table (committed vs gitignored)
  - Enforcement documentation

- **roster.md.template** - Added `/commit` skill to Skills table

- **security-review.md.template** - Added "Credentials Security" checklist (8 items)

- **SECURITY.md.template** - Added credentials management tracking
  - Status table for credential hygiene
  - Environment variables documentation
  - Secrets rotation tracking

**New Agent:**

- **`@template-updater`** - Agent for updating orchestrator templates

**Enforcement Layers:**

| Layer | Mechanism | Catches |
|-------|-----------|---------|
| 1 | `.gitignore` | Accidental staging |
| 2 | Claude hooks | Claude's git operations |
| 3 | `/commit` skill | Manual commits via Claude |
| 4 | CLAUDE.md rules | Hardcoded secrets in code |

### 2.2.0 (2026-01-26)

**Mandatory Agent Creation - Closing Delegation Loopholes**

This release addresses critical issues where orchestrators performed work directly instead of delegating to agents, even when delegation was clearly appropriate.

**Problem Identified:**
- Orchestrators (both Project Builder and created project orchestrators) were doing too much work directly
- "Missing agent" was used as an excuse to bypass delegation
- Vague language ("extensive code") allowed subjective interpretation
- No enforcement mechanism for agent creation when one doesn't exist

**Key User Feedback Addressed:**
> "Missing an agent isn't an excuse to not use an agent. When there isn't an agent then you must create an agent with the necessary context to perform the task."

**Changes to Project Builder:**

- **Created `@project-updater` agent** (`.claude/agents/project-updater.md`)
  - Handles updates to existing projects created by the Project Builder
  - Follows role classification (Coding agent)
  - Includes batching and handoff protocols

- **Updated CLAUDE.md** with stricter delegation rules:
  - Changed "Write extensive code directly" to "Write any code files directly (always delegate)"
  - Added "Read more than 3 files directly without delegating to an agent"
  - Added "Skip creating agents when one doesn't exist for the task"
  - Added new "Mandatory Agent Creation" section with:
    - Explicit triggers for when to create agents
    - 4-step agent creation process
    - Clear principle: "The orchestrator's job is to COORDINATE, not to DO the work"

- **Updated roster.md** with enforcement rules:
  - Added `@project-updater` to agent overview
  - Added "Mandatory Agent Creation Rules" section
  - Added action table mapping task types to required agents
  - Added "Anti-Pattern: Direct Work" section explicitly listing forbidden behaviors

**Changes to Orchestrator Templates (for new projects):**

- **Updated CLAUDE.md.template** with same stricter delegation rules
- **Updated roster.md.template** with:
  - "Mandatory Agent Creation Rules" section
  - Agent template for creating new agents on-the-fly
  - Anti-pattern documentation

**Impact:**
- Orchestrators can no longer claim "no agent exists" as justification for direct work
- Clear enforcement: create the agent first, then delegate
- Quantified limits replace vague qualifiers (>3 files = delegate)
- Both Project Builder and created projects now enforce this pattern

### 2.1.0 (2026-01-26)

**Migration Accuracy Improvements**

This release fixes critical issues discovered during migration testing where created CLAUDE.md files contained incorrect paths and domain syntax.

- **Enhanced `@project-analyzer` agent:**
  - Added Step 3b: Domain-Specific Syntax Detection
    - Scans source files for project-specific syntax (placeholders, DSLs)
    - Documents ACTUAL syntax used, not generic patterns
  - Added Step 3c: Extended Context Identification
    - Identifies rich context files (docs/, README.md, extended AI context)
    - Notes key sections that should be transferred
  - Updated Step 3: Structure Analysis
    - Now requires reading ACTUAL directories (not assuming paths)
    - Output includes path mapping table for CLAUDE.md
  - Updated output template with new required sections

- **Enhanced `@project-migrator` agent:**
  - Added Step 5b: Extract Domain Knowledge from Quarantine
    - Reads quarantined files to extract commands, syntax, conventions
    - Reads extended context files for schemas, env vars, domain concepts
  - Enhanced Step 7: Create CLAUDE.md
    - MUST use actual paths from analysis
    - MUST use actual domain syntax from source files
    - MUST copy commands from quarantined files
  - Added Step 10a: Path Verification
    - Verifies every path in CLAUDE.md exists
    - Requires fixing mismatches before reporting success
  - Added Step 10b: Syntax Verification
    - Verifies domain syntax matches source code
  - Updated verification checklist with Path and Domain Accuracy sections
  - Updated output report to include verification status

- **Updated CRITICAL lists:**
  - "Use ACTUAL directory paths" added to MUST ALWAYS
  - "Use ACTUAL domain syntax" added to MUST ALWAYS
  - "Verify CLAUDE.md paths match actual structure" added
  - "Use assumed directory structures" added to NEVER
  - "Use generic syntax when project has specific syntax" added to NEVER
  - "Skip path verification" added to NEVER
  - "Ignore extended context files" added to NEVER

- **Updated Migration Flow in main CLAUDE.md:**
  - Added verification step (Step 4)
  - Emphasized actual paths and syntax throughout

**Why this matters:**
- Migrations were producing CLAUDE.md files with incorrect paths (e.g., `backend/` instead of `src/server/`)
- Domain-specific syntax was being replaced with generic patterns (e.g., `{{x}}` instead of `[[x]]`)
- Extended context (schemas, env vars) was being lost during migration
- These issues caused confusion when using the migrated project

### 2.0.0 (2026-01-26)

**BREAKING CHANGE: Context Management Architecture**

This major release introduces a comprehensive context management system to prevent AI focus degradation in long sessions.

- **Added Context Management Section to CLAUDE.md**
  - Orchestrator responsibilities (minimal context): greet, clarify, delegate, summarize only
  - Trivial vs non-trivial task detection (auto plan mode for non-trivial)
  - Context shift detection (triggers plan mode and context clearing)
  - Agent role classification: Research, Coding, Testing, Review
  - Structured handoff requirements
  - Context clearing protocol

- **Added Agent Role Classification**
  - **Research Agents** (broad read, docs only write): discovery, architect, tech-validator, analyzer
  - **Coding Agents** (focused read, 15 file max): initializer, migrator
  - Role constraints added to all Project Builder agents
  - Role constraints propagated to created project agent templates

- **Added Structured Handoff System**
  - Created `.claude/templates/handoffs/handoff-full.md` (100 lines max)
  - Created `.claude/templates/handoffs/handoff-mini.md` (20 lines max)
  - Handoffs include: Task, Files to Modify, Reference Files, Critical Context, Anti-Context
  - Research agents MUST identify files and patterns in handoffs

- **Added "Need More Research" Protocol**
  - Coding/Testing agents STOP when they hit knowledge gaps
  - Return `RESEARCH_NEEDED: {question}` instead of exploring
  - Orchestrator spawns Research agent for targeted answer
  - Prevents Coding agents from accumulating exploration context

- **Updated roster.md with Handoff Requirements**
  - Full handoff schema and requirements
  - Mini handoff for targeted research requests
  - Protocol diagram for research requests

- **Propagated to Orchestrator Templates**
  - Added Context Management section to CLAUDE.md.template
  - Added role classifications to roster.md.template
  - Added role constraints to representative dev-* agent templates
  - Created handoff templates for created projects

**Why This Matters:**
- Long contexts cause AI focus degradation
- Main context accumulating exploration wastes resources
- Structured handoffs preserve critical info without bloat
- Role-based constraints keep agents focused

### 1.9.0 (2026-01-26)

- **Added Tech Revalidation for Existing Projects** - Detect and handle validation drift
  - Created `/tech-revalidate` skill for created projects
  - Triggers when: stack.md missing, AI cutoff changed, tech versions changed, >90 days since validation
  - Detects version changes from package manifests (package.json, go.mod, etc.)
  - Performs targeted validation for changed technologies only
  - Updates manifest.json and stack.md with new validation state
- Updated manifest.json.template (v1.2.0):
  - Added `techValidation` section with tracking fields
  - `lastValidated` - timestamp of last validation
  - `aiTrainingCutoffAtValidation` - AI cutoff at validation time
  - `validatedVersions` - snapshot of tech versions
  - `confidenceLevels` - confidence per technology
- Updated stack.md.template:
  - Added Confidence column to AI Version Awareness table
  - Added "Confidence Level Meaning" section
  - Added "When to Revalidate" guidance
  - Changed header from "Last Researched" to "Last Validated"
- Updated CLAUDE.md.template:
  - Added "Tech Stack Revalidation" section documenting /tech-revalidate
  - Updated Session Continuity Protocol for long breaks
  - Updated "When upgrading dependencies" to use /tech-revalidate
- Updated roster.md.template:
  - Added /tech-revalidate to Skills table
  - Added "Using /tech-revalidate" section
- **Why this matters**: Projects evolve - dependencies get upgraded, Claude gets updated. This ensures the tech validation stays current with the actual project state.

### 1.8.0 (2026-01-26)

- **Added Technology Validation Phase (Phase 2b)** - New stage in project creation workflow
  - Created `@project-tech-validator` agent for deep AI knowledge validation
  - Validates AI knowledge even for technologies within training cutoff (sparse data scenarios)
  - Assesses confidence levels: High, Medium, Low, Unknown
  - Generates validation patterns developers can test
  - Creates verification task checklists
  - Documents integration concerns between technologies
  - Produces Technology Validation Report at `.claude/projects/{project-name}-tech-validation.md`
- Updated CLAUDE.md workflow:
  - Added Phase 2b between Architecture and Initialization
  - Updated "Your Role" to include validation step
  - Updated New Project Flow with tech-validator step
  - Updated Agent Delegation section
  - Updated Agent Coordination (scope limits, parallel execution matrix)
  - Added quality gates for technology validation
- Updated roster.md:
  - Added `@project-tech-validator` to agent overview
  - Updated project source diagram
  - Added detailed section for tech-validator
  - Updated initializer to reference validation report
  - Updated parallel execution matrix (now 6x6)
  - Updated directory ownership table
- **Why this matters**: AI training data has uneven coverage. A technology existing before training cutoff doesn't guarantee sufficient training data. This phase identifies knowledge gaps proactively.

### 1.7.0 (2026-01-24)

- Added Terminology section to CLAUDE.md
  - Defines: Project Builder, Created Project, Orchestrator Pattern, Orchestrator Template/Framework (synonyms)
  - Distinguishes Project Builder Agents vs Development Agents
  - Disambiguation examples table for common user requests
  - "When Unsure, Ask" guidance for ambiguous requests
- Added Scope of Changes Reference section to CLAUDE.md
  - Quick reference for modifying Project Builder files
  - Quick reference for modifying Orchestrator Template/Framework files
  - Identifies changes that require updates in both places
- Prevents confusion between Project Builder modifications and template/framework modifications
- Propagated Agent Coordination to Orchestrator Templates (completing 1.6.0 work)
  - Added Agent Coordination section to CLAUDE.md.template
  - Added Agent Coordination Rules section to roster.md.template
  - Added Coordination Rules reference to agents/README.md.template
  - Adapted rules for dev-* agents (vs project-* agents)

### 1.6.0 (2026-01-24)

- Added Agent Coordination section to CLAUDE.md
  - Scope limits per agent (15-20 files max for file-writing agents)
  - Parallel vs sequential execution guidance
  - Batching strategy for large operations
  - Directory ownership table
- Added Agent Coordination Rules section to roster.md
  - Detailed scope limits with rationale
  - Batching strategy with example prompts
  - Parallel execution matrix (5x5 agent compatibility)
  - Directory ownership and conflict risk table
  - Practical workflow examples
- Updated agents/README.md with coordination rules reference
- Added three constraints to orchestrator "You do NOT" section:
  - No >20 files per agent run
  - No parallel file-writing agents
  - No more than 2-3 concurrent agents

### 1.5.0 (2026-01-24)

- Added `/capture` skill for knowledge persistence
  - Agents can self-invoke to save discovered patterns
  - Intelligent file targeting (subdirectory CLAUDE.md, @-mentioned files)
  - Smart editing (updates existing sections, no blind appending)
  - Categories: tech, preference, practice, spec
- Updated LEARNINGS.md template to integrate with /capture
- Updated roster.md template with Skills section
- Updated CLAUDE.md template with Knowledge Capture documentation

### 1.4.0 (2026-01-24)

- Added 6 Task-related tools (TaskCreate, TaskUpdate, TaskList, TaskGet, TaskOutput, TaskStop)
- Added 3 new hook events (PostToolUseFailure, SubagentStart, Setup)
- Added prompt-based hooks documentation (type: "prompt")
- Added 16 new settings fields (sandbox, alwaysThinkingEnabled, etc.)
- Added new skill frontmatter field (`once`)
- Added new subagent frontmatter fields (`permissionMode`, `skills`)
- Added subagent execution modes (foreground/background)
- Added 4 new deployment options (MCP Server, LLM Gateway, Dev Containers, Sandboxing)
- Updated permission patterns with wildcard and subagent examples
- Upgraded Desktop App from "Preview" to "Full release"
- Synced version number with PROJECT VERSION file
