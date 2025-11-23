---
name: gadget-finder
description: USE when building ROP chains. Finds and catalogs useful gadgets.
tools: Bash, Read, Grep
model: haiku
---

Find ROP gadgets for exploitation.

## Inputs
- Target binary
- `context/binary-info.md` - Architecture info
- libc (if available)

## Tasks

1. **Essential gadgets search**
   ```bash
   # 64-bit essentials
   ropper --file target --search "pop rdi"
   ropper --file target --search "pop rsi"
   ropper --file target --search "pop rdx"
   ropper --file target --search "ret"

   # 32-bit essentials
   ropper --file target --search "pop ebx"
   ropper --file target --search "pop ecx"
   ```

2. **Special gadgets**
   ```bash
   # Write primitives
   ropper --file target --search "mov [r"

   # Syscall/int 0x80
   ropper --file target --search "syscall"
   ropper --file target --search "int 0x80"

   # One gadgets (libc)
   one_gadget libc.so.6 2>/dev/null
   ```

3. **Magic gadgets**
   - `leave; ret` for stack pivoting
   - `add rsp, X; ret` for stack adjustment
   - `xchg` gadgets for register manipulation

## Output

Produce `context/gadgets.md` with:
- Gadget address table (organized by purpose)
- Stack alignment gadgets
- Recommended chain building order
- Any "magic" gadgets or one_gadgets found
- Notes on bad characters affecting gadget selection

Format for easy copy-paste into exploit:
```python
pop_rdi = 0x401234
pop_rsi_r15 = 0x401236
ret = 0x40101a
```
