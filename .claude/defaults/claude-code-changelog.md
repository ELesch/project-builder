# Claude Code Baseline - Changelog

> Version history for the Project Builder's Claude Code baseline.
> See @.claude/defaults/claude-code-baseline.md for current capabilities.

---

## Changelog

### 2.22.0 (2026-02-05)

**Claude Code Baseline Accuracy - Tool & AI Confidence Updates**

Fixes discrepancies between baseline documentation and actual Claude Opus 4.6 capabilities. Updates AI version confidence levels.

**Baseline Fixes:**

| Issue | Before | After |
|-------|--------|-------|
| Missing tools | EnterPlanMode, ExitPlanMode not listed | Added to Built-in Tools table |
| Explore/Plan tool access | "Read-only" | All except Task, ExitPlanMode, Edit, Write, NotebookEdit |
| Task tool parameters | Not documented | prompt, model, resume, run_in_background, max_turns |
| Agent selection language | "read-only" | "No file modifications" |

**AI Version Confidence Updates:**

| Technology | Before | After | Reason |
|------------|--------|-------|--------|
| Next.js | 14.x (Major gap) | 15.x (Minor) | Released Oct 2024, within cutoff |
| React | 18.x (Moderate gap) | 19.x (Minor) | Released Dec 2024, good coverage |
| Tailwind CSS | 3.x (Major gap) | v4 (Moderate) | Released Jan 2025, limited early training |
| NextAuth.js | 4.x (Major gap) | Auth.js v5 (Moderate) | Rebranded 2024, patterns changed |
| TypeScript | 5.3 | 5.7 | Minor bump |
| Node.js | 20.x LTS | 22.x LTS | LTS since Oct 2024 |

**Files Changed:**

| File | Change |
|------|--------|
| `claude-code-baseline.md` | Added tools, fixed subagent access, added Task parameters |
| `ai-known-versions.md` | Updated Web Stack and Node.js confidence levels |
| `claude-code-changelog.md` | This entry |
| `VERSION` | 2.21.0 → 2.22.0 |

**Template Version:** 1.14.0 (unchanged)

### 2.21.0 (2026-02-04)

**Skill Naming Standardization - `orc-*` Prefix Convention**

Standardizes orchestrator skill names with a consistent `orc-*` prefix for clarity and discoverability.

**Skill Renames:**

| Old Name | New Name |
|----------|----------|
| `/orchestrator-checkpoint` | `/orc-checkpoint` |
| `/verify-agent` | `/orc-verify` |
| `/recover` | `/orc-recover` |
| `/parallel-check` | `/orc-parallel` |
| `/analyze-orchestrator` | `/orc-analyze` |

**New Skill:**

- **`/orc-framework`** - Verify orchestrator framework integrity and completeness
  - Checks core files (CLAUDE.md, manifest.json, roster.md)
  - Validates knowledge files and domain agents exist
  - Verifies cross-references aren't broken
  - Run after updates or when framework issues suspected

**Benefits:**

- All orchestrator skills easily discoverable with `/orc-` prefix
- Shorter, easier to type
- Clear distinction from app-specific skills
- Consistent naming convention

**Files Changed:**

| Category | Files |
|----------|-------|
| Renamed (Project Builder) | `orc-checkpoint/`, `orc-analyze/` |
| Renamed (Templates) | `orc-checkpoint/`, `orc-verify/`, `orc-recover/`, `orc-parallel/` |
| New (Templates) | `orc-framework/` |
| Updated | `roster.md`, `roster.md.template`, `CLAUDE.md`, `project-initializer.md`, `project-migrator.md` |

**Template Version:** 1.14.0

### 2.20.0 (2026-02-03)

**Communication Discipline - Assumption Surfacing and Failure Mode Prevention**

Adds structured communication patterns inspired by best practices for focused, disciplined agent behavior. Improves assumption handling, confusion management, and completion reporting.

**Key Additions:**

1. **Communication Discipline Section** (CLAUDE.md.template):
   - **Confusion Management Protocol**: STOP → Name → Ask → Wait
   - **Assumption Surfacing Format**: Explicit format for stating assumptions before work
   - **Push Back When Warranted**: Anti-sycophancy guidance
   - **Scope Discipline**: "Touch only what you're asked to touch"
   - **Simplicity Enforcement**: Pre-completion simplicity checks

2. **Failure Modes to Avoid Table** (CLAUDE.md.template):
   - 12 numbered failure modes with prevention strategies
   - Covers: assumptions, confusion, sycophancy, overcomplication, scope creep, dead code

