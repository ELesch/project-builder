# Audit Summary Skill

---
name: audit-summary
description: |
  Analyze session logs and generate a summary report.
  Use for retrospective analysis, debugging patterns, and framework improvement.
disable-model-invocation: true
allowed-tools: Read, Glob, Write
---

You are analyzing audit trail data to generate insights about project activity. This helps identify patterns, diagnose issues, and improve agent performance.

## When to Use This Skill

- **Retrospectives** - Analyze what happened during a project phase
- **Debugging** - Trace what led to a failure or issue
- **Improvement** - Identify patterns for better agent prompts
- **Reporting** - Generate summaries for stakeholders

## Process

### Step 1: Gather Audit Data

Use Glob to find available logs:

```
.claude/audit/sessions/*.md
.claude/audit/decisions/*.md
```

Ask user about scope:
- **All time** - Analyze all available logs
- **Recent** - Last 7 days (default)
- **Specific date range** - User provides dates
- **Single session** - Specific date

### Step 2: Read and Parse Logs

For each session log in scope:
1. Read the file
2. Extract structured entries:
   - Agent delegations (name, task, duration)
   - Agent completions (success/failure)
   - Tool failures (tool, error type)
   - Session boundaries

For decision records:
1. Read each file
2. Extract: date, id, decision, alternatives count

### Step 3: Analyze Patterns

Calculate metrics:

**Agent Usage:**
- Total delegations by agent type
- Success/failure ratio per agent
- Average session count per day

**Failure Analysis:**
- Most common failure types
- Tools with highest failure rates
- Correlation with specific agents

**Activity Patterns:**
- Busiest periods (days/times)
- Average session duration
- Phase transition frequency

**Decision Trends:**
- Decisions per category (tech, arch, tradeoff)
- Most active decision periods

### Step 4: Generate Summary Report

Create report at: `.claude/audit/summary-{YYYY-MM-DD}.md`

Use this format:

```markdown
# Audit Summary Report

**Generated:** {timestamp}
**Period:** {start-date} to {end-date}
**Sessions Analyzed:** {count}
**Decisions Recorded:** {count}

## Executive Summary

{2-3 sentence overview of findings}

## Agent Activity

| Agent | Delegations | Completions | Failures | Success Rate |
|-------|-------------|-------------|----------|--------------|
| {agent} | {n} | {n} | {n} | {%} |

### Most Active Agents

1. **{agent}** - {n} delegations ({description of typical tasks})
2. **{agent}** - {n} delegations ({description of typical tasks})

### Agent Observations

- {Observation 1: e.g., "dev-backend has 15% failure rate, mostly due to TypeScript errors"}
- {Observation 2: e.g., "dev-test is often run multiple times in succession"}

## Failure Analysis

| Error Type | Count | Primary Source |
|------------|-------|----------------|
| {type} | {n} | {tool or agent} |

### Common Failure Patterns

1. **{Pattern}** ({n} occurrences)
   - Typical cause: {description}
   - Typical resolution: {description}

### Recommendations

- {Recommendation 1}
- {Recommendation 2}

## Activity Timeline

| Date | Sessions | Agents Used | Failures |
|------|----------|-------------|----------|
| {date} | {n} | {list} | {n} |

### Activity Patterns

- Peak activity: {day/time pattern}
- Average session: {duration or work volume}

## Decision Records

| Category | Count | Examples |
|----------|-------|----------|
| tech | {n} | {example decisions} |
| arch | {n} | {example decisions} |
| tradeoff | {n} | {example decisions} |

### Recent Decisions

1. **{date}**: {decision summary}
2. **{date}**: {decision summary}

## Insights for Improvement

### Agent Prompt Improvements

Based on failure patterns, consider:
- {Suggestion 1}
- {Suggestion 2}

### Process Improvements

Based on activity patterns:
- {Suggestion 1}
- {Suggestion 2}

### Areas Needing Attention

- {Area 1: e.g., "High failure rate in database migrations"}
- {Area 2: e.g., "Many tool retries in frontend development"}

## Appendix: Raw Statistics

{Detailed numbers if needed}
```

### Step 5: Report Location

After generating, report:
- Summary file location
- Key highlights (3-5 bullet points)
- Suggested follow-up actions

## Output Modes

### Default: Full Summary

Complete analysis as described above.

### Quick Mode

When user says "quick" or "brief":
- Skip detailed tables
- Focus on top 3 patterns
- Keep under 50 lines

### Failure Focus

When user mentions "debug" or "failures":
- Focus primarily on failure analysis
- Include error message excerpts
- Suggest specific fixes

### Agent Focus

When user mentions specific agent:
- Filter to that agent's activity
- Show all delegations and outcomes
- Compare to overall averages

## Examples

### Example 1: Weekly Retrospective

```
/audit-summary
```

"Analyze the last 7 days"

**Generates:** Summary with agent usage, failures, and recommendations.

### Example 2: Debug Session

```
/audit-summary
```

"Why did the build keep failing yesterday?"

**Generates:** Failure-focused report for that date.

### Example 3: Agent Performance

```
/audit-summary
```

"How has dev-backend been performing?"

**Generates:** Agent-specific analysis with comparison to baseline.

## Guidelines

### Objectivity

- Report facts, not judgments
- Include both successes and failures
- Don't hide uncomfortable patterns

### Actionability

- Every insight should suggest an action
- Prioritize by impact
- Be specific about improvements

### Privacy

- Don't include sensitive error content
- Summarize rather than quote extensively
- Focus on patterns, not individual incidents

## Error Handling

If no audit data exists:
- Report "No audit data found for the specified period"
- Suggest running with audit hooks enabled
- Check `.claude/settings.json` for hook configuration

If data is incomplete:
- Note which files could not be parsed
- Report on available data
- Suggest checking hook configuration
