---
name: commit
description: Generate conventional commit messages and create the actual commit. Analyzes staged changes (or all unstaged changes when nothing is staged, grouping them into logical commits). Use when user says "commit", "write a commit", "create commit", or "/commit".
---

# Commit Skill

Analyze changes, craft conventional commit messages, and create the actual git commit(s). Do not ask for confirmation before committing.

## Workflow

### When changes are staged (`git diff --staged` is non-empty):

1. Run `git diff --staged` to inspect staged changes
2. Draft a single commit message following the format below
3. Run `git commit -m "<message>"` to create the commit
4. Report the commit hash and subject line to the user

### When nothing is staged (`git diff --staged` is empty):

1. Run `git status` and `git diff` to inspect all unstaged/untracked changes
2. Group the changes into logical, independently-meaningful sets (e.g. by feature, fix, refactor, or file domain)
3. For each group:
   a. Stage the relevant files with `git add <files>`
   b. Draft a commit message for that group following the format below
   c. Run `git commit -m "<message>"` to create the commit
   d. Report the commit hash and subject line to the user
4. Prefer fewer, coherent commits over many tiny ones — only split when there is a clear logical boundary

## Format - Conventional Commits

- First line: type(scope): subject — under 72 characters, imperative mood ("add", not "added")
- Allowed types: feat, fix, docs, refactor, test, chore, perf, build, ci
- The scope is optional; use it when the change clearly belongs to one component
- If a work item is available, start the first line with the work item ID followed by a colon, then the conventional commit message
- Add a blank line and a short body explaining WHY the change was made when it is not obvious from the subject

## Output format

After each commit, output one line:
  <hash> <subject>

Do not include markdown headers, code fences, or explanatory prose beyond this.

Work Item (if available):
{workItem}
