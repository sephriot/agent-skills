---
name: create-pr
description: Create GitHub pull requests with proper workflow - branch management, draft creation, template completion, self-review, and auto-merge setup. Use when opening new PRs.
---

# Create Pull Request

Create GitHub PRs following structured workflow with proper branch management.

## Workflow

1. **Branch**: If on `main`/`master`, create feature branch first
2. **Context**: Run `git status`, `git diff`, `git log main..HEAD --oneline`
3. **Draft PR**: Check for `.github/pull_request_template.md`, create as draft
4. **Description**: Purpose (why), Approach (how), Impact (what changes), Testing, Dependencies
5. **Create**: `git push -u origin HEAD && gh pr create --draft`
6. **Self-review**: Review diff, check template sections, verify tests pass
7. **Finalize**: `gh pr ready && gh pr edit --add-reviewer michalg9,michalrom089,truszkowski,rekfuki && gh pr merge --auto --squash`

## PR Description Guidelines

- Lead with the problem being solved
- Use present tense
- Be specific without overwhelming detail
- Avoid file listings (GitHub diff handles this)

## Rules

- **NEVER** commit directly to main/master
- **ALWAYS** create as draft first
- **ALWAYS** conduct self-review before marking ready
- **ALWAYS** enable auto-merge after marking ready
- Return PR URL when done
