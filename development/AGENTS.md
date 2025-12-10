# AI Agent Development Guide

## Project Structure

```
project/
├── STATUS.md           # Progress tracking (for restartability)
├── context/            # Context artifacts
│   ├── repo-map.md
│   ├── dependency-graph.md
│   ├── test-coverage-baseline.md
│   └── perf-baseline.md
├── spec.md             # Requirements specification
├── architecture.md     # System design
├── api-design.md       # API/interface design
├── test-plan.md        # Test strategy
├── security-requirements.md  # Security controls (if needed)
├── design-validation.md      # Validation results
└── src/                # Source code
```

## Wave Selection Heuristics

Use this to decide where to start:

| Scenario | Start Wave | Skip? | Notes |
|----------|------------|-------|-------|
| New repo or unfamiliar codebase | A | No skips | Must build context first |
| Major feature (multi-file, new patterns) | A | No skips | Full wave sequence |
| Moderate feature (known patterns) | B | Skip A if context exists | Use existing `context/*.md` |
| Small bugfix in familiar code | D | Skip A-C | Direct implementation |
| One-liner fix or doc update | D | Skip A-C | Shortcut path (see below) |
| Refactoring existing code | A | No skips | Need test baseline first |
| Performance-sensitive work | A | No skips | Need perf baseline |

**Decision tree:**
1. Is this a new repo or has structure changed significantly? → **Wave A**
2. Is `context/repo-map.md` current and sufficient? → If no, **Wave A**
3. Does this require new architecture or API design? → **Wave B**
4. Is this a simple, low-risk change? → **Wave D** (Shortcut Path)

## Global Norms

- Be truthful and avoid fabricating APIs, tools, or behavior.
- Respect the wave model: Wave A → Wave B → Wave C → Wave D → Wave E
- Respect security and safety constraints.

## Development Workflow (Wave Model)

Follow this sequential workflow for non-trivial changes:

### Wave A: Context Gathering

When starting significant work (new feature, major refactor):

**Activities:**
- Read existing code before making changes
- Identify patterns, conventions, and dependencies
- Map inter-module dependencies (if complex)
- Check test coverage metrics (if refactoring or coverage unknown)
- Establish performance baseline (if performance-sensitive work)
- Research unfamiliar libraries/APIs/patterns

**Required artifacts:**
- `context/repo-map.md` — Repository structure and patterns
- `context/dependency-graph.md` — Inter-module dependencies (if complex)
- `context/test-coverage-baseline.md` — Test coverage metrics (if needed)
- `context/perf-baseline.md` — Performance baseline (if needed)
- `context/research.md` — Library/pattern research findings (if needed)
- `STATUS.md` updated with summary and known risks

**Wave A complete when:**
- [ ] `context/repo-map.md` exists and is current
- [ ] Dependencies are understood (documented or trivial)
- [ ] Test coverage is known (documented or acceptable)
- [ ] `STATUS.md` reflects findings and risks

### Wave B: Design & Analysis

When requirements or tickets are ambiguous or non-trivial:

**Activities:**
- Synthesize requirements and acceptance criteria
- Design architecture and component structure
- Define API/interface specifications (if APIs involved)
- Plan test strategy and cases
- Identify security requirements (if auth/data/external exposure)
- Profile performance concerns (if performance matters)

**Required artifacts:**
- `spec.md` — Requirements and acceptance criteria
- `architecture.md` — Component design and data flow
- `api-design.md` — Interface specifications (if APIs involved)
- `test-plan.md` — Test strategy and cases
- `security-requirements.md` — Security controls (if security-sensitive)
- `STATUS.md` updated with design progress

**Wave B complete when:**
- [ ] `spec.md` has clear requirements and acceptance criteria
- [ ] `architecture.md` documents components and data flow
- [ ] `test-plan.md` covers happy path, edge cases, failures
- [ ] Security requirements documented (if applicable)
- [ ] `STATUS.md` reflects design decisions

### Wave C: Design Validation

Before writing significant new code:

