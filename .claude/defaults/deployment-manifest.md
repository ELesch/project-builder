# Deployment Manifest

> **Authoritative list of all orchestrator template files.**
> Referenced by `@project-initializer` and `@project-migrator` as the single source of truth
> for what must be deployed to every created/migrated project.
>
> **Last updated:** 2026-02-16 | **Template count:** 100 files

## How to Use This Manifest

- **Initializer/Migrator:** Walk each batch in order. Deploy every file where the condition is met.
- **After deployment:** Verify all `always` files exist in the created project.
- **When adding templates:** Add an entry here FIRST, then create the template file.
- **Batch size rule:** No batch exceeds 15 deployed files (orchestrator batches agent runs accordingly).

## Condition Reference

| Condition | Meaning | Evaluated From |
|-----------|---------|----------------|
| `always` | Deploy for every project | N/A |
| `new-project-only` | Deploy only for new projects (not migrations) | Creation method |
| `migration-only` | Deploy only for migrations | Creation method |
| `has-database` | Project uses a database | Architecture / analysis |
| `has-web-ui` | Project has a web frontend | Project type (web-app, desktop-app) |
| `has-api` | Project exposes an API | Architecture / analysis |
| `ops-model:developer` | Ops model is "developer" | Discovery |
| `ops-model:ops-team` | Ops model is "ops-team" | Discovery |
| `security-level:confidential+` | Security level is confidential or regulated | Discovery |
| `has-mobile` | Project includes mobile app | Discovery (mobileApps != none) |

---

## Batch 1: Core Structure (always)

> Foundation files every project needs. Deploy first.

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 1 | `CLAUDE.md.template` | `CLAUDE.md` | always |
| 2 | `.claude/manifest.json.template` | `.claude/manifest.json` | always |
| 3 | `.claude/settings.json.template` | `.claude/settings.json` | always |
| 4 | `.claude/roster.md.template` | `.claude/roster.md` | always |
| 5 | `.claude/practices.md.template` | `.claude/practices.md` | always |
| 6 | `CHANGELOG.md.template` | `CHANGELOG.md` | always |
| 7 | `README.md.template` | `README.md` | new-project-only |
| 8 | `ONBOARDING.md.template` | `ONBOARDING.md` | always |
| 9 | `.mcp.json.template` | `.mcp.json` | always |
| 10 | `.claudeignore.template` | `.claudeignore` | always |

**Files in batch:** 10 (9 always + 1 conditional)

---

## Batch 2: Skills (always)

> All skills are framework infrastructure. Deploy every one regardless of project type.

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 10 | `.claude/skills/capture/SKILL.md.template` | `.claude/skills/capture/SKILL.md` | always |
| 11 | `.claude/skills/tech-revalidate/SKILL.md.template` | `.claude/skills/tech-revalidate/SKILL.md` | always |
| 12 | `.claude/skills/commit/SKILL.md.template` | `.claude/skills/commit/SKILL.md` | always |
| 13 | `.claude/skills/app-design/SKILL.md.template` | `.claude/skills/app-design/SKILL.md` | always |
| 14 | `.claude/skills/audit-decision/SKILL.md.template` | `.claude/skills/audit-decision/SKILL.md` | always |
| 15 | `.claude/skills/audit-summary/SKILL.md.template` | `.claude/skills/audit-summary/SKILL.md` | always |
| 16 | `.claude/skills/orc-checkpoint/SKILL.md.template` | `.claude/skills/orc-checkpoint/SKILL.md` | always |
| 17 | `.claude/skills/orc-recover/SKILL.md.template` | `.claude/skills/orc-recover/SKILL.md` | always |
| 18 | `.claude/skills/orc-framework/SKILL.md.template` | `.claude/skills/orc-framework/SKILL.md` | always |
| 19 | `.claude/skills/orc-verify/SKILL.md.template` | `.claude/skills/orc-verify/SKILL.md` | always |
| 20 | `.claude/skills/orc-parallel/SKILL.md.template` | `.claude/skills/orc-parallel/SKILL.md` | always |

**Files in batch:** 11 (all always)

---

## Batch 3: Agents — Core (always) + Conditional

> Core development agents deployed to every project, plus conditional agents based on project type.

