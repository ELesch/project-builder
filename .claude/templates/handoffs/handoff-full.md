# Handoff: {{FROM_PHASE}} -> {{TO_PHASE}}

> **ORCHESTRATOR CHECKPOINT**: Before delegating this handoff:
> - [ ] I am delegating to an AGENT (not doing this myself)
> - [ ] Work is scoped to MAX 20 files
> - [ ] I will REVIEW the result, not continue the work myself

> **Max 100 lines** - This is the ONLY context the next agent receives.
> Drop everything not essential for the next phase.

## Task (5 lines max)
**Goal:** {{What needs to be accomplished}}

**Constraints:** {{Key limitations}}

**Success Criteria:** {{How to know it's done}}

## Files to Modify (REQUIRED)

| File | Action | Reason |
|------|--------|--------|
| {{path/to/file.ts}} | Create | {{Why creating this file}} |
| {{path/to/existing.ts}} | Modify | {{What change needed}} |
| {{path/to/remove.ts}} | Delete | {{Why removing}} |

## Reference Files (vital context for next agent)

| File | Why Needed |
|------|------------|
| {{path/to/pattern.ts}} | Pattern for {{what}} |
| {{path/to/types.ts}} | Type definitions needed |

## Critical Context (10 lines max)

**Tech decisions:**
- {{Decision 1 that affects implementation}}
- {{Decision 2}}

**Patterns to follow:**
- {{Pattern from reference file 1}}
- {{Pattern from reference file 2}}

**Test patterns:** (if TDD)
- Tests location: {{path to similar tests}}
- Test framework: {{jest/pytest/go test/etc}}

## Inputs

| Key | Value |
|-----|-------|
| {{input_name}} | {{value}} |
| {{another_input}} | {{value}} |

## Anti-Context (DROP these - explored but rejected)

- {{Thing researched but not relevant}}
- {{Approach considered but rejected - why}}
- {{Files explored but not needed}}

## Verification Anchor

**To verify you have enough context, answer:**
1. {{Question the agent should be able to answer}}
2. {{Another verification question}}

**If you can't answer, request mini-research for:** {{specific gap}}
