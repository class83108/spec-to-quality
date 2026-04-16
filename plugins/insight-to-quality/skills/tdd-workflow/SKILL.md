---
name: tdd-workflow
description: >
  TDD red-green-refactor workflow for Python projects using Gherkin + pytest-bdd.
  Trigger when a finding's .feature file is ready and implementation should begin.
  Uses finding `type` + `finding-id` prefix routing to enforce skeleton vs feature boundaries.
---

# TDD Workflow (Gherkin + pytest-bdd)

Strict order: Red -> Green -> Refactor.

Before starting, read:
- `../../references/implementation-mindset.md` Part 2 (structure/testability dimensions)
- current finding card (`docs/spec-backlog/{finding-id}.md`)

## Prerequisites

- `.feature` file exists for current finding
- finding card exists with `type` and `tests_path`
- finding card `tests_path/{finding-id}.feature` path is consistent
- corresponding spec-xxx report row is `in-progress`
  - `contract-*` -> `docs/contracts/contracts-report.md`
  - `surface-*` -> `docs/surface/surface-report.md`
  - `behavior-*` -> `docs/behaviors/behavior-report.md`

If any prerequisite fails, stop and fix upstream status/spec first.

## Type Routing

Read finding card `type` and `finding-id` prefix:

| `type` | `finding-id` prefix | TDD mode | Primary objective |
|---|---|---|---|
| `skeleton` | `contract-` | Contract Skeleton | internal handoff schema/boundary guard correctness |
| `skeleton` | `surface-` | Surface Skeleton | external interface shape/error contract correctness |
| `feature` | `behavior-` | Behavior Feature | user behavior/business rule/state transition correctness |

Route validation rules:
- `contract-*` / `surface-*` must be `type: skeleton`
- `behavior-*` must be `type: feature`
- mismatch => stop and request finding card correction

## Phase 0: Preflight (default lightweight)

Default path is fast preflight, then go straight to Red.

Do minimum checks:
- map each Done Criteria to at least one `.feature` scenario and one Then assertion
- classify test level per scenario (`unit`/`integration`)
  - `skeleton` default `unit`, upgrade to `integration` only when crossing real I/O boundary
  - `feature` default `integration`, downgrade to `unit` only for pure in-process logic
- verify type boundary:
  - `skeleton` focuses on schema/boundary guards + declared error contract
  - `feature` focuses on user-observable behavior/business semantics

If all checks pass, proceed directly to Phase 1.

### Escalation: Manual Ledger (exception-only)

Only build a full verification ledger when preflight detects one of:
- any Done Criteria cannot map to scenario/Then (`spec gap`)
- type-intent conflict (`skeleton` testing business semantics, or `feature` re-testing schema shape)
- test-level ownership remains ambiguous after preflight

When escalated, produce a short ledger for unresolved items only, then continue to Red.

## Phase 1: Red

- write/extend step definitions
- prefer reusing existing pytest-bdd shared steps/helpers; add new shared utilities only when duplication appears
- run tests by level/tag (`@unit`, `@integration` as applicable)
- all new tests must fail
- if any unexpectedly pass, stop and discuss

Type-specific Red checklist:
- `skeleton`:
  - include at least one valid-shape pass scenario
  - include at least one invalid-shape rejection scenario with declared error contract
- `feature`:
  - include at least one core success behavior for `Serves` Gx
  - include at least one business-failure scenario from Error Handling Strategy or rule conflicts

## Phase 2: Green

- implement minimum code only
- run tests scenario by scenario, in level order: `unit -> integration`
- confirm target scenarios pass

Type-specific Green boundaries:
- `skeleton`:
  - schema validation and boundary guards can pass with stubbed domain behavior
  - prioritize deterministic contract enforcement over domain completeness
- `feature`:
  - implement real business logic/state transitions needed by scenarios
  - rely on existing skeleton protections; do not re-implement schema validation in feature logic/tests

## Phase 3: Refactor

- run lint/format/type-check using project CLAUDE.md commands
- refactor with green preservation
- rerun `unit + integration`

Refactor guardrails:
- do not expand scope beyond current finding Behavior/Done Criteria
- if new uncovered behavior is discovered, record follow-up finding as `draft` (do not absorb silently)

## Phase 4: Sync Back

Update finding card:
- Done Criteria coverage evidence
- implementation notes affecting boundary/contracts/state transitions
- unresolved integration/e2e gaps (if any) with owner/next action

Do not mark report row `done` here. Final status is decided in design-review release-gate.

## Completion Message

Return:
- routed mode (`Contract Skeleton` / `Surface Skeleton` / `Behavior Feature`)
- executed tests by level and result
- Done Criteria coverage status
- any spec gaps or follow-up draft findings created
