# Project Builder Dependencies

This document lists the dependencies required to run the Project Builder (Claude Project Manager).

## Required Dependencies

| Dependency | Purpose | Install Command |
|------------|---------|-----------------|
| **Node.js 18+** | Hook JSON parsing, project scaffolding | `winget install OpenJS.NodeJS.LTS` (Windows) or `brew install node` (macOS) |
| **Git** | Version control, project initialization | `winget install Git.Git` (Windows) or `brew install git` (macOS) |
| **jq** | JSON parsing in hooks (optional if Node.js present) | `winget install jqlang.jq` (Windows) or `brew install jq` (macOS) |

## Optional Dependencies

| Dependency | Purpose | Install Command |
|------------|---------|-----------------|
| **GitHub CLI (gh)** | GitHub repo creation, PR management | `winget install GitHub.cli` (Windows) or `brew install gh` (macOS) |
| **Vercel CLI** | Vercel deployment (for web projects) | `npm install -g vercel` |
| **pnpm** | Alternative package manager | `npm install -g pnpm` |

## Platform-Specific Notes

### Windows

- Uses **Git Bash** for shell hooks (comes with Git)
- PowerShell available as alternative
- WSL recommended for full Linux compatibility
- **IMPORTANT**: After installing dependencies, restart your terminal/Claude Code session for PATH changes to take effect. Existing console windows won't see newly installed tools until restarted.

### macOS / Linux

- Native bash available
- Homebrew recommended for dependency management

## Verification

Run `/init_cpm` in Claude Code to verify all dependencies are installed and install any missing ones.

## For Created Projects

Created projects may have additional dependencies based on their tech stack:

| Project Type | Additional Dependencies |
|--------------|------------------------|
| Web App (Next.js) | Node.js 18+ |
| Backend API (Python) | Python 3.11+ |
| CLI Tool (Go) | Go 1.21+ |
| Desktop App (Rust) | Rust toolchain |

These are documented in each created project's own `DEPENDENCIES.md` or `README.md`.
