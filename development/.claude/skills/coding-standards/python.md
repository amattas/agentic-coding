# Python Coding Standards

## File Organization & Modularization

**Critical: Files MUST be highly modularized and SHOULD NOT exceed 1000 lines unless absolutely necessary.**

- **Ideal target**: 500-800 lines per file
- **Maximum**: 1000 lines (only when absolutely necessary)
- Break large files into smaller, focused modules
- Use packages to organize related modules

## Code Quality & Linting

**REQUIRED: Run black and ruff before every commit.**

```bash
# Format code with black
black .

# Lint with ruff
ruff check . --fix
```

### Pre-commit Setup
Consider adding a pre-commit hook to enforce linting:
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.x.x
    hooks:
      - id: black
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.x.x
    hooks:
      - id: ruff
        args: [--fix]
```

## Style Guidelines

### PEP 8 Compliance
- Follow PEP 8 conventions (enforced by black/ruff)
- Maximum line length: 88 characters (black default)
- Use 4 spaces for indentation (never tabs)

### Naming Conventions
- `snake_case` for functions, variables, and module names
- `PascalCase` for class names
- `UPPER_CASE` for constants
- Prefix private attributes with single underscore: `_private_var`

### Type Hints
- Use type hints for all function signatures
- Use `typing` module for complex types
```python
from typing import Optional, List, Dict

def process_data(items: List[str], config: Optional[Dict[str, str]] = None) -> bool:
    pass
```

## Code Structure

### Import Organization
Group imports in order:
1. Standard library imports
2. Third-party imports
3. Local application imports

Use blank lines to separate groups.

```python
import os
import sys

import numpy as np
import pandas as pd

from myapp.models import User
from myapp.utils import helper
```

### Function Design
- Keep functions small and focused (single responsibility)
- Aim for functions under 50 lines
- Use early returns to reduce nesting
- Avoid side effects in pure functions

### Class Design
- Use dataclasses for simple data containers
- Prefer composition over inheritance
- Keep class responsibilities narrow and well-defined

## Error Handling

- Use specific exception types (avoid bare `except:`)
- Raise exceptions with descriptive messages
- Use context managers (`with` statements) for resource management

```python
# Good
try:
    result = risky_operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise

# Bad
try:
    result = risky_operation()
except:
    pass
```

## Documentation

### Docstrings
Use Google or NumPy style docstrings:

```python
def calculate_total(items: List[Item], tax_rate: float = 0.0) -> float:
    """Calculate the total cost including tax.

    Args:
        items: List of items to calculate total for
        tax_rate: Tax rate as decimal (e.g., 0.08 for 8%)

    Returns:
        Total cost including tax

    Raises:
        ValueError: If tax_rate is negative
    """
    pass
```

### Inline Comments
- Explain *why*, not *what*
- Keep comments up-to-date with code changes
- Use TODO format: `# TODO [owner]: [description] ([ticket])`

## Testing

- Write tests for all public interfaces
- Use pytest as the testing framework
- Aim for high coverage on business logic (80%+ target)
- Use meaningful test names: `test_user_login_with_invalid_password_returns_error`

## Common Anti-Patterns to Avoid

- **God objects**: Classes that do too much (violates SRP)
- **Long parameter lists**: Use dataclasses or config objects
- **Deep nesting**: Use guard clauses and early returns
- **Magic numbers**: Extract to named constants
- **Mutable default arguments**: Use `None` and initialize in function body

```python
# Bad
def process(items=[]):
    items.append(1)
    return items

# Good
def process(items: Optional[List[int]] = None) -> List[int]:
    if items is None:
        items = []
    items.append(1)
    return items
```

## Performance Considerations

- Use list comprehensions over `map`/`filter` for readability
- Use generators for large datasets
- Profile before optimizing (use `cProfile`, `line_profiler`)
- Consider `@lru_cache` for expensive pure functions

## Security

- Never log sensitive data (passwords, tokens, PII)
- Validate and sanitize all external inputs
- Use parameterized queries for database operations
- Keep dependencies updated (use `pip-audit` or similar)
