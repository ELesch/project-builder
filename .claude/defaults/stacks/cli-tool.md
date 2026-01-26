# CLI Tool Stack

Command-line utilities and developer tools. Use this for applications run from the terminal.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

### Go (Default for CLIs)

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Go | 1.21 | Minor |
| Cobra | 1.8 | Minor |
| Viper | 1.18 | Minor |

### Rust

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Rust | 1.75 | Minor |
| clap | 4.x | Minor |
| tokio | 1.x | Minor |

### Node.js

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Node.js | 20.x LTS | Minor |
| Commander | 12.x | Minor |
| TypeScript | 5.3 | Minor |

**Recommendation**: Go is recommended for CLIs due to single-binary distribution and fast startup. Rust for performance-critical tools. Node.js for tools targeting JS developers.

## Default Stack by Language

### Go (Recommended Default)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Go 1.21+ | Fast compilation, single binary |
| **CLI Framework** | Cobra | Command/subcommand structure |
| **Configuration** | Viper | Config files, env vars, flags |
| **Output** | lipgloss/bubbletea | Terminal UI and styling |
| **Testing** | testing (stdlib) | Built-in testing |
| **Distribution** | goreleaser | Cross-platform releases |
| **Source Control** | GitHub | Repository hosting |

### Rust

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Rust | Performance, safety |
| **CLI Framework** | clap | Argument parsing |
| **Configuration** | config-rs | Configuration management |
| **Output** | colored / indicatif | Terminal colors and progress |
| **Async** | tokio | Async runtime (if needed) |
| **Testing** | cargo test | Built-in testing |
| **Distribution** | cargo-dist | Cross-platform releases |
| **Source Control** | GitHub | Repository hosting |

### Node.js

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Runtime** | Node.js 20 LTS | JavaScript runtime |
| **Language** | TypeScript | Type safety |
| **CLI Framework** | Commander | Command parsing |
| **Configuration** | dotenv / cosmiconfig | Config management |
| **Output** | chalk / ora | Colors and spinners |
| **Bundling** | esbuild / pkg | Single-file distribution |
| **Testing** | Jest | Unit testing |
| **Source Control** | GitHub | Repository hosting |

## Testing Stack

### Go
| Tool | Purpose |
|------|---------|
| testing (stdlib) | Unit testing |
| testify | Assertions and mocks |

### Rust
| Tool | Purpose |
|------|---------|
| cargo test | Built-in testing |
| assert_cmd | CLI integration testing |

### Node.js
| Tool | Purpose |
|------|---------|
| Jest | Unit testing |
| execa | CLI integration testing |

## Logging Stack

| Language | Logger | Notes |
|----------|--------|-------|
| Go | zerolog | JSON logging to stderr |
| Rust | tracing | Structured logging |
| Node.js | Pino | JSON logging |

Note: CLIs typically log to stderr and output data to stdout. Error tracking (Sentry) is less common for CLIs but can be added for distributed tools.

## When to Choose Each Language

| Requirement | Recommended |
|-------------|-------------|
| Fast startup, easy distribution | Go |
| Maximum performance | Rust |
| JS/TS ecosystem integration | Node.js |
| System-level operations | Go or Rust |
| Heavy string processing | Rust or Go |
| Quick prototyping | Node.js |
| Cross-platform GUI elements | Go (with Fyne) or Rust (with egui) |

## Distribution Options

| Method | Best For | Notes |
|--------|----------|-------|
| GitHub Releases | Default | goreleaser/cargo-dist auto-publish |
| Homebrew | macOS users | Tap for easy install |
| npm | Node.js CLIs | `npm install -g` |
| crates.io | Rust tools | `cargo install` |
| go install | Go tools | Direct from source |
| Docker | Complex deps | Container distribution |

## Project Structure

### Go
```
{project}/
├── cmd/
│   └── {name}/
│       └── main.go
├── internal/
│   ├── commands/
│   └── config/
├── pkg/           # Public libraries (if any)
├── go.mod
└── README.md
```

### Rust
```
{project}/
├── src/
│   ├── main.rs
│   ├── cli.rs
│   └── commands/
├── tests/
├── Cargo.toml
└── README.md
```

### Node.js
```
{project}/
├── src/
│   ├── index.ts
│   ├── cli.ts
│   └── commands/
├── bin/
│   └── {name}
├── package.json
├── tsconfig.json
└── README.md
```

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll build this as a command-line tool using Go - it creates a simple, fast program that runs on any computer without needing other software installed."

## For Technical Users

Ask about preferences:

> "For CLI tools, I recommend Go (easy distribution), Rust (max performance), or Node.js (JS ecosystem). What's your preference?"

Then dive into:
- Distribution method? (GitHub Releases, Homebrew, npm, etc.)
- Configuration needs? (flags only, config files, env vars)
- Interactive features? (prompts, progress bars, TUI)
