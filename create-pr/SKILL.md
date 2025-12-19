---
name: create-pr
description: Create GitHub pull requests with proper workflow - branch management, draft creation, template completion, self-review, and auto-merge setup. Use when opening new PRs.
---

# Create Pull Request

You are an assistant that creates GitHub pull requests following a structured workflow. Your output should be a complete, well-structured pull request.

## Core Principles

1. **Formatting**: Use proper Markdown throughout
2. **Completeness**: Never skip or remove template sections
3. **Clarity**: Be concise, informative, and purpose-driven. Be specific and evidence-based.
4. **Context**: Leverage commit messages and file changes without listing files explicitly

---

## Pull Request Workflow

### Step 1: Branch Management

**CRITICAL**: If currently on `main` or `master`, create and switch to a new feature branch before making any commits.

```bash
# Check current branch
git branch --show-current

# If on main/master, create feature branch
git checkout -b feature/descriptive-name
```

### Step 2: Pre-Commit Checks

1. Review unstaged changes with `git status`
2. Review changes with `git diff`
3. Stage relevant changes
4. Commit with meaningful messages following conventional commits
5. **Never commit directly to `main` branch**

### Step 3: Gather Context

Run these commands in parallel to understand the state:

```bash
# See all untracked files
git status

# See staged and unstaged changes
git diff

# Check if branch tracks remote and is up to date
git status -sb

# Understand full commit history for the branch
git log main..HEAD --oneline
git diff main...HEAD
```

### Step 4: Draft PR Creation

1. Check for PR template at `.github/pull_request_template.md`
2. Create PR as **draft** initially for review and refinement
3. Fill every section of the template completely
4. Maintain original template structure

### Step 5: PR Description Best Practices

Structure the description to include:

- **Purpose**: Why this change is needed (business/technical rationale)
- **Approach**: High-level overview of the solution
- **Impact**: What changes for users, developers, or systems
- **Testing**: How the changes were validated
- **Dependencies**: Any related PRs, issues, or external changes

**Writing Guidelines:**
- Lead with the problem being solved
- Use present tense for descriptions
- Be specific about technical changes without overwhelming detail
- Avoid file listings (GitHub's diff view handles this)
- Include relevant context for reviewers

### Step 6: Create the PR

```bash
# Push branch with upstream tracking
git push -u origin HEAD

# Create draft PR
gh pr create --draft --title "the pr title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]
EOF
)"
```

### Step 7: Self-Review

After creating the draft:
1. Review the PR diff yourself
2. Check for obvious issues
3. Verify all template sections are complete
4. Ensure tests pass

### Step 8: Finalization

1. Convert from draft to "ready for review" after self-assessment
2. Add reviewers (teammates: michalg9, michalrom089, truszkowski, rekfuki for backend/exp-infra-as-intent repos)
3. **Enable automatic merge** with squash approach

```bash
# Mark ready for review
gh pr ready

# Add reviewers
gh pr edit --add-reviewer michalg9,michalrom089,truszkowski,rekfuki

# Enable auto-merge with squash
gh pr merge --auto --squash
```

---

## Quality Checklist

Before finalizing, verify:

- [ ] All template sections are complete and meaningful
- [ ] Title accurately reflects the change scope
- [ ] Description explains both "what" and "why"
- [ ] Checkboxes reflect actual status
- [ ] Professional, clear tone throughout
- [ ] Branch is up to date with main
- [ ] CI checks pass
- [ ] Self-review completed

---

## Important Rules

- **NEVER** commit directly to main/master
- **NEVER** skip PR template sections
- **ALWAYS** create as draft first
- **ALWAYS** conduct self-review before marking ready
- **ALWAYS** enable auto-merge after marking ready
- Return the PR URL when done so the user can see it
