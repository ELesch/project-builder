# Project Brief: cc-project-manager

## Project Metadata

- **Project Name**: cc-project-manager
- **Repository**: cc-project-manager
- **Target Platform**: Windows (single .exe binary)
- **Language**: Go
- **User Technical Level**: Technical

## Overview

A command-line interface tool that manages Claude Code projects using the orchestrator pattern. The tool detects whether it's running in an initialized project builder directory or an empty directory, and provides appropriate functionality for each scenario. When uninitialized, it bootstraps the complete project builder infrastructure. When initialized, it serves as a project launcher and manager.

## Purpose

- **Problem**: Setting up the orchestrator pattern for Claude Code projects requires creating multiple directories, agent files, templates, and configuration documents. This manual process is error-prone and time-consuming. Additionally, developers working with multiple Claude Code projects need a way to navigate between them efficiently.
- **Goal**: Provide a zero-dependency Windows executable that can bootstrap project builder environments and manage multiple Claude Code projects from a central location.

## Users

- **Primary**: Developers using Claude Code for project development
- **Scale**: Individual developer tool (single-user)
- **Technical Level**: Technical users comfortable with CLI tools

## Core Features

### 1. First Launch Detection (Uninitialized Directory)

When launched in a directory without project builder infrastructure:

- Detect absence of `.claude/` directory structure or key marker files
- Prompt user: "No project builder found. Initialize this directory as a project builder? [y/N]"
- If confirmed:
  - Create complete project builder directory structure
  - Copy/generate all orchestrator template files
  - Initialize empty project registry
  - Transition immediately to "create first project" mode
- If declined:
  - Exit gracefully with instructions

**Directory structure to create:**
```
.claude/
  agents/
    project-discovery.md
    project-architect.md
    project-initializer.md
  projects/
    .gitkeep
  templates/
    orchestrator/
      [all template files from source]
  defaults/
    web-stack.md
  roster.md
CLAUDE.md
```

### 2. Normal Launch (Initialized Directory)

When launched in an initialized project builder directory:

- Load project registry
- Display interactive menu:
  ```
  cc-project-manager v1.0.0
  Project Builder: C:\path\to\builder

  Projects:
    1. nametag-generator  (modified: 2026-01-22)
    2. task-api           (modified: 2026-01-20)
    3. [Create new project]

  Select project (1-3) or 'q' to quit:
  ```
- On project selection:
  - Change to project directory
  - Launch Claude Code in that directory (spawn `claude` process)
- On "Create new project":
  - Enter project creation mode (interactive prompts)
  - Register new project in registry
  - Optionally launch Claude Code in new project

### 3. Project Registry

Persistent storage of created projects:

**Registry Location Options:**
- Primary: `~/.cc-project-manager/registry.json` (user home directory)
- Alternative: `.claude/registry.json` (project builder local)
- Support both with CLI flag to specify preference

**Registry Schema:**
```json
{
  "version": "1.0",
  "builder_path": "C:\\path\\to\\project-builder",
  "projects": [
    {
      "name": "nametag-generator",
      "path": "C:\\path\\to\\nametag-generator",
      "description": "AI-powered name tag generator",
      "created": "2026-01-22T10:30:00Z",
      "last_accessed": "2026-01-23T14:00:00Z"
    }
  ]
}
```

**Registry Operations:**
- Add project (on creation)
- Remove project (manual cleanup)
- Update last_accessed timestamp
- Validate paths exist (handle moved/deleted projects)

### 4. Project Creation Mode

Interactive project scaffolding:

- Prompt for project name
- Prompt for target directory (default: sibling to project builder)
- Prompt for brief description
- Create project directory structure using templates
- Replace template variables ({{PROJECT_NAME}}, etc.)
- Register in project registry
- Option to launch Claude Code immediately

### 5. Command-Line Interface

