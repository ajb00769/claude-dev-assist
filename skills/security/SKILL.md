---
name: security
description: Security audit for code and cloud infrastructure. Use when user says /security, wants a security review, needs vulnerability assessment, or asks about OWASP compliance. Covers application security (OWASP Top 10) and cloud infrastructure security.
---

# Security Audit

Comprehensive security review covering application code and cloud infrastructure.

## Usage

```
/security              # Inline findings (default)
/security --report     # Full markdown report
/security <path>       # Audit specific file or directory
```

## Audit Workflow

1. **Scope determination** - Identify target (file, directory, or full codebase)
2. **Code audit** - Check OWASP Top 10, secrets, auth patterns
3. **Infrastructure audit** - Review cloud configs, IAM, networking (if applicable)
4. **Report findings** - Output based on format preference

## Code Security Checks

### Critical (Immediate Fix Required)

| Issue | What to Look For | Fix |
|-------|-----------------|-----|
| Hardcoded secrets | Passwords, API keys, tokens in source code | Use secrets manager / env vars |
| Injection | Raw string queries, f-strings/interpolation in SQL/commands | Parameterized queries, safe APIs |
| Command injection | `os.system()`, `exec()`, `eval()` with user input | Sanitize inputs, use safe APIs |
| Path traversal | User input in file paths | Validate/sanitize paths |

### High Priority

| Issue | What to Look For | Fix |
|-------|-----------------|-----|
| Missing auth | Endpoints without authentication | Add auth middleware/decorators |
| Missing rate limiting | Expensive endpoints without limits | Add rate limiting |
| Sensitive data in logs | PII, tokens, passwords in log statements | Redact sensitive data |
| CORS misconfiguration | `allow_origins=["*"]` or equivalent | Restrict to known domains |

### Medium Priority

| Issue | What to Look For | Fix |
|-------|-----------------|-----|
| Missing input validation | No schema validation on inputs | Add validation layer |
| Verbose error messages | Stack traces exposed to users | Generic error responses |
| Missing CSRF protection | State-changing endpoints without CSRF | Add CSRF tokens |
| Insecure cookies | Missing `httponly`, `secure` flags | Set secure cookie flags |

See [references/owasp-checklist.md](references/owasp-checklist.md) for full OWASP Top 10 checklist.

## Infrastructure Security Checks

If cloud configuration files are present (Terraform, Bicep, CloudFormation, Kubernetes manifests, Docker configs):

### Common Issues

| Issue | Check | Fix |
|-------|-------|-----|
| Public storage | Public access enabled on buckets/blobs | Set to private |
| Overly permissive IAM | Admin/Owner roles at broad scope | Use least privilege |
| Missing encryption | Encryption at rest disabled | Enable encryption |
| Open network rules | `0.0.0.0/0` on sensitive ports | Restrict IP ranges |
| Missing secrets management | Secrets in env vars or config files | Use secrets manager |

## Output Formats

### Inline (Default)

```
## Security Findings

**Critical (1)**
- [file.py:42](path) - Hardcoded API key detected

**High (2)**
- [routes/ai.py:15](path) - Missing rate limiting on expensive endpoint
- [config/db.py:8](path) - Database connection string in code

**Medium (1)**
- [main.py:23](path) - CORS allows all origins
```

### Full Report (--report)

```markdown
# Security Audit Report

**Date:** YYYY-MM-DD
**Scope:** <path or "full codebase">
**Auditor:** Claude Code

## Executive Summary
- Critical: X findings
- High: X findings
- Medium: X findings
- Low: X findings

## Critical Findings

### 1. [Title]
**Location:** `file:line`
**Description:** ...
**Impact:** ...
**Recommendation:** ...

## Recommendations
1. ...
2. ...
```

## Execution Steps

1. Determine scope from user input or default to codebase root
2. Run code security checks (grep for patterns, analyze code)
3. If cloud/infra configs exist, run infrastructure checks
4. Categorize findings by severity
5. Output in requested format (inline or report)
