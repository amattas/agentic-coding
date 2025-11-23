# Agentic Coding Workflows

A collection of AI-assisted coding workflows using Claude Code's subagent architecture for software development and security research.

## Background

This repository represents my attempt at consolidating different variations of two agentic workflows I've used over the last 6 months into something useful and reusable.

**This is Gen 3 of these workflows.** Over that time, I experimented with various approaches to AI-assisted coding: different agent structures, wave-based execution models, skills systems, and orchestration patterns. Some worked well, others didn't. What you see here is the distillation of those experiments into two distinct but structurally similar workflows that I've found effective in practice.

> **Status:** Still refining. Once I'm confident in the consolidation, I'll create a GitHub release. Until then, expect changes as I continue to iterate based on real-world usage.

The goal was to create something that:
- **Actually works** in real-world projects rather than just demos
- **Maximizes parallelism** where safe to do so
- **Tracks state** for restartability across sessions
- **Stays out of the way** for simple tasks but provides structure for complex ones
- **Can be customized** without rewriting everything from scratch

Both workflows share a common architecture (wave-based execution, specialized agents, reusable skills) but are tailored to their specific domains. They're designed for Claude Code but include `AGENTS.md` variants for other AI tools.

---

## The Two Workflows

### 1. Development Workflow (`development/`)

A complete Software Development Lifecycle (SDLC) workflow for building and maintaining software projects.

**What it does:**
- Guides AI through the full development cycle: context gathering, design, validation, implementation, and review
- Coordinates 27 specialized agents across 5 sequential waves
- Provides 9 reusable skills for tech stack, coding standards, testing, security, etc.
- Produces structured artifacts (specs, architecture docs, test plans) before writing code

**Wave Model:**
```
Wave A (Context)     → Wave B (Design)      → Wave C (Validate) → Wave D (Implement) → Wave E (Review)
─────────────────────────────────────────────────────────────────────────────────────────────────────
repo-scanner           spec-synthesizer       design-validator     component-impl-*     tester
dependency-mapper      arch-designer                               test-writer          security-scanner
test-coverage-baseline api-designer                                doc-writer           style-reviewer
performance-baseline   test-planner                                optimizer            pr-packager
                       security-designer
```

**When to use:**
- New features requiring design and planning
- Refactoring work where you need baselines first
- Multi-component changes that benefit from validation
- Any project where you want structured artifacts before coding

**When to skip the full workflow:**
- Simple bugfixes (< 50 lines, single file)
- Documentation updates
- Familiar code with straightforward changes

---

### 2. Security Workflow (`security/`)

A framework for AI-assisted security research, CTF challenges, binary exploitation, and malware analysis.

**What it does:**
- Guides AI through security analysis: reconnaissance, vulnerability analysis, exploitation, and documentation
- Coordinates specialized agents for binary scanning, reverse engineering, exploit development
- Provides skills for exploitation techniques (BOF, ROP, format strings), malware patterns, and tool usage
- Supports multi-problem lab/CTF environments with parallel agent spawning

**Wave Model:**
```
Wave A (Recon)      → Wave B (Analysis)       → Wave C (Exploit)     → Wave D (Docs)
────────────────────────────────────────────────────────────────────────────────────
binary-scanner        vulnerability-hunter      gadget-finder          writeup-writer
                      reverse-engineer          payload-crafter
                      crypto-analyzer           exploit-developer
                      malware-analyzer          exploit-tester
```

**When to use:**
- CTF competitions and challenges
- Authorized penetration testing
- Binary exploitation research
- Malware analysis (static analysis only)
- Security training and education

**Important:** This workflow is for authorized security testing only. It includes an authorization gate to verify the context is legitimate (CTF, lab, explicit permission).

---

## Common Architecture

Both workflows share these structural elements:

### Wave-Based Execution
Work flows through sequential phases (waves). Agents within a wave can often run in parallel, but waves must complete before proceeding. This balances speed with safety.

