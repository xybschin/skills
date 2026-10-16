---
name: azure-item-tasks
description: Mirror tickets from the repo's tracker as child Task work items on an Azure DevOps PBI.
disable-model-invocation: true
metadata:
  opencode/slash: "true"
---

# Azure Item Tasks

Create child Tasks on a PBI from tickets that `/to-tickets` already published to the repo's tracker.

## Conventions

- Work items live in the **work item project** (e.g. `40_EMI_Supplier`). Use it for all `mcp_azuredevops_m_wit_*` calls.
- Read `<docs-root>/agents/project-structure.md`, then `issue-tracker.md`: it says where the tickets live and how to read them and add comments.

## Workflow

### 1. Resolve inputs

Take the PBI ID and the feature slug (or ticket references) from the user.

When `issue-tracker.md` names **Azure DevOps**, stop: `/to-tickets` already published the tickets as Tasks under their parent. Report the existing children instead.

### 2. Read the tickets

Follow `issue-tracker.md` for "fetch the relevant ticket" on each ticket. Per ticket collect: title, "What to build", acceptance criteria, "Blocked by", and its `Azure: #<id>` line when an earlier run mirrored it.

### 3. Draft Tasks

Map each ticket per [ticket-mapping.md](ticket-mapping.md). A ticket with an `Azure: #<id>` line is already mirrored: list it as `exists #<id>` and leave it out of creation.

Present a table: ticket, proposed Task title, blocked-by, new or exists. Ask for confirmation, and iterate until approved.

### 4. Create

On an explicit yes:

1. Call `mcp_azuredevops_m_wit_work_item_write` (action `add_child`) with `project`, `parentId`, `workItemType: Task`, and `items`, each carrying `title`, `description`, `areaPath`, `iterationPath`, and the repo's ready-for-agent label (the string mapped in `triage-labels.md`) as tag. Put `areaPath` and `iterationPath` **inside each item**, copied from the parent; as top-level parameters, creation fails with "no permissions to save work items under specified area path".
2. Add a Predecessor/Successor link per blocking edge, via `mcp_azuredevops_m_wit_work_item_link_write`.
3. Record each new Azure ID on its source ticket as the line `Azure: #<id>`, using the tracker's comment operation from `issue-tracker.md`. Leave the source tickets open: the user closes them.

Done when every ticket is `exists #<id>` or newly created. Report the IDs; when some fail, report which succeeded and which failed.
