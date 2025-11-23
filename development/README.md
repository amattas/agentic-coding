# Claude Code SDLC: Parallel Subagent Architecture

A complete SDLC workflow for Claude Code using subagents, skills, and a wave-based execution model designed to maximize safe parallelism.

## Overview

This framework provides a structured approach to software development using Claude Code's agent capabilities. It organizes work into five sequential waves, each with specialized agents that can run in parallel within their wave.

## Configuration Files

This framework uses two different configuration files depending on the AI tool:

| File | Tool | Features |
|------|------|----------|
| `CLAUDE.md` | Claude Code | Full support for subagents, skills, wave-based execution, and advanced orchestration |
| `AGENTS.md` | Other AI tools (Cursor, Copilot, Cody, etc.) | Consolidated, self-contained guide without tool-specific features |

### Why Two Files?

**CLAUDE.md** leverages Claude Code's unique capabilities:
- Subagent spawning and parallel execution
- Skills system for reusable knowledge
- Wave-based workflow orchestration
- Tool-specific directives (MUST USE, USE PROACTIVELY, etc.)

**AGENTS.md** provides a simpler alternative for tools that:
- Don't support subagents or skills
- Need a single, self-contained instruction file
- Work better with consolidated, linear guidelines

## Wave Model

```
Wave A → Wave B → Wave C → Wave D → Wave E
```

| Wave | Purpose | Key Agents |
|------|---------|------------|
| **A** | Context Gathering | repo-scanner, dependency-mapper, test-coverage-baseline, performance-baseline |
| **B** | Design & Analysis | spec-synthesizer, arch-designer, api-designer, test-planner, analyzer, security-designer, performance-profiler |
| **C** | Design Validation | design-validator |
| **D** | Implementation | component-impl-backend, component-impl-frontend, component-impl-worker, test-writer, doc-writer, optimizer |
| **E** | Review & PR Packaging | tester, security-scanner, style-reviewer, perf-reviewer, conflict-resolver, pr-packager |

## Directory Structure

```
# Root-level configuration
CLAUDE.md                        # Claude Code configuration (advanced features)
AGENTS.md                        # Other AI tools configuration (consolidated)

.claude/
├── agents/                      # Agent definitions (27 agents)
│   ├── repo-scanner.md
│   ├── spec-synthesizer.md
│   ├── design-validator.md
│   ├── component-impl-*.md
│   ├── tester.md
│   └── ...
└── skills/                      # Reusable knowledge bases
    ├── tech-stack/
    ├── coding-standards/
    ├── testing-guidelines/
    ├── documentation-standards/
    ├── performance-principles/
    ├── security-baseline/
    ├── project-context/
    ├── domain-glossary/
    └── commit-conventions/

context/                         # Generated context artifacts
├── repo-map.md
├── dependency-graph.md
├── test-coverage-baseline.md
└── perf-baseline.md

templates/                       # Artifact templates
├── spec.md
├── architecture.md
├── api-design.md
├── test-plan.md
├── security-requirements.md
├── design-validation.md
├── security-findings.md
├── review-style.md
├── review-performance.md
└── resolution.md
```

## Quick Start

1. **Customize Skills**: Update the skill files in `.claude/skills/` to match your project's tech stack, coding standards, and domain terminology.

2. **Start with Context (Wave A)**: When beginning significant work, run the context agents to build baseline understanding:
   - `repo-scanner` → produces `context/repo-map.md`
   - `dependency-mapper` → produces `context/dependency-graph.md`
   - `test-coverage-baseline` → produces `context/test-coverage-baseline.md`

3. **Design Before Implementing (Wave B)**: Use design agents to create structured artifacts:
   - `spec-synthesizer` → produces `spec.md`
   - `arch-designer` → produces `architecture.md`
   - `api-designer` → produces `api-design.md`
   - `test-planner` → produces `test-plan.md`

4. **Validate Design (Wave C)**: Run `design-validator` to cross-check all design artifacts before implementation.

5. **Implement (Wave D)**: Use implementation agents scoped to specific areas:
   - Backend, frontend, or worker components
   - Tests based on test-plan.md
   - Documentation updates

6. **Review & Package (Wave E)**: Run review agents in parallel, resolve conflicts, and package the PR.

## Agent Types

### Context Agents (Wave A)
Read-only agents that scan and analyze the codebase to produce context artifacts. Run once per session or major change.

