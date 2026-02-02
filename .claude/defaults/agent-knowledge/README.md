# Agent Knowledge Templates

Pre-built knowledge modules that are **@-referenced** by role-specific domain agents during project creation.

## Purpose

AI training data has uneven coverage. This system provides:

1. **Version-specific patterns** - Correct code for specific technology versions
2. **Do/Don't tables** - Prevent common mistakes
3. **Verification tasks** - Developer checks for AI-generated code
4. **Context7 instructions** - When to use live documentation

## How It Works

```
Knowledge Sources (layered):
┌─────────────────────────────────────────────────────────────┐
│ Pre-built Templates    (this directory)                     │
│   Version-specific patterns, gotchas, verification tasks    │
├─────────────────────────────────────────────────────────────┤
│ Tech-Validator Research   (project-specific findings)       │
│   Confidence levels, integration gotchas, verification      │
├─────────────────────────────────────────────────────────────┤
│ Context7 Instructions     (in agent)                        │
│   "Use context7 for [technology] when unsure"               │
└─────────────────────────────────────────────────────────────┘
                           ↓
              @project-agent-generator
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ .claude/agents/knowledge/     (deployed knowledge files)    │
│   nextjs-15.md, prisma-7.md, logging-pino.md               │
├─────────────────────────────────────────────────────────────┤
│ .claude/agents/               (role-specific agents)        │
│   dev-nextjs-15.md      → @-references knowledge/nextjs-15  │
│   explore-nextjs-15.md  → @-references knowledge/nextjs-15  │
│   debug-nextjs-15.md    → @-references knowledge/nextjs-15  │
│   audit-nextjs-15.md    → @-references knowledge/nextjs-15  │
└─────────────────────────────────────────────────────────────┘
```

## The Four Agent Roles

Each technology can have up to 4 role-specific agents, all sharing the same knowledge file:

| Role | Prefix | Purpose | Tools | Write Scope |
|------|--------|---------|-------|-------------|
| Implementation | `dev-` | Build features | Read, Grep, Edit, Write, Bash | Code (15 max) |
| Research | `explore-` | Investigate codebase | Read, Grep, Glob, WebFetch, WebSearch | Reports only |
| Debugging | `debug-` | Diagnose issues | Read, Grep, Glob, Bash | Reports only |
| Review | `audit-` | Review for patterns | Read, Grep, Glob | Reports only |

### Role Generation by Confidence

| Confidence | Roles Generated | Rationale |
|------------|-----------------|-----------|
| **High** | `dev-` only | AI confident - implementation sufficient |
| **Medium** | All 4 roles | Moderate confidence - need investigation tools |
| **Low** | All 4 roles | Low confidence - full support suite needed |
| **Unknown** | All 4 roles | Unknown - maximum flexibility |

## Directory Structure

```
agent-knowledge/
├── README.md           # This file
├── index.json          # Registry of all templates
├── frameworks/         # Web frameworks
│   ├── nextjs-14.md    # AI-confident version
│   ├── nextjs-15.md    # Post-cutoff patterns
│   ├── react-18.md
│   └── react-19.md
├── databases/          # ORMs and database tools
│   ├── prisma-5.md
│   ├── prisma-7.md     # New defineConfig pattern
│   └── drizzle-0.30.md
├── styling/            # CSS frameworks and component libraries
│   ├── tailwind-3.md
│   ├── tailwind-4.md   # CSS-first approach
│   └── shadcn-ui.md
├── languages/          # Language-specific patterns
│   ├── typescript-5.3.md
│   └── typescript-5.5.md
├── cross-cutting/      # Shared concerns
│   ├── logging-pino.md
│   ├── logging-structlog.md
│   ├── testing-vitest.md
│   └── testing-jest.md
└── integrations/       # Technology combinations
    ├── nextjs-prisma.md
    └── nextjs-supabase.md
```

## Template Format

Each knowledge template uses YAML frontmatter:

```yaml
---
technology: nextjs
version: "15"
versionRange: ">=15.0.0 <16.0.0"
aiConfidence: Medium
context7Available: true
dependencies: [react-19]
supersedes: nextjs-14
lastUpdated: 2026-02-01
---
```

### Frontmatter Fields

