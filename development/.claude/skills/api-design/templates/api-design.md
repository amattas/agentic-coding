# API Design: [Feature Name]

## Base URL

`[base path, e.g., /api/v1]`

## Endpoints

### [METHOD] [path]

**Description**: [what this endpoint does]

**Authentication**: [required/optional/none]

**Request**:
```json
{
  "field": "type - description"
}
```

**Response** (200):
```json
{
  "field": "type - description"
}
```

**Errors**:
| Code | Condition |
|------|-----------|
| 400 | [when returned] |
| 401 | [when returned] |
| 404 | [when returned] |

---

## Schemas

### [SchemaName]

```json
{
  "id": "string - unique identifier",
  "field": "type - description"
}
```

## Error Format

```json
{
  "error": {
    "code": "string - machine-readable error code",
    "message": "string - human-readable message",
    "details": {}
  }
}
```

## Versioning

[Strategy: URL path, header, or query param]

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| [path] | [requests per period] |