### Specialized Agents
Each agent has a focused role (scanner, designer, implementer, reviewer). Agents are defined in `.claude/agents/*.md` with specific tools, models, and instructions.

### Reusable Skills
Domain knowledge lives in `.claude/skills/*/SKILL.md` files. Agents reference skills rather than duplicating knowledge, keeping instructions DRY.

### STATUS.md for Restartability
Both workflows use `STATUS.md` files to track progress. When resuming work, the AI reads status first and continues from the last known state rather than starting over.

### Two Configuration Files
- `CLAUDE.md` - Full configuration for Claude Code (subagents, skills, wave orchestration)
- `AGENTS.md` - Simplified version for other AI tools (Cursor, Copilot, Cody, etc.)

---

## Directory Structure

```
agentic-coding/
├── README.md                 # This file
│
├── development/              # SDLC workflow
│   ├── CLAUDE.md             # Claude Code orchestrator config
│   ├── AGENTS.md             # Other AI tools config
│   ├── README.md             # Development workflow docs
│   ├── .claude/
│   │   ├── agents/           # 27 specialized agents
│   │   └── skills/           # 9 knowledge bases
│   ├── context/              # Generated context artifacts
│   └── templates/            # Artifact templates
│
└── security/                 # Security research workflow
    ├── CLAUDE.md             # Claude Code orchestrator config
    ├── AGENTS.md             # Other AI tools config
    ├── README.md             # Security workflow docs
    ├── .claude/
    │   ├── agents/           # Security-focused agents
    │   └── skills/           # Exploitation techniques, tools
    └── templates/            # Report and status templates
```

---

## Getting Started

### For Development Projects

1. Copy the `development/` directory contents to your project root (or symlink)
2. Customize `.claude/skills/` for your tech stack, coding standards, etc.
3. Start with Wave A if it's a new codebase or major feature
4. Let the workflow guide you through design → validation → implementation → review

### For Security Research / CTF

1. Copy the `security/` directory contents to your challenge/lab directory
2. Ensure you're in an authorized context (CTF, lab, explicit permission)
3. Start with Wave A (recon) for any new binary
4. Use `STATUS.md` to track progress across sessions

### Deployment Options

**Project-level (recommended):** Copy files to each project's root directory

**User-level:** Place in `~/.claude/` for personal defaults across all projects

**Global/Enterprise:** Deploy to system-wide locations for org standards

See individual workflow READMEs for detailed deployment instructions.

---

## Customization

### Adding Agents

Create `.claude/agents/my-agent.md`:

```markdown
---
name: my-agent
description: When to use this agent
tools: Read, Edit, Bash, Grep
model: sonnet
---

[Agent instructions here]
```

### Adding Skills

Create `.claude/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: What this skill provides
---

[Skill content here]
```

### Modifying Wave Structure

Edit `CLAUDE.md` to add/remove agents from waves or change the wave sequence.

---

## Lessons Learned

A few things I discovered while building these:

1. **Wave structure matters.** Early experiments without clear phase separation led to agents stepping on each other or skipping important steps.

2. **STATUS.md is essential.** Without persistent state, every session starts from scratch. The overhead of maintaining status pays off in session restartability.

3. **Skills beat prompt duplication.** Centralizing knowledge in skills files keeps agents consistent and makes updates easier.

4. **Parallelism needs guardrails.** Running everything in parallel sounds fast but causes conflicts. Wave boundaries provide natural checkpoints.

5. **Shortcuts for simple tasks.** Not everything needs the full workflow. The "shortcut path" for trivial changes avoids ceremony where it doesn't help.

6. **Two config files are worth it.** Claude Code's features (subagents, skills) don't translate to other tools. Having both `CLAUDE.md` and `AGENTS.md` covers more use cases.

---

## License

MIT

---

## Acknowledgments

Built with Claude Code. The workflows were developed and refined through many iterations of actual project work, CTF challenges, and experimentation with different agentic patterns.
