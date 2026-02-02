# Project Builder Agent Roster

Quick reference for selecting the right agent during project creation and migration.

## Orchestrator Version

@.claude/VERSION

## Agent Overview

| Agent | Purpose | Role | When to Use |
|-------|---------|------|-------------|
| `@project-discovery` | Gather requirements | Research | Starting any project interaction |
| `@project-architect` | Design structure | Research | After discovery (new projects) |
| `@project-tech-validator` | Validate AI knowledge | Research | After architecture (new projects) |
| `@project-initializer` | Create files | Coding | After tech validation |
| `@project-analyzer` | Analyze existing project | Research | Before migration |
| `@project-migrator` | Migrate existing project | Coding | When user has existing codebase |
| `@project-updater` | Update existing projects to newer framework versions | Coding | When updating created projects |

## Agent Role Classification

Agents are classified by their role to enforce context discipline:

| Role | Read Scope | Write Scope | Context Behavior |
|------|------------|-------------|------------------|
| **Research** | Broad (10+ files) | Handoff docs only | CAN accumulate exploration context |
| **Coding** | Handoff + patterns (15 max) | Code files (15 max) | MUST stay focused, request research if stuck |
| **Testing** | Implementation + test patterns | Test files only | MUST stay focused on test scope |

### Research Agents (Broad Read, Narrow Write)

- `@project-discovery` - Explores user requirements, outputs brief
- `@project-architect` - Explores codebase patterns, outputs architecture doc
- `@project-tech-validator` - Researches technologies, outputs validation report
- `@project-analyzer` - Analyzes existing codebase, outputs analysis doc

**Research agent responsibilities:**
1. Explore broadly (10+ files is fine)
2. Identify specific files to modify
3. Find reference files with patterns
4. Produce structured handoff (100 lines max)
5. Drop exploration context after handoff

### Coding Agents (Narrow Read, Narrow Write)

- `@project-initializer` - Creates project files from handoff
- `@project-migrator` - Creates orchestrator files from handoff

**Coding agent constraints:**
1. Only read files listed in handoff + referenced patterns
2. Maximum 15 files modified per run
3. If knowledge gap: STOP and return `RESEARCH_NEEDED: {question}`
4. Do NOT explore - request mini-research instead

### TDD Workflow Support

Testing agents can run BEFORE coding agents for TDD:

```
Research Agent (identifies files + test patterns)
    │
    ├─→ Handoff includes: files to modify, test pattern locations
    │
    ▼
Testing Agent (RED phase) - Optional, runs first if TDD
    │
    ├─→ Reads: handoff + test pattern files
    ├─→ Writes: failing tests based on requirements
    │
    ▼
Coding Agent (GREEN phase)
    │
    ├─→ Reads: handoff + tests + reference files
    ├─→ Writes: minimum code to pass tests
    │
    ▼
Testing/Coding Agent (REFACTOR phase)
```

## Stack Defaults by Project Type

| Project Type | Stack Reference |
|--------------|-----------------|
| web-app | @.claude/defaults/stacks/web-app.md |
| backend-api | @.claude/defaults/stacks/backend-api.md |
| cli-tool | @.claude/defaults/stacks/cli-tool.md |
| library | @.claude/defaults/stacks/library.md |
| desktop-app | @.claude/defaults/stacks/desktop-app.md |
| data-pipeline | @.claude/defaults/stacks/data-pipeline.md |
| mobile-app | @.claude/defaults/stacks/mobile-app.md |

Use the appropriate stack default based on project type.

## AI Version Baseline

@.claude/defaults/ai-known-versions.md

Know which versions Claude can write confident code for vs. which need gotcha documentation.

## Baselines Reference

@.claude/defaults/security-baseline.md
@.claude/defaults/dependency-policy.md
@.claude/defaults/operational-baseline.md
@.claude/defaults/logging-baseline.md

Reference these during discovery for security, dependency, operational, and logging decisions.

## Project Source Options

```
                         DISCOVERY
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    NEW PROJECT         REPO CLONE         LOCAL EXISTING
         │                   │                   │
    PROJECT TYPE        INITIALIZER          ANALYZER
    (web-app, api,      (clone +               │
    cli, library,       orchestrate)        MIGRATOR
    desktop, data)           │                   │
         │              ┌────┴────┐         ┌────┴────┐
    ARCHITECTURE        │        │         │        │
         │           Clone    Apply      Copy    Apply
    TECH VALIDATOR    Repo     Orch     Local    Orch
    (validate AI
     knowledge)
         │
    INITIALIZER
         │
    ┌────┴────┐
    │        │
 Research  Create
 Versions  Files
```

