# Security Testing Requirements

## Automated Checks

### Static Analysis (SAST)
- Run on every PR
- Tools: [e.g., Semgrep, CodeQL, Snyk Code]
- Block merge on high/critical findings

### Dependency Scanning
- Run on every PR and scheduled daily
- Tools: [e.g., Dependabot, Snyk, npm audit]
- Auto-create PRs for security updates

### Secret Scanning
- Pre-commit hooks to catch secrets before push
- Tools: [e.g., git-secrets, truffleHog, Gitleaks]

### Container Scanning
- Scan images before deployment
- Tools: [e.g., Trivy, Snyk Container]

## Manual Review Checklist

### Authentication Flows
- [ ] Password reset flow tested for account enumeration
- [ ] Session management follows secure practices
- [ ] MFA implementation verified

### Authorization
- [ ] All endpoints enforce proper access control
- [ ] Role escalation tested
- [ ] IDOR vulnerabilities checked

### Data Handling
- [ ] Sensitive data encrypted at rest
- [ ] TLS enforced for data in transit
- [ ] PII handling follows data retention policy

### Input Validation
- [ ] All user inputs validated server-side
- [ ] File uploads restricted and scanned
- [ ] API rate limiting in place

## Penetration Testing

- Schedule: [e.g., annually, before major releases]
- Scope: [e.g., external-facing APIs, authentication flows]
- Remediation SLA: Critical (24h), High (7d), Medium (30d)