### Design Agents (Wave B)
Transform requirements into structured design documents. Can run in parallel after initial spec is created.

### Validation Agents (Wave C)
Cross-check design artifacts for consistency and completeness before implementation.

### Implementation Agents (Wave D)
Write code, tests, and documentation. Scoped to specific directories to enable parallelism.

### Review Agents (Wave E)
Analyze code changes for security, style, and performance issues. Run in parallel over the same diff.

### Git Workflow Agents
- `branch-manager`: Creates and prepares feature branches
- `commit-packager`: Creates atomic commits with conventional messages
- `workflow-monitor`: Diagnoses slow or failed workflows

## Skills Reference

| Skill | Purpose |
|-------|---------|
| `tech-stack` | Languages, frameworks, and conventions |
| `coding-standards` | Code style and structural conventions |
| `testing-guidelines` | Test structure, naming, and coverage expectations |
| `documentation-standards` | Doc types, style, and maintenance rules |
| `performance-principles` | Performance guidelines by layer |
| `security-baseline` | Security requirements and controls |
| `project-context` | Business goals and constraints |
| `domain-glossary` | Domain-specific terminology |
| `commit-conventions` | Commit message format |

## Priorities

When recommendations conflict, follow this priority order:

1. Security and safety (including regulatory/compliance)
2. Correctness and data integrity
3. Performance and reliability
4. Maintainability and readability
5. Minor style preferences

## When NOT to Parallelize

- Multiple agents editing the same file
- One agent depends on another's reasoning (not just artifacts)
- Trivial edits where subagent overhead exceeds benefit

## Customization

### Adding New Agents

Create a new `.md` file in `.claude/agents/` with frontmatter:

```markdown
---
name: my-agent
description: When to USE this agent
tools: Read, Edit, Bash, Grep
model: sonnet
---

[Agent instructions here]
```

### Adding New Skills

Create a new directory in `.claude/skills/` with a `SKILL.md` file:

```markdown
---
name: my-skill
description: What this skill provides
---

[Skill content here]
```

## Deployment Locations

Configuration files can be deployed at three different scopes, listed in order of precedence (highest to lowest):

### Project-Level (Recommended)

Place files in your project's root directory. These settings apply only to that project.

```
your-project/
├── CLAUDE.md              # Claude Code instructions for this project
├── AGENTS.md              # Other AI tools instructions for this project
├── .claude/
│   ├── agents/            # Project-specific agents
│   └── skills/            # Project-specific skills
└── ...
```

**Best for:** Project-specific coding standards, tech stack details, domain terminology, and custom workflows.

### User-Level

Place files in your home directory. These settings apply to all your projects (unless overridden by project-level).

| Platform | Location |
|----------|----------|
| macOS/Linux | `~/.claude/CLAUDE.md` |
| Windows | `%USERPROFILE%\.claude\CLAUDE.md` |

```
~/.claude/
├── CLAUDE.md              # Your personal Claude Code preferences
├── agents/                # Your personal agents
└── skills/                # Your personal skills
```

**Best for:** Personal preferences, common workflows, and settings you want across all projects.

### Global/Enterprise-Level

For organization-wide settings, deploy via:

| Platform | Location |
|----------|----------|
| macOS | `/Library/Application Support/claude-code/CLAUDE.md` |
| Linux | `/etc/claude-code/CLAUDE.md` |
| Windows | `%PROGRAMDATA%\claude-code\CLAUDE.md` |

**Best for:** Organization-wide security policies, compliance requirements, and standard practices.

### Precedence Rules

When the same setting exists at multiple levels:

1. **Project-level** overrides all others
2. **User-level** overrides global
3. **Global-level** is the default

Settings are merged, not replaced entirely. A project can extend user/global settings while overriding specific values.

### AGENTS.md Deployment

For other AI tools using `AGENTS.md`:

| Tool | Typical Location |
|------|------------------|
| Cursor | Project root as `AGENTS.md` or `.cursor/rules` |
| GitHub Copilot | Project root as `AGENTS.md` or `.github/copilot-instructions.md` |
| Cody | Project root as `AGENTS.md` or `.sourcegraph/cody.md` |
| Generic | Project root as `AGENTS.md` |

Most AI coding tools look for instruction files in the project root. Check your specific tool's documentation for supported locations.

## License

[Your license here]
