# Security Research & CTF Guide

> **For general AI coding assistants.** This is a simplified guide for tools like Cursor, Copilot, Codeium, etc. If you're using **Claude Code**, see **[CLAUDE.md](./CLAUDE.md)** instead for advanced features (subagents, skills, parallel orchestration).

## Portability Notes

This guide is designed to work with any AI coding assistant. Tool-specific translations:

| Generic Operation | Claude Code | Other Tools |
|-------------------|-------------|-------------|
| Read file | `Read` tool | Use your file read capability |
| Search files | `Glob` tool | Use file search |
| Search content | `Grep` tool | Use content search |
| Edit file | `Edit` tool | Use your edit capability |
| Run command | `Bash` tool | Use shell/terminal access |
| Run Python | `Bash` with `python3` | Use your Python execution |

**If you're not Claude Code:**
- Ignore references to "subagents" and "skills" - these are Claude-specific
- Follow the wave model using your native capabilities
- Write artifacts to the file locations specified (`STATUS.md`, `REPORT.md`, etc.)
- Use `pwntools` for exploit development regardless of AI tool
- The exploitation patterns and techniques are universal

---

## Quickstart

**If you're a human working with an AI assistant:**
1. Confirm this is authorized testing (CTF, lab, or explicit permission)
2. Use `AGENTS.md` (this file) as the main reference
3. Follow the Wave model: Recon → Analysis → Exploitation → Documentation
4. Always update `STATUS.md` with findings and progress

**If you're a generic AI coding assistant:**
1. **Verify authorization first** - ask user to confirm CTF/lab context if unclear
2. Read `STATUS.md` first if it exists
3. Follow the wave sequence for each challenge
4. Write findings to `STATUS.md`, final docs to `REPORT.md`

**Quick decision:**
- New binary? → Wave A (file, checksec, strings baseline)
- Simple ret2win? → Skip gadget analysis, go straight to exploit
- Unknown protections? → Full wave sequence required

---

## Authorization Gate

**Before doing any exploit development, ALWAYS:**

1. Confirm the user has stated:
   - This is a CTF, lab, or authorized test, AND
   - No real-world third-party targets are involved