```
cc-project-manager [flags] [command]

Commands:
  (none)        Interactive mode (default)
  init          Initialize current directory as project builder
  list          List all registered projects
  add <path>    Add existing project to registry
  remove <name> Remove project from registry
  open <name>   Open project in Claude Code

Flags:
  -v, --version    Show version
  -h, --help       Show help
  --registry=PATH  Use specific registry file
  --builder=PATH   Specify project builder directory
```

### 6. Integration with Orchestrator Templates

The tool must understand and work with the orchestrator template system:

- Read templates from `.claude/templates/orchestrator/`
- Perform variable substitution during project creation
- Support customization based on project type (future: Go, Python, TypeScript presets)

## Tech Stack

- **Language**: Go 1.21+
- **Dependencies**: Minimal, prefer standard library
  - `encoding/json` for registry
  - `os/exec` for launching Claude Code
  - `bufio` for interactive input
  - Consider: `github.com/manifoldco/promptui` for better prompts (optional)
- **Build**: Single static binary via `go build -ldflags "-s -w"`
- **Cross-compilation**: Target `GOOS=windows GOARCH=amd64`

## Requirements

### Platform
- Windows 10/11 (64-bit)
- No prerequisites (statically linked)
- Works from any directory (uses absolute paths internally)

### Claude Code Integration
- Assumes `claude` command is available in PATH
- Launches via `os/exec.Command("claude")`
- Spawns as separate process (does not wait)

### File System
- Handle Windows path conventions (backslashes)
- Support paths with spaces (common in Windows)
- Use appropriate config directory (`%USERPROFILE%` for registry)

### Error Handling
- Graceful handling of missing directories
- Clear error messages for common issues
- Registry corruption recovery (backup before write)

## Implementation Details

### Package Structure
```
cc-project-manager/
  main.go           # Entry point, CLI parsing
  cmd/
    init.go         # Initialize project builder
    list.go         # List projects
    add.go          # Add project to registry
    remove.go       # Remove from registry
    open.go         # Open project in Claude
    interactive.go  # Interactive menu mode
  internal/
    registry/
      registry.go   # Registry CRUD operations
      schema.go     # Registry data structures
    builder/
      detect.go     # Detect initialized builder
      scaffold.go   # Create builder structure
    project/
      create.go     # Create new project
      template.go   # Template variable substitution
    config/
      paths.go      # Path utilities
```

### Key Functions

**Detection Logic:**
```go
func IsInitializedBuilder(path string) bool {
    // Check for required markers:
    // - .claude/agents/ directory exists
    // - .claude/templates/orchestrator/ exists
    // - CLAUDE.md exists
    return dirExists(filepath.Join(path, ".claude", "agents")) &&
           dirExists(filepath.Join(path, ".claude", "templates", "orchestrator")) &&
           fileExists(filepath.Join(path, "CLAUDE.md"))
}
```

**Template Substitution:**
```go
func SubstituteVariables(content string, vars map[string]string) string {
    result := content
    for key, value := range vars {
        placeholder := "{{" + key + "}}"
        result = strings.ReplaceAll(result, placeholder, value)
    }
    return result
}
```

### Embedded Assets

Consider embedding template files using Go's `embed` package:
```go
//go:embed templates/*
var templateFS embed.FS
```

This allows the .exe to be fully self-contained without requiring template files alongside it.

## Constraints

- Must compile to single .exe (no DLLs, no config files required)
- Should work offline (no network requirements)
- Registry file must be human-readable (JSON with indentation)
- Paths stored as absolute paths to avoid ambiguity

## Future Enhancements (Out of Scope for v1)

- Tech stack presets (Go, Python, TypeScript project templates)
- Project archiving/backup
- Multi-builder support (manage multiple project builders)
- Integration with Git (auto-init repos)
- Windows context menu integration
- Project statistics/usage tracking

## Open Questions

None - all requirements captured.

---

Created: 2026-01-23
Status: Ready for Architecture
