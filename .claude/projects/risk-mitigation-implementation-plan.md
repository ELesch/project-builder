# Implementation Plan: Risk Mitigation Features

> Created: 2026-01-23
> Status: Ready for Implementation

## Summary

Add four risk mitigation features to the project builder framework:

1. **Security Checklist System** - OWASP-based security review process
2. **Dependency Health System** - License, vulnerability, and maintenance tracking
3. **Operational Runbooks** - Incident response and operational procedures
4. **Onboarding Guide** - Developer onboarding and knowledge transfer

## Integration Points

These features touch multiple phases of the project lifecycle:

```
DISCOVERY          ARCHITECTURE        INITIALIZATION       CREATED PROJECT
    │                   │                    │                    │
    ├─ Security Qs      ├─ Security arch     ├─ Research          ├─ SECURITY.md
    ├─ Ops model Qs     ├─ Ops decisions     ├─ Create files      ├─ checklists/
    ├─ Team Qs          ├─ Agent selection   ├─ Populate          ├─ runbooks/
    └─ Dep preferences  └─ Conventions       └─ templates         └─ ONBOARDING.md
```

---

## Task Group 1: Foundation - Defaults Files

Create reference documents used during discovery and initialization.

### Task 1.1: Create security-baseline.md
**File:** `.claude/defaults/security-baseline.md`

Content:
- OWASP Top 10 reference with detection guidance
- Framework-specific security patterns (Next.js, Express, etc.)
- Common vulnerability patterns to check
- Security level definitions (public, internal, confidential, regulated)
- Authentication pattern recommendations by use case
- Compliance requirement summaries (GDPR, HIPAA, SOC2, PCI-DSS)

### Task 1.2: Create dependency-policy.md
**File:** `.claude/defaults/dependency-policy.md`

Content:
- License compatibility matrix (MIT, Apache, GPL, etc.)
- Dependency health criteria (maintenance activity, download counts, CVE history)
- Version strategy definitions (LTS, stable, latest)
- Update frequency recommendations
- Abandonment risk indicators
- Vulnerability severity thresholds

### Task 1.3: Create operational-baseline.md
**File:** `.claude/defaults/operational-baseline.md`

Content:
- Operational model definitions (developer-operated, ops team, managed service)
- Monitoring tier recommendations (basic, comprehensive, APM)
- Logging standards (structured logging format, log levels)
- Alerting guidelines (what to alert on, severity levels)
- On-call requirements by project type
- Incident response process templates

---

## Task Group 2: Discovery Phase Updates

Update discovery agent and templates to gather new requirements.

### Task 2.1: Update project-discovery.md
**File:** `.claude/agents/project-discovery.md`

Add new phases:

**Phase 5b: Security Requirements**
Questions to add:
- Data sensitivity level? (public, internal, confidential, regulated)
- Authentication needed? (none, simple login, SSO, MFA required)
- Compliance requirements? (none, GDPR, HIPAA, SOC2, PCI-DSS)
- Expected threat model? (public internet, internal only, high-value target)

**Phase 5c: Operational Model**
Questions to add:
- Who operates this in production? (developer, ops team, managed service)
- On-call requirements? (none, business hours, 24/7)
- Monitoring preferences? (basic, comprehensive, APM)
- Existing incident process? (ad-hoc, documented, formal)

**Phase 6b: Team & Onboarding**
Questions to add:
- Current team size?
- Expected team growth?
- Developer experience levels?
- Onboarding frequency? (rarely, occasionally, frequently)

**Phase 4 Update: Dependency Preferences** (technical users only)
- Version strategy preference? (stable LTS, latest stable, cutting edge)
- Update frequency preference? (conservative, regular, aggressive)
- Specific packages to avoid? (licensing, security, preference)

### Task 2.2: Update project-brief.md template
**File:** `.claude/templates/discovery/project-brief.md`

