---
name: feature-planning
description: >
  Optional backlog integrator for cross-report triage.
  Reads internals/surface alignment reports, merges coupled findings, and maintains
  docs/spec-backlog/index.md ordering and dependencies.
  Use when findings are many or internals/surface interactions are complex.
---

# Feature Planning (Optional Integrator)

This skill is optional. Use it when internals/surface findings interact and need unified execution units.

Before starting, read:
- `references/architect-mindset.md` (traceability, boundary tests)
- `references/implementation-mindset.md` Part 0 (derivation rules)

## Core Purpose

1. Merge related internals/surface findings into one executable unit when needed
2. Set priority/dependency order in `docs/spec-backlog/index.md`
3. Enforce `WIP=1`

## Inputs

- `docs/alignment/internals-report.md` (if exists)
- `docs/alignment/surface-report.md` (if exists)
- `docs/spec-backlog/index.md`
- `SYSTEM_MAP.md`, `goals.md`, `dominant-ops.md`

## Merge Rules

Create `source=both` execution unit when any condition holds:
- same boundary/component
- same journey where one side is contract/schema and the other is interface/behavior
- done criteria of one depends on the other

When merged:
- keep origin IDs in `related` (e.g. `I-003,S-007`)
- create execution ID `U-xxx`

## Derivation Rules (How to Decide Priority/Deps)

1. Priority score
- score = failure impact x frequency x cost
- tie-breakers: user-facing breakage > internal refactor, cross-boundary > single-component

2. Dependency (`deps`)
- contract/schema producer before consumer behavior
- boundary definition before endpoint behavior
- if circular dependency found, split into smaller units and re-evaluate

3. Ready gate
An item can be `ready` only when:
- `serves` includes valid Gx
- `related` includes Dx/APx linkage
- `boundary` maps to SYSTEM_MAP names
- card has initial Done Criteria draft

## Index Status Model

- `draft`: discovered, not execution-ready
- `ready`: execution-ready and dependency-resolved
- `in-progress`: currently executing (unique)
- `done`: passed TDD + design-review/release-gate

## WIP Rule

- at most one `in-progress`
- multiple `draft/ready` allowed
- new discoveries during execution go to `draft`; do not interrupt current `in-progress` unless user cancels it

## Output

Update index and present queue:
- in-progress (0/1)
- ready
- draft
- done (recent)

Then ask user which `ready` item to promote next.
