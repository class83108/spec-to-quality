---
name: align-internals
description: >
  Design or verify data-layer alignment — contracts and persistence shaped by discovery documents.
  In verify mode, produce executable backlog outputs into docs/spec-backlog/.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md.
---

# Align Internals (Contracts and Persistence)

> **Output contract**: Traditional Chinese only. In verify mode, save `docs/alignment/internals-report.md` and sync spec backlog.

Before starting, read:
- `references/architect-mindset.md` (Traceability, boundary abstraction tests)
- `references/implementation-mindset.md` Part 0/1 (derivation + error strategy)

## Required Outputs

- [ ] Mode declared (design / verify)
- [ ] Verify mode audits one seam only
- [ ] Internals report saved/appended
- [ ] Index rows synced for findings
- [ ] If one item enters execution, card created/updated

## Verify Workflow

### Step 0: Scope

- Read SYSTEM_MAP seams
- Pick one seam with user confirmation
- Check `docs/spec-backlog/index.md` for existing `in-progress` item
  - if another item is in-progress, do not promote new item; only add `draft/ready`

### Step 1: Audit and Classify

For selected seam, classify each finding:
- 缺口（code missing behavior/contract）
- Drift（doc/code mismatch）
- 風險（works now but violates anti-pattern or boundary rule）

### Step 2: Derivation Rules (How to Fill Index Fields)

For each finding row, fill fields with these rules:

1. `finding-id`
- internals prefix only: `I-xxx`
- stable once assigned; never reuse old IDs

2. `slice`
- fixed format: `seam:{SYSTEM_MAP seam name}`

3. `serves`
- source: `goals.md`
- method: map finding outcome to at least one Gx objective sentence
- if no direct mapping: set row to `draft` and tag `needs-goal-clarification` in notes

4. `related`
- source: `dominant-ops.md`
- method: map impacted operation chain to Dx/APx IDs; sort by criticality (D1 > D2 > D3)

5. `boundary`
- source: `SYSTEM_MAP.md` boundary map name only
- if boundary missing in SYSTEM_MAP: do not invent; add `draft` and request system-map update

6. `priority`
- score = failure impact x frequency x cost (High/Med/Low)
- convert to `D1` / `D2` / `D3` bucket by dominant operation tie

7. `status`
- default `draft`
- upgrade to `ready` only if `serves/related/boundary/done criteria seed` are complete

### Step 3: Card Sync (when execution starts)

Create `docs/spec-backlog/{finding-id}.md` when `ready -> in-progress`.

Card minimum fields:
- Source
- Traceability (`Serves`, `Related`, `Boundary`)
- Behavior (SHALL/MUST)
- Error Handling Strategy
- Done Criteria
- Out of Scope
- Integration Test Gaps

Done Criteria writing rule:
- each criterion must be observable and testable (can map to Then assertion)
- ban vague criteria (e.g., "improve quality", "handle better")

### Step 4: Report Sync

Append findings to `docs/alignment/internals-report.md` and include backlog linkage (finding-id list).

## Design Mode

Design contracts/persistence from discovery docs. If implementation work is implied, add `draft` rows to index using the same derivation rules.

## Completion Message

Report:
- audited seam
- report update result
- rows added/updated
- whether card was created
