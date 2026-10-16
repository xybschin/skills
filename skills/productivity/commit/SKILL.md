---
name: commit
description: Use when the user says "commit": analyze the changes, group them into logical commits, write Conventional Commits messages, and hold every commit behind user approval.
---

## Workflow

### Tool invariance

The workflow and its approval gates are the same on every Git interface (CLI, Git MCP, IDE, API). A change of interface swaps only the commands used, never a step or a gate.

### Pre-flight

1. Read the current branch (`git rev-parse --abbrev-ref HEAD`).
2. If it is protected (`main`, `master`, `develop`, `release/*`, `hotfix/*`, or a repo-configured protected branch), start a branch named from the dominant change (`feat/add-login`, `fix/cart-total`) and switch to it. Tell the user. Commit only from the new branch.

### The approval gate

The commit waits on **approval**: the exact message(s), shown in full, explicitly accepted by the user. Asking to "commit" approves nothing by itself; a standing approval given earlier in the session covers later commits.

### Staged changes

1. Read the staged diff (`git diff --staged`).
2. Draft one message.
3. Get approval.
4. Commit (`git commit -m "<message>"`).
5. Report the hash and subject.

### Unstaged changes

1. Read the full change set (`git status`, `git diff`).
2. Group it into logical, independently meaningful sets by feature, fix, refactor, or file domain. Prefer fewer commits; split only on a clear boundary.
3. Show the grouping with per-group messages and get approval; the user may edit.
4. For each approved group: stage its files (`git add <files>`), commit, report the hash and subject.

## Message format

- Line 1: `type(scope): subject` — under 72 characters, imperative mood ("add", not "added").
- Types: feat, fix, docs, refactor, test, chore, perf, build, ci.
- Scope is optional; use it when the change belongs to one component.
- Prepend the work item ID when one exists: `<ID>: <message>`.
- After a blank line, add a short body only when the subject leaves the reason for the change unclear.

## Output

After each commit, output exactly one line:

  <hash> <subject>
