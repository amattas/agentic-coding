# Security Research & CTF Assistant

> **For Claude Code users only.** This file uses Claude Code's advanced features (subagents, skills, wave-based orchestration). If you're using a different AI coding tool (Cursor, Copilot, Codeium, etc.), see **[AGENTS.md](./AGENTS.md)** instead for a simplified guide.

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

This file defines how Claude Code should orchestrate subagents and skills for security research, CTF challenges, binary exploitation, and malware analysis. Detailed knowledge lives in skills and agent definition files.

## Authorization Gate

**Before doing any exploit development, ALWAYS verify:**

1. The user has clearly stated one of:
   - This is a CTF challenge
   - This is an authorized lab/training environment
   - This is explicitly authorized penetration testing
   - No real-world third-party targets are involved

2. If authorization is unclear:
   - Ask the user to restate the context and authorization
   - If they refuse or it's ambiguous, switch to **explanation-only mode**
   - In explanation-only mode: analyze, explain, and document, but do not produce working exploits

3. When in doubt, ask. It's better to confirm than to overstep.

---

## Context

**This is for authorized security testing, CTF competitions, and educational purposes only.**

Supported activities:
- CTF (Capture The Flag) challenges
- Binary exploitation research
- Malware analysis and reverse engineering
- Vulnerability research (authorized)
- Security tool development

## Wave Selection Heuristics

Use this to decide where to start:

| Scenario | Start Wave | Skip? | Notes |
|----------|------------|-------|-------|
| Unknown binary | A | No skips | Always run checksec/file/strings first |
| Simple ret2win (obvious win function) | A | Skip gadget-finder | Direct to exploit-developer after recon |
| NX enabled, no direct win | A | No skips | Full wave with gadget-finder |
| Complex protections (PIE+Canary+RELRO) | A | No skips | Full wave, may need multiple analysis passes |
| Format string vulnerability | A | Skip payload-crafter | Format string has its own patterns |
| Heap exploitation | A | No skips | Full wave with heap-specific analysis |
| Multi-problem lab | A | No skips | Use lab-orchestrator for coordination |

**Decision tree:**
1. Have you run `checksec` and `file` on this binary? → If no, **Wave A**
2. Is there an obvious `win()` or `get_flag()` function? → Try simple ret2win first
3. Are there complex protections (PIE, full RELRO, canary)? → Full wave sequence
4. Is the vulnerability class unclear? → **Wave B** analysis required

## Exploit Strategy Ladder

When developing an exploit, **try strategies in order of simplicity**:

1. **Check for trivial wins first:**
   - Look for `win()`, `get_flag()`, `shell()` functions
   - Overwritable return address without canary, PIE disabled?
   - → Use simple ret2win before anything complex

2. **If no direct win but libc is leakable:**
   - → Prefer `ret2libc` or `system("/bin/sh")` over full ROP chains
   - Leak libc base via puts/printf GOT

3. **If control flow is constrained:**
   - → Use `gadget-finder` for ROP or SROP chains
   - Check for one_gadget shortcuts in libc

4. **If protections are unclear or exploit fails:**
   - → Re-run `binary-scanner` and `reverse-engineer`
   - → Update `STATUS.md` with what didn't work and why
   - → Try alternative approach

**Golden rule:** Simplest working exploit wins. Don't build a 50-gadget ROP chain when ret2win would work.

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

## Global Norms

- Be truthful - don't fabricate addresses, offsets, or exploit behavior
- Prefer using subagents and skills when they fit better than single-context work
- Respect the wave model: Wave A → Wave B → Wave C → Wave D
- Respect ethical constraints from the guidelines below
- Keep exploits simple and linear (match examples/ style if available)

## Ethical Guidelines

### DO
- Analyze existing malware behavior and document findings
- Develop exploits for CTF challenges and authorized testing
- Write decryption/extraction/analysis scripts
- Create documentation and technical reports
- Answer questions about code functionality and vulnerabilities
- Develop analysis tools (LD_PRELOAD hooks, parsers, fuzzers)