Add new sections:
```markdown
## Security Requirements

### Data Sensitivity
- Level: {{public/internal/confidential/regulated}}
- Data types handled: {{list}}

### Authentication & Authorization
- Authentication: {{none/simple/SSO/MFA}}
- Authorization model: {{none/role-based/attribute-based}}

### Compliance
- Requirements: {{none/GDPR/HIPAA/SOC2/PCI-DSS}}
- Certifications needed: {{if any}}

## Operational Model

### Operations
- Operated by: {{developer/ops-team/managed-service}}
- On-call: {{none/business-hours/24x7}}

### Monitoring
- Level: {{basic/comprehensive/APM}}
- Alerting: {{what should alert}}

### Incident Response
- Process: {{ad-hoc/documented/formal}}

## Team

### Current State
- Team size: {{number}}
- Experience levels: {{junior/mid/senior mix}}

### Future State
- Expected growth: {{stable/growing/rapid}}
- Onboarding frequency: {{rarely/occasionally/frequently}}

## Dependency Preferences

- Version strategy: {{LTS/stable/latest}}
- Update frequency: {{conservative/regular/aggressive}}
- Packages to avoid: {{list or "none"}}
```

---

## Task Group 3: Architecture Phase Updates

### Task 3.1: Update project-architect.md
**File:** `.claude/agents/project-architect.md`

Add new sections:

**Security Architecture Section**
- Authentication pattern selection based on requirements
- Authorization model design
- Data protection approach (encryption, PII handling)
- Security agent customization needs
- Compliance-specific patterns

**Operational Architecture Section**
- Monitoring strategy based on ops model
- Logging pattern selection
- Alerting requirements
- Runbook selection based on on-call model
- Error handling patterns

**Onboarding Considerations**
- Documentation requirements based on team size/growth
- Knowledge capture patterns
- Handoff procedures

Add to Architecture Document Template:
```markdown
## Security Design

### Authentication
- Pattern: {{chosen pattern}}
- Rationale: {{why}}

### Authorization
- Model: {{role-based/attribute-based/etc.}}
- Enforcement points: {{where checked}}

### Data Protection
- Encryption: {{at-rest/in-transit/both}}
- PII handling: {{approach}}

### Compliance
- {{Requirement}}: {{How addressed}}

## Operational Design

### Monitoring
- Tools: {{recommended tools}}
- Key metrics: {{what to track}}

### Logging
- Format: {{structured JSON/etc.}}
- Retention: {{policy}}

### Alerting
- Critical: {{what alerts}}
- Warning: {{what warns}}

### Runbooks Needed
- {{List of required runbooks}}

## Agents Included

| Agent | Purpose | Customizations |
|-------|---------|----------------|
| dev-security | Security review | {{compliance-specific checks}} |
...
```

---

## Task Group 4: New Templates

Create new template files for created projects.

### Task 4.1: Create SECURITY.md.template
**File:** `.claude/templates/orchestrator/.claude/SECURITY.md.template`

Content:
```markdown
# Security Overview: {{PROJECT_NAME}}

## Security Level
- **Data Sensitivity:** {{SECURITY_LEVEL}}
- **Compliance:** {{COMPLIANCE_REQS}}

## Authentication
- **Method:** {{AUTH_TYPE}}
- **Implementation:** {{auth details}}

## Security Checklist Status

| Review | Last Completed | Status |
|--------|----------------|--------|
| OWASP Top 10 | Never | Pending |
| Dependency Audit | Never | Pending |
| Secret Scan | Never | Pending |
| Access Control | Never | Pending |

## Quick Reference

@.claude/checklists/security-review.md

## Reporting Security Issues

{{Instructions for reporting vulnerabilities}}
```

### Task 4.2: Create security-review.md.template
**File:** `.claude/templates/orchestrator/.claude/checklists/security-review.md.template`

Content:
```markdown
# Security Review Checklist

> Use this before any deployment or security-sensitive release.

## Pre-Review Setup
- [ ] Have access to the codebase
- [ ] Know the security level: {{SECURITY_LEVEL}}
- [ ] Know compliance requirements: {{COMPLIANCE_REQS}}

## OWASP Top 10 Review

### A01: Broken Access Control
- [ ] All endpoints check authentication
- [ ] Users can only access their own data
- [ ] Admin functions restricted to admins
- [ ] No IDOR vulnerabilities (direct object reference)

### A02: Cryptographic Failures
- [ ] No hardcoded secrets in code
- [ ] Secrets stored in environment variables
- [ ] TLS/HTTPS enforced
- [ ] Passwords hashed with bcrypt/argon2

### A03: Injection
- [ ] All SQL uses parameterized queries
- [ ] No eval() on user input
- [ ] HTML output escaped (XSS prevention)
- [ ] Command execution sanitized

### A04: Insecure Design
- [ ] Threat model documented
- [ ] Security requirements defined
- [ ] Fail-secure patterns used

### A05: Security Misconfiguration
- [ ] No default credentials
- [ ] Error messages don't leak info
- [ ] Security headers configured
- [ ] CORS properly restricted

### A06: Vulnerable Components
- [ ] npm audit (or equivalent) clean
- [ ] No known CVEs in dependencies
- [ ] Dependencies recently updated

### A07: Authentication Failures
- [ ] Strong password requirements
- [ ] Account lockout after failures
- [ ] Session management secure
- [ ] MFA available (if required)

### A08: Data Integrity Failures
- [ ] Updates verified/signed
- [ ] Serialization safe
- [ ] CI/CD pipeline secured

### A09: Logging Failures
- [ ] Security events logged
- [ ] Logs don't contain PII/secrets
- [ ] Log injection prevented

### A10: SSRF
- [ ] External URLs validated
- [ ] No internal URLs accessible

## Additional Checks ({{COMPLIANCE_REQS}})

{{Compliance-specific checks if applicable}}

## Sign-Off

- Reviewer: _______________
- Date: _______________
- Findings: [ ] None [ ] See issues below

### Issues Found

| Severity | Description | Location | Remediation |
|----------|-------------|----------|-------------|
| | | | |
```