### 3a: Always-Deploy Agents

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 21 | `.claude/agents/README.md.template` | `.claude/agents/README.md` | always |
| 22 | `.claude/agents/dev-architect.md.template` | `.claude/agents/dev-architect.md` | always |
| 23 | `.claude/agents/dev-reviewer.md.template` | `.claude/agents/dev-reviewer.md` | always |
| 24 | `.claude/agents/dev-test.md.template` | `.claude/agents/dev-test.md` | always |
| 25 | `.claude/agents/dev-refactor.md.template` | `.claude/agents/dev-refactor.md` | always |
| 26 | `.claude/agents/dev-docs.md.template` | `.claude/agents/dev-docs.md` | always |
| 27 | `.claude/agents/dev-analyst.md.template` | `.claude/agents/dev-analyst.md` | always |
| 28 | `.claude/agents/dev-security.md.template` | `.claude/agents/dev-security.md` | always |
| 29 | `.claude/agents/dev-deploy.md.template` | `.claude/agents/dev-deploy.md` | always |
| 30 | `.claude/agents/dev-integration.md.template` | `.claude/agents/dev-integration.md` | always |

### 3b: Conditional Agents

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 31 | `.claude/agents/dev-backend.md.template` | `.claude/agents/dev-backend.md` | has-api |
| 32 | `.claude/agents/dev-frontend.md.template` | `.claude/agents/dev-frontend.md` | has-web-ui |
| 33 | `.claude/agents/dev-api.md.template` | `.claude/agents/dev-api.md` | has-api |
| 34 | `.claude/agents/dev-migration.md.template` | `.claude/agents/dev-migration.md` | has-database |
| 35 | `.claude/agents/dev-database.md.template` | `.claude/agents/dev-database.md` | has-database |
| 36 | `.claude/agents/dev-data.md.template` | `.claude/agents/dev-data.md` | has-database |
| 37 | `.claude/agents/dev-ops.md.template` | `.claude/agents/dev-ops.md` | ops-model:ops-team |
| 38 | `.claude/agents/dev-designer.md.template` | `.claude/agents/dev-designer.md` | has-web-ui |
| 39 | `.claude/agents/dev-ui-designer.md.template` | `.claude/agents/dev-ui-designer.md` | has-web-ui |

### 3c: Agent Templates (always)

> Template agents that get customized per project (explore, debug, audit-tech).

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 40 | `.claude/agents/explore-TEMPLATE.md.template` | `.claude/agents/explore-TEMPLATE.md` | always |
| 41 | `.claude/agents/debug-TEMPLATE.md.template` | `.claude/agents/debug-TEMPLATE.md` | always |
| 42 | `.claude/agents/audit-tech-TEMPLATE.md.template` | `.claude/agents/audit-tech-TEMPLATE.md` | always |

**Files in batch:** 22 total (13 always + 9 conditional) — **split across 2 agent runs**

---

## Batch 4: Auditor Agents (conditional)

> Specialized auditor agents based on project type.

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 43 | `.claude/agents/dev-auditor-performance.md.template` | `.claude/agents/dev-auditor-performance.md` | always |
| 44 | `.claude/agents/dev-auditor-architecture.md.template` | `.claude/agents/dev-auditor-architecture.md` | always |
| 45 | `.claude/agents/dev-auditor-testing.md.template` | `.claude/agents/dev-auditor-testing.md` | always |
| 46 | `.claude/agents/dev-auditor-dependencies.md.template` | `.claude/agents/dev-auditor-dependencies.md` | always |
| 47 | `.claude/agents/dev-auditor-docs.md.template` | `.claude/agents/dev-auditor-docs.md` | always |
| 48 | `.claude/agents/dev-auditor-errors.md.template` | `.claude/agents/dev-auditor-errors.md` | always |
| 49 | `.claude/agents/dev-auditor-accessibility.md.template` | `.claude/agents/dev-auditor-accessibility.md` | has-web-ui |
| 50 | `.claude/agents/dev-auditor-responsive.md.template` | `.claude/agents/dev-auditor-responsive.md` | has-web-ui |
| 51 | `.claude/agents/dev-auditor-api.md.template` | `.claude/agents/dev-auditor-api.md` | has-api |

**Files in batch:** 9 (6 always + 3 conditional)

---

## Batch 5: Checklists + Runbooks

> Operational infrastructure. Checklists are always deployed; runbooks are conditional on ops model.

