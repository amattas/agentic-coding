# Security Requirements: [Feature Name]

## Assets

| Asset | Sensitivity | Protection Needed |
|-------|-------------|-------------------|
| [data/resource] | [high/medium/low] | [what protection] |

## Threat Model

| Threat | Likelihood | Impact | Mitigation |
|--------|------------|--------|------------|
| [threat] | [H/M/L] | [H/M/L] | [control] |

## Authentication

- **Required**: [yes/no]
- **Method**: [JWT/session/API key]
- **MFA**: [required/optional/none]

## Authorization

| Action | Required Permission |
|--------|---------------------|
| [action] | [role/permission] |

## Input Validation

| Input | Validation Rules |
|-------|------------------|
| [field] | [rules] |

## Data Handling

| Data Type | Storage | Transmission | Retention |
|-----------|---------|--------------|-----------|
| [type] | [encrypted/plain] | [TLS/etc] | [duration] |

## Logging

**Log**:
- [security events to log]

**Never Log**:
- [sensitive data to exclude]

## Rate Limiting

| Endpoint | Limit | Window |
|----------|-------|--------|
| [path] | [count] | [period] |

## Compliance

- [Applicable regulations or standards]
