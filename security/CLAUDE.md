# Security Research & CTF Assistant

@include AGENTS.md

---

## Quickstart

**If you're a human using Claude Code:**
1. Ensure this is authorized testing (CTF, lab, or explicit permission)
2. Start in **Wave A** (reconnaissance) for any new binary or challenge
3. Use the agents listed under each wave section below
4. Always update `STATUS.md` before moving between waves
5. For multi-problem labs, use root `STATUS.md` for overview, per-problem `STATUS.md` for details

**If you're Claude Code (the orchestrator):**
1. **Verify authorization first** - if unclear, ask the user to confirm CTF/lab/authorized context
2. Read `STATUS.md` to understand current state
3. Follow the wave sequence: A (recon) → B (analysis) → C (exploit) → D (docs)
4. Use subagents from `.claude/agents/` for specialized work
5. Reference skills from `.claude/skills/` for exploitation techniques
6. Update `STATUS.md` after each wave or significant finding

**Quick decision:**
- New binary? → Always start at Wave A (checksec, file, strings)
- Obvious `win()` function? → May skip gadget-finder, go straight to exploit-developer
- Complex protections? → Full wave sequence with gadget-finder and payload-crafter

---

## Purpose

This file defines how Claude Code should orchestrate subagents and skills for security research, CTF challenges, binary exploitation, and malware analysis. Detailed knowledge lives in skills and agent definition files.

## Claude-Specific Project Structure

### Multi-Problem Lab/CTF Layout (Primary)

When a CTF or lab has multiple problems, use this structure:

```
lab-name/
├── CLAUDE.md           # This file (orchestrator config)
├── AGENTS.md           # For other AI tools (shared content)
├── STATUS.md           # HIGH-LEVEL summary of ALL problems
├── .claude/            # Agents and skills (symlink or copy)
│   ├── agents/         # Subagent definitions
│   └── skills/         # Exploitation techniques
│
├── problem1/
│   ├── README.md       # Problem description (from problem-README template)
│   ├── STATUS.md       # DETAILED tracking for this problem
│   ├── target          # Binary
│   ├── target.asm      # Disassembly
│   ├── exploit.py      # Working exploit
│   └── REPORT.md       # Writeup (when solved)
│
├── problem2/
│   ├── README.md
│   ├── STATUS.md
│   └── ...
│
└── problemN/
    └── ...
```

### Single Challenge Layout

For standalone challenges:

```
challenge/
├── CLAUDE.md           # Challenge-specific context
├── AGENTS.md           # Simplified instructions for other tools
├── STATUS.md           # Progress tracking
├── target              # Binary being analyzed/exploited
├── target.asm          # Disassembly (objdump -M intel -d target)
├── exploit.py          # Working exploit script
├── find.py             # Address discovery script (if needed)
├── REPORT.md           # Technical writeup
└── examples/           # Reference exploits (optional, if provided)
```

### Malware Analysis Layout

```
analysis/
├── CLAUDE.md           # Analysis context
├── SUMMARY.md          # Complete analysis documentation
├── sample.bin          # Malware sample (DO NOT EXECUTE)
├── disassembly/        # Disassembly outputs
├── decompiled/         # Decompiled source
├── extracted/          # Extracted payloads/artifacts
├── scripts/            # Analysis scripts
└── .mcp.json           # MCP server config (pyghidra-mcp, etc.)
```

## Agent Priority

| Agent | Wave | Priority | When to use |
|-------|------|----------|-------------|
| `binary-scanner` | A | **CORE** | First look at any new binary |
| `security-researcher` | A/B | OPTIONAL | Research CVEs, techniques, tool docs, prior writeups |
| `vulnerability-hunter` | B | **CORE** | Identify vulnerability class |
| `reverse-engineer` | B | **CORE** | When behavior is unclear |
| `crypto-analyzer` | B | OPTIONAL | Encryption/obfuscation present |
| `malware-analyzer` | B | OPTIONAL | Malware samples only |
| `gadget-finder` | C | OPTIONAL | Only when ROP/SROP is needed |
| `payload-crafter` | C | OPTIONAL | Custom shellcode needed |
| `exploit-developer` | C | **CORE** | Building working exploit |
| `exploit-tester` | C | **CORE** | Validating exploit works |
| `writeup-writer` | D | **CORE** | After successful or partial solution |
| `lab-orchestrator` | * | **CORE** | Multi-problem labs only |

**CORE** = Always use for appropriate work. **OPTIONAL** = Use when context requires it.

## Wave Model: Agent Assignments

### Wave A – Reconnaissance

When starting any security challenge or analysis:

**Primary agents:**
- `binary-scanner` (**CORE**)
- `security-researcher` (if researching CVEs, techniques, or prior writeups)