**Project Types:**
- `web-app` → Next.js/React/Vue + database + hosting
- `backend-api` → Express/FastAPI/Go + database + hosting
- `cli-tool` → Go/Rust/Node.js CLI framework
- `library` → npm/PyPI/crates.io package
- `desktop-app` → Electron/Tauri
- `data-pipeline` → Python + pandas/Airflow/dbt
- `mobile-app` → React Native/Flutter (cross-cutting)

## Detailed Agent Selection

### @project-discovery

**Use when:**
- User starts any project-related conversation
- Need to determine if new project or migration
- Requirements are vague or incomplete
- Need to understand user's technical level

**Key behaviors:**
- **First asks: new project or existing project migration?**
- Asks for project name and technical level
- Adapts language to user's technical background
- **For non-technical users: makes recommendations with yes/no confirmations**
- For technical users: offers defaults with option to customize
- **Asks about version control (GitHub recommended)**
- **Asks about service provisioning timing**

**Outputs:**
- Project brief document
- Requirements summary
- Tech stack (defaulted or customized)
- Version control choice
- Service provisioning plan
- Or: handoff to migrator if existing project

### @project-architect

**Use when:**
- Discovery is complete (new project)
- Ready to design project structure
- Need to decide on customizations
- Planning which agents to include

**Outputs:**
- Architecture document
- Agent customization specs
- Directory structure plan
- Security and operational design

### @project-tech-validator

**Use when:**
- Architecture is complete (new projects only)
- Before creating any project files
- Need to assess AI knowledge accuracy
- Want to identify verification tasks for developers

**Why this exists (beyond version checking):**
- A technology may have existed before training cutoff but with sparse training data
- Popular frameworks have more training examples than niche libraries
- Pattern evolution happens even within stable API versions
- Integration patterns between technologies may not be well-documented

**Key behaviors:**
- **Researches EVERY technology** in the stack, not just "new" ones
- **Assesses confidence levels** (High/Medium/Low/Unknown)
- **Identifies sparse training data** scenarios
- **Generates validation patterns** developers can test
- **Creates verification tasks** checklist
- **Documents integration concerns** between technologies

**Confidence Level Criteria:**

| Level | Criteria |
|-------|----------|
| **High** | Version in training, widely used, patterns stable |
| **Medium** | Recent version, pattern changes, or niche usage |
| **Low** | Post-cutoff, limited training data, major changes |
| **Unknown** | Cannot determine AI knowledge state |

**Outputs:**
- Technology Validation Report at `.claude/projects/{project-name}-tech-validation.md`
- Validation findings feed into initializer's `.claude/tech/stack.md`

### @project-initializer

**Use when:**
- **Technology validation is complete** (new projects)
- Architecture is approved by user
- Ready to create actual files
- Project location confirmed
- **Cloning from GitHub repository** (validation skipped for clones)

**Key behaviors:**
- **Handles both new projects and GitHub clones**
- **Uses validation report for confidence-aware file generation** (new projects)
- **For clones: performs inline version research** (validation not required)
- Creates `.claude/tech/stack.md` with versions, gotchas, AND confidence levels
- Creates `.claude/manifest.json` with version tracking
- **Sets up logging infrastructure (Pino + optional Sentry)**
- All other files @-mention tech/stack.md for versions
- **Creates risk mitigation files** based on discovery answers
- **Includes verification tasks** from validation report
- **Customizes checklists** for compliance requirements
- **Selects runbooks** based on operational model
- **Guides service provisioning** if during-init selected

**Outputs:**
- Created project directory
- `.claude/manifest.json` (version tracking)
- `.claude/tech/stack.md` (versions + gotchas + confidence levels)
- `src/lib/logger.ts` (logging infrastructure)
- All orchestrator files (referencing tech/stack.md)
- Customized agents and templates

### @project-analyzer

**Use when:**
- User wants to migrate an existing local project
- Need to understand an existing codebase
- Before running migrator

