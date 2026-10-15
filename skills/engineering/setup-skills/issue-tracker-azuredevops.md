# Issue tracker: Azure DevOps

Issues and specs for this repo live as Azure DevOps work items. Use the
[`az boards`](https://learn.microsoft.com/en-us/cli/azure/boards) extension (`az extension add
--name azure-devops`) for all operations. Run `az devops configure --defaults
organization=<org-url> project=<project>` once per machine so commands don't need
`--org`/`--project` on every call.

> This doc assumes the org's process has a hierarchy ceiling of **Product Backlog Item (PBI)**
> — no Epic or Feature above it. If this org's process does have Epic/Feature, use those for
> the map/spec level instead of PBI and adjust the hierarchy below accordingly.

## Conventions

- **Create a work item**: `az boards work-item create --type Task --title "..." --description
  "..."`. Use `--type "Product Backlog Item"` for a spec/map. Descriptions are HTML, not
  markdown — wrap code in `<pre>` and keep formatting simple.
- **Read a work item**: `az boards work-item show --id <id>`. Add `-o json` for
  machine-readable output; always parse JSON, never scrape the table output, which truncates
  fields and omits tags.
- **List / query work items**: `az boards query --wiql "<WIQL query>" -o json`. There is no
  flag-based list filter the way `gh`/`glab` have — everything goes through WIQL.
- **Comment on a work item**: `az boards work-item update --id <id> --discussion "..."`.
- **Apply / remove tags**: `az boards work-item update --id <id> --fields
  "System.Tags=tag-one; tag-two"` — this **replaces** the tag field, so read the current
  value first and merge in the new tag rather than overwriting it.
- **Close**: `az boards work-item update --id <id> --state Closed`. State names are
  process-configurable; confirm the closed-state name for this org's process (commonly
  `Closed` or `Done`) before relying on it.
- **Pull requests**: Azure Repos calls PRs "pull requests," same as GitHub. Use `az repos pr
  create`, `az repos pr show`, etc. If this repo's remote isn't in Azure Repos (e.g. it's on
  GitHub but tracked in Azure Boards for work items only), PR operations belong to that other
  CLI, not `az repos`.

Infer the org/project from `az devops configure --list`, or ask the user once and record it here.

## No sub-type for tasks: use tags

With PBI as the ceiling, every implementation item is created as `Task` — there is no
`chore`/`bug`/`task` type distinction the way beads or local-markdown have. Preserve that
distinction as a tag instead (e.g. `bug`, `chore`), added via `System.Tags` at create time:

```
az boards work-item create --type Task --title "..." --fields "System.Tags=chore"
```

## Hierarchy vs. blocking are different link types

A Task's **Parent** link to its PBI (`System.LinkTypes.Hierarchy-Reverse`) says nothing about
the order siblings run in — same caveat as beads' `parent-child` and GitLab's epic/issue
nesting. Ordering between tasks is always a separate, explicit link:

- **Predecessor / Successor** (`System.LinkTypes.Dependency`) — the traditional dependency
  link, available in every process configuration. Default to this.
- **Blocks / Is blocked by** (`Microsoft.VSTS.Common.TestedBy-Reverse` family, often surfaced
  as a distinct "Blocks" link type) — more literal semantically, but not enabled in every
  process template. Only use it after confirming it's available in this org's process
  (`az boards work-item relation list-type -o table`).

Add a link after creating both work items:

```
az boards work-item relation add --id <successor-id> --relation-type "Predecessor" \
  --target-id <predecessor-id>
```

A ticket is unblocked when every work item linked to it as a Predecessor is Closed.

## When a skill says "publish to the issue tracker"

Create a work item. Map the skill's issue template onto native fields rather than markdown
headings:

| Template section         | Azure DevOps                                                     |
| ------------------------ | ------------------------------------------------------------------ |
| `## Parent`              | `--relation-type "Parent" --target-id <pbi-id>` after create, or `az boards work-item create --fields "System.Parent=<pbi-id>"` |
| `## What to build`       | `--description "..."`                                              |
| `## Acceptance criteria` | `--fields "Microsoft.VSTS.Common.AcceptanceCriteria=<html list>"` (field only present on PBI-level items in most processes — for a Task, fold acceptance criteria into the description) |
| `## Blocked by`          | omit from the body — carried by the Predecessor/Successor links    |

Publish **in dependency order, blockers first**. Each create prints the new work item's id, so
a blocker that already exists can be wired immediately:

```
az boards work-item create --type Task --title "Extract the payment gateway seam" \
  --fields "System.Parent=1042" "System.Tags=chore" \
  --description "<what to build>"
# prints id 1051

az boards work-item create --type Task --title "Charge a card through the seam" \
  --fields "System.Parent=1042" \
  --description "<what to build>"
# prints id 1052

az boards work-item relation add --id 1052 --relation-type "Predecessor" --target-id 1051
```

Read each printed id and paste it literally into the next command — don't capture it in a
shell variable; each `az` invocation is its own process and shell state does not persist.

## When a skill says "fetch the relevant ticket"

```
az boards work-item show --id <id> -o json
```

The user normally passes the id; if they pass a title, find it with a WIQL query:

```
az boards query --wiql "SELECT [System.Id] FROM WorkItems WHERE [System.Title] CONTAINS '<title>'" -o json
```

## Frontier, claim, resolve

The frontier is the open, unblocked, unclaimed work — what can be started right now.
Used by `/implement` to take its next ticket, and by `/wayfinder` to walk a map.

- **Frontier**: a WIQL query for `State <> 'Closed' AND [System.AssignedTo] = ''` scoped to
  the map's PBI, then drop any work item with an open Predecessor link (check via
  `az boards work-item show --id <id> --expand relations -o json` and confirm each
  Predecessor's state is `Closed`). First by id wins. An empty frontier means everything is
  either done or blocked.
- **Claim**: `az boards work-item update --id <id> --assigned-to <email> --state "In
  Progress"` before any work — set both fields in the same call so no window exists where the
  item looks assigned but still "To Do".
- **Close**: `az boards work-item update --id <id> --state Closed --discussion "<what
  landed>"`.
- **Discovered work**: a new work item, with a `--discussion` note naming which ticket
  surfaced it (Azure Boards has no native "discovered-from" link type; record it as text).

### Wayfinding operations

Used by `/wayfinder`. The **map** is a PBI; its tickets are `Task`s linked to it as Parent.

- **Map**: `az boards work-item create --type "Product Backlog Item" --title "<effort name>"
  --fields "System.Tags=wayfinder-map" --description "<Destination / Notes / Decisions so far
  / Not yet specified / Out of scope body>"`. The tag — not the type — identifies a map, since
  every map is a plain PBI. Index them with a WIQL query for `[System.Tags] CONTAINS
  'wayfinder-map'`.
- **Child ticket**: `az boards work-item create --type Task --title "<question as title>"
  --fields "System.Parent=<map-id>" "System.Tags=wayfinder-<type>"
  --description "## Question ..."`, where `<type>` is `research`/`prototype`/`grilling`/`task`.
- **Blocking**: `az boards work-item relation add --id <blocked-id> --relation-type
  "Predecessor" --target-id <blocker-id>` — the native dependency link, so the frontier can be
  computed from the tracker's own graph.
- **Frontier and claim**: as above, scoped to `System.Parent = <map-id>`. Scoping by parent
  also excludes the map PBI itself.
- **Resolve**: `az boards work-item update --id <id> --discussion "<the answer>"`, then
  `--state Closed`, then append a context pointer (ticket title, its id, and the gist) to the
  map's Decisions-so-far. Azure Boards has no append-to-description API call, so that last step
  is read-modify-write: `az boards work-item show --id <map-id> -o json`, edit the
  `System.Description` field, `az boards work-item update --id <map-id> --description "<whole
  new body>"`. Re-read immediately before writing — other sessions edit the map concurrently.