### Task 4.3: Create deployment.md.template (checklist)
**File:** `.claude/templates/orchestrator/.claude/checklists/deployment.md.template`

Content:
```markdown
# Deployment Checklist

## Pre-Deployment

### Code Ready
- [ ] All tests passing
- [ ] No lint errors
- [ ] Security review complete (if required)
- [ ] Code reviewed and approved

### Environment Ready
- [ ] Environment variables set
- [ ] Secrets configured
- [ ] Database migrations ready
- [ ] Rollback plan documented

### Documentation
- [ ] CHANGELOG updated
- [ ] API changes documented
- [ ] Breaking changes communicated

## Deployment

- [ ] Notify team of deployment start
- [ ] Run database migrations (if any)
- [ ] Deploy application
- [ ] Verify health checks pass
- [ ] Run smoke tests

## Post-Deployment

- [ ] Monitor error rates
- [ ] Monitor performance metrics
- [ ] Verify key user flows work
- [ ] Notify team of deployment complete

## Rollback Triggers

Rollback immediately if:
- Error rate exceeds baseline by 2x
- P99 latency exceeds SLA
- Critical functionality broken
- Security issue discovered

## Rollback Procedure

See: @.claude/runbooks/rollback.md
```

### Task 4.4: Create dependency-review.md.template
**File:** `.claude/templates/orchestrator/.claude/checklists/dependency-review.md.template`

Content:
```markdown
# Dependency Review Checklist

## When to Use
- Before adding a new dependency
- Quarterly health review
- After security advisory

## New Dependency Checklist

### Necessity
- [ ] No existing solution in codebase
- [ ] No built-in solution in platform
- [ ] Benefit outweighs dependency cost

### Health Assessment
- [ ] Last commit within 6 months
- [ ] Active maintainer(s)
- [ ] Reasonable issue response time
- [ ] No unaddressed security issues

### License
- [ ] License compatible with project
- [ ] No viral license concerns (GPL)
- [ ] License documented

### Security
- [ ] No known CVEs (npm audit)
- [ ] Reasonable number of dependencies
- [ ] No suspicious code patterns

### Quality
- [ ] TypeScript support (if TS project)
- [ ] Good documentation
- [ ] Test coverage
- [ ] Semantic versioning

## Quarterly Health Review

Run:
```bash
npm audit
npm outdated
```

Check:
- [ ] All high/critical vulnerabilities addressed
- [ ] No abandoned packages (1+ year stale)
- [ ] Major version updates evaluated

## Current Dependencies Status

| Package | Version | Last Audit | Status |
|---------|---------|------------|--------|
| | | | |
```

### Task 4.5: Create runbooks directory with templates
**Directory:** `.claude/templates/orchestrator/.claude/runbooks/`

#### README.md.template
```markdown
# Runbooks

Operational procedures for {{PROJECT_NAME}}.

## Available Runbooks

| Runbook | Use When |
|---------|----------|
| [Deployment](deployment.md) | Deploying to production |
| [Rollback](rollback.md) | Reverting a bad deployment |
| [Incident Response](incident-response.md) | Production incident occurs |
| [Database Recovery](database-recovery.md) | Database issues |
| [On-Call](on-call.md) | On-call reference |

## How to Use

1. Find the relevant runbook
2. Follow steps in order
3. Document any deviations
4. Update runbook if steps are wrong
```

