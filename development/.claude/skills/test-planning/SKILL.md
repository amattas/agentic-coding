---
name: test-planning
description: Design test strategy with concrete test cases. Use before writing tests.
---

# Test Planning

Create a comprehensive test strategy with specific test cases covering requirements.

## Process

1. Review spec.md acceptance criteria
2. Identify test categories needed (unit, integration, e2e)
3. Write concrete test cases for each requirement
4. Include edge cases and error scenarios
5. Define test data and fixtures needed

## Output

Create `test-plan.md` using the template in `templates/test-plan.md`.

## Tips

- Each acceptance criterion should map to tests
- Include both happy path and error cases
- Consider boundary conditions
- Note any test infrastructure needs
- Prioritize tests by risk
