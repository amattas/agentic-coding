---
name: security-scanning
description: Scan code for security vulnerabilities. Use after implementation changes.
---

# Security Scanning

Verify security requirements are implemented and scan for common vulnerabilities.

## Scan Areas

1. **Controls**: Verify required controls are implemented
2. **OWASP Top 10**: Check for common vulnerabilities
3. **Secrets**: Ensure no hardcoded credentials
4. **Input/Output**: Validate proper sanitization

## Output

Create `security-findings.md` using the template in `templates/security-findings.md`.

## Tips

- Check against security-requirements.md
- Look for injection vulnerabilities
- Verify authentication/authorization checks
- Ensure proper error handling (no info leakage)
- Check for hardcoded secrets or sensitive data