**Produces:**
- `context/binary-info.md` with architecture, file type, security mitigations, initial strings and symbols

### Wave B – Analysis

After initial recon, in parallel where possible:

**Primary agents:**
- `vulnerability-hunter` (**CORE**) — Identify potential vulnerabilities
- `reverse-engineer` (**CORE**) — For complex code understanding
- `crypto-analyzer` (if encryption/obfuscation is present)
- `malware-analyzer` (for malware samples instead of above)

**Produces:**
- `context/vulnerability-analysis.md`
- `context/reverse-engineering.md`
- `context/crypto-analysis.md`
- `context/research.md` (if external research was needed)

### Wave C – Exploitation Development

Once vulnerabilities are identified:

**Primary agents:**
- `gadget-finder` (if ROP is needed)
- `payload-crafter` (for custom shellcode)
- `exploit-developer` (**CORE**) — Create working exploit
- `exploit-tester` (**CORE**) — Validate exploit works

**Produces:**
- `context/gadgets.md`
- `exploit.py`
- `find.py` (if needed)

### Wave D – Documentation

After successful exploitation:

**Primary agents:**
- `writeup-writer` (**CORE**)

**Produces:**
- `REPORT.md`
- `STATUS.md` updated with final status

## Skills Usage

When performing any task, USE the appropriate skills:

- `exploitation-techniques` for BOF, ROP, format string, canary, GOT patterns
- `vm-interpreter` for custom interpreter/VM exploitation
- `logic-vulnerabilities` for FD abuse and TOCTOU race conditions
- `side-channels` for timing attacks and oracles
- `binary-analysis` for analysis tools and techniques
- `malware-patterns` for malware behaviors and IOCs
- `tool-usage` for command reference (pwntools, GDB, etc.)
- `reporting-standards` for documentation format

Reference skills directly rather than duplicating their content.

## Multi-Problem Orchestration

When working on a lab/CTF with multiple problems, maximize parallelism:

### Startup Sequence

1. **Read root `STATUS.md`** to understand current state
2. **Identify parallelizable work**:
   - Independent problems can be worked in parallel
   - Different phases of different problems can overlap
3. **Spawn parallel agents** for each independent problem

### Parallel Agent Strategy

```
For a lab with problems A, B, C, D:

IF all are new/not-started:
  - Spawn binary-scanner for A, B, C, D in parallel
  - When recon completes, spawn vulnerability-hunter for each in parallel

IF some are in-progress:
  - Read each problem's STATUS.md
  - Resume from last known state
  - Spawn appropriate agents based on current phase

IF some are blocked:
  - Skip blocked problems
  - Note blockers in root STATUS.md
  - Focus agents on unblocked problems
```

### Parallelization Rules

**CAN parallelize:**
- Wave A (recon) across all problems simultaneously
- Wave B (analysis) across all problems simultaneously
- Wave C (exploitation) for INDEPENDENT problems
- Wave D (docs) for already-solved problems

**CANNOT parallelize:**
- Sequential waves within a single problem (A→B→C→D)
- Problems that share resources (same libc analysis, shared info)

### Status File Discipline

**Root STATUS.md** - Update after:
- Starting work on any problem
- Completing any problem
- Hitting a blocker
- Any session start/end

**Problem STATUS.md** - Update after:
- Each wave completion
- Finding key addresses/offsets
- Failed attempts (document what didn't work!)
- Any breakthrough

## MCP Integration

When pyghidra-mcp is configured:
```json
{
  "mcpServers": {
    "pyghidra-mcp": {
      "command": "uvx",
      "args": ["pyghidra-mcp", "target"],
      "env": { "GHIDRA_INSTALL_DIR": "/usr/share/ghidra" }
    }
  }
}
```

Use MCP tools for:
- `mcp__pyghidra-mcp__decompile_function` - Get pseudo-C
- `mcp__pyghidra-mcp__search_functions_by_name` - Find functions
- `mcp__pyghidra-mcp__search_strings` - Find strings in binary
- `mcp__pyghidra-mcp__list_cross_references` - Find xrefs

## Deliverables Checklist

For CTF challenges:

- [ ] `STATUS.md` - Progress tracking (update frequently)
- [ ] `exploit.py` - Working exploit (matches examples/ style if available)
- [ ] `find.py` - Address discovery (if needed)
- [ ] `REPORT.md` - Combined technical writeup and explanation

For malware analysis:

- [ ] `STATUS.md` - Progress tracking
- [ ] `REPORT.md` - Complete analysis documentation
- [ ] Extracted artifacts in `extracted/`
- [ ] Analysis scripts in `scripts/`
- [ ] Key findings documented with addresses/offsets
