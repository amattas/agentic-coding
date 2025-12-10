# Claude Orchestrator Configuration

@include AGENTS.md

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
- Small bugfix in familiar code? → May skip to Wave D (see Wave Selection Heuristics in AGENTS.md)
- Major feature or refactor? → Full wave sequence required

---

## Purpose

This file defines how Claude should orchestrate subagents and skills for the SDLC in this repository. It specifies the wave-based workflow, global norms, and how to route work to agents. Detailed technical knowledge and role-specific prompts live in skills and agent definition files, not here.

## Claude-Specific Project Structure

```
project/
├── CLAUDE.md           # This file (orchestrator config)
├── AGENTS.md           # For other AI tools (shared content)
├── STATUS.md           # Progress tracking for restartability
├── .claude/            # Agents and skills
│   ├── agents/         # Subagent definitions
│   └── skills/         # Domain knowledge
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

## Wave Model: Agent Assignments

### Wave A – Context Gathering

**Primary agents:**
- `repo-scanner` (**CORE**)
- `dependency-mapper` (if dependencies are non-trivial)
- `test-coverage-baseline` (if refactoring or coverage unknown)
- `performance-baseline` (if performance-sensitive work)
- `web-researcher` (if unfamiliar libraries/APIs/patterns involved)

### Wave B – Design & Analysis

**Primary agents:**
- `spec-synthesizer` (**CORE**) — Must run first
- `arch-designer` (**CORE**) — For architecture decisions
- `api-designer` (if APIs/interfaces involved)
- `test-planner` (**CORE**) — Test strategy
- `security-designer` (if auth/data/external exposure)
- `performance-profiler` (if performance matters)

### Wave C – Design Validation

**Primary agents:**
- `design-validator` (**CORE**)

### Wave D – Implementation

**Primary agents:**
- `component-impl-backend` / `component-impl-frontend` / `component-impl-worker` (**CORE**)
- `test-writer` (**CORE**)
- `doc-writer` (for documentation updates)
- `optimizer` (if performance requirements exist)
- `branch-manager` (for feature branches)

### Wave E – Review & PR Packaging

**Primary agents (parallel):**
- `tester` (**CORE**) — Run tests, report coverage gaps
- `security-scanner` (if security-sensitive changes)
- `style-reviewer` (code style check)
- `perf-reviewer` (if performance-sensitive)
- `conflict-resolver` (if reviews conflict)

**Git agents:**
- `commit-packager` (**CORE**) — Atomic commits
- `pr-packager` (**CORE**) — PR description

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
