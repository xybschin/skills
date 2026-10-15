# Skills

A collection of agent skills, organised under `skills/` into engineering and productivity categories, plus opencode commands in `commands/`.

The engineering workflow builds on [Matt Pocock's skills](https://github.com/mattpocock/skills) — gratefully acknowledged. Those skills are no longer copied into this repo; they are installed straight from upstream with [`npx skills`](https://skills.sh) or Nix (see [Getting Started](#getting-started)). This repo only holds what is custom: a tracker-agnostic `setup-skills` that replaces upstream's `setup-matt-pocock-skills`, plus a `commit` skill.

## Structure

```
skills/
├── commands/          # opencode wrappers for Matt's human-invoked skills
└── skills/
    ├── engineering/   # setup-skills (custom, tracker-agnostic)
    └── productivity/  # general-purpose utilities
```

## Engineering

| Skill | Purpose |
|-------|---------|
| **setup-skills** | Set up the agent skills block and repo config (issue tracker, labels, domain docs) |

`setup-skills` replaces upstream's `setup-matt-pocock-skills`. It discovers the repo's docs root instead of assuming one, and ships issue-tracker adapters for GitHub (`gh`), local markdown (`.scratch/`), [beads](https://github.com/gastownhall/beads) (`bd`), GitLab (`glab`), and Azure DevOps (`az boards`). Run it in a repository before using the upstream engineering skills. For what those skills do, see [Matt's README](https://github.com/mattpocock/skills#readme).

## Productivity

| Skill | Purpose |
|-------|---------|
| **commit** | Generate conventional commit messages and create commits |

## Commands

Opencode commands that let you explicitly invoke Matt's human-invoked skills (the ones he marks `disable-model-invocation: true`) with `/command` in the TUI. Source files live in `commands/`; link them into `~/.config/opencode/commands/` to activate. Each command only loads its skill, so the skill must be installed.

The skills in this repo need no wrapper: their `SKILL.md` sets `metadata: opencode/slash: "true"`, so opencode exposes `/commit` and `/setup-skills` itself.

| Command | What it does |
|---------|--------------|
| `/grill-me` | Interview you relentlessly about a plan or design, without writing docs (for use outside a repo) |
| `/grill-with-docs` | Interview you relentlessly about a plan, sharpening terminology and updating the glossary/ADRs inline as decisions crystallise |
| `/implement` | Execute the work described in a spec or set of tickets, using /tdd at pre-agreed seams, then review and commit |
| `/implement-spec` | Implement the result of /to-spec and /to-tickets in code |
| `/improve-codebase-architecture` | Scan a codebase for deepening opportunities, present them as an HTML report, then grill through the one you pick |
| `/retro` | Conduct a retrospective on a coding session |
| `/to-spec` | Synthesise the current conversation into a spec (PRD) and publish it to the issue tracker — no interview, just capture |
| `/to-tickets` | Break a plan, spec, or conversation into tracer-bullet vertical-slice tickets with declared blocking edges |
| `/triage` | Move incoming issues and external PRs through the triage roles and write agent-ready briefs; don't use it on tickets from /to-tickets |
| `/wayfinder` | Plan work too big for one session as a shared map of investigation tickets on the tracker, resolving one at a time until the route is clear |

## Getting Started

The upstream skills to install are listed in [`upstream-skills.txt`](./upstream-skills.txt). `setup-matt-pocock-skills` is left out on purpose — `setup-skills` from this repo replaces it.

**With Nix:** skip `npx` and pin Matt's repo as a non-flake input:

```nix
inputs.mattpocock-skills = {
  url = "github:mattpocock/skills";
  flake = false;
};
```

Then, for each name in `upstream-skills.txt`, link `${inputs.mattpocock-skills}/skills/*/<name>` into your agent's skills directory (e.g. `~/.claude/skills/<name>`), for example from a home-manager activation script. Update with `nix flake update mattpocock-skills`. Don't mix this with `npx skills` installs into the same directories.

**Without Nix:** from a clone of this repo, install the listed skills from Matt Pocock's repo:

```bash
npx skills@latest add mattpocock/skills -g \
  $(grep -v '^#' upstream-skills.txt | sed 's/^/--skill /')
```

Install the custom skills from this repo:

```bash
npx skills@latest add xybschin/skills -g \
  --skill setup-skills --skill commit
```

Add `-a <agent>` (e.g. `-a opencode`, `-a claude-code`) to skip the agent prompt, or drop `-g` to install into the current project instead of your user directory.

To update `npx` installs, run `npx skills@latest update`. Re-run `add` to pick up new skills.

Then run `/setup-skills` once in each repository before using the engineering skills.

**Note:** The upstream skills tell you to run `/setup-matt-pocock-skills` when they find no tracker config. Run `/setup-skills` instead.