3. **Agent Communication Standards** (agent templates):
   - Before Starting Work: Assumption surfacing format
   - When Confused: CLARIFICATION_NEEDED protocol
   - Scope Discipline: What NOT to touch
   - Simplicity Check: Pre-completion verification

4. **On Completion Report Format** (agent templates):
   - ASSUMPTIONS MADE
   - CHANGES MADE
   - INTENTIONALLY UNCHANGED
   - POTENTIAL CONCERNS
   - DEAD CODE IDENTIFIED
   - TESTS

5. **Handoff Template Updates**:
   - Goal Confirmation block for declarative goal reframing
   - Enhanced On Completion format with structured sections
   - Added CLARIFICATION_NEEDED return type

**Files Changed:**

| File | Change |
|------|--------|
| `CLAUDE.md.template` | Added Communication Discipline section, Failure Modes table |
| `handoff-full.md.template` | Added Goal Confirmation, updated On Completion format |
| `handoff-mini.md.template` | Added structured report format |
| `dev-backend.md.template` | Added Communication Standards, Scope Discipline, On Completion |
| `dev-frontend.md.template` | Added Communication Standards, Scope Discipline, On Completion |
| `dev-test.md.template` | Added Communication Standards, Scope Discipline, On Completion |
| `dev-refactor.md.template` | Added Communication Standards, Scope Discipline, On Completion |
| `dev-migration.md.template` | Added Communication Standards, Scope Discipline, On Completion |
| `TEMPLATE_VERSION` | 1.12.0 → 1.13.0 |
| `VERSION` | 2.19.0 → 2.20.0 |

**Template Version:** 1.13.0

**Inspiration:** Senior Software Engineer prompt patterns for focused, disciplined coding behavior.

### 2.19.0 (2026-02-02)

**Orchestrator Framework Clarifications - Resolving Contradictions**

Fixes 6 issues identified by Snakey project's orchestrator self-analysis that caused confusion and inconsistent behavior.

**Issues Fixed:**

| Issue | Problem | Solution |
|-------|---------|----------|
| 1 | Plan Mode vs EnterPlanMode conflation | Distinguish "planning mindset" (always) from "formal plan mode" (non-trivial) |
| 2 | Agent creation paradox | Current task uses fallback pattern; future sessions get agent files |
| 3 | Verification workflow unclear | Added ASCII workflow diagram with explicit decision points |
| 4 | Research vs Explore confusion | Added decision tree and taxonomy note |
| 5 | Recovery triggers undefined | Added failure type table with severity and actions |
| 6 | Self-reminder is manual | Added micro-checkpoint, turn counter (5 turns / 3 agents), checkpoint in verify-agent |

**Key Clarifications:**

1. **Planning Mindset vs Formal Plan Mode:**
   - Planning mindset (classify task, identify agents) - ALWAYS required
   - Formal plan mode (EnterPlanMode tool) - only for non-trivial tasks
   - Trivial = single file, exact instructions, <10 lines, no design decisions

2. **Agent Creation Paradox Resolved:**
   - Custom agents created mid-session aren't available until restart
   - For current task: MUST use fallback pattern (customize built-in agent)
   - For future sessions: SHOULD create agent file (optional)

3. **Verification Workflow:**
   - Run `/verify-agent` after EVERY single agent (not batches)
   - Clear decision tree: PASS → proceed, FAIL → classify and recover

4. **Research Task Decision Tree:**
   - Built-in Explore: quick, <5 files, no domain knowledge
   - Custom explore-*: technology-specific investigation
   - Custom debug-*: error diagnosis

5. **Failure Triggers Defined:**
   - CRITICAL: Build broken → /recover immediately
   - HIGH: Test failure, scope violation → /recover or escalate
   - MEDIUM: Agent error, timeout, partial completion → appropriate action

6. **Self-Reminder Improvements:**
   - Micro-checkpoint at start of complex responses (3 seconds)
   - Turn-based checkpoints: every 5 turns OR 3 agents
   - Checkpoint reminder added to /verify-agent output
   - Honest acknowledgment: "This is manual - focus requires discipline"

**Files Changed:**

| File | Change |
|------|--------|
| `CLAUDE.md.template` | Rewrote plan mode section, agent creation section, added verification workflow, taxonomy note, failure triggers, self-reminder improvements |
| `roster.md.template` | Added decision tree for research tasks, clarified agent creation paradox, updated anti-patterns |
| `verify-agent/SKILL.md.template` | Added post-verification checkpoint reminder |
| `TEMPLATE_VERSION` | 1.11.1 → 1.12.0 |
| `VERSION` | 2.18.1 → 2.19.0 |