### 5a: Checklists

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 52 | `.claude/checklists/security-review.md.template` | `.claude/checklists/security-review.md` | always |
| 53 | `.claude/checklists/deployment.md.template` | `.claude/checklists/deployment.md` | always |
| 54 | `.claude/checklists/dependency-review.md.template` | `.claude/checklists/dependency-review.md` | always |
| 55 | `.claude/checklists/architecture-review.md.template` | `.claude/checklists/architecture-review.md` | always |
| 56 | `.claude/checklists/performance-review.md.template` | `.claude/checklists/performance-review.md` | always |
| 57 | `.claude/checklists/accessibility-review.md.template` | `.claude/checklists/accessibility-review.md` | has-web-ui |
| 58 | `.claude/checklists/api-review.md.template` | `.claude/checklists/api-review.md` | has-api |
| 59 | `.claude/checklists/responsive-review.md.template` | `.claude/checklists/responsive-review.md` | has-web-ui |

### 5b: Runbooks

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 60 | `.claude/runbooks/README.md.template` | `.claude/runbooks/README.md` | always |
| 61 | `.claude/runbooks/rollback.md.template` | `.claude/runbooks/rollback.md` | always |
| 62 | `.claude/runbooks/deployment.md.template` | `.claude/runbooks/deployment.md` | always |
| 63 | `.claude/runbooks/agent-failure.md.template` | `.claude/runbooks/agent-failure.md` | always |
| 64 | `.claude/runbooks/incident-response.md.template` | `.claude/runbooks/incident-response.md` | ops-model:ops-team |
| 65 | `.claude/runbooks/on-call.md.template` | `.claude/runbooks/on-call.md` | ops-model:ops-team |
| 66 | `.claude/runbooks/database-recovery.md.template` | `.claude/runbooks/database-recovery.md` | has-database |

**Files in batch:** 15 (10 always + 5 conditional)

---

## Batch 6: Handoffs, Templates, Hooks, Audit

> Framework operational infrastructure. All are `always` — these support the orchestrator pattern itself.

### 6a: Handoffs

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 67 | `.claude/handoffs/README.md.template` | `.claude/handoffs/README.md` | always |
| 68 | `.claude/handoffs/research-to-coding.md.template` | `.claude/handoffs/research-to-coding.md` | always |
| 69 | `.claude/handoffs/mini-research.md.template` | `.claude/handoffs/mini-research.md` | always |

### 6b: Templates (plan/results/design)

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 70 | `.claude/templates/README.md.template` | `.claude/templates/README.md` | always |
| 71 | `.claude/templates/plan.md.template` | `.claude/templates/plan.md` | always |
| 72 | `.claude/templates/results.md.template` | `.claude/templates/results.md` | always |
| 73 | `.claude/templates/design-system.md.template` | `.claude/templates/design-system.md` | always |
| 74 | `.claude/templates/handoff-mini.md.template` | `.claude/templates/handoff-mini.md` | always |
| 75 | `.claude/templates/handoff-full.md.template` | `.claude/templates/handoff-full.md` | always |

### 6c: Hooks

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 76 | `.claude/hooks/audit-hooks.sh.template` | `.claude/hooks/audit-hooks.sh` | always |
| 77 | `.claude/hooks/check-secrets.sh.template` | `.claude/hooks/check-secrets.sh` | always |

### 6d: Audit

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 78 | `.claude/audit/README.md.template` | `.claude/audit/README.md` | always |

### 6e: Gitkeep Directories

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 79 | `.claude/plans/.gitkeep` | `.claude/plans/.gitkeep` | always |
| 80 | `.claude/results/.gitkeep` | `.claude/results/.gitkeep` | always |

**Files in batch:** 14 (all always)

---

## Batch 7: Status Files + Tech + Documentation Scaffolding

> Tracking files, tech reference, and documentation directory structure.

### 7a: Status & Tracking Files

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 81 | `.claude/PROJECT_STATUS.md.template` | `.claude/PROJECT_STATUS.md` | always |
| 82 | `.claude/BLOCKERS.md.template` | `.claude/BLOCKERS.md` | always |
| 83 | `.claude/LEARNINGS.md.template` | `.claude/LEARNINGS.md` | always |
| 84 | `.claude/PROCESS_LOG.md.template` | `.claude/PROCESS_LOG.md` | always |
| 85 | `.claude/REQUIREMENTS.md.template` | `.claude/REQUIREMENTS.md` | always |
| 86 | `.claude/TECH_DEBT.md.template` | `.claude/TECH_DEBT.md` | always |
| 87 | `.claude/SECURITY.md.template` | `.claude/SECURITY.md` | always |

### 7b: Tech Reference

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 88 | `.claude/tech/stack.md.template` | `.claude/tech/stack.md` | always |
| 89 | `.claude/tech/dependencies.md.template` | `.claude/tech/dependencies.md` | always |

