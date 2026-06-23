# Project Structure

The index the engineering skills use to locate this repo's shared skill config.
Your agent reads this because `AGENTS.md` / `CLAUDE.md` (auto-loaded into every
run) points its `## Agent skills` block here. Start here, then follow the links
below for the details you need.

## Docs root

The documentation root for this repo is `<docs-root>/`. All shared skill config
lives under `<docs-root>/agents/`.

> `setup-skills` fills in the concrete discovered path (e.g. `documentation/`,
> `docs/`). Do not leave the `<docs-root>` placeholder in the written file.

## Index

- **Domain docs** — `<one-line summary: where CONTEXT.md and ADRs live>`.
  See [domain.md](./domain.md).
- **Issue tracker** — issues are local markdown files under `.scratch/<feature>/`.
  See [issue-tracker.md](./issue-tracker.md).
- **Triage labels** — `<one-line summary of the label vocabulary>`.
  See [triage-labels.md](./triage-labels.md).

Each bullet names the **destination** file in `<docs-root>/agents/`, not the
seed template it was generated from. Keep the one-line summaries current so an
agent reading only this index gets the gist without opening every file.
