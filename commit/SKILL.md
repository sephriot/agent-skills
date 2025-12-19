---
name: commit
description: Generate conventional commit messages from staged changes and create the commit. Use when committing code changes to ensure consistent, well-formatted commit messages.
---

# Git Commit Message Generator

You are an expert software engineer that generates meaningful conventional commit messages based on staged changes.

## Workflow

1. Check for staged changes using `git diff --cached`
2. If no staged changes, check `git status` and ask user what to stage
3. Analyze the diff to understand the changes
4. Generate a commit message following the guidelines below
5. Create the commit

---

## Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/) format:

```
type(scope): description

- bullet point 1
- bullet point 2
- bullet point 3
```

### Supported Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `chore` | Maintenance tasks, dependencies |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or updating tests |
| `docs` | Documentation only changes |
| `style` | Formatting, whitespace (no code change) |
| `perf` | Performance improvements |
| `ci` | CI/CD configuration changes |
| `build` | Build system or external dependencies |

### Scope

- Should be the module, package, or file affected
- Use lowercase
- Keep it short and identifiable
- Examples: `auth`, `api`, `db`, `config`, `tests`

### Description

- Use imperative mood ("add feature" not "added feature")
- Keep under 50 characters
- No period at the end
- Be specific about what changed

### Body (Bullet Points)

- Maximum 5 bullet points
- Each bullet describes a specific change
- Include test additions/updates if applicable
- Focus on "what" not "how"

---

## Examples

### Feature Addition
```
feat(auth): add OAuth2 login support

- Add Google OAuth2 provider integration
- Create OAuth callback handler
- Add session token generation
- Update user model with OAuth fields
- Add integration tests for OAuth flow
```

### Bug Fix
```
fix(api): prevent null pointer on empty response

- Add nil check before accessing response body
- Return appropriate error for empty responses
- Add unit test for empty response case
```

### Refactoring
```
refactor(db): extract connection pooling logic

- Move pool configuration to dedicated module
- Add connection health check function
- Simplify retry logic in query executor
```

### Minor/Formatting
```
style(lint): fix golangci-lint warnings

- Fix unused variable warnings
- Update deprecated function calls
- Format files with gofmt
```

---

## Commit Command

Always use HEREDOC for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
type(scope): description

- bullet point 1
- bullet point 2
EOF
)"
```

---

## Guidelines

1. **Analyze the diff thoroughly** - Understand all changes before writing
2. **Be accurate** - "add" means new, "update" means modified, "fix" means bug fix
3. **Be concise** - Focus on the most important changes
4. **Be consistent** - Follow the same style throughout
5. **Don't over-explain** - The diff shows the "how", commit explains "what" and "why"

---

## Important Rules

- **NEVER** commit files containing secrets (.env, credentials.json, etc.)
- **NEVER** use vague messages like "fix bug" or "update code"
- **ALWAYS** check `git diff --cached` before generating message
- **ALWAYS** use imperative mood in description
- If the change is purely formatting/whitespace, use `style` or `chore` type
