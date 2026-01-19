# Claude Orchestrator Configuration

## Purpose

This file defines how Claude orchestrates subagents and skills for the SDLC in this repository. Agents handle specific tasks; skills provide templates and domain knowledge.

## Quickstart

1. Read `STATUS.md` first to understand current state
2. Follow wave sequence: A → B → C → D → E
3. Use agents listed below for each wave
4. Agents auto-load their activity skills with templates
5. Update `STATUS.md` after completing any wave (see `restartability` skill)

**Quick decision:**
- New repo or unfamiliar codebase? → Start at Wave A
- Small bugfix in familiar code? → Skip to Wave D
- Major feature or refactor? → Full wave sequence

---

## Wave Model

### Wave A: Context Gathering

| Agent | Skill | Output |
|-------|-------|--------|
| `repo-scanner` | repo-scanning | `context/repo-map.md` |
| `dependency-mapper` | dependency-mapping | `context/dependency-graph.md` |
| `test-coverage-baseline` | — | `context/test-coverage-baseline.md` |
| `performance-baseline` | — | `context/perf-baseline.md` |
| `web-researcher` | — | `context/research.md` |

### Wave B: Design & Analysis

| Agent | Skill | Output |
|-------|-------|--------|
| `spec-synthesizer` | spec-synthesis | `spec.md` |
| `arch-designer` | architecture-design | `architecture.md` |
| `api-designer` | api-design | `api-design.md` |
| `test-planner` | test-planning | `test-plan.md` |
| `security-designer` | security-design | `security-requirements.md` |
| `performance-profiler` | — | performance report |

### Wave C: Design Validation

| Agent | Skill | Output |
|-------|-------|--------|
| `design-validator` | design-validation | `design-validation.md` |

### Wave D: Implementation

| Agent | Skills | Scope |
|-------|--------|-------|
| `component-impl-backend` | coding-standards, tech-stack | Backend code |
| `component-impl-frontend` | coding-standards, tech-stack | Frontend code |
| `component-impl-worker` | coding-standards, tech-stack | Workers/jobs |
| `test-writer` | testing-guidelines | Test files |
| `doc-writer` | documentation-standards | Documentation |
| `optimizer` | performance-principles | Performance fixes |
| `branch-manager` | commit-conventions | Git branches |

### Wave E: Review & Packaging

| Agent | Skill | Output |
|-------|-------|--------|
| `tester` | testing-guidelines | Test results |
| `style-reviewer` | code-review | `review-style.md` |
| `perf-reviewer` | code-review | `review-performance.md` |
| `security-scanner` | security-scanning | `security-findings.md` |
| `conflict-resolver` | — | `resolution.md` |
| `commit-packager` | commit-conventions | Atomic commits |
| `pr-packager` | pr-packaging | PR description |

---

## Project Structure

```
project/
├── CLAUDE.md           # This file
├── STATUS.md           # Progress tracking
├── .mcp.json           # MCP server configuration (GitHub, GPT models)
├── .claude/
│   ├── agents/         # Subagent definitions
│   └── skills/         # Activity skills with templates
├── context/            # Wave A outputs
├── spec.md             # Wave B
├── architecture.md     # Wave B
├── api-design.md       # Wave B
├── test-plan.md        # Wave B
└── design-validation.md # Wave C
```

## Global Norms

- Be truthful; avoid fabricating APIs, tools, or behavior
- Respect the wave sequence
- Follow priority order: Security > Correctness > Performance > Maintainability

## Context Optimization

This workflow is optimized for Claude Code's lazy loading:

- **Skills**: Loaded on-demand when agents are invoked (frontmatter only until execution)
- **Agents**: Frontmatter loaded initially; full context loaded when invoked via Task tool
- **Wave Sequence**: Reduces parallel context by executing agents progressively
- **MCP Tools**: Auto-deferred when descriptions exceed 10% of context (v2.1.7+)

**Best practices:**
- Invoke only necessary agents per wave
- Reference skills in agent frontmatter (auto-loaded when needed)
- Use wave sequence to avoid loading all agents simultaneously

## Model Selection Strategy

Agents are configured with specific models based on task complexity and requirements:

### Opus 4.5 (Planning & Complex Reasoning)
- **Wave B Design Agents**: spec-synthesizer, arch-designer, api-designer, test-planner, security-designer
- **Wave C Validation**: design-validator
- **Use when**: Complex reasoning, architectural decisions, specification synthesis required

### Sonnet 4.5 (Implementation & Analysis)
- **Wave D Implementation**: component-impl-backend, component-impl-frontend, component-impl-worker, test-writer, optimizer
- **Wave E Packaging**: conflict-resolver, pr-packager
- **Wave A Analysis**: performance-profiler (when detailed analysis required)
- **Use when**: Code implementation, detailed analysis, or moderate complexity tasks

### Haiku (Simple Tasks & Scans)
- **Wave A Context**: repo-scanner, dependency-mapper, test-coverage-baseline, performance-baseline
- **Wave D Simple Tasks**: doc-writer, branch-manager
- **Wave E Simple Tasks**: tester, commit-packager
- **Utility Agents**: analyzer, workflow-monitor
- **Use when**: Simple scanning, basic operations, quick reviews

### GPT-5.2 and GPT-5.2-codex (Code Review & Debugging)

#### Debugging Tasks
- **Preferred Model**: GPT-5.2 (if available via MCP)
- **First Fallback**: GPT-5.2-codex
- **Final Fallback**: Sonnet 4.5
- **Use when**: Diagnosing bugs, tracing execution, analyzing runtime behavior

#### Code Review Tasks
- **Wave E Code Review**: style-reviewer, perf-reviewer, security-scanner
- **Preferred Model**: GPT-5.2-codex (if available via MCP)
- **Fallback**: Sonnet 4.5
- **Use when**: Reviewing code style, performance, security vulnerabilities

**Note**: Model preferences are configured in each agent's frontmatter. Agents check for GPT model availability via MCP server first, falling back to Claude models if not present.

**MCP Configuration**: See `.mcp.json` in the project root for MCP server setup. This file includes stubs for GitHub integration and GPT model access (gpt-5.2 for debugging, gpt-5.2-codex for code review).

## When NOT to Over-Parallelize

- Multiple agents editing the same file
- One agent depends on another's reasoning (not just artifacts)
- Trivial edits where overhead outweighs benefit
- Changes < ~50 lines in a single file
- Tasks completable directly in < 2 minutes
