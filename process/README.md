# Process Skills

Manage issues and commits.

## Skills

- **triage** — Move issues through a state machine workflow (backlog, ready, in-progress, done, closed)
- **commit** — Generate and create conventional commits with user confirmation

## When to Use

When managing your backlog, triaging incoming issues, and committing changes to version control.

## How to Use

**Triage issues**:
```
/triage
```
Move issues through workflow states. Requires `setup-skills` to have been run (to access triage label vocabulary).

**Create a commit**:
```
/commit
```
Analyzes staged changes, drafts a conventional commit message, presents it for approval, then commits (without pushing).

## Prerequisites

`triage` requires `setup-skills` to have been run (to access triage label vocabulary for your repo).
