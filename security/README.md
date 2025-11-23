# Security Research & CTF Framework

A framework for AI-assisted security research, CTF challenges, binary exploitation, and malware analysis.

## Overview

This framework provides structured guidance for AI coding assistants working on security-related tasks in authorized contexts (CTF competitions, security research, educational purposes).

## Configuration Files

| File | Tool | Purpose |
|------|------|---------|
| `CLAUDE.md` | Claude Code | Full configuration with agents, MCP integration, detailed workflows |
| `AGENTS.md` | Other AI tools | Consolidated guide with patterns and templates |

### Why Two Files?

**CLAUDE.md** leverages Claude Code features:
- Subagent orchestration for parallel analysis
- MCP server integration (pyghidra-mcp)
- Structured workflow phases
- Detailed tool configurations

**AGENTS.md** provides a simpler format for:
- Tools without subagent support
- Quick reference during exploitation
- Self-contained patterns and templates

## Directory Structure

```
security/
├── CLAUDE.md                    # Claude Code orchestrator config
├── AGENTS.md                    # Other AI tools (consolidated)
├── README.md                    # This file
│
├── .claude/
│   ├── agents/                  # Specialized agents
│   │   ├── binary-scanner.md      # Wave A: Initial recon
│   │   ├── vulnerability-hunter.md # Wave B: Find vulns
│   │   ├── reverse-engineer.md    # Wave B: Deep analysis
│   │   ├── crypto-analyzer.md     # Wave B: Crypto analysis
│   │   ├── malware-analyzer.md    # Wave B: Malware analysis
│   │   ├── gadget-finder.md       # Wave C: ROP gadgets
│   │   ├── payload-crafter.md     # Wave C: Shellcode
│   │   ├── exploit-developer.md   # Wave C: Build exploits
│   │   ├── exploit-tester.md      # Wave C: Validate
│   │   └── writeup-writer.md      # Wave D: Documentation
│   │
│   └── skills/                  # Reusable knowledge
│       ├── exploitation-techniques/  # BOF, ROP, fmt string, canary, GOT
│       ├── vm-interpreter/          # Custom VM/interpreter exploitation
│       ├── logic-vulnerabilities/   # FD abuse, TOCTOU
│       ├── side-channels/           # Timing attacks
│       ├── binary-analysis/         # Tools & techniques
│       ├── malware-patterns/        # Behaviors & IOCs
│       ├── tool-usage/              # Command reference
│       └── reporting-standards/     # Doc formats
│
├── context/                     # Generated analysis artifacts
│   ├── binary-info.md             # From binary-scanner
│   ├── vulnerability-analysis.md  # From vulnerability-hunter
│   ├── gadgets.md                 # From gadget-finder
│   └── ...
│
├── templates/                   # Starter templates
│   ├── exploit.py                 # Exploit template
│   ├── REPORT.md                  # Technical writeup
│   ├── README.md                  # Simple explanation
│   ├── STATUS.md                  # Root-level overview (multi-problem)
│   ├── problem-README.md          # Problem description
│   └── problem-STATUS.md          # Per-problem tracking
│
└── combined-agent-claude-docs.md # Source material (reference)
```

### Multi-Problem Lab Structure (Primary Use Case)

```
lab08/
├── CLAUDE.md           # Orchestrator config
├── AGENTS.md           # For other tools
├── STATUS.md           # HIGH-LEVEL: all problems summary
├── .claude/            # Agents and skills
│
├── problem1/
│   ├── README.md       # Problem description (use problem-README template)
│   ├── STATUS.md       # DETAILED: this problem's state
│   ├── target
│   ├── target.asm
│   ├── exploit.py
│   ├── REPORT.md
│   └── examples/       # Reference exploits (optional, if provided)
│
├── problem2/
│   ├── README.md
│   ├── STATUS.md
│   └── ...
│
└── problemN/
    └── ...
```

**Two STATUS.md files:**
- **Root `STATUS.md`**: High-level overview of ALL problems (solved/in-progress/blocked)
- **Problem `STATUS.md`**: Detailed state for ONE problem (addresses, attempts, findings)

### Single Challenge Structure

```
challenge/
├── target              # Binary being analyzed
├── target.asm          # Disassembly
├── exploit.py          # Working exploit
├── find.py             # Address finder (if needed)
├── STATUS.md           # Progress tracking
├── README.md           # Simple explanation
├── REPORT.md           # Technical writeup
└── examples/           # Reference exploits (optional)
```

## Wave Model (Claude Code)

The framework organizes work into sequential waves with specialized agents:

```
Wave A (Recon)     Wave B (Analysis)      Wave C (Exploit)      Wave D (Docs)
     │                    │                     │                    │
binary-scanner ──► vulnerability-hunter ──► gadget-finder ───► writeup-writer
                  reverse-engineer         payload-crafter
                  crypto-analyzer          exploit-developer
                  malware-analyzer         exploit-tester
                         │                      │
                         ▼                      ▼
                  context/*.md              exploit.py
                                           find.py
```

### Wave A – Reconnaissance
- `binary-scanner`: File type, architecture, mitigations, strings
- Output: `context/binary-info.md`

### Wave B – Analysis (parallel where possible)
- `vulnerability-hunter`: Identify vulns from code patterns
- `reverse-engineer`: Deep code understanding
- `crypto-analyzer`: Encryption/obfuscation analysis
- `malware-analyzer`: Malware-specific analysis
- Output: `context/vulnerability-analysis.md`, etc.

### Wave C – Exploitation (sequential)
- `gadget-finder`: ROP gadget discovery
- `payload-crafter`: Custom shellcode
- `exploit-developer`: Write working exploit
- `exploit-tester`: Validate exploit
- Output: `exploit.py`, `find.py`