#### incident-response.md.template
```markdown
# Incident Response Runbook

## Severity Levels

| Level | Definition | Response Time |
|-------|------------|---------------|
| SEV1 | Complete outage | Immediate |
| SEV2 | Major feature broken | 30 minutes |
| SEV3 | Minor degradation | 4 hours |
| SEV4 | Cosmetic issue | Next business day |

## Step 1: Assess & Communicate

- [ ] Determine severity level
- [ ] Notify team (Slack/PagerDuty/etc.)
- [ ] Start incident document

## Step 2: Investigate

- [ ] Check monitoring dashboard
- [ ] Check recent deployments
- [ ] Check error logs
- [ ] Check external dependencies

## Step 3: Mitigate

Options:
- [ ] Rollback deployment (see rollback.md)
- [ ] Toggle feature flag
- [ ] Scale up resources
- [ ] Failover to backup
- [ ] Apply hotfix

## Step 4: Resolve

- [ ] Confirm service restored
- [ ] Verify with monitoring
- [ ] Communicate resolution

## Step 5: Post-Incident

- [ ] Schedule post-mortem (within 48h for SEV1/2)
- [ ] Document timeline
- [ ] Identify action items
- [ ] Update runbooks if needed

## Contacts

| Role | Contact |
|------|---------|
| On-call primary | {{TBD}} |
| On-call backup | {{TBD}} |
| Engineering lead | {{TBD}} |

## Links

- Monitoring: {{URL}}
- Logs: {{URL}}
- Deployment: {{URL}}
```

#### rollback.md.template
```markdown
# Rollback Runbook

## When to Rollback

- Error rate spike (>2x baseline)
- Critical functionality broken
- Security vulnerability discovered
- P99 latency exceeds SLA

## Pre-Rollback

- [ ] Confirm rollback is the right action
- [ ] Notify team
- [ ] Note current version: _______________
- [ ] Note target version: _______________

## Rollback Steps

### Vercel (Default)
```bash
# List recent deployments
vercel list

# Rollback to previous deployment
vercel rollback [deployment-url]
```

### Database Migrations
If rollback includes database changes:
```bash
# Check if migration is reversible
# Run down migration if needed
npx prisma migrate rollback
```

## Post-Rollback

- [ ] Verify service restored
- [ ] Check error rates returning to normal
- [ ] Notify team of completion
- [ ] Document reason for rollback
- [ ] Create ticket for fix

## Rollback Failed?

1. Try rolling back one more version
2. Check database compatibility
3. Escalate to on-call secondary
4. Consider maintenance mode
```

#### deployment.md.template
```markdown
# Deployment Runbook

## Standard Deployment (Vercel)

### Pre-Deployment
- [ ] All tests passing in CI
- [ ] PR approved and merged to main
- [ ] No blocking issues

### Deployment
```bash
# Vercel auto-deploys on merge to main
# Manual deploy if needed:
vercel --prod
```

### Verification
- [ ] Check deployment status
- [ ] Verify health endpoint
- [ ] Run smoke tests
- [ ] Monitor error rates (15 min)

## Database Migration Deployment

### Pre-Deployment
- [ ] Migration tested locally
- [ ] Backup created
- [ ] Rollback plan ready

### Steps
```bash
# 1. Deploy migration
npx prisma migrate deploy

# 2. Verify database state
npx prisma migrate status

# 3. Deploy application code
vercel --prod
```

## Hotfix Deployment

For urgent fixes only:

- [ ] Create hotfix branch from main
- [ ] Minimal change only
- [ ] Get one approval
- [ ] Deploy
- [ ] Verify
- [ ] Document
```

#### database-recovery.md.template
```markdown
# Database Recovery Runbook

## Supabase (Default)

### Check Status
- Dashboard: https://app.supabase.com
- Check project health
- Check recent changes

### Connection Issues

1. Check connection string in env vars
2. Check IP allowlist
3. Check connection pool exhaustion
4. Restart connection pool

### Data Recovery

Supabase maintains point-in-time recovery:

1. Go to Supabase Dashboard
2. Navigate to Database > Backups
3. Select restore point
4. Restore to new database
5. Verify data
6. Update connection strings

### Performance Issues

1. Check slow query log
2. Check for missing indexes
3. Check for lock contention
4. Scale up if needed (Dashboard > Settings)

## Emergency Contacts

- Supabase support: support@supabase.io
- Status page: status.supabase.com
```

