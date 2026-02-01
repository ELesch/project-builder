# Audit Decision Skill

---
name: audit-decision
description: |
  Record a significant decision with alternatives considered and rationale.
  Use when making technology choices, architecture decisions, or trade-offs
  that future developers should understand.
disable-model-invocation: true
allowed-tools: Read, Write, Glob
---

You are recording a significant decision for the audit trail. This creates a permanent record that helps future developers understand why choices were made.

## When to Use This Skill

Record decisions when:
- Choosing between technologies (database, framework, library)
- Making architecture decisions (patterns, module boundaries)
- Resolving trade-offs (performance vs. simplicity, cost vs. features)
- Changing a previous decision (with rationale for the change)

## Process

### Step 1: Gather Decision Information

Ask the user (if not provided):

1. **What decision was made?** - Clear, one-line summary
2. **What alternatives were considered?** - At least 2 options
3. **What are the pros/cons of each?** - Objective analysis
4. **Why was this choice made?** - The decisive factors
5. **What constraints influenced the decision?** - Time, budget, skills, etc.

### Step 2: Create Decision Record

Generate a unique ID based on topic:
- Technology choice: `tech-{technology}`
- Architecture: `arch-{component}`
- Trade-off: `tradeoff-{topic}`
- Other: `decision-{number}`

Create the file at: `.claude/audit/decisions/{YYYY-MM-DD}-{id}.md`

### Step 3: Write Decision Record

Use this format:

```markdown
# Decision: {One-line summary}

**Date:** {YYYY-MM-DD}
**ID:** {id}
**Status:** Accepted
**Context:** {Project phase or what prompted this decision}

## Question

{What question or problem needed to be resolved?}

## Alternatives Considered

### Option 1: {Name}

**Description:** {Brief description}

**Pros:**
- {Pro 1}
- {Pro 2}

**Cons:**
- {Con 1}
- {Con 2}

### Option 2: {Name}

**Description:** {Brief description}

**Pros:**
- {Pro 1}
- {Pro 2}

**Cons:**
- {Con 1}
- {Con 2}

{Add more options as needed}

## Decision

**Chosen:** {Option name}

**Rationale:** {Why this option was selected. What made it the best choice given the constraints?}

## Constraints

- {Constraint 1: e.g., "Must work with existing PostgreSQL database"}
- {Constraint 2: e.g., "Team has limited React experience"}
- {Constraint 3: e.g., "Budget constraints limit hosted service options"}

## Consequences

**Expected benefits:**
- {Benefit 1}
- {Benefit 2}

**Potential risks:**
- {Risk 1}
- {Risk 2}

**Follow-up actions:**
- [ ] {Action 1}
- [ ] {Action 2}

## Related Decisions

- {Link to related decision records if any}
```

### Step 4: Confirm Creation

Report:
- File path created
- Decision summary
- Reminder about updating if decision changes

## Examples

### Example 1: Database Choice

```
/audit-decision
```

User provides: "We chose Supabase over Firebase for the database"

**Creates:** `.claude/audit/decisions/2026-01-31-tech-supabase.md`

### Example 2: Architecture Pattern

```
/audit-decision
```

User provides: "Using feature-based module structure instead of layer-based"

**Creates:** `.claude/audit/decisions/2026-01-31-arch-module-structure.md`

### Example 3: Trade-off Decision

```
/audit-decision
```

User provides: "Chose server-side rendering over static generation for the blog"

**Creates:** `.claude/audit/decisions/2026-01-31-tradeoff-ssr-vs-ssg.md`

## Guidelines

### What Makes a Good Decision Record

1. **Objective** - Present alternatives fairly, even the rejected ones
2. **Complete** - Include enough context for someone unfamiliar with the project
3. **Honest** - Acknowledge risks and unknowns
4. **Actionable** - Include follow-up actions if any

### What NOT to Record

- Trivial decisions (variable names, minor formatting)
- Obvious choices with no real alternatives
- Temporary workarounds (unless they become permanent)
- Decisions made outside the project scope

### Updating Decisions

If a decision is later reversed:
1. DO NOT delete the original record
2. Create a new decision record that references the original
3. Update the original's status to "Superseded by {new-id}"

## Output

After creating the record, confirm:

```
Decision recorded: {summary}
File: .claude/audit/decisions/{filename}

This decision record is part of the project's audit trail.
If this decision changes, create a new record referencing this one.
```
