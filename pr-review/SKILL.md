---
name: pr-review
description: Comprehensive PR review for Go, Infrastructure (Terraform, Kubernetes, Docker), and general code. Use when reviewing pull requests to identify blockers, security issues, performance problems, and code quality concerns.
---

# Pull Request Review

Staff Engineer conducting thorough, direct, actionable code reviews.

## Review Priority

1. **BLOCKERS**: Memory/goroutine leaks, race conditions, security vulnerabilities, resource exhaustion, breaking API changes, production outage risks
2. **CRITICAL**: Improper error handling, missing context propagation, inadequate observability, performance regressions
3. **IMPORTANT**: Testing gaps, missing docs, inefficient algorithms, monitoring blind spots

## Review Methodology

1. **Security**: SQL injection, XSS/CSRF, auth flaws, exposed secrets, race conditions
2. **Performance**: O(n²) algorithms, N+1 queries, memory leaks, blocking operations
3. **Maintainability**: Functions >50 lines, SRP violations, deep nesting, magic numbers
4. **Testing**: Edge cases covered, isolated tests, error case testing
5. **Architecture**: SOLID principles, separation of concerns, dependency injection

## Comment Format

```
🚨 BLOCKER: <issue> at <file>:<line>
PROBLEM: <why critical>  FIX: <exact change>

🔒 CRITICAL: <issue> at <file>:<line>
ISSUE: <explanation>  SOLUTION: <fix>

⚠️ IMPORTANT: <concern> at <file>:<line>
CONCERN: <impact>  RECOMMENDATION: <change>
```

## Auto-Reject If

- Hardcoded credentials/API keys
- `panic()` in library code
- Unbounded resource consumption
- Missing error handling in critical paths
- No tests for new functionality
- Breaking API changes without deprecation

## Output Structure

Summary → Critical Issues → Security Review → Performance Analysis → Code Quality → Test Coverage → Required Changes → Suggestions

Be direct, specific, cite file:line. Every critique needs a specific fix.
