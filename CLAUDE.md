# Global Development Conventions

> **Purpose**: Shared conventions, workflows, and philosophy across all projects. Intended to be symlinked into project `.claude/` directories.

---

## Philosophy

**Professional Objectivity**: Prioritize technical accuracy over validation. Challenge assumptions, disagree when necessary, provide objective guidance.

**Anti-Over-Engineering**:
- Only make changes that are directly requested or clearly necessary
- Don't add features, refactoring, or "improvements" beyond the ask
- Three similar lines > premature abstraction
- Don't add error handling for scenarios that can't happen
- No backwards-compatibility hacks (unused exports, `_vars`, `// removed` comments)
- If unused, delete it completely

**Safety-First Refactoring**: If refactoring code that lacks test coverage, write characterization tests FIRST to lock in current behavior, then refactor against those tests.

**Full-Stack Debugging**: Don't just check server logs. Trace issues end-to-end: browser console → network tab (endpoint, payload, headers, response) → server handler → service → DB → response → frontend state → UI. Identify which boundary the bug crosses.

**Database Design**: Default to normalized schemas (3NF). When a flattened query shape is needed, prefer views or materialized views over denormalized tables. Only denormalize when views aren't sufficient, and document why.

---

## Git Workflow

### Commit Message Format

**Rules**:
- Max 72 characters
- Format: `<type>(<scope>): <short summary>`
- Types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `style`, `ci`

**Examples**:
```
feat(auth): add JWT token validation middleware
fix(api): correct null pointer in user service
refactor(core): simplify pipeline logic
chore(deps): bump dependency versions
docs(readme): clarify setup instructions
test(auth): add login flow coverage
```

### Branch Naming

```
feature/<short-description>
fix/<issue-description>
refactor/<what-refactoring>
chore/<what-chore>
```

### Pull Requests

**Title**: Same format as commit messages

**Body Template**:
```markdown
## Summary
- Bullet point 1
- Bullet point 2
- Bullet point 3

## Test Plan
- [ ] Unit tests pass
- [ ] Manual testing completed
- [ ] No regressions observed

## Related Issues
Closes #123
```

---

## Code Review Standards

### Security (Highest Priority)
- No secrets in code (use environment variables, secrets manager)
- Input validation at system boundaries
- Injection prevention (parameterized queries, ORM)
- XSS prevention (proper escaping, CSP headers)
- CSRF protection where applicable
- Authentication & authorization checks

### Code Quality
- Type safety (use the language's type system fully)
- Single Responsibility Principle
- DRY (Don't Repeat Yourself) - but avoid premature abstraction
- Clear naming (no `data`, `info`, `manager`, `handler` - be specific)
- Self-documenting code > comments (add comments only when "why" isn't obvious)

### Testing Requirements
- Unit tests for business logic
- Integration tests for critical flows
- E2E tests for user journeys
- Coverage >= 80% for new/changed code
- No flaky tests (fix or remove)

---

## Documentation Standards

### When to Document
- Complex algorithms (explain the "why")
- Non-obvious business rules
- Security-critical code
- Public APIs

### When NOT to Document
- Self-explanatory code
- Obvious getters/setters
- Generated code
- Test descriptions (test names should be descriptive)

---

## Anti-Patterns to Avoid

### Hallucination (AI Tools)
- **Never** suggest APIs/patterns without verifying they exist
- **Always** cite specific file paths and line numbers
- **Request clarification** when uncertain vs. guessing

### Over-Engineering
```
BAD:  Abstract base class with one implementation
GOOD: Direct implementation, extract later if needed (rule of three)
```

### Unclear Naming
```
BAD:  handle_data(info), DataManager, process()
GOOD: validate_user_email(email), EmailValidator, parse_csv_row()
```

### Magic Numbers
```
BAD:  if len(password) < 8
GOOD: MIN_PASSWORD_LENGTH = 8; if len(password) < MIN_PASSWORD_LENGTH
```

---

## Cross-Project Patterns

### API Communication
- Use DTOs/schemas for request/response
- Version APIs (`/api/v1/...`, `/api/v2/...`)
- Consistent error response format:
```json
{
  "error": "resource_not_found",
  "message": "User with ID 123 not found",
  "details": {}
}
```

### Authentication
- JWT tokens with refresh tokens
- Access token: 15 minutes expiry
- Refresh token: 7 days expiry
- Secure httpOnly cookies for web apps

### Rate Limiting
- AI/expensive endpoints: 1 request/30 seconds per user
- Auth endpoints: 5 requests/minute per IP
- Standard endpoints: 100 requests/minute per user

---

## Environment Management

### Secrets
- **Never** commit secrets (`.env` in `.gitignore`)
- Use a secrets manager for production
- Use `.env.example` for documentation
- Rotate secrets regularly

### Environment Variables
```bash
# Required for all environments
DATABASE_URL="..."
API_SECRET_KEY="..."
JWT_SECRET_KEY="..."
```

---

## Development Workflow

### Starting Work
1. Check project-specific CLAUDE.md for tech-stack conventions
2. Check project-specific agents for specialized capabilities
3. Use global agents for general tasks (debugging, navigation, refactoring)

### Making Changes
1. Read existing code first
2. Follow established patterns in the codebase
3. Write tests for new functionality
4. Run linters and formatters
5. Commit with semantic commit message

### Before PR
- [ ] All tests pass
- [ ] Linters pass
- [ ] No debug statements left
- [ ] No commented-out code
- [ ] Secrets not committed
- [ ] Documentation updated (if public API changed)

---

## How This Setup Works

This repo is intended to be symlinked into project `.claude/` directories:

```
~/dev/
├── claude-dev-assist/          # This repo
│   ├── CLAUDE.md               # This file (global conventions)
│   ├── agents/                 # Global agents
│   └── skills/                 # Global skills
├── project-a/
│   └── .claude/
│       ├── CLAUDE.md           # Symlink or project-specific overrides
│       ├── agents/ → symlink   # Or project-specific agents
│       └── skills/ → symlink
└── project-b/
    └── .claude/
        ├── CLAUDE.md           # Project-specific conventions
        └── agents/ → symlink
```

### How Claude Code Resolves Configuration

**CLAUDE.md**: Merges parent + child (both are read)
**agents/**: Parent agents always available + child agents when in that project
**settings.local.json**: Child overrides parent (permissions are merged)

### Adding New Conventions
- Cross-project → Update this file
- Tech/project-specific → Update the project's own CLAUDE.md
