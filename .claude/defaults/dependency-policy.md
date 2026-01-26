# Dependency Policy

> Reference for dependency decisions during discovery and initialization.

## License Compatibility

| License | Commercial OK | Copyleft | Notes |
|---------|---------------|----------|-------|
| MIT | Yes | No | Preferred |
| Apache-2.0 | Yes | No | Good |
| ISC | Yes | No | Good |
| BSD-3 | Yes | No | Good |
| LGPL-2.1 | Yes | Weak | Link only |
| GPL-3.0 | Caution | Strong | Viral |
| AGPL-3.0 | No | Strong | Avoid |

## Health Criteria

| Indicator | Healthy | Warning | Avoid |
|-----------|---------|---------|-------|
| Last commit | <6 months | 6-12 months | >12 months |
| Open issues | Responded | Backlog | Ignored |
| Downloads | >10k/week | >1k/week | <100/week |
| Maintainers | Multiple | Single active | Abandoned |

## Version Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| LTS | Long-term support only | Enterprise, stability critical |
| Stable | Latest stable releases | Default for most projects |
| Latest | Cutting edge | Greenfield, early adopters |

## Update Frequency

| Frequency | Cadence | Best For |
|-----------|---------|----------|
| Conservative | Quarterly | Regulated, stable |
| Regular | Monthly | Default |
| Aggressive | Weekly | Active development |

## Vulnerability Thresholds

| Severity | Action | Timeline |
|----------|--------|----------|
| Critical | Must fix | Immediate |
| High | Must fix | 7 days |
| Moderate | Should fix | 30 days |
| Low | Track | Next update cycle |

## Audit Commands

| Package Manager | Audit | Outdated |
|-----------------|-------|----------|
| npm | `npm audit` | `npm outdated` |
| pnpm | `pnpm audit` | `pnpm outdated` |
| yarn | `yarn audit` | `yarn outdated` |

## Red Flags

- No TypeScript support (for TS projects)
- Excessive dependencies (>20 transitive)
- Known security history
- Single maintainer + infrequent updates
- No semantic versioning