### DO NOT
- Create malware for unauthorized deployment
- Improve or augment malicious capabilities for real-world use
- Remove anti-analysis features to make malware more effective
- Execute malware outside controlled environments
- Target systems without explicit authorization

## Project Structure

### Multi-Problem Lab/CTF Layout (Primary)

When a CTF or lab has multiple problems, use this structure:

```
lab-name/
├── CLAUDE.md           # This file (orchestrator config)
├── AGENTS.md           # For other AI tools
├── STATUS.md           # HIGH-LEVEL summary of ALL problems
├── .claude/            # Agents and skills (symlink or copy)
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

**Key files for restartability:**
- `STATUS.md` (root): Which problems are solved/in-progress/blocked
- `*/STATUS.md` (per-problem): Detailed state, findings, failed attempts

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
├── README.md           # 10th-grade level explanation
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

## Wave Model

### Wave A – Reconnaissance

When starting any security challenge or analysis:

- USE `binary-scanner` to gather initial information
- USE `security-researcher` (if researching CVEs, techniques, or prior writeups)
- Produces `context/binary-info.md` with:
  - Architecture, file type, security mitigations
  - Initial strings and symbols

### Wave B – Analysis

After initial recon, in parallel where possible:

- MUST USE `vulnerability-hunter` to identify potential vulnerabilities
- USE `reverse-engineer` for complex code understanding
- USE `crypto-analyzer` if encryption/obfuscation is present
- USE `malware-analyzer` for malware samples (instead of above)

Produces:
- `context/vulnerability-analysis.md`
- `context/reverse-engineering.md`
- `context/crypto-analysis.md`
- `context/research.md` (if external research was needed)

### Wave C – Exploitation Development

Once vulnerabilities are identified:

- USE `gadget-finder` if ROP is needed
- USE `payload-crafter` for custom shellcode
- MUST USE `exploit-developer` to create working exploit
- USE `exploit-tester` to validate

Produces:
- `context/gadgets.md`
- `exploit.py`
- `find.py` (if needed)

### Wave D – Documentation

After successful exploitation:

- MUST USE `writeup-writer` to create documentation
- Creates `README.md` and `REPORT.md`
- Updates `STATUS.md` with final status

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

### Restartability Protocol

When resuming work:

1. **Read root STATUS.md first**
   - What's solved? Skip those
   - What's in progress? Read their STATUS.md
   - What's blocked? Note blockers

2. **For each in-progress problem, read its STATUS.md**
   - What phase is it in?
   - What's already discovered?
   - What failed and why?

3. **Resume from last known good state**
   - Don't re-do completed analysis
   - Use discovered addresses/offsets
   - Avoid repeating failed approaches

4. **Spawn agents based on current state**
   - Match agent to current phase
   - Provide context from STATUS.md

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

## Priorities

When recommendations conflict:

1. Ethical boundaries (authorized testing only)
2. Exploit correctness (it must actually work)
3. Simplicity (match examples/ style if available)
4. Documentation completeness
5. Code elegance

## Workflow (Quick Reference)

### Phase 1: Reconnaissance

1. **Identify the target type**
   - Binary exploitation (ELF, PE)
   - Malware analysis
   - Web application
   - Android/Mobile
   - Crypto challenge

2. **Gather information**
   ```bash
   file target
   checksec target
   strings target | head -50
   readelf -h target
   objdump -M intel -d target > target.asm
   ```

3. **Document initial findings in STATUS.md**

### Phase 2: Analysis

For **binary exploitation**:
- USE `codebase-analyzer` agent to understand binary structure
- USE pyghidra-mcp for decompilation when available
- Identify vulnerability class (see Pattern Library below)
- Map security mitigations (ASLR, NX, canary, PIE, RELRO)

For **malware analysis**:
- USE read-only analysis first (static)
- USE LD_PRELOAD hooks for runtime interception
- Document encryption/obfuscation layers
- Map C2 communication patterns

### Phase 3: Exploitation/Documentation

- USE `implementation-engineer` for exploit development
- USE `tdd-test-writer` for exploit validation scripts
- Keep exploits linear and simple (match examples/ style if available)
- Document in README.md (simple) and REPORT.md (technical)

### Phase 4: Verification

- USE `test-runner-validator` to verify exploits work
- Test locally, then against remote (if CTF server)
- Document any environment-specific requirements

## Exploit Development Style

Based on pwntools conventions:

```python
#!/usr/bin/env python3
from pwn import *
import sys

