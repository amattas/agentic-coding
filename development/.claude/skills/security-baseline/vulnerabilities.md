# Common Vulnerabilities

## OWASP Top 10 Patterns to Avoid

### Injection
- SQL injection: Use parameterized queries, never string concatenation
- Command injection: Avoid shell execution with user input
- LDAP injection: Sanitize LDAP queries

### Broken Authentication
- Weak password policies
- Session fixation
- Credential stuffing vulnerabilities

### Sensitive Data Exposure
- Unencrypted data at rest or in transit
- Weak cryptographic algorithms
- Excessive data exposure in APIs

### XML External Entities (XXE)
- Disable external entity processing in XML parsers

### Broken Access Control
- Missing function-level access control
- IDOR (Insecure Direct Object Reference)
- CORS misconfiguration

### Security Misconfiguration
- Default credentials
- Unnecessary features enabled
- Verbose error messages exposing internals

### Cross-Site Scripting (XSS)
- Reflected XSS
- Stored XSS
- DOM-based XSS

### Insecure Deserialization
- Deserializing untrusted data without validation

### Using Components with Known Vulnerabilities
- Outdated dependencies
- Unpatched libraries

### Insufficient Logging & Monitoring
- Missing audit trails
- No alerting on suspicious activity

## Code Patterns to Flag

- Hardcoded secrets or API keys
- `eval()` or similar dynamic code execution
- Disabled security features (e.g., CSRF protection)
- Overly permissive CORS policies
- Missing rate limiting on sensitive endpoints
