# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `<issue-tracker-root>/`.

> `setup-skills` fills in the concrete discovered path (e.g. `.scratch/`,
> `issues/`). Do not leave the `<issue-tracker-root>` placeholder in the written file.

## Conventions

- One feature per directory: `<issue-tracker-root>/<feature-slug>/`
- The spec is `<issue-tracker-root>/<feature-slug>/SPEC.md`
- Implementation issues are `<issue-tracker-root>/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create the file under `<issue-tracker-root>/<feature-slug>/`, creating the directory if needed:

- A **spec** (`/to-spec`) → `SPEC.md`.
- A **ticket** (`/to-tickets`) → `issues/<NN>-<slug>.md`, numbered from `01` in dependency order so the numbering itself reflects blockers-first.

There are no native blocking links here, so a ticket records its blockers as text: a `**Blocked by:**` line naming the titles it depends on, or "None — can start immediately".

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Frontier, claim, resolve

The frontier is the open, unblocked, unclaimed work — what can be started right now.
Used by `/implement` to take its next ticket, and by `/wayfinder` to walk a map.

- **Frontier**: scan the issue files; a ticket is on the frontier when its `Status:` is
  open and unclaimed and every ticket named on its `**Blocked by:**` line is closed.
  First by number wins. An empty frontier means everything is either done or blocked.
- **Claim**: set `Status: claimed` and save before any work.
- **Close**: set `Status: closed` and append a line naming what landed.
- **Discovered work**: a new issue file, noting in its body which ticket surfaced it.

### Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `<issue-tracker-root>/<effort>/map.md` — the Notes / Decisions-so-far / Fog body.
- **Child ticket**: `<issue-tracker-root>/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. A `Type:` line records the ticket type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved`.
- **Blocking**: a `Blocked by: NN, NN` line near the top. A ticket is unblocked when every file it lists is `resolved`.
- **Frontier and claim**: as above, scanning `<issue-tracker-root>/<effort>/issues/`.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
