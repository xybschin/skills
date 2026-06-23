# Implementation Skills

Write, refactor, and improve code and skills.

## Skills

- **improve-codebase-architecture** — Find deepening opportunities and refactoring targets informed by domain language and architecture decisions
- **write-a-skill** — Create new agent skills with proper structure, resources, and documentation
- **caveman** — Token-efficient communication mode (suite of 7 sub-skills)

## When to Use

During development, refactoring, or when creating new skills. Use across all skills for token efficiency when appropriate.

## How to Use

**Improve architecture**:
```
/improve-codebase-architecture
```
Analyzes your codebase and recommends refactoring opportunities based on existing domain language and architecture decision records.

**Create a skill**:
```
/write-a-skill
```
Scaffolds a new agent skill with proper structure, bundled resources, and documentation.

**Caveman mode** (token efficiency):
```
/caveman
```
Activate terse communication across all skills. Supports intensity levels: `lite`, `full` (default), `ultra`, `wenyan-lite`, `wenyan-full`, `wenyan-ultra`. Switch with `/caveman lite|full|ultra`.

## Prerequisites

`improve-codebase-architecture` requires `setup-skills` to have been run (to access domain docs and ADRs).
