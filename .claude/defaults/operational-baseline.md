# Operational Baseline

> Reference for operational requirements during discovery and initialization.

## Operational Models

| Model | Description | Runbooks Needed | On-Call |
|-------|-------------|-----------------|---------|
| Developer | Dev team operates | Minimal | None |
| Ops Team | Dedicated ops | Comprehensive | Business hours+ |
| Managed | External service | Reference only | Vendor |

## Monitoring Tiers

| Tier | What's Monitored | Tools |
|------|------------------|-------|
| Basic | Uptime, errors | Vercel Analytics, Sentry |
| Comprehensive | + Performance, logs | + Datadog/New Relic |
| APM | + Traces, profiling | Full observability stack |

## Logging Standards

### Log Levels

| Level | Use For |
|-------|---------|
| error | Failures requiring attention |
| warn | Degraded but functional |
| info | Key business events |
| debug | Development only |

### Structured Format

```json
{"level":"info","msg":"...","timestamp":"...","requestId":"..."}
```

## Alerting Guidelines

| Condition | Severity | Response |
|-----------|----------|----------|
| Service down | Critical | Immediate |
| Error rate >1% | High | 15 min |
| Latency p99 >2s | Medium | 1 hour |
| Disk >80% | Low | Next day |

## On-Call Requirements

| Level | Coverage | Response SLA |
|-------|----------|--------------|
| None | N/A | Best effort |
| Business | 9-5 local | 4 hours |
| Extended | 7am-10pm | 1 hour |
| 24/7 | Always | 15 min |

## Incident Severity

| Severity | Definition | Response Time |
|----------|------------|---------------|
| SEV1 | Complete outage | Immediate |
| SEV2 | Major feature broken | 30 min |
| SEV3 | Minor degradation | 4 hours |
| SEV4 | Cosmetic/minor | Next day |

## Runbook Selection Guide

| Ops Model | Include |
|-----------|---------|
| Developer | rollback, deployment |
| Ops Team | + incident-response, on-call, database-recovery |
| Managed | Reference docs only |
