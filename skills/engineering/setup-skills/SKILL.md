---
name: setup-skills
description: Sets up an `## Agent skills` block in AGENTS.md/CLAUDE.md and `<docs-root>/agents/` so the engineering skills know this repo's issue tracker, triage label vocabulary, domain doc layout, and docs root location. Run before first use of `to-tickets`, `to-spec`, `wayfinder`, `domain-modeling`, or `improve-codebase-architecture` — or if those skills appear to be missing context about the issue tracker, triage labels, domain docs, or project structure.
disable-model-invocation: true
---

# Setup Skills

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker** — where issues and specs live (GitHub, GitLab, Azure DevOps, beads, or local markdown under `.scratch/`)
- **Triage labels** — the strings used for the five canonical triage roles
- **Domain docs** — where `GLOSSARY.md` and ADRs live, and the consumer rules for reading them
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
- `GLOSSARY.md` and `GLOSSARY-MAP.md` anywhere near the root. Also look for the legacy names `CONTEXT.md` and `CONTEXT-MAP.md`: the skills no longer read them
- Locate the repo's documentation directory. It may not be named `docs/` — look for common alternatives (`documentation/`, `doc/`, `wiki/`, etc.). Use whichever directory already serves as the documentation root.
- Within the discovered docs directory, look for an ADR subdirectory (commonly `adr/`, but check for variants like `architecture-decisions/` or `decisions/`)
- `<discovered-docs-root>/agents/` — does this skill's prior output already exist? In particular, check for `<discovered-docs-root>/agents/project-structure.md` from a prior run.
- `.scratch/` — a sign that a local-markdown issue tracker is already in use. Also note any other directory already holding issue or spec files (`issues/`, `tickets/`), since the local-markdown tracker only reads `.scratch/`.

Use what you find to infer the layout — don't ask until you've looked.

### 2. Present findings and ask

Summarise what's present and what's missing. Then walk the user through the decisions **one at a time** — present a section, get the user's answer, then move to the next.

Assume the user does not know what these terms mean. Each section starts with a short explainer.

**Section A — Issue tracker.**

> Explainer: The "issue tracker" is where issues and specs live for this repo. Skills like `to-tickets`, `to-spec`, and `wayfinder` read from and write to it. There are five adapters. **GitHub** uses `gh` against the repo's own GitHub issues. **Local markdown** needs nothing installed — issues are files, and blocking edges are prose you honour by hand. **Beads** (`bd`) is a CLI tracker with real dependency edges and a `bd ready` query that returns exactly the unblocked work, at the cost of requiring the `bd` binary. **GitLab** uses `glab` against the repo's own GitLab issues/MRs. **Azure DevOps** uses `az boards` against Azure Boards work items, and is the right choice when a repo's issues live in Azure DevOps rather than beside the code.

Check which adapters are available/applicable before asking:

- Run `bd --version` and look for a `.beads/` directory, for beads.
- Run `git remote -v` and check for a `github.com` remote, for GitHub, or a `gitlab.com` or self-hosted GitLab remote, for GitLab.
- Ask the user, or check for an existing `az devops configure --list` default, if the repo's issues live in Azure DevOps rather than on the code-hosting remote — Azure Boards isn't discoverable from `git remote -v` the way GitLab/GitHub are.

Present what you found and propose accordingly:

- **`.beads/` already exists** → propose beads. The repo is already using it.
- **`bd` is on PATH but no `.beads/`** → offer it alongside other applicable options, noting beads needs `bd init --quiet` first.
- **GitHub remote found** → propose GitHub.
- **GitLab remote found** → propose GitLab.
- **User confirms Azure Boards is where this repo's work items live** → propose Azure DevOps.
- **None of the above** → propose local markdown as the no-install default. Mention the other adapters as options if applicable, but don't push.

For **local markdown**, the root is always `.scratch/`: one feature per directory, specs and issues as markdown files inside. The installed skills (`to-tickets`, `code-review`) look there and nowhere else. If exploration found issue files in another directory (e.g. `issues/`), tell the user and offer to `git mv` them into `.scratch/`.

For **beads**, there is no root path to confirm — the store is `.beads/`. Confirm instead that the repo is initialised, and tell the user to run `bd init --quiet` if it isn't.

