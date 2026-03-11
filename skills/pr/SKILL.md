---
name: pr
description: Create GitHub pull requests with consistent formatting. Use when user says /pr, wants to create a PR, or needs help with PR workflow. Handles branch state checks, PR creation, and issue linking.
---

# Pull Request Creation

Create PRs with consistent formatting and optional issue linking.

## Workflow

1. **Check branch state** - Verify current branch and uncommitted changes
2. **Ask for base branch** - Always prompt user to specify target branch
3. **Analyze commits** - Review commit history since diverging from base
4. **Extract issue references** - Find issue patterns in branch name or commits (e.g., #123, PROJ-456)
5. **Create PR** - Use `gh pr create` with standard format

## Execution Steps

```bash
# 1. Get current state
git branch --show-current
git status
git log --oneline origin/main..HEAD  # adjust base as needed

# 2. Ask user for base branch
# Suggest: main, master, develop (based on what exists)

# 3. Get diff stats
git diff --stat <base>..HEAD

# 4. Extract issue references
# Check branch name and commit messages for patterns like #123, PROJ-456, AB#789

# 5. Generate PR title (semantic format)
# type(scope): summary

# 6. Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
<body from template>
EOF
)"
```

## PR Format

See [references/pr-template.md](references/pr-template.md) for the full template.

**Title format:** `<type>(<scope>): <summary>`
- Types: feat, fix, refactor, chore, docs, test, perf, style, ci
- Max 72 characters

**Body sections:**
- Summary (bullet points)
- Test Plan (checklist)
- Related Issues (if found)

## Error Handling

| Scenario | Action |
|----------|--------|
| No commits to push | Inform user, abort |
| Issue reference not found | Create PR, note no linked issue |
| PR creation fails | Show error, suggest fixes |
| Branch not pushed | Push with `git push -u origin <branch>` first |
