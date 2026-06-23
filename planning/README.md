# Planning Skills

Convert ideas into actionable issues and product requirements.

## Skills

- **setup-skills** — One-time configuration of your repo's issue tracker, domain docs, and triage labels
- **to-prd** — Turn conversation context into a PRD
- **to-issues** — Break a plan into independently-grabbable issues using vertical slices

## When to Use

After a design or plan is solid and you're ready to prepare work for implementation.

## How to Use

1. **Run setup-skills first** (one-time per repository):
   ```
   /setup-skills
   ```
   This configures your repo's issue tracker location, triage labels, and domain docs location. Required before other planning skills can work.

2. **Create a PRD** (if you have a concept that needs formalization):
   ```
   /to-prd
   ```
   Converts the current conversation into a structured PRD and saves it to `.scratch/`.

3. **Break into issues** (to create implementation tickets):
   ```
   /to-issues
   ```
   Takes a plan or PRD and breaks it into vertical-slice issues. Requires `setup-skills` to have been run.

## Prerequisites

`setup-skills` must be run once before using `to-prd` or `to-issues`.
