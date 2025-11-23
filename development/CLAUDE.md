# Claude Orchestrator Configuration

> **For Claude Code users only.** This file uses Claude Code's advanced features (subagents, skills, wave-based orchestration). If you're using a different AI coding tool (Cursor, Copilot, Codeium, etc.), see **[AGENTS.md](./AGENTS.md)** instead for a simplified guide.

---

## Quickstart

**If you're a human using Claude Code:**
1. Start in **Wave A** (context gathering) for any significant new work
2. Use the agents listed under each wave section below
3. Always update `STATUS.md` before moving between waves
4. Check `context/*.md` artifacts exist before proceeding to Wave B

**If you're Claude Code (the orchestrator):**
1. Read `STATUS.md` first to understand current state
2. Follow the wave sequence: A → B → C → D → E
3. Use subagents from `.claude/agents/` for specialized work
4. Reference skills from `.claude/skills/` for domain knowledge
5. Update `STATUS.md` after completing any wave or significant milestone

**Quick decision:**
- New repo or unfamiliar codebase? → Start at Wave A
- Small bugfix in familiar code? → May skip to Wave D (see Wave Selection Heuristics)
- Major feature or refactor? → Full wave sequence required

---

## Purpose

This file defines how Claude should orchestrate subagents and skills for the SDLC in this repository. It specifies the wave-based workflow, global norms, and how to route work to agents. Detailed technical knowledge and role-specific prompts live in skills and agent definition files, not here.

## Project Structure

```
project/
├── CLAUDE.md           # This file (orchestrator config)
├── AGENTS.md           # For other AI tools
├── STATUS.md           # Progress tracking for restartability
├── .claude/            # Agents and skills
│
├── context/            # Context artifacts from Wave A
│   ├── repo-map.md
│   ├── dependency-graph.md
│   ├── test-coverage-baseline.md
│   └── perf-baseline.md
│
├── spec.md             # Requirements specification
├── architecture.md     # System design
├── api-design.md       # API/interface design
├── test-plan.md        # Test strategy
├── security-requirements.md  # Security controls (if needed)
└── design-validation.md      # Validation results
```

**Key file for restartability:**
- `STATUS.md`: Tracks task status, active artifacts, agent activity, and session history

## Restartability Protocol

When resuming work:

1. **Read STATUS.md first**
   - What tasks are done? Skip those
   - What's in progress? Resume from last state
   - What's blocked? Note blockers and skip

2. **Check artifact status**
   - Which context artifacts exist and are current?
   - Which design artifacts are complete vs draft?
   - What's missing that needs to be created?

3. **Resume from last known good state**
   - Don't re-run completed waves
   - Use existing artifacts as inputs
   - Continue from the current wave for each task

4. **Update STATUS.md frequently**
   - After completing any wave
   - After creating/updating artifacts
   - After any blocker or breakthrough
   - At session start and end

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

## Agent Priority

| Agent | Wave | Priority | When to use |
|-------|------|----------|-------------|
| `repo-scanner` | A | **CORE** | New repo, unfamiliar codebase, or major changes |
| `dependency-mapper` | A | OPTIONAL | Complex dependencies, microservices, monorepos |
| `test-coverage-baseline` | A | OPTIONAL | Before refactoring or if coverage is unknown |
| `performance-baseline` | A | OPTIONAL | Performance-sensitive features only |
| `web-researcher` | A/B | OPTIONAL | Research docs, libraries, patterns before implementing |
| `spec-synthesizer` | B | **CORE** | Any non-trivial feature or ambiguous requirements |
| `arch-designer` | B | **CORE** | Multi-component features, new patterns |
| `api-designer` | B | OPTIONAL | Public APIs, external contracts, service interfaces |
| `test-planner` | B | **CORE** | Any feature that needs tests |
| `security-designer` | B | OPTIONAL | Auth, data handling, external exposure |
| `performance-profiler` | B | OPTIONAL | Performance-sensitive designs |
| `design-validator` | C | **CORE** | Before any major implementation work |
| `component-impl-*` | D | **CORE** | Actual implementation |
| `test-writer` | D | **CORE** | Writing tests |
| `doc-writer` | D | OPTIONAL | Documentation updates |
| `optimizer` | D | OPTIONAL | Performance-sensitive implementation |
| `branch-manager` | D | OPTIONAL | Feature branch management |
| `tester` | E | **CORE** | Before marking any task complete |
| `security-scanner` | E | OPTIONAL | Security-sensitive changes |
| `commit-packager` | E | **CORE** | All commits |
| `pr-packager` | E | **CORE** | All PRs |

