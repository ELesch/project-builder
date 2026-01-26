# Library/Package Stack

Reusable code modules published for other developers to use. Use this for npm packages, Python libraries, Go modules, Rust crates, etc.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

### Node.js / TypeScript

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Node.js | 20.x LTS | Minor |
| TypeScript | 5.3 | Minor |
| tsup | 8.x | Minor |
| Vitest | 1.x | Minor |

### Python

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Python | 3.11 | Minor |
| setuptools / poetry | Latest | Minor |
| pytest | 8.x | Minor |

### Go

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Go | 1.21 | Minor |

### Rust

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Rust | 1.75 | Minor |

**Recommendation**: Match the language/ecosystem your target users work in. TypeScript for JS devs, Python for data/ML, Go for infrastructure, Rust for systems.

## Default Stack by Language

### Node.js / TypeScript (npm package)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | TypeScript | Type safety, DX |
| **Build** | tsup | Fast bundling for libs |
| **Testing** | Vitest | Fast unit testing |
| **Linting** | ESLint + Prettier | Code quality |
| **Docs** | TypeDoc | API documentation |
| **Publishing** | npm | Package registry |
| **CI/CD** | GitHub Actions | Automated releases |
| **Source Control** | GitHub | Repository hosting |

### Python (PyPI package)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Python 3.11+ | Runtime |
| **Build** | Poetry / Hatch | Modern packaging |
| **Testing** | pytest | Unit testing |
| **Type Hints** | mypy | Static type checking |
| **Linting** | ruff | Fast linting |
| **Docs** | Sphinx / mkdocs | Documentation |
| **Publishing** | PyPI | Package registry |
| **CI/CD** | GitHub Actions | Automated releases |
| **Source Control** | GitHub | Repository hosting |

### Go (Go module)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Go 1.21+ | Runtime |
| **Testing** | testing (stdlib) | Built-in testing |
| **Linting** | golangci-lint | Linting suite |
| **Docs** | pkg.go.dev | Auto-generated |
| **Publishing** | Go modules | Direct from GitHub |
| **CI/CD** | GitHub Actions | Testing, tagging |
| **Source Control** | GitHub | Repository hosting |

### Rust (crates.io crate)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Rust | Runtime |
| **Testing** | cargo test | Built-in testing |
| **Linting** | clippy | Rust linter |
| **Docs** | rustdoc | Documentation |
| **Publishing** | crates.io | Package registry |
| **CI/CD** | GitHub Actions | Testing, publishing |
| **Source Control** | GitHub | Repository hosting |

## Testing Stack

### Node.js
| Tool | Purpose |
|------|---------|
| Vitest | Fast unit testing |
| c8 / istanbul | Coverage |

### Python
| Tool | Purpose |
|------|---------|
| pytest | Unit testing |
| coverage.py | Coverage |

### Go
| Tool | Purpose |
|------|---------|
| testing (stdlib) | Unit testing |
| go test -cover | Coverage |

### Rust
| Tool | Purpose |
|------|---------|
| cargo test | Unit testing |
| cargo-tarpaulin | Coverage |

## Project Structure

### Node.js / TypeScript
```
{package}/
├── src/
│   ├── index.ts      # Main entry point
│   └── lib/          # Implementation
├── tests/
├── package.json
├── tsconfig.json
├── tsup.config.ts
└── README.md
```

### Python
```
{package}/
├── src/
│   └── {package_name}/
│       ├── __init__.py
│       └── ...
├── tests/
├── pyproject.toml
└── README.md
```

### Go
```
{package}/
├── {package}.go      # Main package file
├── {package}_test.go
├── go.mod
└── README.md
```

### Rust
```
{package}/
├── src/
│   └── lib.rs
├── tests/
├── Cargo.toml
└── README.md
```

## Publishing Setup

### npm (Node.js)
```json
{
  "name": "@scope/package-name",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "files": ["dist"],
  "publishConfig": { "access": "public" }
}
```

### PyPI (Python)
```toml
[tool.poetry]
name = "package-name"
version = "1.0.0"
packages = [{include = "package_name", from = "src"}]
```

### Go modules
```
module github.com/username/package

go 1.21
```

### crates.io (Rust)
```toml
[package]
name = "package-name"
version = "1.0.0"
edition = "2021"

[lib]
name = "package_name"
```

## Documentation Best Practices

| Requirement | Node.js | Python | Go | Rust |
|-------------|---------|--------|-----|------|
| API docs | TypeDoc | Sphinx | pkg.go.dev | rustdoc |
| Examples | /examples | /examples | /examples | /examples |
| README | Essential | Essential | Essential | Essential |
| Changelog | CHANGELOG.md | CHANGELOG.md | CHANGELOG.md | CHANGELOG.md |

## For Non-Technical Users

Libraries are typically built by technical users. If a non-technical user requests this:

> "A library is code that other developers use in their projects. This is usually for technical users. Could you tell me more about what problem you're trying to solve? There might be a simpler approach."

## For Technical Users

Ask about preferences:

> "What language ecosystem is this library for? (npm, PyPI, Go modules, crates.io)"

Then dive into:
- Target audience?
- API style preferences? (functional, OOP, builder pattern)
- Minimum supported versions?
- Monorepo or single package?
