# Reference

Detailed reference for the `azure-item-finalize` skill.

## Two-project split

Azure DevOps separates git repositories from work item tracking. In this setup:

| Concern | Project | Example |
| --- | --- | --- |
| Git repos, PRs | git project | `40_EMI` |
| Work items (PBI, Bug, Task) | work item project | `40_EMI_Supplier` |

Always use the git project for `mcp_azuredevops_m_repo_*` calls and the work item project for `mcp_azuredevops_m_wit_*` calls. Using the wrong project fails (e.g. TF401019 for PRs).

## PR ↔ PBI linking

There are two ways to link a PR to a work item. **Method A is preferred** because it links at creation and cannot produce a broken link.

### Method A — link at PR creation (preferred)

`mcp_azuredevops_m_repo_pull_request_write` (action `create`) accepts a `workItems` parameter (space-separated work item IDs). Passing the PBI ID links the PR to the PBI automatically:

```
workItems: "1796719"
```

### Method B — link post-hoc (fallback)

If the PR already exists without the link, add an `ArtifactLink` relation to the work item.

**Get the exact artifactId** from the PR:

`mcp_azuredevops_m_repo_pull_request` (action `get`) → response field `artifactId`.

Example artifactId:

```
vstfs:///Git/PullRequestId/6f689bc1-dd62-43f8-9096-b6785e83b213%2f0fc47962-18a7-44c8-bdee-d6d645e46180%2f231127
```

Format: `vstfs:///Git/PullRequestId/{projectGUID}%2f{repoGUID}%2f{pullRequestId}`

**Critical**: the short form `vstfs:///Git/PullRequestId/{pullRequestId}` (PR ID only) produces a **broken link** — the work item shows the relation but it does not resolve. Always use the full artifactId from the PR response.

**Add the relation** via `mcp_azuredevops_m_wit_work_item_link_write` (action `add`):

- `id`: work item ID
- `rel`: `ArtifactLink`
- `url`: the full artifactId
- `attributes.name`: `Pull Request`

**Fallback if link tool is disabled**: use `mcp_azuredevops_m_wit_work_item_write` (action `update`) with a JSON patch:

```json
[
  {
    "op": "add",
    "path": "/relations/-",
    "value": {
      "rel": "ArtifactLink",
      "url": "vstfs:///Git/PullRequestId/{projectGUID}%2f{repoGUID}%2f{pullRequestId}",
      "attributes": { "name": "Pull Request" }
    }
  }
]
```

## MCP tool parameter tables

### `mcp_azuredevops_m_repo_pull_request_write` (action `create`)

| Param | Value |
| --- | --- |
| `project` | git project (e.g. `40_EMI`) |
| `repositoryId` | repo name |
| `sourceRefName` | `refs/heads/<branch>` |
| `targetRefName` | `refs/heads/develop` |
| `title` | PR title |
| `description` | PR description |
| `workItems` | space-separated work item IDs to link |

### `mcp_azuredevops_m_wit_work_item` (action `get`)

| Param | Value |
| --- | --- |
| `id` | work item ID |
| `expand` | `Relations` to get child links |

Children appear as relations with `rel: System.LinkTypes.Hierarchy-Forward`.

### `mcp_azuredevops_m_wit_work_item_write` (action `update`)

| Param | Value |
| --- | --- |
| `id` | work item ID |
| `fields` | e.g. `{ "System.State": "Done" }` |

### `mcp_azuredevops_m_wit_work_item_comment_write` (action `add`)

| Param | Value |
| --- | --- |
| `id` | work item ID |
| `text` | comment body |

### `mcp_azuredevops_m_wit_work_item_link_write` (action `add`)

| Param | Value |
| --- | --- |
| `id` | work item ID |
| `rel` | `ArtifactLink` |
| `url` | full artifactId URL |
| `attributes` | `{ "name": "Pull Request" }` |

## Documentation review comment

The PO reviews pre-merge, but the feature-branch link breaks once the branch is deleted after merge. Post both links:

- **Pre-merge** (feature branch, works now):
  `https://dev.azure.com/{org}/{gitProject}/_repos/{repo}?path=/{docPath}&version=GB{branch}`
- **Post-merge** (target branch, stable, resolves after merge):
  `https://dev.azure.com/{org}/{gitProject}/_repos/{repo}?path=/{docPath}&version=GBdevelop`

Comment shape (markdown):

```
**{Doc title}** - decision doc for review.

{1-2 sentence summary of what the doc decides/defines.}

- **Pre-merge** (current state on feature branch): [{Doc title}]({preMergeUrl})
- **Post-merge** (stable link, resolves once merged to develop): [{Doc title}]({postMergeUrl})
```

- One comment per doc, or one comment listing several docs added together.
- Summary stays at 1-2 sentences; the PO reads the summary and follows the link for the full text.
- Derive `{docPath}` from the files the branch added (e.g. `git show --name-only` on the doc commits).
