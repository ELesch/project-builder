# CPM Update Command

Update the Claude Project Manager (Project Builder) when new Claude Code versions are released.

## Your Task

You are performing an update check for the Project Builder system. Follow these steps exactly:

---

## Step 1: Read Current Baseline

Read the current Claude Code baseline that the Project Builder is built against:

**File**: `.claude/defaults/claude-code-baseline.md`

Note the **Baseline Date** and key capabilities documented there.

---

## Step 2: Check for Claude Code Changes

Use the `claude-code-guide` agent to research current Claude Code capabilities:

```
Research the following about Claude Code's CURRENT state:
1. Any new tools or capabilities added recently
2. Any changes to skills, hooks, or MCP configuration
3. Any new subagent types or changes to existing ones
4. Any changes to settings.json schema
5. Any breaking changes or deprecations
6. Current recommended patterns for CLAUDE.md, skills, and agents

Compare against the baseline date noted above and identify what has changed.
```

---

## Step 3: Analyze the Differences

Compare the research results against the baseline document.

Create a summary with two sections:

### Changes Detected

List each change found:
- **Feature**: What changed
- **Type**: New / Modified / Deprecated / Breaking
- **Impact**: How it affects the Project Builder or created orchestrators

### No Changes

If nothing significant has changed since the baseline date, report:

> ✓ Project Builder is up to date with Claude Code capabilities.
>
> Baseline date: [date]
> No updates required.

**Stop here if no changes are detected.**

---

## Step 4: Determine Recommended Updates

For each change detected, determine what needs updating in the Project Builder:

| Change | Files Affected | Update Type |
|--------|---------------|-------------|
| [change] | [files] | [add/modify/remove] |

**Potentially affected files:**
- `.claude/defaults/claude-code-baseline.md` - The baseline itself
- `.claude/defaults/ai-known-versions.md` - If version tracking changed
- `.claude/templates/orchestrator/` - If orchestrator patterns changed
- `.claude/agents/*.md` - If agent patterns changed
- `CLAUDE.md` - If instructions reference outdated patterns
- `.claude/roster.md` - If agent capabilities changed

---

## Step 5: Present Findings to User

Present the findings clearly:

```
## Claude Code Update Report

**Baseline Date**: [date from baseline]
**Check Date**: [today]

### Changes Detected

[List each change with impact assessment]

### Recommended Updates

[List each file that needs updating and what changes are needed]

### Update Plan

1. [First update]
2. [Second update]
...

Would you like me to proceed with these updates?
```

---

## Step 6: Wait for User Approval

Use `AskUserQuestion` to get explicit approval:

- **Option 1**: "Yes, apply all updates"
- **Option 2**: "Show me the changes first"
- **Option 3**: "No, skip this update"

---

## Step 7: Apply Updates (If Approved)

If the user approves:

1. **Update the baseline document** with new/changed capabilities
2. **Update affected files** as identified in Step 4
3. **Update VERSION** if this is a significant change:
   - Patch (1.3.0 → 1.3.1): Minor documentation updates
   - Minor (1.3.0 → 1.4.0): New features incorporated
   - Major (1.3.0 → 2.0.0): Breaking changes to orchestrator pattern
4. **Report completion** with summary of changes made

---

## Step 8: Final Report

Provide a summary:

```
## Update Complete

**Previous Version**: [old version]
**New Version**: [new version]

### Changes Applied

- [List of files updated and what changed]

### What This Means

[Brief explanation of how this affects new projects created by the Project Builder]

### Next Steps

[Any manual steps the user should take, or "None required"]
```

---

## Important Notes

- **Never skip the user approval step** - updates modify the Project Builder
- **Be conservative** - only recommend updates for genuine changes
- **Document everything** - the baseline should reflect exactly what we know
- **Test mentally** - consider how changes affect orchestrator frameworks we create

---

## If Research Fails

If the `claude-code-guide` agent cannot determine current Claude Code state:

1. Report the limitation to the user
2. Suggest manual review of Claude Code documentation
3. Provide the documentation URL: https://docs.anthropic.com/en/docs/claude-code
4. Do not make assumptions about changes
