# Project Builder System

A Claude Code system for creating new projects with the orchestrator pattern.

## Purpose

This system guides users through describing a new project and then initializes a properly structured project directory with a customized orchestrator pattern.

## Quick Start

1. Start a Claude Code session in this directory
2. Describe your project idea
3. Answer discovery questions to refine requirements
4. The system will create a new project directory with:
   - Customized `CLAUDE.md` with project-specific conventions
   - Specialized agents for your tech stack
   - Planning templates
   - Documentation structure

## How It Works

### Phase 1: Discovery
The system asks questions to understand:
- Project purpose and goals
- Target users and use cases
- Tech stack preferences
- Team size and experience
- Integration requirements
- Quality and compliance needs

### Phase 2: Architecture
Based on discovery, the system designs:
- Directory structure
- Module organization
- Agent specializations
- Development workflow

### Phase 3: Initialization
The system creates:
- New project directory
- Customized orchestrator files
- Initial documentation
- Development guidelines

## Directory Structure

```
project/
├── README.md                    # This file
├── CLAUDE.md                    # Project builder instructions
├── .claude/
│   ├── agents/                  # Builder agents
│   │   ├── project-discovery.md
│   │   ├── project-architect.md
│   │   └── project-initializer.md
│   ├── templates/
│   │   ├── discovery/           # Discovery question templates
│   │   └── orchestrator/        # Generic orchestrator template
│   ├── projects/                # Completed project configs
│   └── roster.md                # Agent selection guide
└── docs/
    └── README.md
```

## Origin

Built on patterns from the orchestrator system, which was validated on the Bond Titan project.