context.arch = 'amd64'  # or 'i386'

EXECUTABLE = "./target"
ENV = {}

# Connection - comment out alternatives
p = process([EXECUTABLE], env=ENV)
#gdb.attach(p, gdbscript="b *main")
#p = remote("localhost", PORT)

# ELF setup
elf = ELF(EXECUTABLE)
#libc = ELF("./libc.so.6")

# Key addresses
buffer_size = 0x40
ret_offset = buffer_size + 8  # 64-bit: +8, 32-bit: +4

# Gadgets (from ropper or pwntools)
pop_rdi = 0x401234
ret = 0x40101a  # for stack alignment

# Payload construction
payload = b'A' * ret_offset
payload += p64(pop_rdi)
payload += p64(next(elf.search(b'/bin/sh')))
payload += p64(elf.symbols['system'])

# Send and interact
p.sendline(payload)
p.interactive()
```

**Style rules:**
- Linear flow, minimal functions
- Lowercase_with_underscores for variables
- Print addresses for debugging: `print(f"libc_base: {hex(libc_base)}")`
- Comment out alternative connection methods
- Keep it simple - match examples/ structure if available

## Pattern Library Reference

Exploitation patterns are organized into skills:

| Skill | Patterns |
|-------|----------|
| `exploitation-techniques` | BOF, canary bypass, GOT overwrite, format string, ret2libc, ROP, heap |
| `vm-interpreter` | Interpreter/VM arbitrary R/W exploitation |
| `logic-vulnerabilities` | FD abuse, TOCTOU race conditions |
| `side-channels` | Timing attacks, oracles |

USE the appropriate skill when developing exploits.

## Tool Reference

### Static Analysis
```bash
# ELF analysis
file target
checksec target
readelf -h target
readelf -S target
objdump -M intel -d target > target.asm

# String extraction
strings target
strings -a -t x target  # with offsets

# Binary diffing
diff <(xxd file1) <(xxd file2)
```

### Dynamic Analysis
```bash
# Tracing
strace -f ./target
ltrace ./target

# GDB with pwndbg
gdb ./target
> checksec
> vmmap
> telescope $rsp
> x/20gx $rsp

# LD_PRELOAD hooks
gcc -shared -fPIC -o hook.so hook.c -ldl
LD_PRELOAD=./hook.so ./target
```

### Exploitation Tools
```bash
# Gadget finding
ropper --file target --search "pop rdi"
ROPgadget --binary target

# Pattern generation
pwn cyclic 200
pwn cyclic -l 0x61616168

# Shellcode
pwn shellcraft amd64.linux.sh
```

### MCP Integration

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

## Security Mitigation Quick Reference

| Mitigation | Check | Bypass Approach |
|------------|-------|-----------------|
| **NX/DEP** | `checksec` shows NX enabled | ROP, ret2libc |
| **Stack Canary** | Canary found | Leak canary, brute force (fork), off-by-one |
| **ASLR** | System-wide | Leak addresses, partial overwrite |
| **PIE** | PIE enabled | Leak code base, partial overwrite |
| **Full RELRO** | RELRO: Full | Can't overwrite GOT, target other pointers |
| **Partial RELRO** | RELRO: Partial | GOT is writable |

## Standard Artifact Schema

### Required Artifacts

All security work must produce artifacts in a consistent format:

**`report.md`** (technical details):
```markdown
# Technical Report: [Challenge/Target Name]

