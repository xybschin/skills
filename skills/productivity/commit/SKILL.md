---
name: commit
description: Generate conventional commit messages and create the actual commit. Analyzes staged changes (or all unstaged changes when nothing is staged, grouping them into logical commits). Use when user says "commit", "write a commit", "create commit", or "/commit".
---

# Commit Skill

Analyze changes, craft conventional commit messages, and create the actual git commit(s). Require user confirmation before any commit is made.

## Workflow

### Tool invariance

These rules apply regardless of Git interface: CLI, Git MCP, IDE, or API. Tool availability changes only the interface used, never the workflow order or approval gates.

- Treat Git MCP operations such as add, commit, branch, checkout, and reset as equivalent to their CLI commands.
- Never invoke any commit-producing tool before showing the exact message(s) and receiving explicit approval.
- The initial request to "commit" is not approval of an unseen commit message.
- Use read-only Git operations to inspect state. Use mutating operations only at their corresponding approved workflow step.
- If the available Git interface cannot perform a required operation, use another available interface without skipping any rule.

### Pre-flight (always)

1. Detect the current branch using the available Git interface (CLI equivalent: `git rev-parse --abbrev-ref HEAD`)
2. Protected branches: `main`, `master`, `develop`, `release/*`, `hotfix/*` (also any listed in repo's branch protection if discoverable)
3. If the current branch is protected, create and switch to a new branch whose name reflects the changes:
   - Derive a short kebab-case slug from the dominant change type and subject (e.g. `feat/add-login`, `fix/cart-total`)
   - If multiple logical groups, use the primary one; do not branch per group
   - Create and switch to the branch using the available Git interface (CLI equivalent: `git checkout -b <name>`) and inform the user
   - Do not proceed with commits until off the protected branch

### When changes are staged (the staged diff is non-empty)

1. Inspect the staged changes using the available Git interface (CLI equivalent: `git diff --staged`)
2. Draft a single commit message following the format below
3. Present the message to the user and ask for confirmation
4. On approval, create the commit using the available Git interface (CLI equivalent: `git commit -m "<message>"`)
5. Report the commit hash and subject line

### When nothing is staged

1. Inspect all unstaged and untracked changes using the available Git interface (CLI equivalents: `git status` and `git diff`)
2. Group the changes into logical, independently-meaningful sets (by feature, fix, refactor, or file domain)
3. Present the proposed grouping and per-group messages to the user; ask for confirmation to proceed (allow editing)
4. For each approved group:
   a. Stage the relevant files using the available Git interface (CLI equivalent: `git add <files>`)
   b. Create the commit using the available Git interface (CLI equivalent: `git commit -m "<message>"`)
   c. Report the commit hash and subject line
5. Prefer fewer, coherent commits over many tiny ones — only split when there is a clear logical boundary

## Confirmation rule

Never commit without explicit user confirmation of the message(s). If the user only asked for a commit but has not seen/approved the message, show it and wait. A prior standing approval in the session satisfies this rule.

## Format - Conventional Commits

- First line: `type(scope): subject` — under 72 characters, imperative mood ("add", not "added")
- Allowed types: feat, fix, docs, refactor, test, chore, perf, build, ci
- Scope is optional; use it when the change clearly belongs to one component
- If a work item is available, start the first line with the work item ID followed by a colon, then the conventional commit message
- Add a blank line and a short body explaining WHY the change was made when it is not obvious from the subject

## Output format

After each commit, output one line:
  <hash> <subject>

Do not include markdown headers, code fences, or explanatory prose beyond this.

Work Item (if available):
{workItem}
