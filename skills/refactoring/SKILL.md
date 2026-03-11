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
2. GREP    → Find all usages (use Grep tool, not bash grep)
3. PLAN    → Identify smallest safe changes
4. CHANGE  → Make one change at a time
5. TEST    → Run tests after each change
6. VERIFY  → Compare before/after behavior
```

## Safe Patterns

### Extract Function

```
Before:
  function process() {
    // ... 50 lines of code ...
    result = complexCalculation(a, b, c)
    // ... more code ...
  }

After - extract without changing behavior:
  function _complexCalculation(a, b, c) {
    // Extracted from process() - same logic
    return complexCalculation(a, b, c)
  }

  function process() {
    // ... 50 lines of code ...
    result = _complexCalculation(a, b, c)
    // ... more code ...
  }
```

### Rename Symbol

```
1. Use Grep to find all usages of "old_name" in the codebase
2. Check imports/requires referencing old_name
3. Check test files for old_name
4. Rename everywhere atomically
5. Run tests
```

### Move Function to New Module

```
1. Create new module with the function
2. Add re-export in old location (temporary backward compat)
   // old_module: export { movedFunction } from './new_module'
3. Update all imports over time
4. Remove re-export when all updated
```

### Change Function Signature

```
Safe:    Add optional parameter with default → backward compatible
Unsafe:  Change parameter order or make new param required → BREAKS callers
```

## Backward Compatibility Rules

### DO
- Add optional parameters with defaults
- Add new functions alongside old
- Use deprecation warnings/annotations before removal
- Keep re-exports during migration

### DON'T
- Remove or rename public functions
- Change parameter order
- Change return types
- Delete unused-looking code without grepping

## Verification Steps

### After Each Change

```
1. Run tests (project's test runner)
2. Run type checker (if the project uses one)
3. Run linter (project's linter)
4. Verify specific functionality (manual or integration test)
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

Mark deprecated code using the language's idiomatic mechanism:

```
- Python: warnings.warn("...", DeprecationWarning)
- TypeScript/JavaScript: @deprecated JSDoc tag
- Java: @Deprecated annotation
- Go: // Deprecated: comment convention
- Rust: #[deprecated(note = "...")]
- C#: [Obsolete("...")]
```

The deprecated function should delegate to the new function so callers still work.
