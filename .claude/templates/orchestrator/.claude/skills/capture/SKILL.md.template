# Knowledge Capture Skill

---
name: capture
description: |
  Capture important knowledge for future reference. Invoke when discovering:
  - Tech patterns or gotchas (version-specific behaviors)
  - User preferences (code style, architecture choices)
  - Best practices (patterns that worked well)
  - Specifications (business rules, requirements)

  Agents should invoke this when they discover something important to remember.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Glob
---

You are capturing knowledge that should persist for future sessions. This knowledge will be saved to project files so it's available in future conversations.

## Input

The capture request includes:
- **Knowledge**: What was learned (required)
- **Category** (optional): tech | preference | practice | spec
- **Target** (optional): @-mentioned file or directory context

## Process

### Step 1: Categorize the Knowledge

If category not provided, determine from content:

| Content Type | Category |
|--------------|----------|
| Version-specific behavior, API changes, framework gotchas | `tech` |
| User's stated preferences, coding style, architecture choices | `preference` |
| Patterns that worked well, solutions to problems | `practice` |
| Business rules, feature requirements, specifications | `spec` |

### Step 2: Determine Target File

**Priority order:**

1. **Explicit @-mention**: If a file was @-mentioned in the capture request, use that file

2. **Directory context**: If knowledge relates to a specific directory:
   - Check if `{directory}/CLAUDE.md` exists using Glob
   - If exists → target that file
   - If not → consider creating it (only for significant, recurring patterns)

3. **Category-based fallback**:

| Category | Target File | Section |
|----------|-------------|---------|
| `tech` | `.claude/tech/stack.md` | "Version Gotchas" or "Discovered Patterns" |
| `preference` | `CLAUDE.md` (root) | "Project-Specific Conventions" |
| `practice` | `.claude/LEARNINGS.md` | "Effective Patterns" or "Solutions Found" |
| `spec` | `.claude/REQUIREMENTS.md` | Create if doesn't exist |

### Step 3: Read and Analyze Target File

1. **Read the target file** completely
2. **Analyze the content** to find:
   - Does similar knowledge already exist? → Skip (report "already captured")
   - Is there a related section? → Plan to edit/enhance that section
   - Does it contradict existing content? → Plan to update (replace outdated)
   - Is it a completely new topic? → Plan to add in logical location

### Step 4: Edit Intelligently

**CRITICAL: Never just append. Always edit surgically.**

**If updating existing section:**
- Use Edit tool to modify the specific relevant portion
- Merge new knowledge with existing content
- Enhance, don't duplicate

**If adding new content to existing section:**
- Find the end of the relevant section
- Add as a new bullet point or subsection
- Maintain consistent formatting

**If creating new section:**
- Find the logical location (after related sections)
- Add with clear heading
- Use consistent formatting with rest of file

**If creating new subdirectory CLAUDE.md:**
```markdown
# {Directory Name} Conventions

<!-- Auto-maintained by /capture skill -->

## Patterns

{captured knowledge}
```

### Step 5: Report What Was Done

Output a brief summary:
- What was captured (one line)
- Where it was saved (file path + section name)
- How it was saved (updated existing / added to section / created new section)

## Knowledge Format Guidelines

### For Tech Gotchas

Format as a do/don't when possible:
```markdown
### {Technology} {Version}

| Do | Don't |
|----|-------|
| {correct pattern} | {outdated pattern} |
```

### For User Preferences

Format as clear directives:
```markdown
- **{Topic}**: {Preference statement}
```

### For Best Practices

Format with context:
```markdown
### {Pattern Name}
- **Context**: When this applies
- **Pattern**: What to do
- **Reason**: Why this works
```

### For Specifications

Format as testable requirements:
```markdown
- **{Feature}**: {Requirement statement}
  - Acceptance: {How to verify}
```

## Examples

### Example 1: Tech Gotcha Discovery

**Input**: "Prisma 7 requires prisma.config.ts with defineConfig(), not environment variables in schema.prisma"

**Process**:
1. Category: `tech` (version-specific pattern)
2. Target: `.claude/tech/stack.md`
3. Read file, find Prisma section in Version Gotchas
4. Edit: Add/update the configuration pattern in do/don't format
5. Report: "Updated Prisma gotchas in .claude/tech/stack.md"

### Example 2: User Preference

**Input**: "User prefers named exports over default exports"

**Process**:
1. Category: `preference` (coding style)
2. Target: `CLAUDE.md` (root)
3. Read file, find "Project-Specific Conventions" section
4. Edit: Add bullet under exports convention
5. Report: "Added export preference to CLAUDE.md conventions"

### Example 3: Codebase Pattern Discovery

**Input**: "This codebase uses barrel exports (index.ts) in each feature folder" (discovered while working in src/features/)

**Process**:
1. Category: `practice` (codebase pattern)
2. Check: Does `src/features/CLAUDE.md` exist?
3. If no, check if pattern is significant enough to create file
4. If significant → Create `src/features/CLAUDE.md` with pattern
5. If not significant → Add to `.claude/LEARNINGS.md` under Effective Patterns
6. Report: "Created src/features/CLAUDE.md with barrel export pattern"

### Example 4: Specification Captured

**Input**: "Authentication must support both email/password and OAuth (Google, GitHub)"

**Process**:
1. Category: `spec` (requirement)
2. Target: `.claude/REQUIREMENTS.md` (create if needed)
3. Add to Authentication section
4. Report: "Added auth requirement to .claude/REQUIREMENTS.md"

### Example 5: Agent Self-Capture

**Context**: dev-frontend agent discovers Next.js 15 requires 'use client' directive

**Agent invokes**: `/capture`
```
Knowledge: Next.js 15 App Router requires 'use client' directive at top of any component using React hooks (useState, useEffect, etc.)
Category: tech
```

**Process**:
1. Category: `tech`
2. Target: `.claude/tech/stack.md`
3. Read file, find Next.js section
4. Edit: Add to Next.js gotchas if not already present
5. Report: "Updated Next.js section in .claude/tech/stack.md"

## Critical Rules

1. **NEVER duplicate** - If knowledge already exists, enhance it or skip
2. **NEVER append blindly** - Always read first, find the right location, edit surgically
3. **Prefer updates over new sections** - If a related topic exists, add to it
4. **Be concise** - Capture actionable knowledge, not verbose explanations
5. **Include context** - Note when the knowledge applies
6. **Maintain formatting** - Match the existing style of the target file

## When NOT to Capture

- Temporary workarounds (unless they become permanent)
- One-off debugging information
- Information already in official documentation (just reference it)
- Speculative or unverified patterns

## File Size Awareness

If a file is getting large (>300 lines):
- Consider splitting knowledge to subdirectory CLAUDE.md files
- For tech/stack.md, consider creating tech/{technology}.md files
- Report the size concern in output