#### on-call.md.template
```markdown
# On-Call Reference

## Responsibilities

- Monitor alerts during shift
- Respond within SLA
- Escalate when needed
- Document incidents

## Alert Response

### When Paged

1. Acknowledge alert
2. Assess severity
3. Start incident (if SEV1/2)
4. Investigate and mitigate
5. Document

### Escalation

Escalate to backup when:
- Cannot resolve within 30 minutes
- Need domain expertise
- Multiple systems affected
- SEV1 incident

## Key Links

| Resource | URL |
|----------|-----|
| Monitoring | {{URL}} |
| Logs | {{URL}} |
| Runbooks | This folder |
| Incident docs | {{URL}} |

## Common Issues

| Symptom | Likely Cause | First Step |
|---------|--------------|------------|
| High error rate | Bad deployment | Check recent deploys |
| Slow responses | Database | Check DB metrics |
| Timeouts | External API | Check dependencies |
| OOM | Memory leak | Restart, investigate |

## Handoff

At end of shift:
- [ ] Note any ongoing issues
- [ ] Brief incoming on-call
- [ ] Transfer pager/alerts
```

### Task 4.6: Create ONBOARDING.md.template
**File:** `.claude/templates/orchestrator/ONBOARDING.md.template`

Content:
```markdown
# Developer Onboarding: {{PROJECT_NAME}}

Welcome to {{PROJECT_NAME}}! This guide will help you get set up and productive.

## Prerequisites

Before you begin, ensure you have:

{{PREREQUISITES_LIST}}

## Quick Start

### 1. Clone the Repository
```bash
git clone {{REPO_URL}}
cd {{PROJECT_SLUG}}
```

### 2. Install Dependencies
```bash
{{INSTALL_COMMAND}}
```

### 3. Set Up Environment
```bash
cp .env.example .env
# Edit .env with your values
```

### 4. Start Development
```bash
{{DEV_COMMAND}}
```

## Project Structure

```
{{DIRECTORY_STRUCTURE}}
```

## Key Files to Read

1. `CLAUDE.md` - Development conventions
2. `.claude/tech/stack.md` - Technology versions and gotchas
3. `README.md` - Project overview

## Development Workflow

### Making Changes
1. Create feature branch from `main`
2. Make changes
3. Run tests: `{{TEST_COMMAND}}`
4. Create PR
5. Get review
6. Merge

### Code Style
- {{STYLE_GUIDELINES}}

### Testing
- {{TESTING_APPROACH}}

## Architecture Overview

{{ARCHITECTURE_SUMMARY}}

## Common Tasks

### Adding a New Feature
1. {{Step 1}}
2. {{Step 2}}
3. {{Step 3}}

### Fixing a Bug
1. {{Step 1}}
2. {{Step 2}}
3. {{Step 3}}

### Deploying
See: `.claude/runbooks/deployment.md`

## Getting Help

- Documentation: This folder
- Team lead: {{CONTACT}}
- Slack channel: {{CHANNEL}}

## FAQ

### How do I run tests?
```bash
{{TEST_COMMAND}}
```

### How do I access the database?
{{DATABASE_ACCESS}}

### How do I deploy?
{{DEPLOYMENT_INFO}}

## First Week Checklist

- [ ] Complete local setup
- [ ] Read CLAUDE.md
- [ ] Read key documentation
- [ ] Complete first small task
- [ ] Attend team standup
- [ ] Meet with mentor/lead
```

### Task 4.7: Create TECH_DEBT.md.template
**File:** `.claude/templates/orchestrator/.claude/TECH_DEBT.md.template`

Content:
```markdown
# Technical Debt Tracker

## How to Use

Add items as they're discovered. Review quarterly to prioritize.

## Priority Levels

| Priority | Impact | When to Fix |
|----------|--------|-------------|
| P1 | Blocks features or causes incidents | Next sprint |
| P2 | Slows development significantly | This quarter |
| P3 | Minor inconvenience | When convenient |

## Current Debt

### P1 - High Priority

| Item | Description | Impact | Added |
|------|-------------|--------|-------|
| | | | |

### P2 - Medium Priority

| Item | Description | Impact | Added |
|------|-------------|--------|-------|
| | | | |

### P3 - Low Priority

| Item | Description | Impact | Added |
|------|-------------|--------|-------|
| | | | |

## Resolved Debt

| Item | Resolution | Resolved |
|------|------------|----------|
| | | |

## Debt Review Log

| Date | Reviewer | Actions Taken |
|------|----------|---------------|
| | | |
```

