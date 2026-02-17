# Claude Code Baseline

> This document captures what the Project Builder knows about Claude Code.
> Used by `/cpm_update` to detect when updates are needed.

**Baseline Date**: 2026-02-05
**Claude Code Version**: Latest (as of February 2026)
**Project Builder Version**: 2.22.0

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
| EnterPlanMode | Transition into plan mode | No |
| ExitPlanMode | Signal plan completion for approval | No |

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
| Explore | Haiku | Fast codebase exploration | All except Task, ExitPlanMode, Edit, Write, NotebookEdit |
| Plan | Inherited | Research for Plan Mode | All except Task, ExitPlanMode, Edit, Write, NotebookEdit |
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

### Task Tool Parameters

| Parameter | Type | Purpose |
|-----------|------|---------|
| `prompt` | string | Instructions for the subagent (required) |
| `subagent_type` | string | Agent type: Explore, Plan, general-purpose, Bash, etc. |
| `model` | string | "sonnet", "opus", or "haiku" (default: inherited) |
| `resume` | string | Agent ID to continue previous context |
| `run_in_background` | boolean | Run concurrently; returns `output_file` path |
| `max_turns` | number | Limit agent round-trips |

**Background agents:** Return `output_file` path. Check with Read or tail. Multiple can launch in one message. Non-approved actions auto-denied.

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
| Research/investigate | `Explore` | Fast (Haiku), no file modifications, can run Bash/WebSearch |
| Multi-step implementation | `general-purpose` | Full tool access, can read/write/test |
| Run commands | `Bash` | Isolated terminal context |
| Plan mode research | `Plan` | No file modifications, prevents recursion |

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

> Version history for the Project Builder's Claude Code baseline.
> See @.claude/defaults/claude-code-changelog.md for full version history.