**Key behaviors:**
- Detects tech stack from manifest files (package.json, go.mod, etc.)
- Identifies existing orchestrator framework if present
- Documents project structure and patterns
- Lists files that need quarantine

**Outputs:**
- Project analysis document
- Tech stack detection
- Files to quarantine list
- Recommended agents

### @project-migrator

**Use when:**
- User has existing local project to add orchestrator to
- After analyzer has run

**Key behaviors:**
- **Copies project to projects directory** (never modifies original)
- **Quarantines conflicting files to `_pre_migration/`**
- Uses quarantined files as context for customization
- Creates fresh orchestrator framework
- Creates manifest.json with migration metadata

**Outputs:**
- Migrated project in new location
- `_pre_migration/` folder with old files (for reference)
- Fresh orchestrator framework
- `.claude/manifest.json` with migration info

## Discovery Topics

Discovery gathers information across these phases:

| Phase | Topic | Non-Technical | Technical |
|-------|-------|---------------|-----------|
| 0 | Project Source | New or existing? | Same |
| 0b | Metadata | Name, technical level | Same |
| 0c | **Project Type** | "Website, tool, data...?" | web-app, backend-api, cli-tool, library, desktop-app, data-pipeline |
| 0d | **Mobile Apps** | "Also need phone app?" | none, react-native, flutter, native |
| 1 | Purpose | Problem, goal | Same |
| 2 | Users | Who, how many | Same |
| 3 | Features | Must-haves | Same |
| 4 | Tech Stack | Automatic (yes/no needs) | Language, framework, preferences |
| 4b | **Version Control** | "Use GitHub?" (yes/no) | GitHub/GitLab/Bitbucket, new/existing |
| 5 | Quality | "Include testing?" (yes/no) | Testing strategy |
| 5b | Security | "Handle sensitive data?" (yes/no) | Level, compliance, auth |
| 5c | Operations | "Have tech team?" (yes/no) | Ops model, on-call |
| 5d | **Logging** | Automatic (language-specific) | Library, aggregation |
| 6 | Constraints | Timeline, budget | Same |
| 6b | Team | "Others working on this?" | Size, growth, onboarding |
| 6c | **Provisioning** | "Set up services now or later?" | Database, hosting, error tracking |

## Non-Technical User Flow

For non-technical users, discovery uses yes/no recommendations:

```
1. "What would you like to build?" → Understand idea
2. "What should we call it?" → Get name
3. Questions about app → Purpose, users, features
4. Recommendations with yes/no:
   ✓ "Use GitHub for code storage?" (recommended)
   ✓ "Include automated testing?" (catches bugs early)
   ✓ "Include error tracking?" (alerts you to issues)
   ✓ "Will users need accounts?"
   ✓ "Handle sensitive data?"
5. "Set up services now or later?" → Provisioning
6. Summary and confirmation → Review before proceeding
```

## Workflow Rules

1. **Always start with discovery** - determines new vs migration vs GitHub clone
2. **Ask for project name and technical level** - these shape the conversation
3. **Adapt to user's technical level** - yes/no for non-technical, discuss for technical
4. **Get user approval** before moving to next phase
5. **One phase at a time** - don't skip steps
6. **Research current versions** - don't use stale training data
7. **Document version gaps** - create gotchas for newer versions
8. **Never modify original** - migration copies, doesn't modify
9. **Document everything** - create records in `.claude/projects/`
10. **Always set up logging** - every project needs AI-readable logs

## Handoff Requirements

All phase transitions MUST use structured handoffs. Handoff templates are in `.claude/templates/handoffs/`.

### Full Handoff (100 lines max)

Used for phase transitions: Research → Coding, Coding → Testing, etc.

**Required sections:**
1. **Task** (5 lines) - Goal, constraints, success criteria
2. **Files to Modify** (table) - Path, action, reason
3. **Reference Files** (table) - Path, why needed
4. **Critical Context** (10 lines) - Tech decisions, patterns
5. **Inputs** (table) - Key-value pairs
6. **Anti-Context** (list) - What to DROP
7. **Verification Anchor** - Questions to confirm context

**Template:** `.claude/templates/handoffs/handoff-full.md`

### Mini Handoff (20 lines max)

Used when Coding/Testing agent requests targeted research.

**Required sections:**
1. **Question** - The specific question asked
2. **Answer** (10 lines) - Direct answer
3. **Files Identified** (table) - Relevant files
4. **Resume With** - Next action for calling agent

