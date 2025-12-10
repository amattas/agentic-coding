# Security Research & CTF Guide

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

| Scenario | Start Wave | Skip? | Notes |
|----------|------------|-------|-------|
| Unknown binary | A | No skips | Always run checksec/file/strings first |
| Simple ret2win (obvious win function) | A | Skip gadget analysis | Direct to exploit after recon |
| NX enabled, need ROP | A | No skips | Full wave with gadget analysis |
| Complex protections (PIE+Canary+RELRO) | A | No skips | Full wave, may need multiple passes |
| Format string vulnerability | A | Skip payload crafting | Format string has its own patterns |
| Heap exploitation | A | No skips | Full wave with heap-specific analysis |
| Multi-problem lab | A | No skips | Use root STATUS.md for overview |

**Decision tree:**
1. Have you run `checksec` and `file` on this binary? → If no, **Wave A**
2. Is there an obvious `win()` or `get_flag()` function? → Try simple ret2win first
3. Are there complex protections (PIE, full RELRO, canary)? → Full wave sequence
4. Is the vulnerability class unclear? → **Wave B** analysis required

## Exploit Strategy Ladder

Try strategies in order of simplicity:

1. **Trivial wins first:**
   - Look for `win()`, `get_flag()`, `shell()` functions
   - Overwritable return address without canary, PIE disabled?
   - → Use simple ret2win before anything complex

2. **If no direct win but libc is leakable:**
   - → Prefer `ret2libc` or `system("/bin/sh")` over full ROP chains
   - Leak libc base via puts/printf GOT

3. **If control flow is constrained:**
   - → Use gadget analysis for ROP or SROP chains
   - Check for one_gadget shortcuts in libc

4. **If protections are unclear or exploit fails:**
   - → Re-analyze the binary
   - → Update `STATUS.md` with what didn't work and why
   - → Try alternative approach

**Golden rule:** Simplest working exploit wins. Don't build a 50-gadget ROP chain when ret2win would work.

## Global Norms

- Be truthful - don't fabricate addresses, offsets, or exploit behavior
- Respect the wave model: Wave A → Wave B → Wave C → Wave D
- Respect ethical constraints from the guidelines above
- Keep exploits simple and linear (match examples/ style if available)

## Workflow (Wave Model)

Follow this sequential workflow for each challenge:

### Wave A: Reconnaissance

Run recon tools (file, checksec, strings, disassembly). Save disassembly to `target.asm`.

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
| Canary | Leak it, brute force (fork server), off-by-one |
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

- REPORT.md (combined technical writeup and explanation)
- Update STATUS.md to mark complete with end time

**Output**: Complete documentation, STATUS.md marked solved

## Exploit Template

```python
#!/usr/bin/env python3
from pwn import *

context.arch = 'amd64'  # or 'i386'

EXECUTABLE = "./target"
HOST = "localhost"
PORT = 9999

# CRITICAL: Stack consistency
ARGV0 = "/pwn"  # Fixed argv[0] = consistent stack addresses
ENV = {}        # Empty environment

def conn():
    if args.REMOTE:
        return remote(HOST, PORT)
    elif args.GDB:
        return gdb.debug([EXECUTABLE], env=ENV, argv=[ARGV0], gdbscript='b *main\nc')
    else:
        return process([EXECUTABLE], env=ENV, argv=[ARGV0])

p = conn()

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
│   └── REPORT.md
├── problem2/
│   └── ...
└── problemN/
    └── ...
```

## Restartability (STATUS.md)

**Always maintain STATUS.md** to enable picking up work after interruptions.

**Root STATUS.md** (for multi-problem labs): Table with problem, status, start/end times, difficulty, notes.

**Problem STATUS.md** must include:
- Timeline (started/completed)
- Current phase (recon/analysis/exploitation/documentation)
- Key findings (buffer size, offsets, leaked values)
- What's working vs what's not working
- Next steps

When resuming: read STATUS.md first, check what's discovered, review failed attempts, continue from last good state.

## CRITICAL: Stack Consistency for Debugging

**This is the #1 cause of "my addresses don't match GDB" issues.**

When Linux starts a process, it pushes onto the stack (from high to low addresses):
1. Environment variable strings
2. Argument strings (including argv[0] - the program path)
3. Pointers to the above

Without fixing these, every run has different stack addresses - making debugging impossible.

### The Problem

```
Run 1:  ./target                    → argv[0] = "./target" (8 bytes)
Run 2:  python3 exploit.py          → argv[0] = different
GDB:    gdb ./target                → argv[0] = different again

Result: Stack addresses are DIFFERENT every time = can't debug reliably
```

Your shell also adds environment variables (PATH, HOME, TERM, etc.) to the stack.

### The Solution

**ALWAYS use a fixed argv[0] and empty environment:**

```python
# Fixed argv[0] - use any short consistent value
ARGV0 = "/pwn"
ENV = {}  # Empty environment

def conn():
    if args.REMOTE:
        return remote(HOST, PORT)
    elif args.GDB:
        return gdb.debug([EXECUTABLE], env=ENV, argv=[ARGV0], gdbscript='...')
    else:
        return process([EXECUTABLE], env=ENV, argv=[ARGV0])
```

### Why This Works

- `env={}` removes all shell environment variables from the stack
- `argv=[ARGV0]` spoofs a fixed program name regardless of how you invoke
- Result: Stack addresses are **IDENTICAL** between normal run and GDB debug

### Remote Considerations

| Exploit Type | Does stack matter for remote? |
|--------------|------------------------------|
| ROP / ret2win (PIE off) | No - code addresses are fixed |
| ret2libc | No - libc addresses from leak |
| Stack buffer / shellcode | Yes - need leak OR match remote argv[0] |

If you need absolute stack addresses for remote:
- **Best**: Leak stack address from the binary
- **Alternative**: Match remote's argv[0] (check Dockerfile for binary path)

## REPORT.md Contents

Include: overview, binary properties table, vulnerability details, key addresses, exploitation approach, payload structure, flag, and lessons learned.

## Priority Order

When recommendations conflict:

1. Ethical boundaries (authorized testing only)
2. Exploit correctness (it must actually work)
3. Simplicity (match examples/ style if available)
4. Documentation completeness
5. Code elegance


