# Skills

A collection of agent skills, organised under `skills/` into engineering and productivity categories, plus opencode commands in `commands/`.

The engineering workflow builds on [Matt Pocock's skills](https://github.com/mattpocock/skills) — gratefully acknowledged. Those skills are no longer copied into this repo; they are installed straight from upstream with [`npx skills`](https://skills.sh) (see [Getting Started](#getting-started)). This repo only holds what is custom: a tracker-agnostic `setup-skills` that replaces upstream's `setup-matt-pocock-skills`, plus a few productivity skills.

## Structure

```
skills/
├── commands/          # opencode command wrappers (/command in the TUI)
└── skills/
    ├── engineering/   # setup-skills (custom, tracker-agnostic)
    └── productivity/  # general-purpose utilities
```

## Engineering

| Skill | Purpose |
|-------|---------|
| **setup-skills** | Set up the agent skills block and repo config (issue tracker, labels, domain docs) |

`setup-skills` replaces upstream's `setup-matt-pocock-skills`. It discovers the repo's docs root instead of assuming one, and ships issue-tracker adapters for local markdown, [beads](https://github.com/gastownhall/beads) (`bd`), GitLab (`glab`), and Azure DevOps (`az boards`). Run it in a repository before using the upstream engineering skills.

The upstream skills used with it:

| Skill | Purpose |
|-------|---------|
| **domain-modeling** | Build and sharpen a project's domain model (glossary + ADRs) |
| **grill-with-docs** | Run /grilling + /domain-modeling to grill a plan and update docs inline |
| **implement** | Implement a piece of work based on a spec or set of tickets |
| **improve-codebase-architecture** | Find architecture improvement opportunities in a codebase |
| **to-tickets** | Break a plan into tracer-bullet tickets with blocking edges |
| **to-spec** | Turn conversation context into a spec and publish it to the tracker |
| **triage** | Move issues through the triage state machine and write agent-ready briefs |
| **wayfinder** | Plan huge work as a shared map of investigation tickets, resolved one at a time |
| **codebase-design**, **code-review**, **prototype**, **research**, **tdd** | Called by the skills above |

## Productivity

| Skill | Source | Purpose |
|-------|--------|---------|
| **commit** | this repo | Generate conventional commit messages and create commits |
| **write-a-skill** | this repo | Create new agent skills |
| **grilling** | upstream | Interview relentlessly about a plan until shared understanding |
| **wait-what** | upstream | Re-pitch the last step in simple language using the project glossary |

## Commands

Opencode commands that let you explicitly invoke skills with `/command` in the TUI. Source files live in `commands/`; link them into `~/.config/opencode/commands/` to activate. Each command only loads its skill, so the skill must be installed (from this repo or upstream).

| Command | Skill | What it does |
|---------|-------|--------------|
| `/commit` | commit | Analyse staged or unstaged changes, draft conventional commit message(s), and create the commit after user confirmation |
| `/grill-with-docs` | grill-with-docs | Interview you relentlessly about a plan, sharpening terminology and updating the glossary/ADRs inline as decisions crystallise |
| `/implement` | implement | Execute the work described in a spec or set of tickets, using /tdd at pre-agreed seams, then review and commit |
| `/setup-skills` | setup-skills | Configure a repo for the engineering skills: discover and record the issue tracker root, triage label vocabulary, and domain-doc layout |
| `/to-tickets` | to-tickets | Break a plan, spec, or conversation into tracer-bullet vertical-slice tickets with declared blocking edges |
| `/to-spec` | to-spec | Synthesise the current conversation into a spec (PRD) and publish it to the issue tracker — no interview, just capture |
| `/wayfinder` | wayfinder | Plan work too big for one session as a shared map of investigation tickets on the tracker, resolving one at a time until the route is clear |

## Getting Started

Install the upstream skills from Matt Pocock's repo. Leave out `setup-matt-pocock-skills` — `setup-skills` from this repo replaces it.

```bash
npx skills@latest add mattpocock/skills -g \
  --skill domain-modeling --skill grill-with-docs --skill implement \
  --skill improve-codebase-architecture --skill to-spec --skill to-tickets \
  --skill triage --skill wayfinder --skill grilling --skill wait-what \
  --skill codebase-design --skill code-review --skill prototype \
  --skill research --skill tdd
```

Install the custom skills from this repo:

```bash
npx skills@latest add xybschin/skills -g \
  --skill setup-skills --skill commit --skill write-a-skill
```

Add `-a <agent>` (e.g. `-a opencode`, `-a claude-code`) to skip the agent prompt, or drop `-g` to install into the current project instead of your user directory.

To update, run `npx skills@latest update`. Re-run `add` to pick up new skills.

Then run `/setup-skills` once in each repository before using the engineering skills.

**Note:** The upstream skills tell you to run `/setup-matt-pocock-skills` when they find no tracker config. Run `/setup-skills` instead.
