# Claude Code Baseline

> This document captures what the Project Builder knows about Claude Code.
> Used by `/cpm_update` to detect when updates are needed.

**Baseline Date**: 2026-01-31
**Claude Code Version**: Latest (as of January 2026)
**Project Builder Version**: 2.6.0

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
