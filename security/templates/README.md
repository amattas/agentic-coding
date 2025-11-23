# [Challenge Name]

## What's the vulnerability?

[Explain in simple terms what's wrong with the program. Imagine explaining to someone who knows basic programming but not security.]

Example: "The program reads your name into a small box that can only hold 32 letters. But it doesn't check how many letters you type - if you type more than 32, the extra letters spill over and overwrite other important data."

## How does the exploit work?

[Step-by-step walkthrough in plain English]

1. **Find the weakness**: First, I figured out that I could type more data than the program expected...

2. **Measure the distance**: By sending a special pattern, I found that after typing 72 characters, the next thing I write will control where the program goes next...

3. **Redirect the program**: Instead of letting the program go back to normal, I made it jump to a function that prints the secret flag...

## What did I learn?

- **[Concept 1]**: Brief explanation
- **[Concept 2]**: Brief explanation

## Running the exploit

```bash
# Local
python3 exploit.py

# Remote
python3 exploit.py REMOTE

# With debugging
python3 exploit.py GDB
```

## Files

- `exploit.py` - The working exploit
- `target` - The vulnerable program
- `REPORT.md` - Detailed technical writeup
