# AI-Confident Versions

> **AI Training Cutoff**: May 2025
>
> These are versions Claude can write correct code for based on training data.
> For newer versions, consult the gotchas in `tech/stack.md`.

## Web Stack (Default)

| Technology | Confident Version | Risk Level |
|------------|-------------------|------------|
| Next.js | 14.x | Major gap if using 15+ |
| React | 18.x | Moderate gap if using 19+ |
| TypeScript | 5.3 | Minor - mostly compatible |
| Prisma | 7.x | Moderate - new config patterns (defineConfig, adapters) |
| Tailwind CSS | 3.x | Major gap if using 4+ (CSS-first) |
| shadcn/ui | 0.8.x | Moderate - components evolve |
| Zod | 3.22 | Minor - stable API |
| NextAuth.js | 4.x | Major gap if using 5+ (Auth.js) |

## .NET / Azure Stack

| Technology | Confident Version | Risk Level |
|------------|-------------------|------------|
| .NET | 8.x LTS | **Major gap if using 10+** (C# 14, new APIs) |
| C# | 12 | **Major gap if using 14+** (new language features) |
| ASP.NET Core | 8.x | **Major gap if using 10+** |
| .NET Aspire | 8.x | **Major gap if using 13+** (Aspire CLI, deploy commands) |
| Entity Framework Core | 8.x | Moderate gap if using 10+ |
| Azure SDK | 1.x | Minor - stable patterns |
| Blazor | 8.x | Moderate gap if using 10+ |

### .NET 10 / Aspire 13 Gotchas (Major Gap)

| Do | Don't |
|----|-------|
| Use `aspire deploy` and `aspire publish` CLI commands | Use `dotnet run --publisher manifest` (deprecated) |
| Use deployment state management (persists across runs) | Expect prompts every deployment |
| Use `WithDeploymentSlot()` for staging slots | Manually configure slots |
| Use Azure Developer CLI (`azd up`) for simple deploys | Over-engineer deployment pipelines |
| Use `DeployingCallbackAnnotation` for custom deploy | Expect built-in deployment annotations |
| Require .NET SDK 10.0.100+ for Aspire CLI | Use older SDK versions |
| Use Visual Studio 2026 (v18.0+) for .NET 10 | Try to target .NET 10 from VS 2022 |

### C# 14 New Features (Verify before using)

- Field keyword in properties
- Extension members (new syntax)
- First-class Span<T> support
- Null-conditional assignment
- Lambda improvements

## Other Common Technologies

| Technology | Confident Version | Risk Level |
|------------|-------------------|------------|
| Node.js | 20.x LTS | Minor |
| Go | 1.21 | Minor - stable |
| Python | 3.11 | Minor |
| FastAPI | 0.109 | Minor |
| PostgreSQL | 15 | Minor - SQL stable |

## Risk Levels Explained

| Level | Meaning | Action |
|-------|---------|--------|
| **Minor** | API compatible, new features optional | Code likely works |
| **Moderate** | Some patterns changed | Review gotchas, test carefully |
| **Major** | Breaking changes likely | Must reference gotchas, verify all code |

## Sparse Training Data Risk

**Important**: Version within training cutoff does not guarantee accurate AI knowledge.

Some technologies may have existed before cutoff but with insufficient training data:

| Scenario | Risk | Example |
|----------|------|---------|
| **Niche library** | Limited examples in training | Specialized ORMs, domain-specific tools |
| **New API patterns** | Not widely documented yet | React 18 hooks at release |
| **Enterprise tools** | Proprietary documentation | Internal frameworks |
| **Recent major release** | Limited community content | Framework released 3 months before cutoff |
| **Integration patterns** | Cross-technology usage | Framework A + Library B together |

**The `@project-tech-validator` agent addresses this by:**
1. Researching actual API behavior, not just version numbers
2. Checking community discussions for common pitfalls
3. Validating integration patterns between technologies
4. Generating test patterns developers can verify

## How This Is Used

1. **During project creation**: Tech-validator assesses AI knowledge for EVERY technology
2. **Confidence levels assigned**: High/Medium/Low/Unknown based on training data quality
3. **Validation report created**: Includes test patterns and verification tasks
4. **Initializer uses report**: Creates `tech/stack.md` with confidence-aware gotchas
5. **By agents**: Reference `tech/stack.md` before writing version-sensitive code

## Updating This File

When Claude's training is updated, this file should be revised to reflect the new baseline.