**Template Version:** 1.12.0

**Issue Identified By:** Snakey project orchestrator self-analysis

### 2.18.1 (2026-02-02)

**Bugfix: Remove Contradictory Direct Work Exception**

Removes stale rule that allowed orchestrators to make "small single-file edits (< 20 lines)" directly, which contradicted the stricter v2.2.0 rule that ALL code file edits must be delegated.

**Files Changed:**

| File | Change |
|------|--------|
| `CLAUDE.md.template` | Removed "Small single-file edits (< 20 lines)" from direct work list |
| `roster.md.template` | Changed "Edit multiple files" to "Edit ANY code files" in anti-patterns |

**Template Version:** 1.11.1

**Issue Identified By:** Snakey project orchestrator self-analysis

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

### 2.17.0 (2026-02-02)

**Task-Level Orchestrator Analyzer - Cross-Session Compliance**

Adds task-level analysis that groups related sessions together for more accurate compliance evaluation.

**Problem Solved:**

Session-level analysis produces false positives when:
- Planning happens in session A, execution in session B
- User accepts plan (context clears for execution session)
- Multi-session workflows are used intentionally

**New Files:**

| File | Purpose |
|------|---------|
| `task-linker.mjs` | Groups sessions into tasks using linking signals |
| `task-rules.mjs` | Task-level compliance rules (evaluates full lifecycle) |
| `analyze-task.mjs` | CLI for task-level analysis |

**Task Linkage Signals:**

| Signal | Reliability |
|--------|-------------|
| Slug match | High |
| Plan content | Very High |
| Transcript reference | High |
| Timing proximity | Medium |

**Template Version:** N/A (scripts only)

### 2.16.0 (2026-02-02)

**Version-Aware Session Analysis - Orchestrator Version in Transcripts**

Adds orchestrator version to CLAUDE.md template and implements version-aware compliance analysis.

**Problem Solved:**

Session analysis was applying v2.15.0 rules to sessions from older orchestrator versions, producing false positives. Without version info, we couldn't determine which rules to apply.

**Key Changes:**

- **CLAUDE.md.template** - Added version header
- **TEMPLATE_VERSION file** - New single source of truth
- **manifest.json.template** - Uses `{{TEMPLATE_VERSION}}` placeholder
- **transcript-parser.mjs** - Version extraction from project manifest
- **orchestrator-rules.mjs** - Version-aware rule evaluation

**Template Version:** 1.11.0

### 2.15.0 (2026-02-02)

**Mandatory Plan Mode - Domain Agent Selection for Every Prompt**

Removes the "extremely simple" exception from plan mode. Plan mode is now **mandatory for every user prompt** to ensure proper domain agent selection.

**Why This Change:**

AI training data has a knowledge gap with current package versions. Domain agents embed version-specific patterns that the orchestrator may lack. Skipping plan mode risks using stale patterns.

**Key Changes:**

- Removed "extremely simple" exception
- Plan mode is MANDATORY for every prompt
- Expanded declaration format to include task type, technologies, agent selection
- Removed hardcoded agent names from templates (use template variables)
- Updated project-agent-generator to populate template variables

**Template Version:** N/A (behavior change)

### 2.14.0 (2026-02-01)

**Role-Specific Domain Agents with Shared Knowledge**

Refactors the domain agent system so that version-specific knowledge is stored **once** in shared knowledge files, and multiple role-specific agents @-reference that shared knowledge.

**Architecture Change:**

- **Before:** Knowledge embedded into `dev-*` agents only
- **After:** Knowledge in `knowledge/` folder, referenced by `dev-`, `explore-`, `debug-`, `audit-` agents

**The Four Agent Roles:**

| Role | Prefix | Purpose | Write Scope |
|------|--------|---------|-------------|
| Implementation | `dev-` | Build features | Code (15 max) |
| Research | `explore-` | Investigate | Reports only |
| Debugging | `debug-` | Diagnose issues | Reports only |
| Review | `audit-` | Code review | Reports only |

**Template Version:** 1.10.0

### 2.13.0 (2026-02-01)

**Domain-Specific Agents - Version-Aware Code Generation**

Creates technology-specific agents with embedded version knowledge instead of generic agents referencing shared stack.md.

**New Agent:**

