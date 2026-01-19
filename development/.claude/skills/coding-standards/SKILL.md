---
name: coding-standards
description: Coding style and structural conventions for this codebase.
---

# Coding Standards

## General Principles

### Structure

- Prefer small, focused functions.
- Avoid deeply nested conditionals; use early returns.
- Extract reusable logic into shared modules.

### Comments & Documentation

- Write comments to explain *why*, not *what*.
- Keep docstrings in sync with behavior; avoid outdated comments.

### Error & Logging

- Use structured logging with consistent fields:
  - `requestId`, `userId`, `component`, `severity`
- Never log secrets, tokens, or sensitive PII.

### Misc

- Avoid long parameter lists; group related values into objects.
- Prefer pure functions where practical.
- Add TODOs with owner and context, e.g.:
  - `// TODO [owner]: [short description] ([ticket link])`

---

## Language-Specific Standards

For detailed language-specific conventions, see:

- **[Python](python.md)** - Includes modularization guidelines (500-800 lines ideal, 1000 max), black/ruff usage, PEP 8, type hints, and testing conventions
- **[Swift](swift.md)** - iOS/macOS development standards, optionals, protocol-oriented programming, SwiftUI, and memory management
- **[Kotlin](kotlin.md)** - Android development standards, coroutines, idiomatic Kotlin, null safety, and Jetpack Compose patterns
