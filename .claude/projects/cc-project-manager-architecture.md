# Architecture: cc-project-manager

## Overview

A self-contained Windows CLI tool written in Go that manages Claude Code projects using the orchestrator pattern. The tool detects its execution context (initialized vs uninitialized directory) and provides appropriate functionality: bootstrapping project builder infrastructure or serving as a project launcher/manager.

**Key Architectural Decision:** All templates are embedded directly in the executable using Go's `//go:embed` directive, making the .exe fully self-contained with zero external dependencies.

## Directory Structure

```
cc-project-manager/
├── main.go                         # Entry point, CLI parsing with flag package
├── go.mod                          # Module definition
├── go.sum                          # Dependency checksums
├── Makefile                        # Build automation
├── .goreleaser.yaml                # Release automation config
├── .github/
│   └── workflows/
│       ├── build.yml               # CI build/test
│       └── release.yml             # Automated releases
├── cmd/
│   ├── root.go                     # Root command, mode detection
│   ├── init.go                     # Initialize project builder
│   ├── list.go                     # List registered projects
│   ├── add.go                      # Add existing project to registry
│   ├── remove.go                   # Remove project from registry
│   ├── open.go                     # Open project in Claude Code
│   └── interactive.go              # Interactive menu mode (default)
├── internal/
│   ├── builder/
│   │   ├── detect.go               # Detect initialized builder directory
│   │   ├── detect_test.go
│   │   ├── scaffold.go             # Create builder directory structure
│   │   └── scaffold_test.go
│   ├── registry/
│   │   ├── registry.go             # Registry CRUD operations
│   │   ├── registry_test.go
│   │   ├── schema.go               # Registry data structures
│   │   └── backup.go               # Registry backup/recovery
│   ├── project/
│   │   ├── create.go               # Create new project from templates
│   │   ├── create_test.go
│   │   ├── template.go             # Template variable substitution
│   │   └── template_test.go
│   ├── config/
│   │   ├── paths.go                # Path utilities (Windows-aware)
│   │   ├── paths_test.go
│   │   └── version.go              # Version info (set at build time)
│   ├── ui/
│   │   ├── menu.go                 # Interactive menu rendering
│   │   ├── prompt.go               # User input prompts
│   │   └── spinner.go              # Activity indicator (optional)
│   └── claude/
│       ├── launcher.go             # Launch Claude Code process
│       └── launcher_test.go
├── templates/                      # EMBEDDED via //go:embed
│   ├── builder/                    # Project builder infrastructure templates
│   │   ├── CLAUDE.md
│   │   ├── .claude/
│   │   │   ├── agents/
│   │   │   │   ├── project-discovery.md
│   │   │   │   ├── project-architect.md
│   │   │   │   └── project-initializer.md
│   │   │   ├── projects/
│   │   │   │   └── .gitkeep
│   │   │   ├── templates/
│   │   │   │   └── orchestrator/
│   │   │   │       └── [all orchestrator template files]
│   │   │   ├── defaults/
│   │   │   │   └── web-stack.md
│   │   │   └── roster.md
│   └── orchestrator/               # Templates for new projects (copied from current project)
│       ├── CLAUDE.md.template
│       ├── CHANGELOG.md.template
│       ├── README.md.template
│       └── .claude/
│           └── [all template files from current project]
└── docs/
    ├── USAGE.md                    # User documentation
    └── DEVELOPMENT.md              # Developer documentation
```

## Agents Included

This is a Go CLI project. The following agents are recommended for development:

| Agent | Purpose | Go-Specific Customizations |
|-------|---------|---------------------------|
| `dev-backend` | Core Go logic implementation | Go idioms, error handling patterns, struct design |
| `dev-cli` | CLI-specific patterns | Cobra/flag patterns, subcommand structure, help text |
| `dev-test` | Test implementation | Go testing patterns, table-driven tests, testify assertions |
| `dev-reviewer` | Code review | Go code review checklist, idiomatic Go |
| `dev-docs` | Documentation | Go doc comments, README, usage examples |

