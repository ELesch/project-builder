# Package Orchestrator Skill

Compiles the entire Project Builder + Orchestrator Framework into a single text file for expert review, auditing, or sharing.

## When to Use

- Before sending the framework for external review
- When documenting the current state of the framework
- After major version updates
- When comparing versions

## Process

### Step 1: Read Version

Read `.claude/VERSION` to get the current version number.

### Step 2: Compile Package

Run the following Python script from the project root to compile all files:

```python
import os
import glob as g
from datetime import datetime

BASE = os.getcwd()
version_file = os.path.join(BASE, '.claude', 'VERSION')
with open(version_file, 'r') as f:
    version = f.read().strip()

OUT = os.path.join(BASE, f'orchestrator-package-v{version}.txt')

sections = [
    ('SECTION 1: PROJECT BUILDER CORE', [
        'CLAUDE.md', 'DEPENDENCIES.md', 'package.json',
        '.claude/VERSION', '.claude/roster.md',
    ]),
    ('SECTION 2: PROJECT BUILDER AGENTS', ['.claude/agents/**/*']),
    ('SECTION 3: PROJECT BUILDER SKILLS', ['.claude/skills/**/*']),
    ('SECTION 4: DEFAULTS - Root Configuration', [
        '.claude/defaults/ai-known-versions.md',
        '.claude/defaults/claude-code-baseline.md',
        '.claude/defaults/claude-code-changelog.md',
        '.claude/defaults/deployment-manifest.md',
        '.claude/defaults/dependency-policy.md',
        '.claude/defaults/logging-baseline.md',
        '.claude/defaults/operational-baseline.md',
        '.claude/defaults/security-baseline.md',
        '.claude/defaults/web-stack.md',
    ]),
    ('SECTION 5: DEFAULTS - Stack Definitions', ['.claude/defaults/stacks/**/*']),
    ('SECTION 6: DEFAULTS - Agent Knowledge Templates', ['.claude/defaults/agent-knowledge/**/*']),
    ('SECTION 7: DEFAULTS - Provider Guides', ['.claude/defaults/providers/**/*']),
    ('SECTION 8: ORCHESTRATOR TEMPLATE - Project Root Files', [
        '.claude/templates/orchestrator/CLAUDE.md.template',
        '.claude/templates/orchestrator/README.md',
        '.claude/templates/orchestrator/README.md.template',
        '.claude/templates/orchestrator/TEMPLATE_VERSION',
        '.claude/templates/orchestrator/CHANGELOG.md.template',
        '.claude/templates/orchestrator/ONBOARDING.md.template',
        '.claude/templates/orchestrator/SECURITY.md.template',
        '.claude/templates/orchestrator/.env.example.template',
        '.claude/templates/orchestrator/.gitignore.template',
        '.claude/templates/orchestrator/.mcp.json.template',
    ]),
    ('SECTION 9: ORCHESTRATOR TEMPLATE - .claude Core', [
        '.claude/templates/orchestrator/.claude/manifest.json.template',
        '.claude/templates/orchestrator/.claude/settings.json.template',
        '.claude/templates/orchestrator/.claude/roster.md.template',
        '.claude/templates/orchestrator/.claude/practices.md.template',
        '.claude/templates/orchestrator/.claude/BLOCKERS.md.template',
        '.claude/templates/orchestrator/.claude/LEARNINGS.md.template',
        '.claude/templates/orchestrator/.claude/PROJECT_STATUS.md.template',
        '.claude/templates/orchestrator/.claude/PROCESS_LOG.md.template',
        '.claude/templates/orchestrator/.claude/REQUIREMENTS.md.template',
        '.claude/templates/orchestrator/.claude/TECH_DEBT.md.template',
    ]),
    ('SECTION 10: ORCHESTRATOR TEMPLATE - Agents', [
        '.claude/templates/orchestrator/.claude/agents/**/*',
    ]),
    ('SECTION 11: ORCHESTRATOR TEMPLATE - Skills', [
        '.claude/templates/orchestrator/.claude/skills/**/*',
    ]),
    ('SECTION 12: ORCHESTRATOR TEMPLATE - Checklists', [
        '.claude/templates/orchestrator/.claude/checklists/**/*',
    ]),
    ('SECTION 13: ORCHESTRATOR TEMPLATE - Runbooks', [
        '.claude/templates/orchestrator/.claude/runbooks/**/*',
    ]),
    ('SECTION 14: ORCHESTRATOR TEMPLATE - Handoffs & Templates', [
        '.claude/templates/orchestrator/.claude/handoffs/**/*',
        '.claude/templates/orchestrator/.claude/templates/**/*',
    ]),
    ('SECTION 15: ORCHESTRATOR TEMPLATE - Tech, Hooks, Audit, Docs', [
        '.claude/templates/orchestrator/.claude/tech/**/*',
        '.claude/templates/orchestrator/.claude/hooks/**/*',
        '.claude/templates/orchestrator/.claude/audit/**/*',
        '.claude/templates/orchestrator/docs/**/*',
    ]),
    ('SECTION 16: ORCHESTRATOR TEMPLATE - Source Scaffolding', [
        '.claude/templates/orchestrator/prisma/**/*',
        '.claude/templates/orchestrator/prisma.config.ts.template',
        '.claude/templates/orchestrator/src/**/*',
    ]),
]

def resolve_files(patterns):
    files = []
    for pat in patterns:
        full = os.path.join(BASE, pat)
        if '*' in pat:
            for m in sorted(g.glob(full, recursive=True)):
                if os.path.isfile(m):
                    files.append(m)
        else:
            if os.path.isfile(full):
                files.append(full)
    return files

toc_lines = []
all_content = []
total_files = 0

for title, patterns in sections:
    files = resolve_files(patterns)
    total_files += len(files)
    toc_lines.append(f'  {title} ({len(files)} files)')
    parts = [f'\n{"=" * 80}\n{title}\n{"=" * 80}\n']
    for fpath in files:
        rel = os.path.relpath(fpath, BASE).replace(os.sep, '/')
        parts.append(f'--- FILE: {rel} ---')
        try:
            with open(fpath, 'r', encoding='utf-8', errors='replace') as f:
                parts.append(f.read())
        except Exception as e:
            parts.append(f'[ERROR: {e}]')
        parts.append(f'--- END: {rel} ---\n')
    all_content.append('\n'.join(parts))

header = f'''{"#" * 80}
#
# ORCHESTRATOR FRAMEWORK PACKAGE
# Version: {version}
# Generated: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}
# Total Files: {total_files}
#
# This file contains the complete Project Builder + Orchestrator Framework.
# It is intended for expert review of the system's design, structure,
# templates, agents, skills, and defaults.
#
{"#" * 80}

TABLE OF CONTENTS
{"=" * 80}

{chr(10).join(toc_lines)}

{"=" * 80}
'''

with open(OUT, 'w', encoding='utf-8') as f:
    f.write(header)
    f.write('\n'.join(all_content))

size_kb = os.path.getsize(OUT) / 1024
print(f'Package created: orchestrator-package-v{version}.txt')
print(f'Total files: {total_files} | Size: {size_kb:.1f} KB')
```

### Step 3: Report Results

Report the output file path, total files included, and file size to the user.

## Output

| Output | Location |
|--------|----------|
| Package file | `orchestrator-package-v{VERSION}.txt` in project root |

## Notes

- The script uses Python (required dependency) to bypass tool output limits
- Files are organized into 16 sections matching the framework architecture
- Each file has clear `--- FILE:` / `--- END:` delimiters for parsing
- Binary files and `.gitkeep` files are included but may show as empty
- The package excludes user-local files (`.local.md`, `.env`) and session data