### Task 4.8: Create dependencies.md.template
**File:** `.claude/templates/orchestrator/.claude/tech/dependencies.md.template`

Content:
```markdown
# Dependency Health: {{PROJECT_NAME}}

> Last audit: {{DATE}}

## Summary

| Category | Count | Status |
|----------|-------|--------|
| Direct dependencies | {{COUNT}} | {{OK/WARNING/CRITICAL}} |
| Known vulnerabilities | {{COUNT}} | {{OK/WARNING/CRITICAL}} |
| Outdated (major) | {{COUNT}} | {{OK/WARNING/CRITICAL}} |

## Vulnerability Status

Last `npm audit` result:

```
{{AUDIT_RESULT}}
```

### Outstanding Issues

| Package | Severity | CVE | Status |
|---------|----------|-----|--------|
| | | | |

## License Matrix

| License | Packages | Compatible |
|---------|----------|------------|
| MIT | {{list}} | Yes |
| Apache-2.0 | {{list}} | Yes |
| ISC | {{list}} | Yes |

## Key Dependencies Health

| Package | Version | Last Update | Maintenance |
|---------|---------|-------------|-------------|
| {{package}} | {{version}} | {{date}} | {{active/slow/stale}} |

## Update Strategy

- **Version strategy:** {{LTS/stable/latest}}
- **Update frequency:** {{conservative/regular/aggressive}}
- **Automation:** {{none/dependabot/renovate}}

## Review Schedule

- **Security vulnerabilities:** Weekly (automated)
- **Outdated packages:** Monthly
- **Full health review:** Quarterly

## Packages to Avoid

{{List of packages to avoid and why, or "None"}}
```

---

## Task Group 5: Agent Updates

### Task 5.1: Enhance dev-security.md.template
**File:** `.claude/templates/orchestrator/.claude/agents/dev-security.md.template`

Add to existing content:

```markdown
## Security Resources

@.claude/SECURITY.md
@.claude/checklists/security-review.md
@.claude/tech/dependencies.md

## Pre-Deployment Security Review

Before any deployment to production, complete the security review checklist:

1. Read the current security level and requirements in SECURITY.md
2. Use the checklist in checklists/security-review.md
3. Check dependency health in tech/dependencies.md
4. Update SECURITY.md with review date

## Compliance-Specific Checks

### GDPR
- [ ] Data inventory documented
- [ ] Consent mechanisms in place
- [ ] Right to deletion implemented
- [ ] Data export available

### HIPAA
- [ ] PHI identified and protected
- [ ] Access controls implemented
- [ ] Audit logging enabled
- [ ] BAA in place with vendors

### SOC2
- [ ] Access controls documented
- [ ] Change management process
- [ ] Incident response plan
- [ ] Monitoring in place

### PCI-DSS
- [ ] Cardholder data identified
- [ ] Network segmentation
- [ ] Encryption in place
- [ ] Access logging enabled
```

### Task 5.2: Create dev-ops.md.template (optional agent)
**File:** `.claude/templates/orchestrator/.claude/agents/dev-ops.md.template`

Content:
```markdown
# Operations Agent

## Role

Manage operational aspects including runbooks, monitoring, and incident response procedures.

## Scope

- **Primary:** Runbooks, monitoring configuration, operational documentation
- **Output:** Operational procedures, alerts, dashboards
- **Focus areas:** Reliability, observability, incident management
- **Off-limits:** Application feature development

## Operational Resources

@.claude/runbooks/README.md
@.claude/checklists/deployment.md

## CRITICAL: YOU MUST ALWAYS

1. Keep runbooks up to date with actual procedures
2. Document any operational changes
3. Ensure monitoring covers critical paths
4. Maintain incident response procedures
5. Test recovery procedures periodically

## CRITICAL: NEVER DO THESE

1. Skip runbook updates when procedures change
2. Remove monitoring without replacement
3. Ignore alerts or alert fatigue
4. Deploy without rollback plan

## Key Tasks

### Update Runbooks
- Review procedures are accurate
- Test procedures work
- Update contacts and links
- Version significant changes

### Configure Monitoring
- Ensure critical paths monitored
- Set appropriate thresholds
- Configure alerting
- Avoid alert fatigue

### Incident Management
- Document incidents
- Conduct post-mortems
- Track action items
- Update procedures based on learnings

## Output Format

When updating operational docs:
1. Note what changed
2. Note why it changed
3. Note who should be informed
```

