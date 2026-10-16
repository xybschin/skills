# Examples

Worked example of finalizing an Azure DevOps PBI.

## Example: PBI #1796719 "[MTTB] Usage Analysis ParOS"

Context: PBI in project `40_EMI_Supplier`, git repo in project `40_EMI`. Implementation done on branch `docs/agent-setup-and-adr-bundle-storage`. Three child tasks created earlier via `azure-item-tasks`.

### Step 1 — Verify state

Fetched PBI #1796719 with `mcp_azuredevops_m_wit_work_item` (action `get`, expand `Relations`). Confirmed:

- Branch pushed to remote
- No existing PR for the branch
- Children: #1837997, #1837998, #1837999 (relations `System.LinkTypes.Hierarchy-Forward`)

Presented plan to user; approved.

### Step 2 — Create the PR

`mcp_azuredevops_m_repo_pull_request_write` (action `create`):

```
project: 40_EMI
repositoryId: ManipTools.ManipToolsNEO.Toolbox
sourceRefName: refs/heads/docs/agent-setup-and-adr-bundle-storage
targetRefName: refs/heads/develop
title: docs(adr): ADR-0001 bundle storage in ParOS + agent setup
description: Adds engineering skills setup and ADR-0001 defining release bundle storage in ParOS. Closes PBI #1796719.
workItems: 1796719
```

Result: PR #231127 created, PBI linked at creation (method A).

### Step 3 — Close child tasks

For each child (#1837997, #1837998, #1837999):

1. `mcp_azuredevops_m_wit_work_item_comment_write` (action `add`) — result comment: what was done, PR link, ADR path
2. `mcp_azuredevops_m_wit_work_item_write` (action `update`) — `fields: { "System.State": "Done" }`

### Step 4 — Summarize in the PBI

`mcp_azuredevops_m_wit_work_item_comment_write` (action `add`) on #1796719 — summary: tasks completed, PR #231127 link, ADR-0001 path, next steps.

### Step 5 — Post documentation review links

The branch added `Documentation/adr/0001-bundle-storage-in-paros.md`. The PO reviews from the PBI, so post a comment with both a pre-merge (feature branch) and a post-merge (develop) link:

```
**ADR-0001: Bundle storage in ParOS** — decision doc for review.

Defines the JSON format for release bundle configs and the mapping to ParOS File Definitions.

- **Pre-merge** (current state on feature branch): [ADR-0001](https://dev.azure.com/ZEISSgroup-SMT/40_EMI/_repos/ManipTools.ManipToolsNEO.Toolbox?path=/Documentation/adr/0001-bundle-storage-in-paros.md&version=GBdocs/agent-setup-and-adr-bundle-storage)
- **Post-merge** (stable link, resolves once merged to develop): [ADR-0001](https://dev.azure.com/ZEISSgroup-SMT/40_EMI/_repos/ManipTools.ManipToolsNEO.Toolbox?path=/Documentation/adr/0001-bundle-storage-in-paros.md&version=GBdevelop)
```

The pre-merge link works now; the post-merge link is the one that survives branch deletion.

### Step 6 — Link PR to PBI (post-hoc variant)

If the PR had been created without `workItems`, link post-hoc:

1. `mcp_azuredevops_m_repo_pull_request` (action `get`) → `artifactId: vstfs:///Git/PullRequestId/6f689bc1-...%2f0fc47962-...%2f231127`
2. `mcp_azuredevops_m_wit_work_item_link_write` (action `add`) with the full artifactId URL

**Anti-example**: adding `vstfs:///Git/PullRequestId/231127` (short form) creates a relation that shows on the PBI but does not resolve — a broken link. Always use the full artifactId.