2. If authorization is unclear:
   - Ask the user to restate context and authorization
   - If refused or ambiguous, switch to explanation-only mode (analyze but don't exploit)

---

Guidelines for AI coding assistants working on security research, CTF challenges, and binary exploitation.

**Context**: Authorized security testing, CTF competitions, and educational purposes only.

## Ethical Boundaries

**Allowed:**
- CTF challenge exploitation
- Malware analysis and documentation
- Authorized vulnerability research
- Security tool development
- Writing analysis scripts

**Not Allowed:**
- Creating malware for unauthorized use
- Improving malicious code capabilities
- Targeting systems without authorization

## Wave Selection Heuristics

Use this to decide where to start:

| Scenario | Start Wave | Skip? |
|----------|------------|-------|
| Unknown binary | A | No skips |
| Simple ret2win | A | Skip gadget analysis |
| NX enabled, need ROP | A | No skips |
| Complex protections | A | No skips |

**Decision tree:**
1. Have you run `checksec` and `file`? → If no, **Wave A**
2. Obvious `win()` function? → Try ret2win first
3. Complex protections? → Full wave sequence

## Exploit Strategy Ladder

Try strategies in order of simplicity:

1. **Trivial wins first:** Look for `win()`, `get_flag()` → simple ret2win
2. **Libc leakable?** → ret2libc or `system("/bin/sh")`
3. **Control flow constrained?** → ROP chain with gadget-finder
4. **Exploit failing?** → Re-analyze, update STATUS.md with what didn't work

**Rule:** Simplest working exploit wins.

## Workflow (Wave Model)

Follow this sequential workflow for each challenge:

### Wave A: Reconnaissance
```bash
file target                    # File type
checksec target                # Security mitigations
strings target | head -50      # Interesting strings
objdump -M intel -d target > target.asm
```
**Output**: Initial findings in STATUS.md

### Wave B: Analysis
Identify vulnerability class:
- Buffer overflow (no bounds checking)
- Format string (user input as format)
- Use-after-free / heap corruption
- Integer overflow
- Race condition (TOCTOU)
- Logic bug

Check mitigations and plan bypass:
| Mitigation | Bypass |
|------------|--------|
| NX enabled | ROP, ret2libc |
| Canary | Leak it, brute force, off-by-one |
| ASLR | Leak addresses |
| PIE | Leak code base |
| Full RELRO | Can't use GOT overwrites |

**Output**: Vulnerability identified, bypass strategy in STATUS.md

### Wave C: Exploitation
- Keep it simple and linear
- Use pwntools library
- Test locally first, then remote
- Update STATUS.md with key addresses/offsets found

**Output**: Working `exploit.py`

### Wave D: Documentation
- README.md (simple explanation - 10th grade level)
- REPORT.md (technical details with addresses)
- Update STATUS.md to mark complete with end time

**Output**: Complete documentation, STATUS.md marked solved

## Exploit Template

```python
#!/usr/bin/env python3
from pwn import *

context.arch = 'amd64'

# Connection
p = process("./target")
#p = remote("host", PORT)

# Setup
elf = ELF("./target")
offset = 72  # bytes to return address

# Payload
payload = b'A' * offset
payload += p64(elf.symbols['win'])  # or ROP chain

# Send
p.sendline(payload)
p.interactive()
```

## Common Exploitation Patterns

### 1. Basic Buffer Overflow
**When**: No canary, unchecked input
```python
payload = b'A' * offset + p64(target_address)
```

### 2. ret2libc
**When**: NX enabled, need shell
```python
# Leak libc address first, then:
libc_base = leaked_addr - libc.symbols['puts']
system = libc_base + libc.symbols['system']
binsh = libc_base + next(libc.search(b'/bin/sh'))
# ROP: pop rdi; ret -> binsh -> system
```

### 3. Format String
**When**: printf(user_input)
```python
# Leak: %p %p %p %p
# Write: use %n to write to addresses
# Leak canary: %11$p (adjust offset)
```

### 4. ROP Chain
**When**: NX enabled, need to chain functions
```python
from pwn import *
rop = ROP(elf)
rop.call('open_flag')
rop.call('read_flag')
rop.call('show_flag')
payload = b'A' * offset + rop.chain()
```

### 5. GOT Overwrite
**When**: Partial RELRO, arbitrary write
```python
# Overwrite GOT entry to redirect function call
# e.g., puts@got -> system
```

### 6. Canary Bypass
**When**: Canary present but exploitable
- **Leak**: Use format string or info leak
- **Brute force**: Fork server (canary constant per child)
- **Off-by-one**: Corrupt adjacent variable instead

## File Structure

### Single Challenge
```
challenge/
├── STATUS.md       # Progress tracking (for restartability)
├── target          # Binary
├── target.asm      # Disassembly
├── exploit.py      # Working exploit
├── find.py         # Address finder (if needed)
├── README.md       # Simple explanation
└── REPORT.md       # Technical writeup
```

### Multi-Problem Lab/CTF
```
lab-name/
├── STATUS.md           # Overview of ALL problems (start/end times)
├── problem1/
│   ├── STATUS.md       # Detailed tracking for this problem
│   ├── target
│   ├── exploit.py
│   └── README.md
├── problem2/
│   └── ...
└── problemN/
    └── ...
```

## Restartability (STATUS.md)

**Always maintain STATUS.md** to enable picking up work after interruptions.

### Root STATUS.md (for multi-problem labs)
Track all problems with start/completion times:
```markdown
| Problem | Status | Started | Completed | Difficulty | Notes |
|---------|--------|---------|-----------|------------|-------|
| prob1 | ✅ Solved | 2024-01-15 09:00 | 2024-01-15 11:30 | Easy | ret2win |
| prob2 | 🔄 In Progress | 2024-01-15 12:00 | - | Medium | ROP needed |
| prob3 | ❌ Not Started | - | - | Hard | PIE + RELRO |
```

### Problem STATUS.md
Track detailed progress for each problem:
```markdown
## Timeline
- Started: 2024-01-15 12:00
- Completed: -

## Current Phase
[x] Reconnaissance
[x] Analysis
[ ] Exploitation
[ ] Documentation

## Key Findings
- Buffer size: 64 bytes
- Offset to RIP: 72
- Canary: leaked via format string

## What's Working
- Libc leak successful
- Canary bypass working

## What's Not Working
- ROP chain crashes (stack alignment?)

## Next Steps
1. Add ret gadget for alignment
2. Test full chain
```

### When Resuming Work
1. Read STATUS.md to understand current state
2. Check what's already discovered (addresses, offsets)
3. Review failed attempts to avoid repeating them
4. Continue from last known good state

## Tool Commands

### GDB/pwndbg
```
checksec              # Show mitigations
vmmap                 # Memory map
telescope $rsp 20     # Stack contents
x/20gx $rsp           # Raw stack
cyclic 200            # Generate pattern
cyclic -l 0x61616168  # Find offset
```

### ropper
```bash
ropper --file target --search "pop rdi"
ropper --file target --search "ret"
```

### pwntools
```python
cyclic(200)                    # Pattern
cyclic_find(0x61616168)        # Offset
elf.symbols['main']            # Symbol address
elf.got['puts']                # GOT entry
elf.plt['puts']                # PLT entry
next(elf.search(b'/bin/sh'))   # Find string
```

## Security Check Quick Reference

```bash
checksec target
# Output interpretation:
# RELRO: Full/Partial/None     - GOT protection
# Stack: Canary found/No canary - Stack protection
# NX: NX enabled/disabled       - Executable stack
# PIE: PIE enabled/No PIE       - Position independent
```

## Debugging Tips

1. **Local debugging**
   ```python
   p = process("./target")
   gdb.attach(p, gdbscript="b *main+50")
   ```

2. **Stack alignment** (x86-64)
   - Add extra `ret` gadget before function calls
   - Stack must be 16-byte aligned

3. **Environment differences**
   - Use `env={}` for clean environment
   - Remote may have different stack layout

4. **SUID binaries**
   - Can't use ptrace/gdb directly
   - Create symlink: `ln -s /path/to/target ./t`

## Deliverables

For each challenge:

1. **exploit.py** - Working exploit
   - Simple, linear code
   - Match style of reference examples (if available)
   - Include commented alternatives

2. **README.md** - Simple explanation
   - What the vulnerability is
   - How the exploit works
   - Could a 10th grader understand it?

3. **REPORT.md** - Technical details
   - Key addresses found
   - Tools used
   - Exploitation technique

## Resources

- CTF writeups: https://ctftime.org/writeups
- Heap exploitation: https://github.com/shellphish/how2heap
- ROP: http://crypto.stanford.edu/~blynn/rop/
- Format string: https://cs155.stanford.edu/papers/formatstring-1.2.pdf
