# Security Baseline

> Reference for security requirements during discovery and initialization.

## Security Levels

| Level | Data Types | Auth Required | Examples |
|-------|------------|---------------|----------|
| Public | Non-sensitive | Optional | Marketing site, docs |
| Internal | Business data | Required | Internal tools, dashboards |
| Confidential | PII, financial | Required + audit | Customer data, billing |
| Regulated | PHI, PCI data | MFA + compliance | Healthcare, payments |

## OWASP Top 10 Quick Reference

| ID | Risk | Key Checks | Framework Pattern |
|----|------|------------|-------------------|
| A01 | Broken Access Control | Role checks on all routes | Middleware auth guards |
| A02 | Cryptographic Failures | TLS, hashed passwords, encrypted PII | Use bcrypt, enforce HTTPS |
| A03 | Injection | Parameterized queries, input validation | Prisma ORM, Zod validation |
| A04 | Insecure Design | Threat modeling, least privilege | Review data flows |
| A05 | Security Misconfiguration | Disable debug, secure headers | CSP, HSTS, env validation |
| A06 | Vulnerable Components | Dependency scanning, updates | npm audit, Dependabot |
| A07 | Auth Failures | Strong passwords, rate limiting | NextAuth.js, brute-force protection |
| A08 | Data Integrity Failures | Verify signatures, secure CI/CD | Lock dependencies, sign commits |
| A09 | Logging Failures | Log auth events, no sensitive data | Structured logging, audit trails |
| A10 | SSRF | Validate URLs, allowlist domains | Block internal IPs, validate redirects |

## Authentication Patterns

| Use Case | Recommended | Notes |
|----------|-------------|-------|
| Public site | None or optional | Guest access OK |
| SaaS app | NextAuth.js | Default stack choice |
| Enterprise | SSO/SAML | Okta, Auth0, Azure AD |
| High security | MFA required | TOTP, WebAuthn preferred |
| API access | API keys or OAuth | Rotate keys, scope limits |

## Compliance Quick Reference

### GDPR

| Requirement | Implementation |
|-------------|----------------|
| Data inventory | Document all PII fields and flows |
| Consent | Explicit opt-in, record timestamp |
| Right to deletion | Soft delete + hard delete workflow |
| Data export | JSON export endpoint |
| Breach notification | 72-hour process documented |

### HIPAA

| Requirement | Implementation |
|-------------|----------------|
| PHI identification | Tag PHI fields in schema |
| BAA with vendors | Verify Supabase, Vercel BAAs |
| Access audit logs | Log all PHI access |
| Encryption | At-rest and in-transit required |
| Minimum necessary | Role-based data filtering |

### SOC2

| Requirement | Implementation |
|-------------|----------------|
| Access controls | RBAC, least privilege |
| Change management | PR reviews, deployment logs |
| Incident response | Documented runbook |
| Monitoring | Error tracking, uptime alerts |
| Vendor management | Security reviews for deps |

### PCI-DSS

| Requirement | Implementation |
|-------------|----------------|
| Cardholder scope | Use Stripe/payment processor |
| Network segmentation | Isolate payment flows |
| Encryption | TLS 1.2+, no card storage |
| Access logging | All payment access logged |
| Vulnerability scans | Quarterly ASV scans |

## Framework Security Patterns

### Next.js

| Pattern | Implementation |
|---------|----------------|
| Server Actions | Prefer over API routes for forms |
| Validation | Zod on server, never trust client |
| Cookies | httpOnly, secure, sameSite=strict |
| CSP headers | Configure in next.config.js |
| Environment | Validate with t3-env or similar |

### Prisma

| Pattern | Implementation |
|---------|----------------|
| Query safety | Always use parameterized queries |
| Sensitive fields | Use @map for column obfuscation |
| Row-level security | Filter by userId/orgId in queries |
| Audit fields | Add createdAt, updatedAt, deletedAt |

### Supabase

| Pattern | Implementation |
|---------|----------------|
| RLS policies | Enable on all tables |
| Service role | Server-only, never expose |
| Anon key | Client-safe, limited access |
| Auth hooks | Validate tokens server-side |

## Discovery Questions

Ask during requirements gathering based on data sensitivity:

| If User Mentions | Ask About |
|------------------|-----------|
| User accounts | Password policy, MFA needs |
| Customer data | GDPR, data retention |
| Payments | PCI scope, payment processor |
| Healthcare | HIPAA, BAA requirements |
| Enterprise sales | SSO, SOC2 requirements |
| File uploads | Size limits, type validation, scanning |

## Security Checklist by Level

### Public
- [ ] HTTPS enforced
- [ ] Security headers configured
- [ ] Dependencies updated

### Internal
- [ ] All Public checks
- [ ] Authentication required
- [ ] Session management
- [ ] CSRF protection

### Confidential
- [ ] All Internal checks
- [ ] Audit logging
- [ ] Data encryption
- [ ] Access controls
- [ ] Backup encryption

### Regulated
- [ ] All Confidential checks
- [ ] MFA enforced
- [ ] Compliance documentation
- [ ] Vendor agreements
- [ ] Incident response plan