**Template:** `.claude/templates/handoffs/handoff-mini.md`

### "Need More Research" Protocol

When a Coding or Testing agent encounters a knowledge gap:

```
Coding Agent encounters gap
    │
    ├─→ STOP immediately (do not explore)
    ├─→ Return: RESEARCH_NEEDED: {specific question}
    │
    ▼
Orchestrator receives request
    │
    ├─→ Spawn Research agent with targeted question
    │
    ▼
Research Agent
    │
    ├─→ Answer specific question only
    ├─→ Return mini-handoff (20 lines max)
    │
    ▼
Coding Agent resumes
    │
    └─→ Continue with targeted answer only
```

**Why this matters:** Prevents Coding agents from accumulating exploration context.

## Mandatory Agent Creation Rules

**The orchestrator MUST NOT work directly when an agent could be created.**

| If Task Requires... | Action |
|---------------------|--------|
| Reading >3 files | Create Research agent or use existing |
| Writing any code files | Create Coding agent or use existing |
| Analyzing existing code | Use `@project-analyzer` or create specific agent |
| Updating existing projects | Use `@project-updater` |
| Any specialized work | Create agent with necessary context |

### Creating a New Agent

When no suitable agent exists:

1. **Determine agent type**: Research, Coding, Testing, or Review
2. **Create agent file**: `.claude/agents/{descriptive-name}.md`
3. **Define clearly**:
   - Role and purpose
   - Role classification (with constraints)
   - Inputs expected
   - Outputs expected
   - CRITICAL: YOU MUST ALWAYS (musts)
   - CRITICAL: NEVER DO THESE (nevers)
4. **Delegate via Task tool**

### Anti-Pattern: Direct Work

The orchestrator should NEVER:
- Read 10+ files to "understand" the codebase (delegate to Research agent)
- Edit multiple files directly (delegate to Coding agent)
- Say "there's no agent for this" and do it directly (CREATE the agent)

## Agent Coordination Rules

**Critical constraint:** Agents cannot spawn sub-agents. Only the orchestrator can delegate.

### Scope Limits per Agent

| Agent | Max Files | Max Scope | When to Batch |
|-------|-----------|-----------|---------------|
| `@project-discovery` | 1 | Single document | Never - always 1 output |
| `@project-architect` | 1 | Single document | Never - always 1 output |
| `@project-tech-validator` | 1 | Single document | Never - always 1 output |
| `@project-initializer` | 15-20 | One directory tree | >20 files or multiple concerns |
| `@project-analyzer` | 1 | Single document | Never - always 1 output |
| `@project-migrator` | 15-20 | One directory tree | >20 files or multiple concerns |

**Why 15-20 files?**
- Keeps agent context focused
- Ensures quality per file (not rushing)
- Allows orchestrator to review between batches
- Prevents agent from running out of context

### Batching Strategy for Large Projects

When initializer or migrator would create >20 files, the **orchestrator** batches:

```
Run 1: "Create core orchestrator structure"
  Scope: CLAUDE.md, .claude/manifest.json, .claude/roster.md, .claude/agents/README.md
  Files: ~5

Run 2: "Create agent definitions"
  Scope: .claude/agents/*.md
  Files: ~8-10

Run 3: "Create status and tracking files"
  Scope: .claude/PROJECT_STATUS.md, .claude/BLOCKERS.md, .claude/LEARNINGS.md, etc.
  Files: ~6

Run 4: "Create checklists and runbooks"
  Scope: .claude/checklists/*.md, .claude/runbooks/*.md
  Files: ~6-10

Run 5: "Create source scaffolding"
  Scope: src/lib/logger.ts, src/app/layout.tsx, etc.
  Files: ~5-10

Run 6: "Create configuration files"
  Scope: package.json, tsconfig.json, .env.example, etc.
  Files: ~5
```

**Orchestrator prompts to agent:**
- "Create only the files in Batch 1: [list]. Do not create other files."
- "Continue with Batch 2: [list]. The following files already exist: [list from Batch 1]."

### Parallel Execution Matrix

**Legend:** ✅ Safe | ⚠️ Caution | ❌ Never

