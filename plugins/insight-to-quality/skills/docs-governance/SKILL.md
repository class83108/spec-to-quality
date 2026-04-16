---
name: docs-governance
description: >
  Sparse governance audit for cross-document consistency.
  Not a per-finding execution step. Use periodically or when drift/conflict is suspected.
  Modes: detect/sync/route/archive-check for one scope per run.
---

# Docs Governance (Sparse Audit, Not Mainline)

> **Output contract**: Traditional Chinese only.

## Positioning

This skill is a governance/audit tool, not a delivery pipeline step.
Do **not** run it for every finding by default.

Primary value:
- catch cross-document drift that per-finding skills may not see
- maintain consistency across architecture docs, spec reports, and delivery visibility docs
- provide archive recommendations without mutating active status flow

## When To Use

Use this skill in sparse cadence:
- before release / milestone close
- after major architecture updates (SYSTEM_MAP / dominant-ops / goals changes)
- when status/link inconsistencies are suspected (especially delivery-map drift)
- when you need archive recommendation for long-complete findings

Do not use this skill as a substitute for:
- `spec-contract` / `spec-surface` / `spec-behavior`
- `spec-to-gherkin` / `tdd-workflow` / `design-review`

## Governance Surface

Audit links and status across:
- `docs/SYSTEM_MAP.md`
- `docs/delivery-map.md`
- `docs/contracts/contracts-report.md`
- `docs/surface/surface-report.md`
- `docs/behaviors/behavior-report.md`
- `docs/spec-backlog/{finding-id}.md`
- codebase evidence (only as needed)

## Status Model (Source of Truth)

Active status flow is report-row driven:
- `draft -> ready -> in-progress -> done`

Rules:
- report main tables are source of truth for active status
- `done` requires evidence links in finding card/review outputs
- `in-progress` should be unique by default (`WIP=1`) unless linked-batch exception is documented

Archive policy:
- `archive-check` produces recommendation metadata only
- do not change report-row status from `done` to `archived`
- recommendation requires:
  - done for >= 30 days
  - no unresolved deferred E2E gaps
  - no reopen in last 2 release cycles

## Required Input Contract

Every run declares exactly one `scope` and one `mode`.

Scope (pick one):
- `finding:{id}` (e.g. `finding:contract-005`)
- `journey:{Dx}` (e.g. `journey:D2`)
- `seam:{id}` (e.g. `seam:B`)
- `doc:{path}` (e.g. `doc:docs/delivery-map.md`)

Mode (pick one):
- `detect`: detect drift/gap only (no edits)
- `sync`: update links/status fields only (no redesign)
- `route`: recommend one next skill per issue
- `archive-check`: evaluate archive recommendation for done findings

If scope or mode is missing, stop and ask for one.

## Analysis Budget

Per run limits:
- default max 8 files
- up to 12 files only for `seam:*` / `journey:*` or necessary cross-checks
- max 3 findings
- max 3 actions

If more work exists, stop and propose next scoped run.

## Workflow

### Step 0: Validate Inputs

- confirm one scope + one mode
- confirm required base docs exist (`goals`, `dominant-ops`, `SYSTEM_MAP`)

### Step 1: Minimal Read Set

Prefer this order:
1. scope target file
2. `docs/delivery-map.md`
3. corresponding spec report row(s)
   - `contract-*` -> `docs/contracts/contracts-report.md`
   - `surface-*` -> `docs/surface/surface-report.md`
   - `behavior-*` -> `docs/behaviors/behavior-report.md`
4. related finding card(s)
5. code truth file(s) only if needed

### Step 2: Execute Mode

- `detect`:
  - report drift/missing-link/stale-status
- `sync`:
  - update only links/status fields
  - keep semantics unchanged
- `route`:
  - map each issue to one next skill:
    - internal handoff/schema mismatch -> `spec-contract`
    - external interface shape mismatch -> `spec-surface`
    - behavior/goal semantics mismatch -> `spec-behavior`
    - scenario/test-level gap -> `spec-to-gherkin`
    - implementation evidence gap -> `tdd-workflow`
    - release-gate/evidence quality gap -> `design-review`
- `archive-check`:
  - evaluate done findings against archive policy
  - if pass, add archive recommendation note to finding card and/or delivery-map
  - do not mutate report-row status

### Step 3: Output Format

Return concise report:

| Item | Result |
|---|---|
| Scope | |
| Mode | |
| Files checked | |
| Findings (<=3) | |
| Actions (<=3) | |
| Next skill route | |

If budget exceeded, include:
- `Next suggested scope run`

## Handoff Rules

- Never auto-run downstream implementation skills here.
- Provide exact next-skill suggestion with one-sentence reason.
- Keep each run independently understandable (no hidden dependency).
