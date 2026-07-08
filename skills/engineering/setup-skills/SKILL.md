---
name: setup-skills
description: Sets up an `## Agent skills` block in AGENTS.md/CLAUDE.md and `<docs-root>/agents/` so the engineering skills know this repo's issue tracker, triage label vocabulary, domain doc layout, and docs root location. Run before first use of `to-tickets`, `to-spec`, `wayfinder`, `domain-modeling`, or `improve-codebase-architecture` — or if those skills appear to be missing context about the issue tracker, triage labels, domain docs, or project structure.
disable-model-invocation: true
---

# Setup Skills

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker** — where issues and specs live (local markdown, discovered path)
- **Triage labels** — the strings used for the five canonical triage roles
- **Domain docs** — where `CONTEXT.md` and ADRs live, and the consumer rules for reading them
- **Project structure** — where the docs root lives, so other skills know where to read/write `agents/*.md` files

This skill is **project-agnostic**: it never forces a structure. It explores the repo first to find where things already live, then confirms what it found with the user before writing anything. Every path below is *discovered*, not assumed.

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## How the skills find this config

The engineering skills do **not** hardcode where this config lives. They rely on
`AGENTS.md` / `CLAUDE.md` being **auto-loaded into the agent's context on every
run** — that's the load-bearing contract. This skill writes a short
`## Agent skills` block into that file pointing at a single index,
`<docs-root>/agents/project-structure.md`, which in turn links to the issue
tracker, triage label, and domain-doc config. If a skill ever appears to be
missing this context, the repo either hasn't run `setup-skills` or its
`AGENTS.md` / `CLAUDE.md` isn't being loaded.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `AGENTS.md` and `CLAUDE.md` at the repo root — does either exist? Is there already an `## Agent skills` section in either?
- `CONTEXT.md` and `CONTEXT-MAP.md` anywhere near the root
- Locate the repo's documentation directory. It may not be named `docs/` — look for common alternatives (`documentation/`, `doc/`, `wiki/`, etc.). Use whichever directory already serves as the documentation root.
- Within the discovered docs directory, look for an ADR subdirectory (commonly `adr/`, but check for variants like `architecture-decisions/` or `decisions/`)
- `<discovered-docs-root>/agents/` — does this skill's prior output already exist? In particular, check for `<discovered-docs-root>/agents/project-structure.md` from a prior run.
- Existing scratch / issues directory — look for `.scratch/`, `issues/`, `tickets/`, or any directory already holding issue or spec files. Note its path.

Use what you find to infer the layout — don't ask until you've looked.

### 2. Present findings and ask

Summarise what's present and what's missing. Then walk the user through the decisions **one at a time** — present a section, get the user's answer, then move to the next.

Assume the user does not know what these terms mean. Each section starts with a short explainer.

**Section A — Issue tracker.**

> Explainer: The "issue tracker" is where issues and specs live for this repo. Skills like `to-tickets`, `to-spec`, and `wayfinder` read from and write to it. The default is local markdown files in a scratch directory. The skill discovers where that directory is (or should be) rather than forcing a fixed location.

From your exploration, present what you found:

- If a scratch/issues directory already exists (e.g. `.scratch/`, `issues/`), propose using it. State the evidence: "I found `.scratch/` with existing feature directories — using this as the issue tracker root."
- If none exists, propose `.scratch/` as a default and ask the user to confirm or name a different path.

Confirm the **issue tracker root** path with the user. This is a local-markdown tracker: one feature per directory under the root, specs and issues as markdown files inside.

**Section B — Triage label vocabulary.**

> Explainer: When the `to-tickets` skill publishes tickets, it applies a `ready-for-agent` label so AFK agents can find fully-specified work. The triage state machine also uses labels for its other roles. The skill applies labels (as frontmatter fields or filename conventions) that match strings *you've actually configured*. If your project already uses different label names (e.g. `bug:triage` instead of `needs-triage`), map them here so the skill applies the right ones instead of creating duplicates.

The five canonical roles:

