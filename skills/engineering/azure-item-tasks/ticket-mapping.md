# Ticket → Azure Task Mapping

Azure field rules for mirroring `to-tickets` tickets as child Tasks.

## Title

The ticket's title, verbatim.

## Description

```markdown
## What to build

<ticket's "What to build">

## Acceptance criteria

- [ ] <ticket's criteria>

Source ticket: <path or reference from the tracker>
```

## Blocking edges

Each "Blocked by" entry becomes a **Predecessor/Successor** link: the blocking ticket's Task is the Predecessor of the blocked one. Create all Tasks first, then add the links.

## Fields

- `areaPath` / `iterationPath`: copied from the parent PBI, inside each item.
- Tag: the ready-for-agent label string from `triage-labels.md`.
- `workItemType`: `Task`.