**CORE** = Always use for appropriate work. **OPTIONAL** = Use when context requires it.

## Global Norms

- Be truthful and avoid fabricating APIs, tools, or behavior.
- Prefer using subagents and skills when they are a better fit than doing everything in a single context.
- Respect the wave model:
  - Wave A → Wave B → Wave C → Wave D → Wave E
- Respect security and safety constraints from the `security-baseline` skill.

## Wave Model

### Wave A – Context Gathering

When starting significant work (new feature, major refactor):

**Primary agents:**
- `repo-scanner` (**CORE**)
- `dependency-mapper` (if dependencies are non-trivial)
- `test-coverage-baseline` (if refactoring or coverage unknown)
- `performance-baseline` (if performance-sensitive work)
- `web-researcher` (if unfamiliar libraries/APIs/patterns involved)

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

### Wave B – Design & Analysis

When requirements or tickets are ambiguous or non-trivial:

**Primary agents:**
- `spec-synthesizer` (**CORE**) — Must run first
- `arch-designer` (**CORE**) — For architecture decisions
- `api-designer` (if APIs/interfaces involved)
- `test-planner` (**CORE**) — Test strategy
- `security-designer` (if auth/data/external exposure)
- `performance-profiler` (if performance matters)

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

### Wave C – Design Validation

Before writing significant new code:

**Primary agents:**
- `design-validator` (**CORE**)

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

**If validation fails:** Re-run relevant Wave B agents, then re-validate.

### Wave D – Implementation

For actual feature or bugfix implementation:

**Primary agents:**
- `component-impl-backend` / `component-impl-frontend` / `component-impl-worker` (**CORE**)
- `test-writer` (**CORE**)
- `doc-writer` (for documentation updates)
- `optimizer` (if performance requirements exist)
- `branch-manager` (for feature branches)

**Required artifacts:**
- Source code changes (in appropriate directories)
- Test implementations (mirroring source structure)
- Updated documentation (if behavior changed)
- `STATUS.md` updated with implementation progress

**Wave D rules:**
- Keep write-capable agents scoped to specific directories/file types to avoid conflicts
- Implement tests defined in `test-plan.md`
- Update docs in parallel with code changes

**Wave D complete when:**
- [ ] All specified functionality implemented
- [ ] Tests written for new code
- [ ] Documentation updated
- [ ] `STATUS.md` reflects implementation status

### Wave E – Review & PR Packaging

Once implementation and tests are in place:

**Primary agents (parallel):**
- `tester` (**CORE**) — Run tests, report coverage gaps
- `security-scanner` (if security-sensitive changes)
- `style-reviewer` (code style check)
- `perf-reviewer` (if performance-sensitive)
- `conflict-resolver` (if reviews conflict)

**Git agents:**
- `commit-packager` (**CORE**) — Atomic commits
- `pr-packager` (**CORE**) — PR description

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

## Skills Usage

When performing any task, prefer to use the appropriate skills:

- `project-context` for understanding goals and stakeholders.
- `tech-stack` and `coding-standards` for implementation.
- `testing-guidelines` for tests.
- `documentation-standards` for docs.
- `performance-principles` for perf-sensitive work.
- `security-baseline` for anything involving data, auth, or external exposure.
- `domain-glossary` for domain-specific terminology.
- `commit-conventions` for commit messages.

Do not duplicate skills content in prompts; instead, reference skills and follow their guidance.

## Priorities

When recommendations or goals conflict:

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

## When Not to Over-Parallelize

- Do not parallelize when multiple agents would be editing the same file.
- Do not parallelize when one agent depends on another's reasoning rather than just its output artifacts.
- Do not spawn subagents for trivial or tiny edits where the overhead outweighs the benefit.

**Additional anti-spawning rules:**
- Do not create new subagents just to rephrase or restate existing artifacts
- Prefer reusing `arch-designer` + `api-designer` outputs instead of re-deriving architecture ad hoc
- If a change touches only a single file and is < ~50 lines, do not involve multiple agents
- If `context/repo-map.md` exists and is current, do not re-run `repo-scanner`
- Never spawn agents for tasks you can do directly in < 2 minutes

Use judgment to balance speed with clarity and stability.
