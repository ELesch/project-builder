# Desktop Application Stack

Native or cross-platform applications installed on computers. Use this for apps that run on Windows, macOS, and/or Linux desktops.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

### Electron

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Electron | 28.x | Moderate if 30+ |
| React | 18.x | Moderate if 19+ |
| TypeScript | 5.3 | Minor |

### Tauri

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Tauri | 1.5 | Major if 2.0+ |
| Rust | 1.75 | Minor |
| React/Vue/Svelte | Various | Minor |

**Recommendation**: Electron for maximum compatibility and JS ecosystem. Tauri for smaller bundles and better performance. Native frameworks for platform-specific apps.

## Default Stack by Framework

### Electron (Default for cross-platform)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Electron | Cross-platform desktop |
| **Language** | TypeScript | Type safety |
| **UI Framework** | React | Component-based UI |
| **Styling** | Tailwind CSS | Utility-first CSS |
| **State** | Zustand | Simple state management |
| **Build** | electron-builder | Packaging and distribution |
| **Updates** | electron-updater | Auto-updates |
| **Testing** | Playwright | E2E testing |
| **Source Control** | GitHub | Repository hosting |

### Tauri (Lightweight alternative)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Tauri | Rust-based desktop |
| **Backend** | Rust | System operations |
| **Language** | TypeScript | Frontend type safety |
| **UI Framework** | React / Vue / Svelte | Component-based UI |
| **Styling** | Tailwind CSS | Utility-first CSS |
| **Build** | Tauri bundler | Native packaging |
| **Updates** | Tauri updater | Auto-updates |
| **Source Control** | GitHub | Repository hosting |

### Native Options

| Platform | Framework | Language |
|----------|-----------|----------|
| Windows | WinUI 3 / WPF | C# |
| macOS | SwiftUI / AppKit | Swift |
| Linux | GTK / Qt | C++, Rust, Python |
| Cross-platform | Qt | C++, Python (PyQt) |

## Testing Stack

### Electron
| Tool | Purpose |
|------|---------|
| Playwright | E2E testing |
| Vitest | Unit testing |

### Tauri
| Tool | Purpose |
|------|---------|
| Playwright | E2E testing |
| cargo test | Rust backend testing |
| Vitest | Frontend testing |

## Logging Stack

| Framework | Logger | Notes |
|-----------|--------|-------|
| Electron | electron-log | File + console logging |
| Tauri | tracing (Rust) | Structured logging |

Error tracking (Sentry) works with both Electron and Tauri for crash reporting.

## When to Choose Each Framework

| Requirement | Recommended |
|-------------|-------------|
| Max compatibility | Electron |
| Small bundle size | Tauri |
| Web tech familiarity | Electron |
| Rust ecosystem | Tauri |
| System-level access | Tauri (Rust) or native |
| macOS-only | SwiftUI |
| Windows-only | WinUI 3 |
| Performance critical | Native or Tauri |

## Project Structure

### Electron
```
{project}/
├── src/
│   ├── main/           # Main process
│   │   └── main.ts
│   ├── preload/        # Preload scripts
│   │   └── preload.ts
│   └── renderer/       # React app
│       ├── App.tsx
│       └── index.tsx
├── resources/          # App icons, assets
├── electron-builder.yml
├── package.json
└── README.md
```

### Tauri
```
{project}/
├── src/                # Frontend
│   ├── App.tsx
│   └── main.tsx
├── src-tauri/          # Rust backend
│   ├── src/
│   │   └── main.rs
│   ├── Cargo.toml
│   └── tauri.conf.json
├── package.json
└── README.md
```

## Distribution Options

| Method | Platforms | Notes |
|--------|-----------|-------|
| GitHub Releases | All | Direct download |
| Microsoft Store | Windows | Official store |
| Mac App Store | macOS | Official store |
| Homebrew Cask | macOS | Developer-friendly |
| Snapcraft | Linux | Snap packages |
| Flatpak | Linux | Sandboxed apps |
| Auto-updater | All | In-app updates |

## Security Considerations

| Concern | Electron | Tauri |
|---------|----------|-------|
| Context isolation | Required | Built-in |
| Node integration | Disable in renderer | N/A (Rust backend) |
| CSP | Configure strictly | Configure strictly |
| Code signing | Required for distribution | Required for distribution |

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll build this as a desktop app using Electron - it works on Windows, Mac, and Linux, and provides a native app experience. You'll be able to install it just like any other program."

## For Technical Users

Ask about preferences:

> "For desktop apps, I recommend Electron (web tech, max compatibility) or Tauri (smaller, faster, Rust backend). What's your preference?"

Then dive into:
- Target platforms? (Windows, macOS, Linux, all)
- Distribution method? (direct download, app stores)
- Auto-updates needed?
- System integration needs? (file system, notifications, tray)
