---
name: azure-item-finalize
description: "Close out a finished Azure DevOps PBI or Bug: create the PR, close child tasks, comment a summary, link the PR, post doc (ADR) review links for the PO."
---

# Finalize Item

## Conventions

- **Two-project split**: git/PRs live in the **git project** (e.g. `40_EMI`); work items live in the **work item project** (e.g. `40_EMI_Supplier`). Mixing them fails (TF401019).
- PRs target `refs/heads/develop` (protected).
- Tools are `mcp_azuredevops_m_*`; parameter tables are in [REFERENCE.md](REFERENCE.md).

## Workflow

Each mutating step (2-5) starts only after the user approved it. Collect approvals in one plan at step 1, and re-ask only when the plan changes.

### 1. Verify state and get approval

Fetch the work item via `mcp_azuredevops_m_wit_work_item` (action `get`, expand `Relations`). Done when:

- the branch with the changes is pushed to the remote
- no PR is open for that branch (`mcp_azuredevops_m_repo_pull_request` list, filter by source branch)
- the work item has child Tasks. Without any, tell the user `/azure-item-tasks` hasn't run, and ask whether to continue with step 3 skipped.

Present the plan: PR title/description/target, child tasks to close, summary text, and doc review comment (if any docs). Proceed on approval.

### 2. Create the PR

Generate the PR body with the `/pr` skill. Then call `mcp_azuredevops_m_repo_pull_request_write` (action `create`) with:

- `project`: the **git project**
- `repositoryId`: repo name
- `sourceRefName`: `refs/heads/<branch>`, `targetRefName`: `refs/heads/develop`
- `title`: from the conventional commits and the work item title
- `workItems`: the work item ID(s). This links PBI and PR at creation (REFERENCE.md, method A).

Report the PR ID and URL.

### 3. Close child tasks

Find children via relations (`System.LinkTypes.Hierarchy-Forward`). Source tickets in the repo's tracker stay open: the user closes them. Done when every child task has a result comment (what was done, PR link, ADR path) via `mcp_azuredevops_m_wit_work_item_comment_write` and state `Done` via `mcp_azuredevops_m_wit_work_item_write`.

### 4. Summarize in the work item

Comment on the work item with: tasks completed, PR link, ADR/decision reference, next steps.

### 5. Post documentation review links

If the PBI produced docs the PO reviews (ADRs, design docs, specs under `Documentation/` or similar), post a comment with pre-merge and post-merge links plus a 1-2 sentence summary. Template and URL formats: REFERENCE.md, "Documentation review comment".

### 6. Link PR to work item (only if step 2 omitted `workItems`)

Add the `ArtifactLink` with the **full** artifactId URL. The short form produces a broken link. Procedure and fallback: REFERENCE.md, method B. Ask the user before adding it.

See [EXAMPLES.md](EXAMPLES.md) for a worked example.
