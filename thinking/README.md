# Thinking Skills

Stress-test designs with rigorous questioning.

## Skills

- **grill-me** — Interview yourself exhaustively about a plan or design until reaching shared understanding
- **grill-with-docs** — Stress-test a plan against your project's domain model (CONTEXT.md and ADRs)

## When to Use

Before committing to a plan, major design decision, or high-stakes architecture change. Use to identify gaps, dependencies, and edge cases.

## How to Use

**General stress-testing**:
```
/grill-me
```
Asks relentless, focused questions about your plan or design, walking down each branch of the decision tree.

**Domain-aware stress-testing**:
```
/grill-with-docs
```
Challenges your plan against your project's existing domain language and architecture decisions (CONTEXT.md and ADRs). Also updates documentation as decisions crystallize.

## When to Use Each

- **grill-me**: Any design or plan; general-purpose rigorous questioning
- **grill-with-docs**: When working within an established project with documented domain language and architecture decisions
