---
name: debug
description: Structured debugging methodology. Use when user says /debug, is troubleshooting an error, investigating unexpected behavior, or needs help diagnosing a bug. Provides systematic hypothesis-driven debugging workflow.
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

### Backend / Server-Side
- **Stack trace**: Full trace, not just the error line
- **Server logs**: Search for errors around the timestamp of the failure
- **Recent changes**: `git log --oneline -20` and `git diff` — did a recent change cause this?
- **Configuration**: Environment variables, config files, feature flags
- **Dependencies**: Any recent version changes? (`git diff <lockfile>`)

### Frontend / Client-Side
- **Browser console**: JavaScript errors, warnings, failed assertions
- **Network tab**: Which endpoint was called? What HTTP method, headers, query params?
- **Request payload**: Is the frontend sending the correct data shape and types?
- **Response body + status**: What did the server actually return? 200 with wrong data? 4xx? 5xx?
- **Response headers**: CORS issues, auth token presence, content-type mismatches
- **Timing**: Is this a race condition where the UI renders before the API responds?
- **State**: What does the application state (Redux, signals, stores) look like at the point of failure?

### Full-Stack (When Frontend + Backend Interact)
- **Trace the request end-to-end**: Browser → network request → server handler → service → DB → response → frontend handler → UI
- **Identify which boundary the bug crosses**: Is the frontend sending the wrong thing, or is the backend returning the wrong thing?
- **Check serialization**: Field name mismatches (camelCase vs snake_case), missing fields, type coercion issues
- **Check auth flow**: Token present in request? Token valid? Permissions correct?

### Targeted search
- Grep for the error message in the codebase to find where it's thrown
- Grep for the failing function/method to understand all callers
- Read the full function/module where the error occurs (not just the error line)
- Check test files — is there a test that should have caught this?

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

## Phase 4: Isolate

Test hypotheses from most to least likely:

- **Add logging/prints** at key decision points to trace execution flow
- **Check inputs** at the failing function — are they what you expect?
- **Bisect**: If a recent change caused it, use `git bisect` or manually check recent commits
- **Simplify**: Remove complexity until the bug disappears, then add back to find the trigger
- **Compare**: If it works in one environment but not another, diff the configurations

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

## Quick Reference: Error Categories

| Error Type | First Things to Check |
|------------|----------------------|
| Null/undefined | Trace where the value comes from, check all branches |
| Type error | Check serialization boundaries, API contracts |
| Timeout | Check external dependencies, connection pools, query performance |
| Auth failure | Check token expiry, permissions, environment config |
| 500 / unhandled | Read the full stack trace, check error handling |
| CORS error | Check server CORS config, request origin, preflight headers |
| 404 on API call | Check URL construction, route registration, proxy/base path config |
| Wrong data in UI | Check network tab response body, then trace frontend state binding |
| Intermittent | Look for race conditions, caching, connection pools |
| Works locally, fails in CI/prod | Environment vars, file paths, dependency versions |

## Anti-Patterns

- **Shotgun debugging**: Making random changes hoping something works
- **Printf and pray**: Adding logs everywhere instead of forming hypotheses
- **Fixing symptoms**: Catching/suppressing the error instead of fixing the cause
- **Scope creep**: Refactoring unrelated code while debugging
- **Assuming**: "It can't be X" without actually checking X