---

## Task Group 6: Initializer Updates

### Task 6.1: Update project-initializer.md
**File:** `.claude/agents/project-initializer.md`

Add new steps:

```markdown
### Step 2c: Dependency Health Check (NEW)

Before creating files, assess dependency health:

1. Research known vulnerabilities in chosen packages
2. Check license compatibility
3. Note any packages to avoid
4. Prepare initial dependencies.md content

### Step 4c: Create Security Files (NEW)

Based on security requirements from discovery:

1. Create `.claude/SECURITY.md` with:
   - Security level from discovery
   - Compliance requirements
   - Authentication details
   - Checklist status (all pending)

2. Create `.claude/checklists/` directory with:
   - security-review.md (customized for compliance)
   - deployment.md
   - dependency-review.md

### Step 4d: Create Operations Files (NEW)

Based on operational model from discovery:

1. Create `.claude/runbooks/` directory with:
   - README.md (index)
   - incident-response.md (if on-call)
   - rollback.md (always)
   - deployment.md (always)
   - database-recovery.md (if database)
   - on-call.md (if 24/7)

2. Select runbooks based on:
   - Ops model: developer → fewer runbooks
   - Ops model: ops-team/managed → more runbooks
   - On-call: none → skip on-call.md
   - Database: none → skip database-recovery.md

### Step 4e: Create Onboarding Guide (NEW)

Create `ONBOARDING.md` in project root with:
- Prerequisites populated from tech research
- Install commands from tech research
- Project structure from architecture
- Workflow from conventions

### Step 4f: Create Dependency Health File (NEW)

Create `.claude/tech/dependencies.md` with:
- Initial audit results
- License matrix for chosen dependencies
- Update strategy from discovery
```

Add to Verification Checklist:
```markdown
- [ ] `.claude/SECURITY.md` created with correct security level
- [ ] `.claude/checklists/security-review.md` customized for compliance
- [ ] `.claude/runbooks/` has appropriate runbooks for ops model
- [ ] `ONBOARDING.md` has correct commands and structure
- [ ] `.claude/tech/dependencies.md` has initial audit
```

Add to Template Variables:
```markdown
**Security variables** (from discovery):

| Variable | Replace With |
|----------|--------------|
| `{{SECURITY_LEVEL}}` | public/internal/confidential/regulated |
| `{{AUTH_TYPE}}` | none/simple/SSO/MFA |
| `{{COMPLIANCE_REQS}}` | none/GDPR/HIPAA/SOC2/PCI-DSS |

**Operations variables** (from discovery):

| Variable | Replace With |
|----------|--------------|
| `{{OPS_MODEL}}` | developer/ops-team/managed-service |
| `{{ON_CALL}}` | none/business-hours/24x7 |
| `{{MONITORING_LEVEL}}` | basic/comprehensive/APM |

**Team variables** (from discovery):

| Variable | Replace With |
|----------|--------------|
| `{{TEAM_SIZE}}` | number |
| `{{ONBOARDING_FREQ}}` | rarely/occasionally/frequently |

**Dependency variables** (from research):

| Variable | Replace With |
|----------|--------------|
| `{{DEP_STRATEGY}}` | LTS/stable/latest |
| `{{AUDIT_RESULT}}` | npm audit output |
| `{{LICENSE_MATRIX}}` | license compatibility table |
```

---

## Task Group 7: Integration Updates

### Task 7.1: Update roster.md
**File:** `.claude/roster.md`

Add:
```markdown
## New Discovery Topics

The discovery phase now includes:
- Security requirements (Phase 5b)
- Operational model (Phase 5c)
- Team & onboarding (Phase 6b)
- Dependency preferences (Phase 4 for technical users)

## New Created Project Files

Projects now include:
- `.claude/SECURITY.md` - Security overview and checklist status
- `.claude/checklists/` - Security, deployment, dependency checklists
- `.claude/runbooks/` - Operational procedures
- `.claude/tech/dependencies.md` - Dependency health tracking
- `ONBOARDING.md` - Developer onboarding guide
- `.claude/TECH_DEBT.md` - Technical debt tracker

## Agent Availability

| Agent | When Included |
|-------|---------------|
| dev-ops | On-call = business-hours or 24x7 |
| dev-security | Always (compliance affects customization) |
```

