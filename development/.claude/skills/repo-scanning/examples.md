# Repository Scanning Examples

## Example: Node.js API Project

```markdown
# Repository Map

## Overview

A REST API service for user management built with Express.js and PostgreSQL.

## Directory Structure

```
├── src/
│   ├── controllers/    # Route handlers
│   ├── models/         # Database models (Sequelize)
│   ├── routes/         # Express route definitions
│   ├── middleware/     # Auth, validation, error handling
│   ├── services/       # Business logic
│   └── utils/          # Helper functions
├── tests/
│   ├── unit/           # Unit tests (Jest)
│   └── integration/    # API integration tests
├── migrations/         # Database migrations
└── config/             # Environment configs
```

## Key Components

| Component | Location | Purpose |
|-----------|----------|---------|
| UserController | src/controllers/user.js | CRUD operations for users |
| AuthMiddleware | src/middleware/auth.js | JWT validation |
| Database | src/models/index.js | Sequelize connection setup |

## Conventions Observed

- **Naming**: camelCase for files, PascalCase for classes
- **Structure**: Feature-based organization within src/
- **Testing**: `*.test.js` suffix, mirrors src/ structure
- **Config**: dotenv with environment-specific overrides

## Entry Points

- **Main**: `src/index.js`
- **Tests**: `npm test`
- **Build**: `npm run build`

## Dependencies

- express: HTTP server
- sequelize: ORM
- jsonwebtoken: Auth tokens
- jest: Testing

## Notes

- No TypeScript, but JSDoc comments throughout
- Some legacy code in src/legacy/ - marked for removal
```
