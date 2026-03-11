---
name: debug
description: Structured debugging methodology. Use when user says /debug, is troubleshooting an error, investigating unexpected behavior, or needs help diagnosing a bug. Provides systematic hypothesis-driven debugging workflow covering frontend, backend, database, infrastructure, and full-stack issues.
---

# Structured Debugging

Systematic approach to diagnosing and fixing bugs.

## Usage

```
/debug                 # Start debugging workflow for the current issue
/debug <error message> # Debug a specific error
/debug <file:line>     # Debug starting from a specific location
```

## Debugging Workflow

```
1. REPRODUCE  → Confirm the bug and capture exact error
2. GATHER     → Collect context (logs, stack traces, recent changes)
3. HYPOTHESIZE → Form ranked hypotheses
4. ISOLATE    → Narrow down with targeted experiments
5. FIX        → Apply minimal fix
6. VERIFY     → Confirm fix + no regressions
7. PREVENT    → Add test, improve error message, or document
```

## Phase 1: Reproduce

- Get the exact error message, stack trace, or unexpected behavior description
- Identify the minimal steps to reproduce
- Note: does it happen consistently or intermittently?
- Check: is it environment-specific (dev vs prod, OS, versions)?

## Phase 2: Gather Context

### Frontend / Client-Side
- **Browser console**: JavaScript errors, warnings, failed assertions
- **Network tab**: Which endpoint was called? What HTTP method, headers, query params?
- **Request payload**: Is the frontend sending the correct data shape and types?
- **Response body + status**: What did the server actually return? 200 with wrong data? 4xx? 5xx?
- **Response headers**: CORS issues, auth token presence, content-type mismatches
- **Timing**: Is this a race condition where the UI renders before the API responds?
- **State**: What does the application state (stores, signals, context) look like at failure?

### Backend / Server-Side
- **Stack trace**: Full trace, not just the error line
- **Server logs**: Search for errors around the timestamp of the failure
- **Request context**: What endpoint was hit? What were the params, headers, body?
- **Authentication/Authorization**: Is the user/token valid? Are permissions correct?
- **Service layer**: Trace through the handler → service → data access chain
- **External calls**: Are downstream APIs/services responding? Check timeouts, retries

### Database / Data Layer
- **Query logs**: Enable query logging to see what's actually being executed
- **Slow queries**: Check for N+1 queries, missing indexes, full table scans
- **Connection pool**: Are connections exhausted? Check pool size and active connections
- **Deadlocks**: Check for competing transactions locking the same rows
- **Data integrity**: Are foreign keys, constraints, or uniqueness violations involved?
- **Migration state**: Is the schema in the expected state? Any failed/partial migrations?
- **Stale data**: Is caching returning outdated results?

### Infrastructure / Systems
- **Resource exhaustion**: CPU, memory, disk, file descriptors — is anything maxed out?
- **Container/process health**: Is the service running? Has it been OOM-killed or restarted?
- **Network**: DNS resolution, firewall rules, security groups, proxy configs
- **Environment variables**: Are all required config values set correctly in this environment?
- **Certificates/TLS**: Expired certs, self-signed certs, TLS version mismatches
- **Logs**: Check orchestrator logs (Docker, Kubernetes, systemd) for restarts or failures
- **Dependencies**: Are external services (cache, queue, storage, AI APIs) reachable?

### Full-Stack (When Multiple Layers Interact)
- **Trace end-to-end**: Browser → network request → server handler → service → DB → response → frontend handler → UI
- **Identify which boundary the bug crosses**: Is the frontend sending the wrong thing, or is the backend returning the wrong thing?
- **Check serialization**: Field name mismatches (camelCase vs snake_case), missing fields, type coercion
- **Check auth flow**: Token present in request? Token valid? Permissions correct?

### Targeted Search
- Grep for the error message in the codebase to find where it's thrown
- Grep for the failing function/method to understand all callers
- Read the full function/module where the error occurs (not just the error line)
- Check test files — is there a test that should have caught this?

### Recent Changes
- `git log --oneline -20` and `git diff` — did a recent change cause this?
- Check dependency lockfile diffs — any version bumps?
- Check config/environment changes

## Phase 3: Hypothesize

Form **3-5 hypotheses** ranked by likelihood. For each:

```
Hypothesis: [What you think is wrong]
Evidence for: [What supports this]
Evidence against: [What contradicts this]
Test: [How to confirm or eliminate]
```

**Common root causes** (check these first):
- Null/undefined/None where a value was expected
- Off-by-one or boundary condition
- Race condition or timing issue
- State mutation from an unexpected caller
- Missing error handling on external call
- Type mismatch (especially across serialization boundaries)
- Configuration/environment mismatch
- Recent dependency update with breaking change
- Stale cache or stale compiled output
- Connection pool exhaustion or resource leak
- Permissions/IAM misconfiguration
- DNS or network routing issue

## Phase 4: Isolate

Test hypotheses from most to least likely:

- **Add logging/prints** at key decision points to trace execution flow
- **Check inputs** at the failing function — are they what you expect?
- **Bisect**: If a recent change caused it, use `git bisect` or manually check recent commits
- **Simplify**: Remove complexity until the bug disappears, then add back to find the trigger
- **Compare**: If it works in one environment but not another, diff the configurations
- **Reproduce in isolation**: Can you reproduce with a minimal test case or curl command?

**Stop as soon as you find the root cause.** Don't keep investigating after you have confirmation.

## Phase 5: Fix

- Apply the **minimal fix** that addresses the root cause
- Don't fix symptoms — fix the underlying issue
- Don't refactor surrounding code in the same change
- If the fix is complex, explain the root cause in a code comment

## Phase 6: Verify

- Confirm the original error no longer occurs
- Run the full test suite — no regressions
- If the bug was environment-specific, verify in that environment
- Check related code paths — could the same bug exist elsewhere?

## Phase 7: Prevent

- **Add a regression test** that reproduces the original bug and verifies the fix
- **Improve the error message** if the original was unclear
- **Grep for similar patterns** — fix all instances, not just the one reported
- **Consider a lint rule** if this is a class of error that could recur
- **Add monitoring/alerting** if this class of failure should be caught early

## Quick Reference: Error Categories

| Error Type | First Things to Check |
|------------|----------------------|
| Null/undefined | Trace where the value comes from, check all branches |
| Type error | Check serialization boundaries, API contracts |
| Timeout | External dependencies, connection pools, query performance |
| Auth failure | Token expiry, permissions, environment config |
| 500 / unhandled | Full stack trace, error handling gaps |
| CORS error | Server CORS config, request origin, preflight headers |
| 404 on API call | URL construction, route registration, proxy/base path config |
| Wrong data in UI | Network tab response body, then frontend state binding |
| Intermittent | Race conditions, caching, connection pools, resource limits |
| Works locally, fails in CI/prod | Env vars, file paths, dependency versions, resource limits |
| Connection refused | Service running? Port correct? Firewall/security group rules? |
| OOM / crash loop | Memory leaks, unbounded data structures, resource limits |
| Slow response | Query performance, N+1, missing indexes, external call latency |
| Deadlock | Competing transactions, lock ordering, connection pool exhaustion |

## Anti-Patterns

- **Shotgun debugging**: Making random changes hoping something works
- **Printf and pray**: Adding logs everywhere instead of forming hypotheses
- **Fixing symptoms**: Catching/suppressing the error instead of fixing the cause
- **Scope creep**: Refactoring unrelated code while debugging
- **Assuming**: "It can't be X" without actually checking X
- **Ignoring infrastructure**: Only looking at code when the issue is environmental