| Field | Type | Description |
|-------|------|-------------|
| `technology` | string | Technology identifier (lowercase) |
| `version` | string | Major version (e.g., "15", "7.x") |
| `versionRange` | string | Semver range this template covers |
| `aiConfidence` | enum | `High`, `Medium`, `Low`, `Unknown` |
| `context7Available` | boolean | Whether Context7 has docs for this |
| `dependencies` | array | Other templates that should be included |
| `supersedes` | string | Older template this replaces |
| `lastUpdated` | date | When template was last updated |

## Template Content Structure

Each template should include:

### 1. AI Training Context

```markdown
## AI Training Context

| Aspect | Status |
|--------|--------|
| **AI Trained On** | 14.x |
| **Gap Level** | Major |
| **Confidence** | Medium |
| **Context7** | Available |
```

### 2. Critical Patterns

```markdown
## Critical Patterns (Embed in Agent)

### Server Components (Default)
[Code examples and explanations]
```

### 3. Do/Don't Table

```markdown
## Do/Don't Table

| Do | Don't |
|----|-------|
| Use Server Actions for mutations | Create API routes for forms |
```

### 4. Verification Tasks

```markdown
## Verification Tasks

- [ ] Server Actions work without API routes
- [ ] params/cookies are awaited
```

### 5. Context7 Usage

```markdown
## Context7 Usage

Say: `use context7 for nextjs app router`
```

## Using Templates

### During Project Creation

1. `@project-tech-validator` identifies which templates match the stack
2. `@project-agent-generator`:
   - Deploys knowledge files to `.claude/agents/knowledge/`
   - Generates role-specific agents that @-reference knowledge files
   - Determines how many roles to generate based on confidence level
3. Generated agents @-reference knowledge files + include Context7 instructions

### Template Selection Logic

The agent generator selects templates based on:

| Project Type | Templates Selected |
|--------------|-------------------|
| web-app | framework + orm + styling + shared |
| backend-api | framework + orm + shared |
| cli-tool | language + shared |
| library | language + testing |

### Shared Templates

Cross-cutting templates (logging, testing, language) are included based on:

- Detected logging library → `logging-{library}.md`
- Detected test framework → `testing-{framework}.md`
- Primary language → `languages/{language}-{version}.md`

These are deployed as shared knowledge files but typically don't get dedicated agents (High confidence).

## Maintenance

### Adding New Templates

1. Create template file in appropriate directory
2. Add entry to `index.json` registry
3. Include all required sections
4. Set accurate `aiConfidence` level

### Updating Templates

When AI training is updated:

1. Review confidence levels
2. Update `aiConfidence` if training improved
3. Update `lastUpdated` date
4. Verify patterns against current docs

### Version Bump Guidelines

| Scenario | Create New Template? |
|----------|---------------------|
| Minor version (1.x → 1.y) | No - update existing |
| Major version with breaking changes | Yes - new template |
| Major version, compatible patterns | Update existing, bump version |

## Integration with Project Builder

The Project Builder uses these templates via:

1. **Tech Validator** - Selects matching templates, determines confidence levels
2. **Agent Generator** - Deploys knowledge files + generates role-appropriate agents
3. **Initializer/Migrator** - Creates agent files in project

Templates feed into:
- Knowledge files (`.claude/agents/knowledge/{tech}.md`)
- Role-specific domain agents (`.claude/agents/{role}-{tech}-{version}.md`)
- Tech stack documentation (`.claude/tech/stack.md`)
- Manifest tracking (`.claude/manifest.json`)

## Output Structure in Created Projects

```
.claude/agents/
├── knowledge/                   # Deployed knowledge files
│   ├── nextjs-15.md            # Next.js 15 patterns
│   ├── prisma-7.md             # Prisma 7 patterns
│   ├── tailwind-4.md           # Tailwind v4 patterns
│   ├── logging-pino.md         # Shared logging patterns
│   └── testing-vitest.md       # Shared testing patterns
├── dev-nextjs-15.md            # Implementation agent
├── explore-nextjs-15.md        # Research agent
├── debug-nextjs-15.md          # Debugging agent
├── audit-nextjs-15.md          # Review agent
├── dev-prisma-7.md
├── explore-prisma-7.md
├── debug-prisma-7.md
├── audit-prisma-7.md
└── ... (generic agents like dev-backend, dev-frontend)
```

Each role agent is small (~50-100 lines) because the knowledge is @-referenced, not embedded.