For **GitHub**, confirm `gh` is installed and authenticated (`gh auth status`). The seed's PRs-as-request-surface flag defaults to off; leave it off unless the user asks.

For **GitLab**, confirm `glab` is installed and authenticated (`glab auth status`), and ask whether external merge requests should be treated as a triage surface (see the MRs-as-request-surface flag in the seed).

For **Azure DevOps**, confirm `az` with the `azure-devops` extension is installed and `az devops configure --list` shows the right org/project default. Also confirm the process's hierarchy ceiling — if it has Epic/Feature above Product Backlog Item, note that the map/spec level should use those instead of PBI, and flag this as a deviation from the seed when writing `issue-tracker.md`.

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

> Explainer: Some skills (`improve-codebase-architecture`, `domain-modeling`, `grill-with-docs`) read a `GLOSSARY.md` file to learn the project's domain language, and an ADR directory for past architectural decisions. The layout of these files determines where the skills look.

Infer the layout from your exploration in step 1:

- A `GLOSSARY.md` at the repo root (or wherever it currently lives)
- An ADR directory under the discovered docs root (commonly `adr/`, but check for variants like `architecture-decisions/` or `decisions/`)

If exploration found a legacy `CONTEXT.md` or `CONTEXT-MAP.md`, offer to `git mv` it to `GLOSSARY.md` / `GLOSSARY-MAP.md` in the same directory. If the map file links to per-context `CONTEXT.md` files, rename those too and update the links. Record the new names in `domain.md`.

Present what you found and the evidence behind it (e.g. "I found `documentation/` as the docs root and `documentation/adr/` for ADRs — using these for domain docs"). Ask the user to confirm or correct.

If after exploring you genuinely cannot determine the layout — no `GLOSSARY.md`, no docs directory, no obvious ADR location — ask the user directly:

> I couldn't find an existing docs directory, `GLOSSARY.md`, or ADR location. Where should domain context and ADRs live in this repo?

In either case, the `<discovered-docs-root>/agents/domain.md` you write must reflect the actual discovered paths (e.g. `documentation/GLOSSARY.md`, `documentation/adr/`), not generic placeholders.

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
- The contents of `<discovered-docs-root>/agents/project-structure.md`, `<discovered-docs-root>/agents/issue-tracker.md`, `<discovered-docs-root>/agents/triage-labels.md`, `<discovered-docs-root>/agents/domain.md`

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which one to create; don't pick for them.

Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa), and never write the `## Agent skills` block into both files.

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
- The issue tracker seed chosen in Section A → write as `agents/issue-tracker.md`. Use [issue-tracker-github.md](./issue-tracker-github.md) for GitHub, [issue-tracker-local.md](./issue-tracker-local.md) for local markdown, [issue-tracker-beads.md](./issue-tracker-beads.md) for beads, [issue-tracker-gitlab.md](./issue-tracker-gitlab.md) for GitLab, or [issue-tracker-azuredevops.md](./issue-tracker-azuredevops.md) for Azure DevOps. Write exactly one of them.
- [triage-labels.md](./triage-labels.md) → write as `agents/triage-labels.md` — label mapping
- [domain.md](./domain.md) → write as `agents/domain.md` — domain doc consumer rules + layout

`project-structure.md` must state the discovered docs root as a concrete repo-relative path (e.g. `documentation/`), and that `<docs-root>/agents/` is where this and other skills' shared config files live. It must link to the other three files by their **destination** names (`domain.md`, `issue-tracker.md`, `triage-labels.md`), each with a one-line summary so an agent reading only the index gets the gist. This file is the canonical pointer — `AGENTS.md` / `CLAUDE.md` points only here, and other skills read it first to locate the rest.

The `domain.md` you write must include concrete paths discovered during exploration, not generic placeholders. For example, if the repo has `documentation/GLOSSARY.md` and `documentation/adr/`, those paths should appear explicitly.

If no docs directory exists anywhere in the repo, ask the user where to create one — don't default to `docs/`.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `<discovered-docs-root>/agents/*.md` directly later — re-running this skill is only necessary if they want to change the triage label vocabulary, domain doc layout, project structure, issue tracker, or restart from scratch.
