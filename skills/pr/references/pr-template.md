# PR Template

## GitHub PR Body Template

```markdown
## Summary
- [First change description]
- [Second change description]
- [Third change description]

## Test Plan
- [ ] Unit tests pass
- [ ] Manual testing completed
- [ ] No regressions observed

## Related Issues
Closes #<issue-id>

---
Generated with Claude Code
```

## Commit Message Format

```
<type>(<scope>): <summary>

<optional body>

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code change that neither fixes nor adds
- `chore`: Maintenance tasks
- `docs`: Documentation only
- `test`: Adding or updating tests
- `perf`: Performance improvement
- `style`: Formatting, missing semi-colons, etc
- `ci`: CI/CD changes

### Examples

**Feature:**
```
feat(auth): add JWT token refresh endpoint

Implement automatic token refresh when access token expires.
Includes rate limiting and logging.

Closes #1234
```

**Bug fix:**
```
fix(api): correct null pointer in user service

Handle case where user has no previous records.

Closes #5678
```