### Wave D – Documentation
- `writeup-writer`: README.md, REPORT.md
- Updates STATUS.md

## Multi-Problem Orchestration

For labs with multiple problems, use `lab-orchestrator` agent to coordinate:

### Parallel Agent Spawning

```
Lab with 4 problems (A, B, C, D):

Session 1 (Fresh Start):
┌─────────────────────────────────────────────────────────┐
│ lab-orchestrator reads STATUS → all "Not Started"       │
│                                                         │
│ Spawns in PARALLEL:                                     │
│   binary-scanner(A)  binary-scanner(B)                  │
│   binary-scanner(C)  binary-scanner(D)                  │
│                                                         │
│ Then PARALLEL:                                          │
│   vuln-hunter(A)  vuln-hunter(B)                        │
│   vuln-hunter(C)  vuln-hunter(D)                        │
└─────────────────────────────────────────────────────────┘

Session 2 (Resume):
┌─────────────────────────────────────────────────────────┐
│ lab-orchestrator reads STATUS:                          │
│   A: ✅ Solved    → Skip                                │
│   B: 🔄 Analysis  → spawn exploit-developer(B)          │
│   C: 🔄 Recon     → spawn vuln-hunter(C)                │
│   D: ⏸️ Blocked   → Skip, note blocker                  │
└─────────────────────────────────────────────────────────┘
```

### STATUS Files for Restartability

**Root STATUS.md** (high-level):
```markdown
| Problem | Status | Notes |
|---------|--------|-------|
| prob1 | ✅ Solved | Basic BOF |
| prob2 | 🔄 In Progress | Have leak, building ROP |
| prob3 | ❌ Not Started | |
| prob4 | ⏸️ Blocked | Need remote libc |
```

**Problem STATUS.md** (detailed):
```markdown
## Key Findings
- Offset: 72 bytes
- Canary at: %11$p
- Libc leak working

## What Failed
- Direct ret2win: ASLR prevents
- Static ROP: PIE enabled

## Next Steps
1. Leak PIE base
2. Build dynamic ROP
```

## Quick Start

### For CTF Challenges

1. **Copy the appropriate config** to your challenge directory:
   - Use `CLAUDE.md` for Claude Code
   - Use `AGENTS.md` for other tools

2. **Set up the challenge**:
   ```bash
   mkdir challenge && cd challenge
   cp /path/to/target .
   objdump -M intel -d target > target.asm
   checksec target
   ```

3. **Create STATUS.md** to track progress:
   ```markdown
   # Status
   ## Working
   - [completed steps]
   ## Not Working
   - [current blockers]
   ## Next Steps
   - [planned actions]
   ```

4. **Develop exploit** following the templates in AGENTS.md

5. **Document** with README.md and REPORT.md

### For Malware Analysis

1. **Set up analysis environment** (isolated VM/container)

2. **Create project structure**:
   ```bash
   mkdir analysis && cd analysis
   # Copy sample (DO NOT EXECUTE)
   mkdir extracted scripts
   ```

3. **Static analysis first**:
   ```bash
   file sample.bin
   strings -a sample.bin > strings.txt
   objdump -M intel -d sample.bin > disassembly.asm
   ```

4. **Document findings** in SUMMARY.md

## Exploitation Patterns Reference

Patterns are organized into skills in `.claude/skills/`:

| Skill | Patterns |
|-------|----------|
| `exploitation-techniques` | BOF, canary bypass, GOT overwrite, format string, ret2libc, ROP, heap |
| `vm-interpreter` | Custom interpreter/VM arbitrary R/W |
| `logic-vulnerabilities` | FD abuse, TOCTOU race conditions |
| `side-channels` | Timing attacks, byte-by-byte oracles |

## Deployment Locations

### Project-Level (Recommended)

Place in each project/challenge directory:
```
challenge/
├── CLAUDE.md    # or AGENTS.md
└── ...
```

### User-Level

For personal defaults across projects:

| Platform | Location |
|----------|----------|
| macOS/Linux | `~/.claude/CLAUDE.md` |
| Windows | `%USERPROFILE%\.claude\CLAUDE.md` |

### Team/Lab Environment

Share via git repository or lab infrastructure:
```
lab-configs/
├── security/
│   ├── CLAUDE.md
│   └── AGENTS.md
```

## Tool Integration

### pyghidra-mcp

For automated Ghidra analysis, add to `.mcp.json`:
```json
{
  "mcpServers": {
    "pyghidra-mcp": {
      "command": "uvx",
      "args": ["pyghidra-mcp", "target"],
      "env": {
        "GHIDRA_INSTALL_DIR": "/usr/share/ghidra"
      }
    }
  }
}
```

### pwntools

Standard exploit development library:
```python
from pwn import *
context.arch = 'amd64'
p = process("./target")
# or: p = remote("host", port)
```

### ropper/ROPgadget

For ROP gadget discovery:
```bash
ropper --file target --search "pop rdi"
ROPgadget --binary target
```

## Ethical Guidelines

This framework is for **authorized security testing only**:

- CTF competitions
- Authorized penetration testing
- Security research with permission
- Educational purposes

**Never use** for unauthorized access, malware deployment, or malicious purposes.

## Resources

### Learning
- CTF writeups: https://ctftime.org/writeups
- LiveOverflow: https://www.youtube.com/c/LiveOverflow
- pwn.college: https://pwn.college/

### Tools
- pwntools: https://docs.pwntools.com/
- Ghidra: https://ghidra-sre.org/
- Binary Ninja: https://binary.ninja/

### References
- how2heap: https://github.com/shellphish/how2heap
- ROP Emporium: https://ropemporium.com/
- Phrack: http://phrack.org/
