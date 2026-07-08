# Skills

A collection of agent skills, organised under `skills/` into engineering and productivity categories, plus opencode commands in `commands/`.

Much of the engineering skill set is adapted from [Matt Pocock's skills](https://github.com/mattpocock/skills) — gratefully acknowledged. Local modifications include a local-markdown-only issue tracker and project-agnostic placement discovery in `setup-skills`.

## Structure

```
skills/
├── commands/          # opencode command wrappers (/command in the TUI)
└── skills/
    ├── engineering/   # code-work skills
    └── productivity/  # general-purpose utilities
```

## Engineering

Skills for code work. Several require `setup-skills` to be run first in your repository.

| Skill | Purpose |
|-------|---------|
| **domain-modeling** | Build and sharpen a project's domain model (glossary + ADRs) |
| **grill-with-docs** | Run /grilling + /domain-modeling to grill a plan and update docs inline |
| **improve-codebase-architecture** | Find architecture improvement opportunities in a codebase |
| **setup-skills** | Set up the agent skills block and repo config (issue tracker, labels, domain docs) |
| **to-tickets** | Break a plan into tracer-bullet tickets with blocking edges |
| **to-spec** | Turn conversation context into a spec and publish it to the tracker |
| **wayfinder** | Plan huge work as a shared map of investigation tickets, resolved one at a time |

## Productivity

General-purpose utilities.

| Skill | Purpose |
|-------|---------|
| **commit** | Generate conventional commit messages and create commits |
| **grilling** | Interview relentlessly about a plan until shared understanding |
| **write-a-skill** | Create new agent skills |

## Commands

Opencode commands that let you explicitly invoke skills with `/command` in the TUI. Source files live in `commands/`; link them into `~/.config/opencode/commands/` to activate.

| Command | Skill | What it does |
|---------|-------|--------------|
| `/grill-with-docs` | grill-with-docs | Interview you relentlessly about a plan, sharpening terminology and updating `CONTEXT.md`/ADRs inline as decisions crystallise |
| `/implement` | implement | Execute the work described in a spec or set of tickets, using /tdd at pre-agreed seams, then review and commit |
| `/setup-skills` | setup-skills | Configure a repo for the engineering skills: discover and record the issue tracker root, triage label vocabulary, and domain-doc layout |
| `/to-tickets` | to-tickets | Break a plan, spec, or conversation into tracer-bullet vertical-slice tickets with declared blocking edges |
| `/to-spec` | to-spec | Synthesise the current conversation into a spec (PRD) and publish it to the issue tracker — no interview, just capture |
| `/wayfinder` | wayfinder | Plan work too big for one session as a shared map of investigation tickets on the tracker, resolving one at a time until the route is clear |

## Getting Started

**Note**: Several engineering skills require `setup-skills` to be run first in your repository.
