---
name: azure-item-grill
description: Start from an assigned Azure DevOps PBI or Bug and grill the user on it, as the input for @to-spec and @to-tickets.
---

# Grill Azure Item

The PBI or Bug is the seed. This skill ends when the grill ends; `@to-spec` and `@to-tickets` run afterwards on the same conversation, and publish to the repo's configured tracker.

## Conventions

- Work items live in the **work item project** (e.g. `40_EMI_Supplier`). Use it for all `mcp_azuredevops_m_wit_*` calls.
- Repo config lives under `<docs-root>/agents/`. Read `project-structure.md` (index, reached from the `## Agent skills` block in AGENTS.md/CLAUDE.md), then `domain.md` for the paths of `GLOSSARY.md` and the ADRs.

## Workflow

### 1. Pick the item

Use the ID the user passed. Otherwise call `mcp_azuredevops_m_wit_work_item` (action `my`, type `assignedtome`), present the PBIs and Bugs, and ask which one.

### 2. Load context

Fetch the item (`get`, expand `Relations`) for title, description, acceptance criteria, and existing children. Read `GLOSSARY.md` and the ADRs in the area the item touches, via `domain.md`. When the repo lacks the `agents/` config, tell the user to run `@setup-skills` and continue without it.

### 3. Grill

Use the `grilling` skill with the work item as the plan and the glossary as background. Ask only what the item, glossary, and ADRs leave open.

Done when each topic is resolved or explicitly out of scope:

- Scope boundaries
- Data model or schema changes
- API / service contract changes
- Error handling and edge cases
- Testing expectations (unit, integration, e2e)

### 4. Propose the spec

Propose **spec: yes/no** with the reason. Yes when any holds:

- the grill surfaced a schema or API contract change
- a new module seam is needed
- an ADR is needed
- more than one developer will split the work
- the PO should review decisions before work starts

No is the default for a Bug, or a PBI whose acceptance criteria already pin the behavior.

### 5. Hand off

Report `Grill done for #<id> <title>. Spec: <yes/no>, <reason>.` followed by the next steps, in order:

1. `@to-spec` (spec yes only)
2. `@to-tickets` (tickets reference #<id> as parent)
3. `@azure-item-tasks #<id> <feature-slug>` (skip when the tracker is Azure DevOps)

List the existing child Tasks found in step 2 so the tickets don't duplicate them.
