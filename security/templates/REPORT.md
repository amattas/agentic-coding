# Challenge: [NAME]

**Category**: Binary Exploitation / Pwn / Reverse Engineering
**Points**: XX
**Solves**: XX

## Summary

[One paragraph technical summary of the vulnerability and exploitation approach]

## Analysis

### Binary Properties

| Property | Value |
|----------|-------|
| Architecture | x86-64 / i386 |
| NX | Enabled / Disabled |
| Stack Canary | Found / Not found |
| PIE | Enabled / Disabled |
| RELRO | Full / Partial / None |

### Vulnerability

- **Type**: [Buffer overflow, format string, use-after-free, etc.]
- **Location**: Function `name` at `0xADDRESS`
- **Root Cause**: [Why the vulnerability exists]
- **Trigger**: [How to reach the vulnerable code]

### Key Addresses

| Symbol | Address | Purpose |
|--------|---------|---------|
| main | 0x... | Entry point |
| vuln | 0x... | Vulnerable function |
| win | 0x... | Target function |
| pop rdi | 0x... | ROP gadget |

## Exploitation

### Strategy

1. [Step 1]
2. [Step 2]
3. [Step 3]

### Payload Structure

```
[description of payload layout]

Offset 0:    [AAAA... padding]
Offset N:    [return address / ROP chain]
```

### Challenges Encountered

- [Any obstacles and how they were overcome]

## Exploit Code

```python
# See exploit.py
```

## Flag

```
flag{...}
```

## Mitigations

This vulnerability could be prevented by:

1. [Mitigation 1]
2. [Mitigation 2]

## Tools Used

- pwntools
- GDB / pwndbg
- ropper
- [Other tools]

## References

- [Any references used]