- **`@project-agent-generator`** - Assembles domain agents from pre-built knowledge templates

**Template Version:** 1.9.0

### 2.12.0 (2026-02-01)

**UI Design & Responsive Layout Improvements**

Adds professional UI design and responsive layout expertise to the orchestrator framework.

**New Agents:**

| Agent | Role | Purpose |
|-------|------|---------|
| `dev-ui-designer` | Research | Visual design, design systems, responsive strategy |
| `dev-auditor-responsive` | Review | Verify responsive implementation, mobile testing |

**Template Version:** 1.8.0

### 2.11.0 (2026-02-01)

**Orchestrator Pattern Improvements - Verification, Recovery, and Checkpoints**

**New Skills:**

| Skill | Purpose |
|-------|---------|
| `/verify-agent` | Mandatory verification after every agent delegation |
| `/recover` | Structured error recovery |
| `/parallel-check` | Pre-flight safety check for parallel execution |

**Template Version:** 1.7.0

### 2.10.0 (2026-02-01)

**Orchestrator Performance Analyzer - Automated Compliance Analysis**

Adds `/analyze-orchestrator` skill that parses Claude Code session transcripts to evaluate orchestrator pattern compliance.

**New Files:**

- `.claude/scripts/transcript-parser.mjs`
- `.claude/scripts/orchestrator-rules.mjs`
- `.claude/scripts/analyze-session.mjs`
- `.claude/skills/analyze-orchestrator/SKILL.md`

### 2.9.0 (2026-01-31)

**Audit Trail System - Process Tracking for Framework Improvement**

Adds comprehensive audit trail to track activity during project creation and development.

**What Gets Tracked:**

| Component | Capture Method |
|-----------|----------------|
| Session Log | Automatic (hooks) |
| Decision Log | Manual (`/audit-decision`) |
| Audit Summary | Manual (`/audit-summary`) |

**Template Version:** 1.6.0

### 2.8.0 (2026-01-31)

**App Design Phase + Mandatory Connection Verification**

Adds App Design Phase to created projects and makes database connection verification mandatory before project handoff.

**Template Version:** 1.5.0

### 2.7.0 (2026-01-31)

**User Preferences Memory - Faster Project Creation**

Adds user preferences system that remembers choices from previous projects.

### 2.6.0 (2026-01-31)

**Context7 MCP Integration - Live Documentation for LLMs**

Adds Context7 MCP server integration to all created projects.

**Template Version:** 1.4.0

### 2.5.0 (2026-01-31)

**Auditor Agents - Comprehensive Review Perspectives**

Adds 8 specialized auditor agents for focused code and project reviews.

**Template Version:** 1.3.0

### 2.4.0 (2026-01-31)

**Always Plan Mode + Orchestrator Self-Reminder**

Inverts plan mode behavior and adds mechanisms to prevent orchestrator role drift.

### 2.3.0 (2026-01-28)

**Credentials Security - Defense in Depth**

Adds comprehensive credentials and secrets protection to all created projects.

### 2.2.0 (2026-01-26)

**Mandatory Agent Creation - Closing Delegation Loopholes**

Addresses critical issues where orchestrators performed work directly instead of delegating.

### 2.1.0 (2026-01-26)

**Migration Accuracy Improvements**

Fixes critical issues where created CLAUDE.md files contained incorrect paths and domain syntax.

### 2.0.0 (2026-01-26)

**BREAKING CHANGE: Context Management Architecture**

Major release introducing comprehensive context management system to prevent AI focus degradation.

### 1.9.0 (2026-01-26)

**Tech Revalidation for Existing Projects**

Detect and handle validation drift with `/tech-revalidate` skill.

**Template Version:** 1.2.0

### 1.8.0 (2026-01-26)

**Technology Validation Phase (Phase 2b)**

New stage in project creation workflow for deep AI knowledge validation.

### 1.7.0 (2026-01-24)

**Terminology Section and Scope of Changes Reference**

Added terminology definitions and quick reference for modifying files.

### 1.6.0 (2026-01-24)

**Agent Coordination**

Added scope limits, parallel/sequential execution guidance, and batching strategy.

### 1.5.0 (2026-01-24)

**Knowledge Capture Skill**

Added `/capture` skill for persisting important discoveries.

### 1.4.0 (2026-01-24)

**Tool and Hook Updates**

- Added 6 Task-related tools
- Added 3 new hook events
- Added prompt-based hooks
- Added 16 new settings fields
- Added new skill and subagent frontmatter fields
- Added 4 new deployment options
