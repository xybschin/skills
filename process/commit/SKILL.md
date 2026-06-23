---
name: commit
description: Generate and create conventional commits with user confirmation. Analyzes staged changes, drafts commit messages, presents for approval, then commits without pushing. Use when user says "commit", "write a commit", "create commit", or "/commit".
---

# Commit Skill

Generate conventional commits from staged changes with required user confirmation.

## Format - Conventional Commits:

- First line: type(scope): subject - under 72 characters, imperative mood ("add", not "added")
- Allowed types: feat, fix, docs, refactor, test, chore, perf, build, ci
- The scope is optional; use it when the change clearly belongs to one component
- If a work item is available, start the first line with the work item ID followed by a colon, then the conventional commit message
- You may add a blank line and a short body explaining WHY the change was made, when it is not obvious

IMPORTANT: Return ONLY the raw commit message text. Do not include:

- Headers like "### Commit Message" or "## Message"
- Wrapping quotes, backticks, or code fences
- Markdown formatting (no #, \*, \*\*, etc.)
- Preamble text like "Here is the commit message:"
- Explanations or commentary

Work Item (if available):
{workItem}

## Security Rules

- **ALWAYS** request user confirmation before committing
- Present full message for review
- **NEVER** push to any branch
- **NEVER** force-commit ignored files
- **NEVER** commit without explicit user approval
