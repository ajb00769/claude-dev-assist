# OWASP Top 10 Security Checklist

## A01:2021 - Broken Access Control

- [ ] All endpoints require authentication (unless explicitly public)
- [ ] Authorization checks verify user can access specific resource
- [ ] CORS configured with specific allowed origins (not `*`)
- [ ] Directory listing disabled
- [ ] Metadata and backup files not accessible
- [ ] Rate limiting on authentication endpoints
- [ ] Tokens validated on every request
- [ ] Session invalidation on logout

**Patterns to check:**
```
GOOD: Auth required on every protected route, ownership check in queries
BAD:  Routes without auth middleware, queries without user_id filter
```

## A02:2021 - Cryptographic Failures

- [ ] Sensitive data encrypted at rest
- [ ] TLS 1.2+ for all connections
- [ ] Passwords hashed with bcrypt/argon2 (not MD5/SHA1)
- [ ] Secrets stored in secrets manager (not in code or env files)
- [ ] No sensitive data in URLs or logs
- [ ] Proper key rotation policies

**Patterns to check:**
```
BAD:  Hardcoded secret, API_KEY = "sk-1234567890"
GOOD: Loaded from secrets manager or environment variable
```

## A03:2021 - Injection

- [ ] All SQL uses parameterized queries or ORM
- [ ] No string concatenation in queries
- [ ] User input validated before use in commands
- [ ] No `eval()` or `exec()` with user input
- [ ] LDAP queries use parameterized filters

**Patterns to check:**
```
BAD:  f"SELECT * FROM users WHERE id = {user_id}" or `${user_id}` in query
GOOD: ORM query with typed parameters, parameterized prepared statement
```

## A04:2021 - Insecure Design

- [ ] Threat modeling completed for critical features
- [ ] Rate limiting on expensive operations
- [ ] Business logic validated (not just input format)
- [ ] Fail-secure defaults
- [ ] Defense in depth (multiple layers)

## A05:2021 - Security Misconfiguration

- [ ] Debug mode disabled in production
- [ ] Default credentials changed
- [ ] Unnecessary features disabled
- [ ] Error messages don't leak stack traces
- [ ] Security headers configured (CSP, X-Frame-Options, etc.)
- [ ] Cloud resources use private endpoints where possible

## A06:2021 - Vulnerable Components

- [ ] Dependencies updated regularly
- [ ] No known vulnerabilities in dependencies
- [ ] Unused dependencies removed
- [ ] Software Bill of Materials (SBOM) maintained

**Check commands (varies by ecosystem):**
```bash
# Python: pip-audit, safety check
# Node.js: npm audit
# Go: govulncheck
# Rust: cargo audit
# Java: mvn dependency-check:check
```

## A07:2021 - Authentication Failures

- [ ] Strong password policy enforced
- [ ] Account lockout after failed attempts
- [ ] MFA available for sensitive operations
- [ ] Session timeout configured
- [ ] Secure session management (httponly, secure cookies)
- [ ] Password reset uses secure tokens

## A08:2021 - Data Integrity Failures

- [ ] Code and data integrity verified (checksums, signatures)
- [ ] CI/CD pipeline secured
- [ ] Deserialization of untrusted data avoided
- [ ] No unsafe deserialization with user input (pickle, yaml.load, etc.)
- [ ] Software updates verified before installation

## A09:2021 - Logging Failures

- [ ] All authentication events logged
- [ ] Access control failures logged
- [ ] Input validation failures logged
- [ ] Logs don't contain sensitive data (PII, passwords, tokens)
- [ ] Log injection prevented (sanitize user input in logs)
- [ ] Centralized logging configured
- [ ] Alerts for suspicious patterns

**Patterns to check:**
```
BAD:  logger.info(f"User login: {username}, password: {password}")
GOOD: logger.info(f"User login attempt: {username}")
```

## A10:2021 - SSRF

- [ ] URL validation on user-provided URLs
- [ ] Allowlist for external services
- [ ] No raw URL fetching with user input
- [ ] Internal metadata endpoints blocked

## Cloud Infrastructure Checks

### Secrets Management
- [ ] Secrets manager used (not hardcoded)
- [ ] Soft delete / versioning enabled
- [ ] Access policies follow least privilege
- [ ] Network access restricted

### Storage
- [ ] Public access disabled by default
- [ ] Encryption enabled
- [ ] Secure transfer required
- [ ] Network rules configured

### Databases
- [ ] Encryption at rest enabled
- [ ] Service account / managed identity auth preferred
- [ ] Firewall rules restricted
- [ ] Auditing enabled

### Compute / App Hosting
- [ ] HTTPS only
- [ ] Managed identity used (not connection strings with credentials)
- [ ] Latest runtime version
- [ ] Diagnostic logs enabled
