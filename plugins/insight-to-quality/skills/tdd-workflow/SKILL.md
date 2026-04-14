---
name: tdd-workflow
description: >
  TDD red-green-refactor workflow for Python projects using Gherkin + pytest-bdd.
  Trigger when a finding's .feature file is ready and implementation should begin.
---

# TDD Workflow (Gherkin + pytest-bdd)

Strict order: Red -> Green -> Refactor.

Before starting, read:
- `references/implementation-mindset.md` Part 2 (structure/testability dimensions)
- current finding card (`docs/spec-backlog/{finding-id}.md`)

## Prerequisites

- `.feature` file exists for current finding
- finding card exists
- index row is `in-progress`

## Phase 0: Verification Ledger

From finding card extract:
- Error Handling Strategy
- Behavior (SHALL/MUST)
- Done Criteria
- Boundary constraints

Build a ledger mapping each SHALL/Done Criteria to test ownership.
If behavior is underspecified, infer from `.feature` Then steps and mark:
`SHALLs inferred from .feature (spec gap)`.

After user confirms ledger, update card `## Integration Test Gaps`.

## Phase 1: Red

- write/extend step definitions
- run tests
- all new tests must fail
- if any unexpectedly pass, stop and discuss

## Phase 2: Green

- implement minimum code only
- run tests scenario by scenario
- confirm target scenarios pass

## Phase 3: Refactor

- run lint/format/type-check using project CLAUDE.md commands
- refactor with green preservation
- rerun tests after refactor

## Phase 4: Sync Back

Update finding card:
- Done Criteria coverage evidence
- Integration Test Gaps status
- implementation notes affecting boundary/contracts

Do not mark index row `done` here. Final status is decided in design-review release-gate.
