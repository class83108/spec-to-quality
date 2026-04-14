---
name: align-surface
description: >
  Design or verify surface-layer alignment — interfaces and infrastructure shaped by dominant-ops.
  In verify mode, produce executable backlog outputs into docs/spec-backlog/.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md.
---

# Align Surface (Interfaces and Infrastructure)

> **Output contract**: Traditional Chinese only. In verify mode, save `docs/alignment/surface-report.md` and sync spec backlog.

Before starting, read:
- `references/architect-mindset.md` (dominant operations thinking)
- `references/implementation-mindset.md` Part 0/1 (derivation + error strategy)

## Required Outputs

- [ ] Mode declared (design / verify)
- [ ] Verify mode audits one scope only (one Dx journey or Infrastructure)
- [ ] Surface report saved/appended
- [ ] Index rows synced for findings
- [ ] If one item enters execution, card created/updated

## Verify Workflow

### Step 0: Scope

- List pending Dx journeys + Infrastructure scope
- Pick one scope with user confirmation
- Respect `WIP=1`: if another finding is in-progress, add only `draft/ready`

### Step 1: Audit and Classify

For selected scope, classify findings:
- 缺口（journey/endpoint missing）
- Drift（behavior/docs mismatch）
- 風險（classification wrong, missing idempotency/reconnect/capacity)

### Step 2: Derivation Rules (How to Fill Index Fields)

For each finding row, fill fields with these rules:

1. `finding-id`
- surface prefix only: `S-xxx`
- stable once assigned; never reuse old IDs

2. `slice`
- `journey:{Dx-name}` or `infra:{scope-name}`

3. `serves`
- source: `goals.md`
- method: map user-visible outcome to Gx sentence
- if ambiguous: keep `draft` + `needs-goal-clarification`

4. `related`
- source: `dominant-ops.md`
- method: map scope to Dx/APx; sort by criticality

5. `boundary`
- source: `SYSTEM_MAP.md` component/boundary names
- missing boundary -> do not invent; keep `draft` and request system-map update

6. `priority`
- score = failure impact x frequency x cost
- map to D1/D2/D3 bucket

7. `status`
- default `draft`
- `ready` only if `serves/related/boundary/done criteria seed` complete

### Step 3: Card Sync (when execution starts)

Create `docs/spec-backlog/{finding-id}.md` when `ready -> in-progress`.

Surface-specific card rules:
- include endpoint category assumption (Trigger/Query/Push) when relevant
- Done Criteria must include user-visible confirmation signal (status code, UI state, event)

### Step 4: Report Sync

Append findings to `docs/alignment/surface-report.md` and include backlog linkage (finding-id list).

## Design Mode

Design interfaces/infrastructure from dominant operations and constraints. Add implied implementation work as `draft` rows using the same derivation rules.

## Completion Message

Report:
- audited scope
- report update result
- rows added/updated
- whether card was created