- `needs-triage` — maintainer needs to evaluate
- `needs-info` — waiting on reporter
- `ready-for-agent` — fully specified, AFK-ready (an agent can pick it up with no human context)
- `ready-for-human` — needs human implementation
- `wontfix` — will not be actioned

Default: each role's string equals its name. Ask the user if they want to override any.

**Section C — Domain docs.**

> Explainer: Some skills (`improve-codebase-architecture`, `domain-modeling`, `grill-with-docs`) read a `CONTEXT.md` file to learn the project's domain language, and an ADR directory for past architectural decisions. The layout of these files determines where the skills look.

Infer the layout from your exploration in step 1:

- A `CONTEXT.md` at the repo root (or wherever it currently lives)
- An ADR directory under the discovered docs root (commonly `adr/`, but check for variants like `architecture-decisions/` or `decisions/`)

Present what you found and the evidence behind it (e.g. "I found `documentation/` as the docs root and `documentation/adr/` for ADRs — using these for domain docs"). Ask the user to confirm or correct.

If after exploring you genuinely cannot determine the layout — no `CONTEXT.md`, no docs directory, no obvious ADR location — ask the user directly:

> I couldn't find an existing docs directory, `CONTEXT.md`, or ADR location. Where should domain context and ADRs live in this repo?

In either case, the `<discovered-docs-root>/agents/domain.md` you write must reflect the actual discovered paths (e.g. `documentation/CONTEXT.md`, `documentation/adr/`), not generic placeholders.

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
- The contents of `<discovered-docs-root>/agents/project-structure.md`, `<discovered-docs-root>/agents/issue-tracker.md`, `<discovered-docs-root>/agents/triage-labels.md`, `<discovered-docs-root>/agents/domain.md`

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- Always create or edit `AGENTS.md` at the repo root
- If `CLAUDE.md` exists, edit it in addition to `AGENTS.md` for backward compatibility
- Never create duplicate `## Agent skills` blocks in multiple files

If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block points at a **single index file** — keep it minimal so AGENTS.md stays
stable across re-runs. All per-topic detail lives in
`<discovered-docs-root>/agents/project-structure.md`, not inline here:

```markdown
## Agent skills

This repo is configured for the engineering skills. The index of shared skill
config lives at `<discovered-docs-root>/agents/project-structure.md` — read it
first to locate the issue tracker, triage label vocabulary, and domain-doc
layout.
```

Then write the four docs files into `<discovered-docs-root>/agents/` using the seed templates in this skill folder as a starting point:

- [project-structure.md](./project-structure.md) — the index: docs root location + links and one-line summaries pointing at the other three files
- [issue-tracker-local.md](./issue-tracker-local.md) → write as `agents/issue-tracker.md` — local-markdown issue tracker (always use this one)
- [triage-labels.md](./triage-labels.md) → write as `agents/triage-labels.md` — label mapping
- [domain.md](./domain.md) → write as `agents/domain.md` — domain doc consumer rules + layout

`project-structure.md` must state the discovered docs root as a concrete repo-relative path (e.g. `documentation/`), and that `<docs-root>/agents/` is where this and other skills' shared config files live. It must link to the other three files by their **destination** names (`domain.md`, `issue-tracker.md`, `triage-labels.md`), each with a one-line summary so an agent reading only the index gets the gist. This file is the canonical pointer — `AGENTS.md` / `CLAUDE.md` points only here, and other skills read it first to locate the rest.

The `domain.md` you write must include concrete paths discovered during exploration, not generic placeholders. For example, if the repo has `documentation/CONTEXT.md` and `documentation/adr/`, those paths should appear explicitly.

The `issue-tracker.md` you write must state the **discovered issue tracker root** as a concrete repo-relative path (e.g. `.scratch/`), not a hardcoded default. If no existing scratch directory was found, the path confirmed with the user in Section A is the one to record.

If no docs directory exists anywhere in the repo, ask the user where to create one — don't default to `docs/`.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `<discovered-docs-root>/agents/*.md` directly later — re-running this skill is only necessary if they want to change the triage label vocabulary, domain doc layout, project structure, issue tracker root, or restart from scratch.
