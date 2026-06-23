# Domain Docs
How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

The locations of domain docs vary by repo. Before doing any work, discover where they live:

- Look for a file named `CONTEXT.md` anywhere near the root — it may be at the root, or one level down inside a source or documentation directory
- Look for a file named `CONTEXT-MAP.md` — if present, read it; it will point at the per-context `CONTEXT.md` files and their locations
- Look for ADR files — they are typically markdown files with a numeric prefix (e.g. `0001-*.md`) grouped in a directory. That directory may be called `docs/adr/`, `Documentation/decisions/`, `adr/`, or something else entirely. Search for it rather than assuming a path.
- In multi-context repos, each context may have its own ADR directory alongside its own `CONTEXT.md`

Once found, read:
- The `CONTEXT.md` relevant to the area you're working in
- Any ADRs that touch that area

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The producer skill (`/grill-with-docs`) creates them lazily when terms or decisions actually get resolved.

## Inferring the layout

Explore the repo before assuming anything:

- List the root directory and note what the top-level directories are actually called
- Search for any file named `CONTEXT.md` or `CONTEXT-MAP.md`
- Search for directories containing files that match `[0-9]*-*.md` (ADR naming pattern)
- Note the naming conventions in use — source code, documentation, and decision directories can be named anything

Infer the layout from what you find:

- **Single-context** — one `CONTEXT.md`, one ADR directory, no per-package context files
- **Multi-context** — `CONTEXT-MAP.md` exists, or multiple separate directories each contain their own `CONTEXT.md` or ADR directory

Present your inference and the evidence behind it (e.g. "I found `CONTEXT.md` at the root and `Documentation/decisions/` containing ADRs — treating this as single-context"). Ask the user to confirm or correct before proceeding.

If you cannot determine the layout after exploring — no `CONTEXT.md`, no `CONTEXT-MAP.md`, no ADR-like files — ask the user directly:

> I couldn't find any `CONTEXT.md`, `CONTEXT-MAP.md`, or ADR files. Can you point me at where domain context and architectural decisions are documented, or confirm they don't exist yet?

Whatever layout is confirmed, record the **concrete paths** actually discovered, not assumed ones.

## Use the glossary's vocabulary
When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal — either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/grill-with-docs`).

## Flag ADR conflicts
If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders) — but worth reopening because…_
