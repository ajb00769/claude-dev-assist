---
name: code-review
description: Code review checklist for security, performance, and best practices. Use when user says /code-review, wants to review a PR, needs to check code quality before merging, or wants to post inline review comments on a GitHub PR.
---

# Code Review Checklist

Systematic review for security, performance, and maintainability.

## Review Discipline

**Verify, don't speculate.** If a review finding raises a question about the broader codebase (e.g. "are there other endpoints that have this issue?"), use Grep/Glob to check before commenting. Post definitive findings, not suggestions to "do a quick grep." The reviewer should provide certainty so the PR author gets clear, actionable guidance.

## Quick Review Order

1. **Security** - Can this be exploited?
2. **Correctness** - Does it work?
3. **Performance** - Is it efficient?
4. **Maintainability** - Can others understand it?

## Security Checklist

### Authentication & Authorization
- [ ] All endpoints require appropriate auth
- [ ] User can only access their own resources
- [ ] Sensitive operations require re-authentication
- [ ] Tokens validated correctly

### Input Validation
- [ ] All user inputs validated
- [ ] Queries use parameterized inputs (no string interpolation)
- [ ] File uploads validate type and size
- [ ] URLs validated before fetching

### Data Protection
- [ ] No secrets in code or logs
- [ ] Sensitive data not exposed in responses
- [ ] PII not logged
- [ ] Error messages don't leak internals

### Rate Limiting
- [ ] Expensive endpoints rate limited
- [ ] Auth endpoints protected from brute force

## Performance Checklist

### Database
- [ ] No N+1 queries (use eager loading)
- [ ] Batch operations used for bulk writes
- [ ] Indexes on frequently queried columns
- [ ] Pagination for large result sets

### Code Efficiency
- [ ] No unnecessary loops
- [ ] No blocking operations in async code
- [ ] Expensive computations cached if repeated
- [ ] No unbounded data structures

## Correctness Checklist

### Logic
- [ ] Edge cases handled (empty, null, max values)
- [ ] Error handling on external calls
- [ ] Transactions for atomic operations
- [ ] Correct HTTP status codes / error codes

### Types
- [ ] Type annotations on public functions
- [ ] Input schemas validate inputs
- [ ] Return types match schema

## Maintainability Checklist

### Code Quality
- [ ] Functions do one thing
- [ ] Clear naming (no abbreviations)
- [ ] No magic numbers
- [ ] Complex logic has comments explaining WHY

### Architecture
- [ ] Business logic in services (not route handlers)
- [ ] Follows existing patterns in codebase
- [ ] No circular imports/dependencies
- [ ] Tests for critical paths

## Review Comments Template

### Critical (Block Merge)
```
**Critical:** [Issue]

**Location:** `file.py:42`
**Impact:** [Security/Data loss/Breaking change]
**Fix:** [What to do]
```

### Suggestion (Non-blocking)
```
**Suggestion:** [Improvement]

**Location:** `file.py:42`
**Reason:** [Why this is better]
```

### Question
```
**Question:** [What you're unsure about]

**Location:** `file.py:42`
```

## Review Workflow

**Always present findings to the user first before posting to GitHub.** Show:
1. Summary of what passed the checklist
2. Each finding with the relevant code snippet, file path, and line number
3. Your reasoning for why it's an issue

Wait for the user to review and approve the findings. Only then post inline PR comments.

## Posting Inline PR Comments

After the user approves findings, post them on the PR using `gh api`.

### Step 1: Get the HEAD commit SHA

```bash
gh pr view <PR_NUMBER> --repo <OWNER/REPO> --json headRefOid --jq '.headRefOid'
```

### Step 2: Post inline review comments

```bash
gh api repos/<OWNER/REPO>/pulls/<PR_NUMBER>/reviews \
  --method POST \
  --input - <<'EOF'
{
  "commit_id": "<HEAD_COMMIT_SHA>",
  "event": "COMMENT",
  "body": "Code review summary here.",
  "comments": [
    {
      "path": "src/example.py",
      "line": 42,
      "body": "**Critical:** Issue description here.\n\n**Fix:** What to do."
    }
  ]
}
EOF
```

### Step 3: Post general comments for non-diff items

```bash
gh pr comment <PR_NUMBER> --repo <OWNER/REPO> --body "$(cat <<'EOF'
### Additional review notes

**Issue title**

Description of the concern and recommendation.
EOF
)"
```

### Tips
- Multiple inline comments can be batched in a single review (one API call)
- Use markdown formatting in comment bodies for readability
- `event` — use `COMMENT` for neutral, `APPROVE` to approve, `REQUEST_CHANGES` to block

## Before Approving

- [ ] Tests pass
- [ ] No security issues
- [ ] No obvious bugs
- [ ] Follows project patterns
- [ ] No unnecessary complexity