### 7c: Documentation Directories

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 90 | `docs/DECISIONS/README.md.template` | `docs/DECISIONS/README.md` | always |
| 91 | `docs/DESIGNS/README.md.template` | `docs/DESIGNS/README.md` | always |
| 92 | `docs/RFCS/README.md.template` | `docs/RFCS/README.md` | always |

**Files in batch:** 12 (all always)

---

## Batch 8: Source Scaffolding (new-project-only)

> Source code scaffolding. Only deployed for new projects — migrations already have source code.

| # | Template Path | Deployed Path | Condition |
|---|--------------|---------------|-----------|
| 93 | `.env.example.template` | `.env.example` | new-project-only |
| 94 | `.gitignore.template` | `.gitignore` | new-project-only |
| 95 | `prisma.config.ts.template` | `prisma.config.ts` | new-project-only AND has-database |
| 96 | `prisma/schema.prisma.template` | `prisma/schema.prisma` | new-project-only AND has-database |
| 97 | `src/lib/db/client.ts.template` | `src/lib/db/client.ts` | new-project-only AND has-database |

**Files in batch:** 5 (all new-project-only)

---

## Non-Deployed Files

> These files exist in the template directory but are NOT deployed to projects.

| File | Purpose |
|------|---------|
| `README.md` | Template directory documentation (for Project Builder maintainers) |
| `TEMPLATE_VERSION` | Template version tracking (consumed by initializer, not deployed) |

---

## Deployment Summary

| Category | Always | Conditional | Total |
|----------|--------|-------------|-------|
| Core Structure | 9 | 1 | 10 |
| Skills | 11 | 0 | 11 |
| Agents (core) | 13 | 0 | 13 |
| Agents (conditional) | 0 | 9 | 9 |
| Auditor Agents | 6 | 3 | 9 |
| Checklists | 5 | 3 | 8 |
| Runbooks | 4 | 3 | 7 |
| Handoffs | 3 | 0 | 3 |
| Templates (plan/results) | 6 | 0 | 6 |
| Hooks | 2 | 0 | 2 |
| Audit | 1 | 0 | 1 |
| Gitkeep dirs | 2 | 0 | 2 |
| Status & tracking | 7 | 0 | 7 |
| Tech reference | 2 | 0 | 2 |
| Documentation dirs | 3 | 0 | 3 |
| Source scaffolding | 0 | 5 | 5 |
| **Totals** | **74** | **24** | **98** |

> **Note:** 98 deployed files + 2 non-deployed (README.md, TEMPLATE_VERSION) = 100 template files total.
> Domain-specific agents created by `@project-agent-generator` are additional and not tracked here.

## Post-Deployment Verification

After deploying all batches, verify:

### Minimum Required (always files)

- [ ] `CLAUDE.md` exists and has project-specific content
- [ ] `.claudeignore` exists and excludes audit sessions
- [ ] `.claude/manifest.json` exists and is valid JSON
- [ ] `.claude/settings.json` exists
- [ ] `.claude/roster.md` exists
- [ ] `.claude/practices.md` exists
- [ ] `CHANGELOG.md` exists
- [ ] `ONBOARDING.md` exists
- [ ] All 11 skills directories exist with `SKILL.md`
- [ ] `.claude/agents/` has at least 13 core agent files + README
- [ ] `.claude/checklists/` has at least 5 files
- [ ] `.claude/runbooks/` has at least 4 files (README + rollback + deployment + agent-failure)
- [ ] `.claude/handoffs/` has 3 files
- [ ] `.claude/templates/` has 6 files
- [ ] `.claude/hooks/` has 2 files
- [ ] `.claude/audit/README.md` exists
- [ ] `.claude/plans/` and `.claude/results/` directories exist
- [ ] All 7 status/tracking files exist in `.claude/`
- [ ] `.claude/tech/stack.md` and `.claude/tech/dependencies.md` exist
- [ ] `docs/DECISIONS/`, `docs/DESIGNS/`, `docs/RFCS/` directories exist with READMEs

### Conditional Verification

- [ ] If `has-database`: database agents, migration agent, database-recovery runbook exist
- [ ] If `has-web-ui`: frontend agent, designer agents, accessibility/responsive checklists exist
- [ ] If `has-api`: backend agent, api-review checklist, api auditor exist
- [ ] If `ops-model:ops-team`: incident-response and on-call runbooks exist
- [ ] If `new-project-only`: .env.example, .gitignore, and source scaffolding exist