| Agent A ↓ / Agent B → | discovery | architect | tech-validator | initializer | analyzer | migrator |
|----------------------|-----------|-----------|----------------|-------------|----------|----------|
| **discovery** | ❌ | ⚠️ | ❌ | ❌ | ✅ | ❌ |
| **architect** | ⚠️ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **tech-validator** | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **initializer** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **analyzer** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **migrator** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

**Notes:**
- ✅ **discovery + analyzer**: Both gather info, no file conflicts
- ✅ **architect + analyzer**: Both produce separate documents
- ✅ **tech-validator + analyzer**: Both read-only, separate documents
- ⚠️ **discovery + architect**: Only if discovery is complete first
- ❌ **tech-validator + architect**: Validator needs architect output
- ❌ **tech-validator + initializer**: Initializer needs validation report
- ❌ **initializer + anything**: Creates files, must be exclusive
- ❌ **migrator + anything**: Creates files, must be exclusive
- ❌ **Same agent twice**: Never run same agent in parallel

### Directory Ownership

Each agent "owns" specific directories to prevent conflicts:

| Directory | Owner(s) | Access Type | Conflict Risk |
|-----------|----------|-------------|---------------|
| `.claude/projects/` | discovery, architect, tech-validator | Write briefs/docs | Low (different files) |
| `.claude/agents/` | initializer, migrator | Write agent defs | High (same files) |
| `.claude/checklists/` | initializer, migrator | Write checklists | High |
| `.claude/runbooks/` | initializer, migrator | Write runbooks | High |
| `.claude/tech/` | initializer, migrator | Write stack.md | High |
| `src/` | initializer only | Write source | Medium |
| `_pre_migration/` | migrator only | Write backups | None (exclusive) |
| Project root | initializer, migrator | Write configs | High |

**Rule:** If two agents share a directory ownership, they cannot run in parallel.

**Note:** Tech-validator writes to `.claude/projects/` (validation report), not `.claude/tech/`. The initializer reads the validation report and uses it when creating `.claude/tech/stack.md`.

### When to Run Agents in Parallel

**Parallel is appropriate when:**
1. Gathering information from different sources (discovery + analyzer)
2. Both agents are read-only
3. Outputs go to completely different locations
4. Neither agent needs the other's output

**Sequential is required when:**
1. Agent B needs Agent A's output (dependency chain)
2. Both agents write to same directory
3. User approval is needed between steps
4. Order matters for correctness

### Practical Examples

**New Project Flow (all sequential):**
```
discovery → [user approval] → architect → [user approval] → tech-validator → initializer (batched)
```

**Migration Flow with Parallel Opportunity:**
```
discovery ──┐
            ├──→ [merge info] → migrator (batched)
analyzer ───┘
```
Here, discovery and analyzer CAN run in parallel since:
- Discovery gathers user requirements (writes to `.claude/projects/`)
- Analyzer reads existing code (writes to `.claude/projects/`)
- No file overlap, different concerns

**Large Project Initialization (batched):**
```
initializer(batch1) → initializer(batch2) → initializer(batch3) → ...
```
Each batch is sequential. Never run initializer batches in parallel (same directories).

## Files in Created Projects

### Core Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Development conventions, orchestrator instructions |
| `README.md` | Project overview, setup |
| `ONBOARDING.md` | Developer onboarding guide |
| `.env.example` | Environment variable template |
| `src/lib/logger.ts` | Configured logging infrastructure |

### Orchestrator Files

| File | Purpose |
|------|---------|
| `.claude/manifest.json` | Version tracking, project metadata |
| `.claude/tech/stack.md` | Current versions, gotchas |
| `.claude/tech/dependencies.md` | Dependency health tracking |
| `.claude/PROJECT_STATUS.md` | Sprint planning, tasks |
| `.claude/BLOCKERS.md` | Active blockers |
| `.claude/LEARNINGS.md` | Accumulated insights |
| `.claude/PROCESS_LOG.md` | Session notes |
| `.claude/roster.md` | Agent selection guide |
| `.claude/SECURITY.md` | Security overview |
| `.claude/TECH_DEBT.md` | Technical debt tracker |
| `.claude/agents/` | Development agents |
| `.claude/checklists/` | Review checklists |
| `.claude/runbooks/` | Operational procedures |

### Runbook Selection

| Ops Model | On-Call | Runbooks Included |
|-----------|---------|-------------------|
| Developer | none | rollback, deployment |
| Developer | business-hours | + on-call |
| Ops Team | any | + incident-response, on-call, database-recovery |
| Managed | any | Reference docs only (vendor links) |

