---
name: design-review
description: >
  Refactor-focused design verification after TDD green, plus release-gate checks.
  Verifies refactor safety under test protection, type-specific quality gates, and completion evidence
  before marking backlog item done.
---

# Design Review (+ Release Gate)

This stage is a post-TDD refactor gate.
TDD has already validated baseline behavior; this skill verifies refactor safety and release readiness.

Before starting, read:
- `../../references/architect-mindset.md` (traceability and boundary discipline)
- `../../references/implementation-mindset.md` Part 2 (structural checks)
- current finding card (`docs/spec-backlog/{finding-id}.md`)

## Prerequisites

- finding card exists with `type` and `tests_path`
- corresponding spec-xxx report row is `in-progress`
  - `contract-*` -> `docs/contracts/contracts-report.md`
  - `surface-*` -> `docs/surface/surface-report.md`
  - `behavior-*` -> `docs/behaviors/behavior-report.md`
- TDD is green for this finding
- `.feature` scenarios carry level tags (`@unit`/`@integration`) consistent with test execution evidence

If any prerequisite fails, stop and fix upstream status/evidence first.

## Core Intent

- perform refactor under test protection (not feature expansion)
- rerun targeted checks after refactor to detect regressions
- only mark `done` when post-refactor evidence is complete

## Type Routing

Read finding card `type` and `finding-id` prefix:

| `type` | `finding-id` prefix | Review mode | Primary gate |
|---|---|---|---|
| `skeleton` | `contract-` | Contract Skeleton Gate | internal handoff schema and boundary guard integrity |
| `skeleton` | `surface-` | Surface Skeleton Gate | external interface shape/error contract integrity |
| `feature` | `behavior-` | Behavior Feature Gate | user behavior/business rule/state transition integrity |

Route validation rules:
- `contract-*` / `surface-*` must be `type: skeleton`
- `behavior-*` must be `type: feature`
- mismatch => stop and request finding card correction

## Part 1: Declared Decisions Verification

Verify post-refactor implementation still matches finding card declarations:
- catch boundary
- domain errors
- infrastructure recovery
- boundary rules
- anti-pattern constraints

Type-specific decision gates:
- Contract Skeleton Gate:
  - handoff schema constraints are enforced at declared boundaries
  - no silent coercion that can cause downstream data drift
- Surface Skeleton Gate:
  - request/response/event shape and error contract are consistent with declared boundary contract
  - boundary tests do not depend on domain-specific semantics to pass
- Behavior Feature Gate:
  - behavior semantics match `Serves` goals and declared business rules
  - state transitions (if declared) are consistent and observable
  - feature behavior does not duplicate schema-shape ownership already covered by skeleton

If violations exist, report directly with `file:line`.

## Part 2: Structural Checks

Assess:
- separation of concerns
- dependency direction
- naming semantics
- testability
- consistency

Type-specific structural intent:
- `skeleton`: validation/guard code is isolated from business orchestration
- `feature`: business decisions are explicit and not buried in transport/schema plumbing

Use questions for improvements; use direct statements for clear violations.

## Part 2.5: Code Risk Review (mandatory)

Scan changed code paths for:
- correctness (logic and edge-condition behavior)
- state/transaction integrity
- concurrency/race/order sensitivity
- error handling and failure recovery
- security and data exposure
- performance and capacity regressions
- observability (logs/metrics needed for diagnosis)
- migration/backward compatibility impact

Type-specific risk anchors:
- `contract-*`: schema drift risk, boundary bypass risk, downstream compatibility risk
- `surface-*`: client-visible contract break risk, error-shape inconsistency risk
- `behavior-*`: business rule regression risk, state-machine inconsistency risk

Every discovered issue must be reported with:
- severity (`P0`/`P1`/`P2`/`P3`)
- `file:line`
- risk statement
- fix suggestion

## Part 2.6: Test Adequacy Review (mandatory)

For each high-risk change, verify post-refactor test ownership and coverage level:
- `unit` / `integration`
- explicit mapping to Done Criteria or finding behavior

Also verify:
- every Scenario has exactly one level tag (`@unit` or `@integration`)
- scenario tags align with executed test evidence
- for `behavior-*`, tests focus on semantics; schema-shape assertions remain in skeleton scope

## Part 2.7: Scope Control Review (mandatory)

Check whether code changes exceed current finding scope:
- if no: mark `in-scope`
- if yes: mark `scope-creep` and create follow-up finding/task reference

Do not hide out-of-scope work under current finding `done`.

## Part 3: Release Gate (mandatory)

Run and record:
1. post-refactor tests by level (`unit`, `integration`) for current finding scenarios
2. lint/format
3. type check
4. `git diff --stat` + `git status`
5. finding card sync status (Done Criteria evidence)
6. discovery sync check (SYSTEM_MAP / dominant-ops / goals updates if needed)

## Completion Rules

Mark report row to `done` only when:
- no blocking Part 1 violations
- no blocking findings from Part 2.5/2.6/2.7
- Part 3 commands executed with evidence
- unresolved integration gaps are tracked (test/issue/manual checklist)

Blocking criteria (must fail gate):
- contract or boundary breakage that can corrupt data/flow
- state-machine inconsistency or transaction integrity risk
- missing recovery/error handling for declared failure strategy
- high-risk code path without required test ownership
- scenario level tags (`@unit`/`@integration`) missing or inconsistent with evidence
- unresolved `scope-creep` without follow-up tracking

## Refactor Discipline

- treat refactor as structure/quality improvement under unchanged acceptance behavior
- if behavior changes are required, route back to new/updated finding before claiming `done`
- avoid redoing full TDD design work here; focus on regression detection and safety evidence

## Output Order (mandatory)

1. Findings first (ordered by severity, highest first)
2. Open questions/assumptions
3. Output table summary

Findings format:

| Severity | File:Line | Category | Risk | Suggested Fix |
|---|---|---|---|---|
| P1 | `path/to/file.py:120` | correctness | ... | ... |

## Output Table

| Item | Status | Notes |
|---|---|---|
| Routed mode | Contract Skeleton / Surface Skeleton / Behavior Feature | |
| Declared decisions | Complies / Violates | |
| Structural checks | OK / Issues found | |
| Code risk review | PASS / FAIL | |
| Test adequacy | PASS / FAIL | |
| Scope control | In-scope / Scope-creep | |
| Tests | PASS / FAIL | |
| Lint/Format | PASS / FAIL | |
| Type check | PASS / FAIL | |
| Change hygiene | PASS / FAIL | |
| Finding card sync | PASS / FAIL | |
| Discovery sync | Updated / No changes / N/A | |
| Integration gaps tracking | Tracked / None / Missing | |