### Custom Agent: dev-cli

A new agent specific to CLI development:

```markdown
# CLI Agent

## Role

Implement CLI commands, flags, and user interactions following Go CLI best practices.

## CRITICAL: YOU MUST ALWAYS

- Use consistent flag naming (kebab-case for long, single letter for short)
- Provide helpful --help output for all commands
- Return appropriate exit codes (0=success, 1=error, 2=usage error)
- Handle Ctrl+C gracefully
- Validate all user inputs before processing

## CRITICAL: NEVER DO THESE

- Panic on user input errors (return errors instead)
- Print to stdout for non-data output (use stderr for messages)
- Ignore Windows path conventions
- Block on user input without timeout in non-interactive mode

## Patterns

### Command Structure
- Each command in its own file under cmd/
- Commands return errors, main.go handles exit codes
- Use flag package (stdlib) for simplicity, or cobra for complex CLIs

### User Input
- Prompt clearly with defaults shown: "Project name [my-project]: "
- Validate immediately after input
- Support both interactive and flag-based input
```

## Conventions

### File Organization

- **cmd/** contains CLI command implementations (one file per command)
- **internal/** contains all internal packages (not importable externally)
- **templates/** contains embedded files only
- Package names match directory names (e.g., `internal/registry` = package `registry`)
- Test files are colocated: `foo.go` + `foo_test.go`

### Naming Conventions

- **Files**: lowercase with underscores (`project_create.go`)
- **Packages**: short, lowercase, no underscores (`registry`, `builder`)
- **Exported functions**: PascalCase (`CreateProject`)
- **Unexported functions**: camelCase (`validatePath`)
- **Constants**: PascalCase for exported, camelCase for unexported
- **Interfaces**: verb-er pattern (`Reader`, `ProjectCreator`)
- **Structs**: noun pattern (`Registry`, `Project`)

### Go Patterns

**Error Handling:**
```go
// Return errors, don't panic
func LoadRegistry(path string) (*Registry, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        if os.IsNotExist(err) {
            return &Registry{}, nil // Empty registry is valid
        }
        return nil, fmt.Errorf("reading registry: %w", err)
    }
    // ...
}
```

**Struct Construction:**
```go
// Use functional options for complex construction
type Option func(*Config)

func WithRegistryPath(path string) Option {
    return func(c *Config) {
        c.RegistryPath = path
    }
}
```

**Dependency Injection:**
```go
// Accept interfaces, return structs
type ProjectCreator interface {
    Create(name, path string) error
}

type Builder struct {
    creator ProjectCreator
    fs      afero.Fs  // For testing (optional)
}
```

## Developer Prerequisites

These tools must be installed before working on this project.

| Tool | Version | Install (Windows) | Verify |
|------|---------|-------------------|--------|
| **Go** | 1.21+ | `winget install GoLang.Go` or https://go.dev/dl/ | `go version` |
| **Git** | 2.x+ | `winget install Git.Git` or https://git-scm.com | `git --version` |
| **Make** | any | `choco install make` or use Git Bash | `make --version` |
| **Claude Code** | latest | (for testing launcher) | `claude --version` |
| **goreleaser** | latest | `go install github.com/goreleaser/goreleaser@latest` | `goreleaser --version` |

### Installation Notes (Windows)

**Option 1: winget (recommended)**
```powershell
winget install GoLang.Go
winget install Git.Git
# Make is optional - can use go build directly
```

**Option 2: Chocolatey**
```powershell
choco install golang
choco install git
choco install make
```

**Option 3: Manual**
- Go: Download MSI from https://go.dev/dl/
- Git: Download from https://git-scm.com/download/win
- Make: Included in Git Bash, or install MinGW

### Post-Installation

1. Verify Go is in PATH: `go version`
2. Set GOPATH if needed (usually auto-configured)
3. Verify `go env GOPATH` points to valid directory

## Tech Stack Details

### Go Version
- **Version**: Go 1.21+ (required for `slices`, `maps` packages and improved generics)
- **Purpose**: Modern Go with all stdlib improvements
- **Rationale**: No need for older compatibility; single-developer tool

### Standard Library (Preferred)
- **`flag`**: CLI argument parsing (simple, no dependencies)
- **`encoding/json`**: Registry serialization
- **`os/exec`**: Launching Claude Code
- **`embed`**: Template embedding
- **`text/template`**: Template processing (for complex templates)
- **`strings`**: Simple variable substitution
- **`filepath`**: Cross-platform path handling
- **`bufio`**: Interactive input reading

### Optional Dependencies (Evaluate During Development)

| Dependency | Purpose | When to Add |
|------------|---------|-------------|
| `github.com/spf13/cobra` | CLI framework | Only if flag package becomes unwieldy |
| `github.com/manifoldco/promptui` | Better prompts | If stdin prompts feel clunky |
| `github.com/charmbracelet/bubbletea` | TUI framework | Only for rich interactive UI |
| `github.com/stretchr/testify` | Test assertions | If Go's testing feels verbose |

**Recommendation**: Start with stdlib only. Add dependencies only when clear benefit emerges.

## Key Technical Decisions

### 1. Template Embedding

**Decision**: Use `//go:embed` to bundle all templates inside the executable.

**Rationale**:
- Single .exe distribution with no external files
- Templates versioned with code
- No file path resolution issues

**Implementation**:
```go
// internal/templates/embed.go
package templates

import "embed"

//go:embed builder/* orchestrator/*
var FS embed.FS

// Usage: templates.FS.ReadFile("builder/CLAUDE.md")
```

**Template Structure**:
- `templates/builder/` - Files for initializing a project builder directory
- `templates/orchestrator/` - Files for initializing a new project (with variable substitution)

### 2. Registry Location

**Decision**: Store registry at `~/.cc-project-manager/registry.json` by default.

**Rationale**:
- User-global (works across multiple builder directories)
- Standard Windows config location (`%USERPROFILE%`)
- Human-readable JSON for manual editing if needed

**Implementation**:
```go
func DefaultRegistryPath() string {
    home, _ := os.UserHomeDir()
    return filepath.Join(home, ".cc-project-manager", "registry.json")
}
```

**Registry Schema**:
```json
{
  "version": "1.0",
  "builders": [
    {
      "path": "C:\\dev\\project-builder",
      "name": "main-builder",
      "last_used": "2026-01-23T14:00:00Z"
    }
  ],
  "projects": [
    {
      "name": "nametag-generator",
      "path": "C:\\dev\\nametag-generator",
      "builder_path": "C:\\dev\\project-builder",
      "description": "AI-powered name tag generator",
      "created": "2026-01-22T10:30:00Z",
      "last_accessed": "2026-01-23T14:00:00Z"
    }
  ]
}
```

### 3. Claude Code Launching

**Decision**: Use `os/exec.Command` to spawn Claude Code as a detached process.

**Rationale**:
- Claude Code must run independently (not tied to our process)
- User can close cc-project-manager while Claude runs
- Standard Go approach for process spawning

**Implementation**:
```go
func LaunchClaude(projectPath string) error {
    cmd := exec.Command("claude")
    cmd.Dir = projectPath

    // Detach from parent process on Windows
    cmd.SysProcAttr = &syscall.SysProcAttr{
        CreationFlags: syscall.CREATE_NEW_PROCESS_GROUP,
    }

    if err := cmd.Start(); err != nil {
        return fmt.Errorf("launching claude: %w", err)
    }

    // Don't wait - let Claude run independently
    return nil
}
```

### 4. Interactive Prompts

**Decision**: Start with simple `bufio.Scanner` for prompts; add library if needed.

**Rationale**:
- Minimal dependencies
- Full control over UX
- Can upgrade to promptui/bubbletea if users want richer interaction

**Implementation**:
```go
func Prompt(question, defaultValue string) string {
    if defaultValue != "" {
        fmt.Printf("%s [%s]: ", question, defaultValue)
    } else {
        fmt.Printf("%s: ", question)
    }

    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    input := strings.TrimSpace(scanner.Text())

    if input == "" {
        return defaultValue
    }
    return input
}
```

### 5. Builder Detection

**Decision**: Check for specific marker files to determine if directory is initialized.

**Markers checked**:
- `.claude/agents/` directory exists
- `.claude/templates/orchestrator/` directory exists
- `CLAUDE.md` file exists

**Implementation**:
```go
func IsInitializedBuilder(path string) bool {
    checks := []string{
        filepath.Join(path, ".claude", "agents"),
        filepath.Join(path, ".claude", "templates", "orchestrator"),
        filepath.Join(path, "CLAUDE.md"),
    }

    for _, check := range checks {
        if _, err := os.Stat(check); os.IsNotExist(err) {
            return false
        }
    }
    return true
}
```

### 6. Template Variable Substitution

**Decision**: Use simple `{{VARIABLE}}` syntax with `strings.ReplaceAll`.

**Rationale**:
- Consistent with existing orchestrator templates
- Simple to implement and understand
- No complex template logic needed

**Variables supported**:
- `{{PROJECT_NAME}}` - Project name
- `{{PROJECT_DESCRIPTION}}` - Brief description
- `{{DATE}}` - Creation date
- `{{LANGUAGE}}` - Primary language (Go, TypeScript, etc.)
- `{{FRAMEWORK}}` - Framework if any
- `{{DATABASE}}` - Database if any
- `{{EXT}}` - File extension (.go, .ts, etc.)
- `{{CONVENTIONS}}` - Project-specific conventions
- `{{COMMANDS}}` - Development commands
- `{{KEY_FILES}}` - Important file locations

## Build & Distribution

### Build Process

**Makefile**:
```makefile
VERSION ?= $(shell git describe --tags --always --dirty)
LDFLAGS := -s -w -X main.version=$(VERSION)

.PHONY: build
build:
	go build -ldflags "$(LDFLAGS)" -o bin/cc-project-manager.exe .

.PHONY: test
test:
	go test -v ./...

.PHONY: lint
lint:
	golangci-lint run

.PHONY: release
release:
	goreleaser release --clean
```

**Build Command**:
```bash
# Development build
go build -o cc-project-manager.exe .

# Production build (smaller binary)
go build -ldflags "-s -w" -o cc-project-manager.exe .
```

### goreleaser Configuration

```yaml
# .goreleaser.yaml
version: 2

builds:
  - env:
      - CGO_ENABLED=0
    goos:
      - windows
    goarch:
      - amd64
      - arm64
    ldflags:
      - -s -w -X main.version={{.Version}}
    binary: cc-project-manager

archives:
  - formats: ['zip']
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"

checksum:
  name_template: 'checksums.txt'

changelog:
  sort: asc
  filters:
    exclude:
      - '^docs:'
      - '^test:'
```

### GitHub Actions

**Build Workflow** (`.github/workflows/build.yml`):
```yaml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      - run: go test -v ./...
      - run: go build -v .
```

**Release Workflow** (`.github/workflows/release.yml`):
```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      - uses: goreleaser/goreleaser-action@v5
        with:
          version: latest
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Quality Standards

### Testing Strategy

- **Coverage Target**: 80% for internal packages
- **Test Style**: Table-driven tests (Go idiom)
- **Mocking**: Use interfaces for external dependencies (filesystem, exec)
- **Integration Tests**: Test full command flows with temp directories

**Example Table-Driven Test**:
```go
func TestIsInitializedBuilder(t *testing.T) {
    tests := []struct {
        name     string
        setup    func(t *testing.T, dir string)
        expected bool
    }{
        {
            name:     "empty directory",
            setup:    func(t *testing.T, dir string) {},
            expected: false,
        },
        {
            name: "full structure",
            setup: func(t *testing.T, dir string) {
                os.MkdirAll(filepath.Join(dir, ".claude", "agents"), 0755)
                os.MkdirAll(filepath.Join(dir, ".claude", "templates", "orchestrator"), 0755)
                os.WriteFile(filepath.Join(dir, "CLAUDE.md"), []byte("test"), 0644)
            },
            expected: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            dir := t.TempDir()
            tt.setup(t, dir)
            got := IsInitializedBuilder(dir)
            if got != tt.expected {
                t.Errorf("got %v, want %v", got, tt.expected)
            }
        })
    }
}
```

### Code Review Checklist

- [ ] Error handling: All errors wrapped with context
- [ ] No panics in library code
- [ ] Windows paths handled correctly (use `filepath`, not `path`)
- [ ] Resources cleaned up (defer for file handles)
- [ ] Exported functions have doc comments
- [ ] Table-driven tests for functions with multiple cases
- [ ] No hardcoded paths (use `os.UserHomeDir()`, etc.)

### Documentation Requirements

- **README.md**: Installation, quick start, usage examples
- **Go doc comments**: All exported types and functions
- **Usage.md**: Detailed command reference
- **CHANGELOG.md**: Version history following Keep a Changelog format

## Customization Specifications

Instructions for the initializer agent when creating this project:

### Files to Create

1. **Go module initialization**:
   - `go.mod` with module `github.com/user/cc-project-manager`
   - Go version 1.21

2. **Main entry point** (`main.go`):
   - Version variable (set via ldflags)
   - Parse flags and delegate to cmd package
   - Exit code handling

3. **Command files** (`cmd/*.go`):
   - `root.go` - Root command, auto-detection logic
   - `init.go` - `init` subcommand
   - `list.go` - `list` subcommand
   - `add.go` - `add` subcommand
   - `remove.go` - `remove` subcommand
   - `open.go` - `open` subcommand
   - `interactive.go` - Interactive menu (default mode)

4. **Internal packages**:
   - Stub implementations with interfaces defined
   - Test files with at least one test per package

5. **Embedded templates** (`templates/`):
   - Copy all files from current project's `.claude/templates/orchestrator/`
   - Create builder templates (CLAUDE.md, agents, etc.)

6. **Build files**:
   - `Makefile`
   - `.goreleaser.yaml`
   - `.github/workflows/build.yml`
   - `.github/workflows/release.yml`

7. **Project orchestrator files** (`.claude/`):
   - Custom CLAUDE.md for Go CLI development
   - Go-specific agents (dev-backend, dev-cli, dev-test, dev-reviewer)
   - roster.md with Go agent guidance
   - tech/stack.md with Go version info

### Template Variables for This Project

When creating CLAUDE.md and agent files:

- `{{PROJECT_NAME}}` = `cc-project-manager`
- `{{PROJECT_DESCRIPTION}}` = `CLI tool for managing Claude Code projects`
- `{{LANGUAGE}}` = `Go`
- `{{FRAMEWORK}}` = `stdlib (flag, embed, os/exec)`
- `{{DATABASE}}` = `JSON file (registry)`
- `{{EXT}}` = `go`

### Special Considerations

1. **Windows-first**: All path handling must use `filepath` package
2. **Self-referential**: This tool creates project builders that contain templates for creating projects
3. **Template nesting**: The embedded templates include templates (meta!)
4. **Test isolation**: Tests must use `t.TempDir()` for filesystem operations

---

Based on: cc-project-manager-brief.md
Created: 2026-01-23
Status: Pending Approval
