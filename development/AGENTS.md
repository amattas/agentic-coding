# AI Agent Development Guide

> **For general AI coding assistants.** This is a simplified guide for tools like Cursor, Copilot, Codeium, etc. If you're using **Claude Code**, see **[CLAUDE.md](./CLAUDE.md)** instead for advanced features (subagents, skills, parallel orchestration).

## Portability Notes

This guide is designed to work with any AI coding assistant. Tool-specific translations:

| Generic Operation | Claude Code | Cursor/Copilot | Codeium | Other |
|-------------------|-------------|----------------|---------|-------|
| Read file | `Read` tool | Built-in | Built-in | Use your file read capability |
| Search files | `Glob` tool | Built-in | Built-in | Use file search |
| Search content | `Grep` tool | Built-in | Built-in | Use content search |
| Edit file | `Edit` tool | Built-in | Built-in | Use your edit capability |
| Run command | `Bash` tool | Terminal | Terminal | Use shell access |

**If you're not Claude Code:**
- Ignore references to "subagents" and "skills" - these are Claude-specific
- Follow the wave model using your native capabilities
- Write artifacts to the file locations specified
- Use your tool's equivalent for file operations

---

## Quickstart

**If you're a human working with an AI assistant:**
1. Use `AGENTS.md` (this file) as the main reference for your AI tool
2. Follow the Wave model: Wave A → B → C → D → E
3. Always update `STATUS.md` before switching tasks or ending a session

**If you're a generic AI coding assistant:**
1. Read `STATUS.md` first if it exists
2. Follow the wave sequence for non-trivial changes
3. Write artifacts to `context/*.md` for analysis, root for design docs
4. Update `STATUS.md` after each major step

**Quick decision:**
- New repo? → Wave A (read code first)
- Small fix? → Skip to Wave D, update STATUS.md when done
- Multi-file feature? → Full wave sequence

---

This document provides guidelines for AI coding assistants working in this codebase.

## Project Structure

```
project/
├── STATUS.md           # Progress tracking (for restartability)
├── context/            # Context artifacts
│   └── repo-map.md
├── spec.md             # Requirements specification
├── architecture.md     # System design
├── api-design.md       # API/interface design
├── test-plan.md        # Test strategy
└── src/                # Source code
```

## Wave Selection Heuristics

Use this to decide where to start:

| Scenario | Start Wave | Skip? |
|----------|------------|-------|
| New repo or unfamiliar codebase | A | No skips |
| Major feature (multi-file) | A | No skips |
| Moderate feature (known patterns) | B | Skip A if context exists |
| Small bugfix | D | Skip A-C |
| One-liner or doc update | D | Shortcut path |

**Decision tree:**
1. New repo or significant structure change? → **Wave A**
2. Need new architecture/API design? → **Wave B**
3. Simple, low-risk change? → **Wave D**

## Development Workflow (Wave Model)

Follow this sequential workflow for non-trivial changes:

### Wave A: Context Gathering
- Read existing code before making changes
- Identify patterns, conventions, and dependencies
- Check for existing tests and documentation
- **Update STATUS.md** with initial findings

### Wave B: Design (for significant changes)
- Create `spec.md` with requirements and acceptance criteria
- Create `architecture.md` for component design
- Create `api-design.md` for interface definitions
- Create `test-plan.md` with test cases
- **Update STATUS.md** with design progress

### Wave C: Design Validation
- Cross-check spec vs architecture vs test plan
- Ensure all requirements are covered
- Identify gaps before implementation

### Wave D: Implementation
- Write tests first (TDD when appropriate)
- Implement in small, focused commits
- Follow existing code patterns and conventions
- Update documentation alongside code changes
- **Update STATUS.md** with implementation progress

### Wave E: Review & Verification
- Run all tests and ensure they pass
- Check for security issues (input validation, auth, secrets)
- Review for performance concerns
- Verify documentation is accurate
- **Update STATUS.md** to mark task complete

## Restartability (STATUS.md)

**Always maintain STATUS.md** to enable picking up work after interruptions.

### STATUS.md Structure
```markdown
# Development Status: [PROJECT NAME]
**Last Updated**: [timestamp]
**Tasks**: X completed / Y total

## Overview
| Task | Status | Wave | Notes |
|------|--------|------|-------|
| Feature A | ✅ Done | E | PR merged |
| Feature B | 🔄 In Progress | D | Writing tests |
| Bug fix C | ⏳ Blocked | B | Need clarification |

## Active Artifacts
| Artifact | Status |
|----------|--------|
| spec.md | ✅ Approved |
| architecture.md | 🔄 Draft |
| test-plan.md | ❌ Missing |

## Session History
### [Date]
- Completed: Feature A implementation
- Progress: Feature B tests 50% done
- Blocked: Bug fix C (need stakeholder input)
```

### When Resuming Work
1. Read STATUS.md to understand current state
2. Check artifact status for stale/missing docs
3. Review blocked tasks for resolution
4. Continue from last known good state

## Coding Standards

### General Principles
- Prefer small, focused functions
- Use early returns to avoid deep nesting
- Write comments to explain *why*, not *what*
- Follow existing naming conventions in the codebase

### Error Handling
- Use structured logging with consistent fields
- Never log secrets, tokens, or sensitive PII
- Handle errors at appropriate boundaries

### Testing
- Mirror source structure: `src/foo/bar.ts` → `tests/foo/bar.test.ts`
- Use descriptive test names reflecting behavior
- Follow Arrange-Act-Assert pattern
- Prefer factory functions over static fixtures

## Security Requirements

### Always
- Validate all untrusted input at boundaries
- Sanitize or encode output appropriately
- Use parameterized queries (never string concatenation for SQL)
- Store secrets in environment variables or secret managers

### Never
- Commit secrets, tokens, or credentials to code
- Log sensitive data (passwords, tokens, PII)
- Use custom password hashing (use established libraries)
- Trust client-side validation alone

### Common Vulnerabilities to Avoid
- SQL Injection
- Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)
- Insecure Direct Object References
- Security Misconfiguration

## Commit Conventions

Format: `<type>(<scope>): <short summary>`

Types:
- `feat` – new feature
- `fix` – bug fix
- `refactor` – code change without behavior change
- `docs` – documentation only
- `test` – adding or fixing tests
- `chore` – tooling, CI, config

Example: `feat(auth): add MFA to login flow`

## Priority Order

When recommendations conflict:
1. Security and safety
2. Correctness and data integrity
3. Performance and reliability
4. Maintainability and readability
5. Style preferences

## Artifact Templates

When creating design documents, use these structures:

### spec.md
- Problem Statement
- Functional Requirements (numbered)
- Non-Functional Requirements
- Acceptance Criteria
- Out of Scope
- Dependencies & Risks

### architecture.md
- Overview
- Components (with responsibilities)
- Data Flow
- Control Flow
- Boundaries & Integrations
- Trade-offs & Decisions

### test-plan.md
- Scope
- Test Categories (unit, integration, e2e)
- Test Cases (happy path, edge cases, failures)
- Data & Fixtures
- Exit Criteria

## Quick Reference

**Before writing code:**
1. Read the relevant existing code
2. Understand the patterns in use
3. Check for existing tests

**When implementing:**
1. Write/update tests first
2. Make small, focused changes
3. Run tests frequently
4. Commit logical chunks

**Before committing:**
1. All tests pass
2. No security issues introduced
3. Documentation updated if needed
4. Commit message follows conventions
