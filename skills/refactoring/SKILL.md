---
name: refactoring
description: Safe refactoring with backward compatibility checks. Use when user says /refactoring, needs to restructure code, extract functions, or clean up tech debt. Ensures changes don't break existing behavior.
---

# Safe Refactoring

Refactor code while preserving backward compatibility.

## Pre-Refactoring Checklist

### 1. Establish Baseline
- [ ] Tests pass before starting
- [ ] Note current behavior (screenshots, API responses)
- [ ] Identify all callers of code being changed
- [ ] Check for dynamic usage (reflection, string-based calls)

### 2. Assess Impact
- [ ] List all files that import/use target code
- [ ] Check for external API consumers
- [ ] Identify database schema dependencies
- [ ] Review configuration files referencing the code

## Refactoring Workflow

```
1. VERIFY  → Tests pass, behavior documented
2. GREP    → Find all usages
3. PLAN    → Identify smallest safe changes
4. CHANGE  → Make one change at a time
5. TEST    → Run tests after each change
6. VERIFY  → Compare before/after behavior
```

## Safe Patterns

### Extract Function

```python
# Before
def process():
    # ... 50 lines of code ...
    result = complex_calculation(a, b, c)
    # ... more code ...

# After - extract without changing behavior
def _complex_calculation(a, b, c):
    """Extracted from process() - same logic."""
    return complex_calculation(a, b, c)

def process():
    # ... 50 lines of code ...
    result = _complex_calculation(a, b, c)
    # ... more code ...
```

### Rename Symbol

```bash
# 1. Find all usages
grep -r "old_name" --include="*.py" app/

# 2. Check imports
grep -r "from .* import.*old_name" app/

# 3. Check tests
grep -r "old_name" tests/

# 4. Rename everywhere atomically
# 5. Run tests
```

### Move Function to New Module

```python
# 1. Create new module with function
# 2. Add re-export in old location (temporary)
# old_module.py
from new_module import moved_function  # Re-export

# 3. Update all imports over time
# 4. Remove re-export when all updated
```

### Change Function Signature

```python
# Safe: Add optional parameter with default
def func(a, b, new_param=None):  # OK - backward compatible
    pass

# Unsafe: Change parameter order or make required
def func(new_param, a, b):  # BREAKS callers
    pass
```

## Backward Compatibility Rules

### DO
- Add optional parameters with defaults
- Add new functions alongside old
- Use deprecation warnings before removal
- Keep re-exports during migration

### DON'T
- Remove or rename public functions
- Change parameter order
- Change return types
- Delete unused-looking code without grepping

## Verification Steps

### After Each Change

```bash
# 1. Run tests
pytest

# 2. Type check (if using)
mypy app/

# 3. Lint
ruff check .

# 4. Verify specific functionality
# (manual test or integration test)
```

### Before Committing

- [ ] All tests pass
- [ ] No new lint errors
- [ ] Manual verification of affected features
- [ ] Diff review - no unintended changes

## Common Refactorings

| Refactoring | Risk | Mitigation |
|-------------|------|------------|
| Rename variable | Low | IDE rename, grep verify |
| Extract function | Low | Keep same logic, add tests |
| Move to new file | Medium | Re-export from old location |
| Change signature | High | Add new function, deprecate old |
| Delete code | High | Grep all usages first |

## Deprecation Pattern

```python
import warnings

def old_function():
    """Deprecated: Use new_function instead."""
    warnings.warn(
        "old_function is deprecated, use new_function",
        DeprecationWarning,
        stacklevel=2
    )
    return new_function()
```