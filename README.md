# Skills

A collection of agent skills, organised under `skills/` into engineering and productivity categories.

The engineering workflow builds on [Matt Pocock's skills](https://github.com/mattpocock/skills) — gratefully acknowledged. Those skills are no longer copied into this repo; they are installed straight from upstream with [`npx skills`](https://skills.sh) or Nix (see [Getting Started](#getting-started)). This repo only holds what is custom: a tracker-agnostic `setup-skills` that replaces upstream's `setup-matt-pocock-skills`, plus a `commit` skill.

## Structure

```
skills/
├── engineering/   # setup-skills and the azure-item skills (custom)
└── productivity/  # commit (general-purpose)
```

## Engineering

| Skill | Purpose |
|-------|---------|
| **setup-skills** | Set up the agent skills block and repo config (issue tracker, labels, domain docs) |
| **azure-item-grill** | Grill the user on an assigned Azure DevOps PBI/Bug, and propose whether a spec is needed, before `@to-spec` and `@to-tickets` |
| **azure-item-tasks** | Mirror tickets from the repo's tracker as child Tasks on the PBI (`@azure-item-tasks`) |
| **azure-item-finalize** | Close out a PBI/Bug: create the PR, close child Tasks, summarize, link the PR, post doc review links |

Azure flow: `@azure-item-grill` → `@to-spec` (optional) → `@to-tickets` → `@azure-item-tasks` → implement → `@azure-item-finalize`. The Azure skills use the Azure DevOps MCP server.

`setup-skills` replaces upstream's `setup-matt-pocock-skills`. It discovers the repo's docs root instead of assuming one, and ships issue-tracker adapters for GitHub (`gh`), local markdown (`.scratch/`), [beads](https://github.com/gastownhall/beads) (`bd`), GitLab (`glab`), and Azure DevOps (`az boards`). Run it in a repository before using the upstream engineering skills. For what those skills do, see [Matt's README](https://github.com/mattpocock/skills#readme).

## Productivity

| Skill | Purpose |
|-------|---------|
| **commit** | Generate conventional commit messages and create commits |

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

In PowerShell:

```powershell
$skills = Get-Content upstream-skills.txt |
  Where-Object { $_ -and -not $_.StartsWith('#') } |
  ForEach-Object { '--skill', $_ }
npx skills@latest add mattpocock/skills -g $skills
```

Install the custom skills from this repo (same command in bash and PowerShell):

```bash
npx skills@latest add xybschin/skills -g --skill setup-skills --skill commit --skill azure-item-grill --skill azure-item-tasks --skill azure-item-finalize
```

Add `-a <agent>` (e.g. `-a opencode`, `-a claude-code`) to skip the agent prompt, or drop `-g` to install into the current project instead of your user directory.

To update `npx` installs, run `npx skills@latest update`. Re-run `add` to pick up new skills.

Then run `@setup-skills` once in each repository before using the engineering skills.

**Note:** The upstream skills tell you to run `@setup-matt-pocock-skills` when they find no tracker config. Run `@setup-skills` instead.
