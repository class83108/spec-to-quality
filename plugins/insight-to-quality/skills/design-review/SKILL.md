---
name: design-review
description: >
  Design quality verification after TDD green, plus release-gate checks.
  Verifies finding-card decisions, structural quality, and completion evidence
  before marking backlog item done.
---

# Design Review (+ Release Gate)

This stage combines design verification and final release checks.

Before starting, read:
- `references/architect-mindset.md` (traceability and boundary discipline)
- `references/implementation-mindset.md` Part 2 (structural checks)
- current finding card

## Prerequisites

- current finding is `in-progress` in index
- TDD is green for this finding

## Part 1: Declared Decisions Verification

Verify implementation against finding card:
- catch boundary
- domain errors
- infrastructure recovery
- boundary rules
- anti-pattern constraints

If violations exist, report directly with `file:line`.

## Part 2: Structural Checks

Assess:
- separation of concerns
- dependency direction
- naming semantics
- testability
- consistency

Use questions for improvements; use direct statements for clear violations.

## Part 3: Release Gate (mandatory)

Run and record:
1. tests
2. lint/format
3. type check
4. `git diff --stat` + `git status`
5. finding card sync status (Done Criteria evidence + gaps tracking)
6. discovery sync check (SYSTEM_MAP / dominant-ops / goals updates if needed)

## Completion Rules

Mark index row to `done` only when:
- no blocking Part 1 violations
- Part 3 commands executed with evidence
- unresolved integration gaps are tracked (test/issue/manual checklist)

## Output Table

| Item | Status | Notes |
|---|---|---|
| Declared decisions | Complies / Violates | |
| Structural checks | OK / Issues found | |
| Tests | PASS / FAIL | |
| Lint/Format | PASS / FAIL | |
| Type check | PASS / FAIL | |
| Change hygiene | PASS / FAIL | |
| Finding card sync | PASS / FAIL | |
| Discovery sync | Updated / No changes / N/A | |
| Integration gaps tracking | Tracked / None / Missing | |