## Manifest Structure

Created projects include `.claude/manifest.json` tracking:

```json
{
  "orchestrator": { "version": "...", "templateVersion": "1.1.0" },
  "project": {
    "name": "...",
    "slug": "...",
    "type": "web-app|backend-api|cli-tool|library|desktop-app|data-pipeline",
    "primaryLanguage": "typescript|python|go|rust|swift|kotlin|...",
    "mobileApps": "none|react-native|flutter|native"
  },
  "created": { "date": "...", "method": "initialization|repo-clone|migration" },
  "ai": { "trainingCutoff": "2025-05", "knownVersionsBaseline": "..." },
  "versionControl": { "provider": "github|gitlab|bitbucket|none", "repository": "...", "clonedFrom": null },
  "services": { "database": "supabase|firebase|aws-rds|...", "hosting": "vercel|railway|aws|...", "errorTracking": "sentry|..." },
  "logging": { "library": "pino|structlog|zerolog|...", "errorTracking": "sentry", "aggregation": "..." },
  "migration": { "migratedFrom": null, "originalPath": null, "preMigrationBackup": null }
}
```

## Skills

Project Builder skills for common workflows:

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/capture` | Persist knowledge to project files | When discovering patterns to remember |
| `/audit-decision` | Record significant decisions | Before/after major architectural choices |
| `/audit-summary` | Generate session retrospective | After project creation/migration |
| `/analyze-orchestrator` | Evaluate session-level compliance | After single-session tasks |
| `/analyze-task` | Evaluate task-level compliance | After multi-session tasks (plan in one, execute in another) |
| `/cpm_update` | Update Project Builder | When Claude Code capabilities change |

### /analyze-orchestrator

Parses Claude Code session transcripts to detect orchestrator anti-patterns:

**Usage:**
```
/analyze-orchestrator [session-id] [--batch] [--list]
```

**Rules evaluated:**
- Plan mode usage for non-trivial tasks
- Agent delegation (vs. direct work)
- File read limits (≤3 in main context)
- No direct code writes in main context
- Explore agent usage for >5 reads
- Batch file limits (≤20 per delegation)

**Output:** Markdown report in `.claude/audit/analysis/`

**When to use:**
- After completing project creation/migration
- During retrospectives
- Periodically with `--batch` for trends

### /analyze-task (Task-Level)

Analyzes orchestrator compliance at the **task level** rather than session level. Use this when work spans multiple sessions.

**Why Task-Level Analysis?**

Session-level analysis can produce false violations when:
- Planning happens in session A, execution in session B
- User accepts plan (context clears for execution session)
- Multi-session workflows are used intentionally

Task-level analysis groups related sessions and evaluates compliance across the full task lifecycle.

**Usage:**
```bash
# Analyze most recent task
node .claude/scripts/analyze-task.mjs

# Analyze task by slug
node .claude/scripts/analyze-task.mjs --slug goofy-twirling-orbit

# Find and analyze task containing a specific session
node .claude/scripts/analyze-task.mjs --session c497b649

# List all tasks with summaries
node .claude/scripts/analyze-task.mjs --list

# Show task timeline visualization
node .claude/scripts/analyze-task.mjs --timeline

# Batch analyze all tasks
node .claude/scripts/analyze-task.mjs --batch
```

**Task Linkage Signals:**

| Signal | Reliability | Description |
|--------|-------------|-------------|
| Slug match | High | Same slug across sessions |
| Plan content | Very High | `planContent` field in execution session |
| Transcript reference | High | Execution session references planning transcript |
| Timing proximity | Medium | Sessions within 5 minutes |

**Task Rules Evaluated:**

| Rule | Description |
|------|-------------|
| Plan Mode Used | Plan mode in ANY session of task |
| Plan Approved | ExitPlanMode called |
| Agent Delegation | Delegations across task lifecycle |
| No Direct Code Writes | Aggregated across all sessions |
| File Exploration Delegated | Explore agent usage |
| Task Completion | Task reached completion status |

**Output:** Task reports saved to `.claude/audit/analysis/task-{slug}.md`

**When to use:**
- After completing multi-session tasks
- When session-level analysis shows unexpected violations
- To understand cross-session task compliance
- Periodically with `--batch` for trends
