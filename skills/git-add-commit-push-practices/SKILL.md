---
name: git-add-commit-push-practices
description: Guides agents on Git best practices for staging changes (add), creating commits with clear messages (commit), and pushing code safely (push). Use when an agent needs to perform or advise on Git add-commit-push workflows.
---

# GitHub Best Practices: Add, Commit, Push

This skill provides guidance on the fundamental Git workflow: staging changes, creating commits, and pushing to remote repositories. Follow these best practices to maintain clean commit history and avoid common pitfalls.

## When to Use This Skill

**Use this skill when:**
- Preparing to stage and commit code changes
- Writing commit messages
- Pushing changes to a remote repository
- Advising users on Git workflow best practices
- Reviewing changes before committing

**Do not use this skill for:**
- Branching, merging, or rebasing operations
- Pull request creation or review
- Repository initialization or configuration
- Git history manipulation beyond basic commits

## Git Add: Staging Changes

### Best Practices

1. **Stage selectively**: Review and stage only the changes relevant to your current commit
   - Use `git add <specific-file>` for individual files
   - Use `git add <directory>` for entire directories
   - Avoid `git add .` or `git add -A` without first reviewing what will be staged

2. **Review before staging**: Always check what you're about to stage
   - Use `git status` to see modified, new, and deleted files
   - Use `git diff` to review unstaged changes
   - Use `git diff --staged` to review staged changes

3. **Use interactive staging** for complex changes: `git add -p` allows you to stage hunks selectively

4. **Check for sensitive data** before staging:
   - API keys, tokens, passwords
   - Private keys or certificates
   - Personal identifiable information (PII)
   - Internal URLs or system paths
   - Configuration files with credentials

5. **Avoid staging**:
   - Build artifacts (compiled code, binaries)
   - Dependency directories (node_modules, venv, vendor)
   - IDE configuration files (unless team-standardized)
   - Log files and temporary files
   - Large binary files without proper LFS setup

6. **Use .gitignore**: Ensure your repository has a proper .gitignore file to prevent accidental staging of unwanted files

### Common Mistakes to Avoid

- Staging everything blindly without review
- Committing sensitive credentials or API keys
- Adding large binary files that bloat repository size
- Staging unrelated changes together (breaks atomic commit principle)

## Git Commit: Creating Meaningful Commits

### Best Practices

1. **Write clear, descriptive commit messages** following this structure:
   ```
   <type>: <short summary in present tense>
   
   <optional detailed description>
   
   <optional footer with issue references>
   ```

2. **Use conventional commit types**:
   - `feat:` A new feature
   - `fix:` A bug fix
   - `docs:` Documentation changes
   - `style:` Code style changes (formatting, no logic change)
   - `refactor:` Code refactoring (no feature change or bug fix)
   - `test:` Adding or updating tests
   - `chore:` Maintenance tasks, dependency updates
   - `perf:` Performance improvements

3. **Write good commit summaries**:
   - Keep first line under 50-72 characters
   - Use imperative mood: "Add feature" not "Added feature" or "Adds feature"
   - Be specific: "Fix login validation error" not "Fix bug"
   - Don't end with a period

4. **Add detailed descriptions when needed**:
   - Leave a blank line after the summary
   - Explain the "why" not just the "what"
   - Reference related issues: "Fixes #123" or "Related to #456"

5. **Make atomic commits**: Each commit should represent one logical change
   - Don't mix unrelated changes in one commit
   - If you can't describe the commit in one sentence, it's probably too large

6. **Verify before committing**:
   - Review staged changes: `git diff --staged`
   - Ensure all intended changes are staged
   - Check that no unintended changes are included

### Commit Message Examples

**Good:**
```
feat: add user authentication endpoint

Implements JWT-based authentication for the /api/login endpoint.
Includes token generation, validation, and refresh functionality.

Fixes #234
```

**Good (simple):**
```
fix: resolve null pointer in user profile loader
```

**Poor:**
```
fixed stuff
```

**Poor:**
```
Updated files, added new feature, fixed some bugs, and refactored code
```

### Common Mistakes to Avoid

- Vague messages: "Update file" or "Fix issue"
- Mixing multiple unrelated changes in one commit
- Committing commented-out code or debug statements
- Committing broken or untested code
- Not reviewing staged changes before committing

## Git Push: Sending Changes to Remote

### Best Practices

1. **Verify your current branch** before pushing:
   - Use `git branch` or `git status` to confirm you're on the correct branch
   - Never push directly to protected branches (main, master, production) unless explicitly authorized

2. **Pull before push** to avoid conflicts:
   - Use `git pull` or `git fetch` + `git merge` to update your local branch
   - Resolve any merge conflicts locally before pushing

3. **Review what you're about to push**:
   - Use `git log origin/<branch>..<branch>` to see commits that will be pushed
   - Verify commit messages and content

4. **Use the appropriate push command**:
   - Standard push: `git push origin <branch-name>`
   - First-time push with upstream: `git push -u origin <branch-name>`
   - Force push (DANGEROUS): Only when absolutely necessary and after team coordination

5. **Understand force push risks**:
   - `git push --force` or `git push -f` **overwrites remote history**
   - Can cause data loss for other team members
   - Only use when:
     - Working on a personal feature branch
     - After explicit team coordination
     - You understand the consequences
   - Prefer `git push --force-with-lease` as a safer alternative

6. **Handle push rejections properly**:
   - If push is rejected, pull and merge first
   - Don't immediately force push to override the rejection
   - Understand why the push was rejected before proceeding

### Common Mistakes to Avoid

- Pushing to the wrong branch
- Force pushing to shared branches
- Pushing without pulling recent changes first
- Pushing untested or broken code
- Pushing sensitive data (credentials, keys)
- Ignoring push rejection messages

## Security Checklist

Before any add, commit, or push operation, verify:

- [ ] No credentials, API keys, or tokens in code
- [ ] No private keys or certificates
- [ ] No hardcoded passwords or secrets
- [ ] No personal or sensitive data
- [ ] Environment variables used for sensitive configuration
- [ ] .gitignore properly configured
- [ ] No internal URLs or system paths exposed
- [ ] Large files handled properly (LFS if needed)

## Complete Workflow Example

```bash
# 1. Check status and review changes
git status
git diff

# 2. Stage specific files after review
git add src/authentication.py
git add tests/test_auth.py

# 3. Review staged changes
git diff --staged

# 4. Create meaningful commit
git commit -m "feat: add JWT authentication

Implements token-based authentication with refresh capability.
Includes validation and expiration handling.

Fixes #123"

# 5. Pull latest changes before pushing
git pull origin feature-branch

# 6. Push to remote
git push origin feature-branch
```

## Troubleshooting

### Staged wrong files
- Unstage specific file: `git reset HEAD <file>`
- Unstage all: `git reset HEAD`

### Need to modify last commit message
- Use: `git commit --amend` (before pushing)
- Never amend commits already pushed to shared branches

### Pushed sensitive data
- **Stop immediately**
- Rotate/revoke any exposed credentials
- Contact your team lead or security team
- Do not simply delete and re-commit; history may still contain the data

## Summary

The add-commit-push workflow is fundamental to Git. Success requires:
1. **Selective staging** - only stage relevant, reviewed changes
2. **Meaningful commits** - atomic changes with clear, descriptive messages
3. **Safe pushing** - verify branch, pull first, avoid force push on shared branches
4. **Security awareness** - always check for sensitive data before committing

When in doubt, review your changes one more time before pushing.