**Activities:**
- Cross-check spec vs architecture vs test plan
- Ensure all requirements are covered
- Identify gaps before implementation

**Required artifacts:**
- `design-validation.md` — Cross-check results

**Validates:**
- `spec.md` ↔ `architecture.md` consistency
- `architecture.md` ↔ `api-design.md` consistency
- `test-plan.md` covers all requirements
- `security-requirements.md` addressed in design (if present)

**Wave C complete when:**
- [ ] `design-validation.md` shows no critical issues
- [ ] All design artifacts are internally consistent
- [ ] If issues found: resolved and re-validated

**If validation fails:** Re-run relevant Wave B activities, then re-validate.

### Wave D: Implementation

For actual feature or bugfix implementation:

**Activities:**
- Write tests first (TDD when appropriate)
- Implement in small, focused commits
- Follow existing code patterns and conventions
- Update documentation alongside code changes

**Required artifacts:**
- Source code changes (in appropriate directories)
- Test implementations (mirroring source structure)
- Updated documentation (if behavior changed)
- `STATUS.md` updated with implementation progress

**Wave D rules:**
- Keep changes scoped to specific directories/file types to avoid conflicts
- Implement tests defined in `test-plan.md`
- Update docs in parallel with code changes

**Wave D complete when:**
- [ ] All specified functionality implemented
- [ ] Tests written for new code
- [ ] Documentation updated
- [ ] `STATUS.md` reflects implementation status

### Wave E: Review & PR Packaging

Once implementation and tests are in place:

**Activities:**
- Run all tests and ensure they pass
- Check for security issues (input validation, auth, secrets)
- Review for performance concerns
- Verify documentation is accurate
- Create atomic commits
- Prepare PR description

**Required artifacts:**
- All tests passing
- `STATUS.md` updated to mark task complete
- PR description with risks, testing notes, follow-up actions

**Wave E complete when:**
- [ ] All tests pass
- [ ] No critical security issues
- [ ] Code style reviewed
- [ ] Commits are atomic and well-described
- [ ] PR is ready for review
- [ ] `STATUS.md` marked complete

## Restartability (STATUS.md)

**Always maintain STATUS.md** to enable picking up work after interruptions.

```markdown
# Status: [PROJECT NAME]
**Updated**: [timestamp] | **Tasks**: X/Y complete

## Tasks
| Task | Status | Wave | Notes |
|------|--------|------|-------|
| Feature A | ✅ Done | E | PR merged |
| Feature B | 🔄 Active | D | Writing tests |
| Bug fix C | ⏳ Blocked | B | Need input |

## Artifacts
| File | Status |
|------|--------|
| spec.md | ✅ Done |
| architecture.md | 🔄 Draft |
| test-plan.md | ❌ Missing |

## Session Log
- Completed: [what]
- In progress: [what]
- Blocked: [what + why]
```

When resuming: read STATUS.md first, skip completed work, resume from last good state, update frequently.

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

1. Security and safety (including regulatory/compliance)
2. Correctness and data integrity
3. Performance and reliability
4. Maintainability and readability
5. Minor style preferences

## Shortcut Path: Trivial Changes

For very small, low-risk changes (one-liner bugfix, doc comment, simple refactor):

1. **Skip Waves A–C entirely**
2. Edit the file directly
3. Run tests relevant to the change
4. Update `STATUS.md` with:
   - What changed
   - Tests run
   - Any follow-up needed

**Use shortcut path when:**
- Change is < 50 lines in a single file
- No architectural impact
- No new dependencies
- Tests are straightforward
- You understand the surrounding code

**Do NOT use shortcut path when:**
- Refactoring code you haven't analyzed
- Touching security-sensitive code
- Change affects multiple components
- Tests are missing or unclear

## Documentation Requirements

**All design documents (spec.md, architecture.md, api-design.md) must use Mermaid diagrams** for visualizing:
- Component relationships
- Data flows
- Sequence diagrams
- State machines

Keep diagrams simple and focused. Prefer multiple small diagrams over one complex one.