### Task 7.2: Update main CLAUDE.md
**File:** `CLAUDE.md`

Add to documentation:

```markdown
## Risk Mitigation Features

The project builder now addresses lifecycle risks:

### Security
- Security requirements gathered during discovery
- OWASP Top 10 security review checklist
- Compliance-specific checks (GDPR, HIPAA, SOC2, PCI-DSS)
- Security agent with @-mentions to checklists

### Dependencies
- Dependency policy baseline
- Health tracking (licenses, vulnerabilities, maintenance)
- Audit results in tech/dependencies.md

### Operations
- Operational model captured during discovery
- Runbooks for deployment, rollback, incidents
- On-call guide for projects with on-call requirements

### Onboarding
- ONBOARDING.md with prerequisites and setup
- Populated automatically from tech research
- Updated when conventions change
```

---

## File Creation Summary

### New Files to Create (15)

| File | Location |
|------|----------|
| security-baseline.md | .claude/defaults/ |
| dependency-policy.md | .claude/defaults/ |
| operational-baseline.md | .claude/defaults/ |
| SECURITY.md.template | .claude/templates/orchestrator/.claude/ |
| security-review.md.template | .claude/templates/orchestrator/.claude/checklists/ |
| deployment.md.template | .claude/templates/orchestrator/.claude/checklists/ |
| dependency-review.md.template | .claude/templates/orchestrator/.claude/checklists/ |
| README.md.template | .claude/templates/orchestrator/.claude/runbooks/ |
| incident-response.md.template | .claude/templates/orchestrator/.claude/runbooks/ |
| rollback.md.template | .claude/templates/orchestrator/.claude/runbooks/ |
| deployment.md.template (runbook) | .claude/templates/orchestrator/.claude/runbooks/ |
| database-recovery.md.template | .claude/templates/orchestrator/.claude/runbooks/ |
| on-call.md.template | .claude/templates/orchestrator/.claude/runbooks/ |
| ONBOARDING.md.template | .claude/templates/orchestrator/ |
| TECH_DEBT.md.template | .claude/templates/orchestrator/.claude/ |
| dependencies.md.template | .claude/templates/orchestrator/.claude/tech/ |
| dev-ops.md.template | .claude/templates/orchestrator/.claude/agents/ |

### Files to Update (8)

| File | Updates |
|------|---------|
| project-discovery.md | Add phases 5b, 5c, 6b, update phase 4 |
| project-brief.md | Add security, ops, team, dependency sections |
| project-architect.md | Add security/ops architecture sections |
| project-initializer.md | Add steps 2c, 4c-4f, new variables |
| dev-security.md.template | Add @-mentions, compliance checks |
| roster.md | Document new features |
| CLAUDE.md | Document risk mitigation features |
| VERSION | Bump to 1.1.0 |

---

## Implementation Order

### Phase 1: Foundation (No Dependencies)
1. Task 1.1: security-baseline.md
2. Task 1.2: dependency-policy.md
3. Task 1.3: operational-baseline.md

### Phase 2: Templates (Depends on Phase 1)
4. Task 4.1: SECURITY.md.template
5. Task 4.2-4.4: Checklists
6. Task 4.5: Runbooks directory
7. Task 4.6: ONBOARDING.md.template
8. Task 4.7: TECH_DEBT.md.template
9. Task 4.8: dependencies.md.template
10. Task 5.2: dev-ops.md.template

### Phase 3: Agent Updates (Depends on Phase 2)
11. Task 5.1: Enhance dev-security.md.template
12. Task 2.1: Update project-discovery.md
13. Task 2.2: Update project-brief.md
14. Task 3.1: Update project-architect.md
15. Task 6.1: Update project-initializer.md

### Phase 4: Integration (Depends on Phase 3)
16. Task 7.1: Update roster.md
17. Task 7.2: Update CLAUDE.md
18. Bump VERSION to 1.1.0

### Phase 5: Testing
19. Create test project to verify all features work
20. Test migration flow
21. Document any issues found

---

## Success Criteria

- [ ] All 15 new files created
- [ ] All 8 existing files updated
- [ ] New project includes security, ops, onboarding files
- [ ] Security checklist is customized based on compliance
- [ ] Runbooks are selected based on ops model
- [ ] Onboarding guide has correct commands
- [ ] Dependency health is tracked
- [ ] VERSION bumped to 1.1.0