## Binary Metadata
- File: `target` (or sample name)
- Architecture: amd64 / i386 / arm
- Checksec output: (paste full output)

## Identified Vulnerabilities
- Type: (BOF, format string, heap, logic, etc.)
- Location: (function name, offset)
- Trigger: (how to reach vulnerable code)

## Key Addresses
| Symbol/Gadget | Address | Notes |
|---------------|---------|-------|
| buffer_start | 0x... | |
| ret_offset | 72 | bytes to overwrite RIP |
| pop_rdi | 0x... | from ropper |
| libc_base | 0x... | leaked via ... |

## Exploit Strategy
1. Step-by-step approach
2. Why this strategy was chosen
3. Alternatives considered

## Commands and Tools Used
- `checksec target`
- `ropper --file target --search "pop rdi"`
- etc.
```

**`writeup.md`** (narrative explanation):
```markdown
# [Challenge Name] Writeup

## Overview
High-level description suitable for someone learning.

## Approach
1. First, I examined the binary...
2. I noticed that...
3. To exploit this...

## Solution
```python
# key parts of exploit
```

## Lessons Learned
- What made this interesting
- What would you do differently
```

**`STATUS.md`** (progress tracking):
```markdown
## Timeline
- Started: YYYY-MM-DD HH:MM
- Completed: YYYY-MM-DD HH:MM (or "-")

## Phase
- [x] Reconnaissance
- [x] Analysis
- [ ] Exploitation
- [ ] Documentation

## Key Findings
- Buffer: 64 bytes
- Offset: 72
- Protections: NX, no canary, no PIE

## What's Working
- Libc leak successful

## What's NOT Working
- ROP chain crashes (stack alignment?)

## Next Steps
1. Add ret gadget
2. Test full chain
```

### Artifact Naming Convention

| Artifact | Filename | Location |
|----------|----------|----------|
| Progress tracking | `STATUS.md` | root or problem dir |
| Technical report | `report.md` | problem dir |
| Narrative writeup | `writeup.md` | problem dir |
| Exploit code | `exploit.py` | problem dir |
| Address finder | `find.py` | problem dir (if needed) |
| Disassembly | `target.asm` | problem dir |

**Note:** Use lowercase filenames (`report.md` not `REPORT.md`) for consistency.

## Deliverables Checklist

For CTF challenges:

- [ ] `STATUS.md` - Progress tracking (update frequently)
- [ ] `exploit.py` - Working exploit (matches examples/ style if available)
- [ ] `find.py` - Address discovery (if needed)
- [ ] `writeup.md` - Simple explanation (10th-grade level)
- [ ] `report.md` - Technical writeup (addresses, techniques)

For malware analysis:

- [ ] `STATUS.md` - Progress tracking
- [ ] `report.md` - Complete analysis documentation
- [ ] Extracted artifacts in `extracted/`
- [ ] Analysis scripts in `scripts/`
- [ ] Key findings documented with addresses/offsets

## Common Pitfalls

1. **Stack alignment** - x86-64 requires 16-byte alignment before calls
2. **Virtual vs file offsets** - ELF loading changes addresses
3. **Remote vs local** - Stack layout differs (env vars, argv)
4. **Endianness** - x86 is little-endian
5. **Null bytes** - Many functions terminate on \x00
6. **Bad characters** - Some inputs filter certain bytes
7. **SUID binaries** - Can't use ptrace, need alternative debugging

## Resources

### General
- https://ctftime.org/writeups
- https://github.com/shellphish/how2heap

### ROP
- http://crypto.stanford.edu/~blynn/rop/
- https://hovav.net/ucsd/dist/blackhat08.pdf

### Format String
- https://cs155.stanford.edu/papers/formatstring-1.2.pdf
- http://phrack.org/issues/59/7.html

### Stack Protection Bypass
- http://phrack.org/issues/56/5.html
- https://www.coresecurity.com/sites/default/private-files/publications/2016/05/StackguardPaper.pdf
