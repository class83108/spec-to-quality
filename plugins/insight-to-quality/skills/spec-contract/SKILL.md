---
name: spec-contract
description: >
  Define or audit internal data-flow handoff schema contracts, prioritizing high-pressure Dx paths.
  Design mode: inventory all handoffs and set status/order.
  Verify mode: incrementally audit one handoff and update the same contracts report and backlog.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md.
---

# Spec Contract (Data-Flow Contracts)

> **Output contract**：繁體中文。Design mode 建立/重寫 baseline，Verify mode 增量更新同一份 `docs/contracts/contracts-report.md`。

Read before starting:
- `../../references/architect-mindset.md` (traceability, boundary abstraction tests)
- `../../references/implementation-mindset.md` Part 0/1 (derivation + error strategy)
- `./references/contracts-report-template.md` (contracts-report structure and field spec)

## Scan Unit

The scan unit for spec-contract is: **internal data-flow handoff schema contracts**.

A handoff is a data exchange point between components. spec-contract focuses on whether the exchanged data is protected by a stable and verifiable schema boundary (field shape, required constraints, boundary guards, and error strategy), so schema drift does not spread through internal chains.

**Split of responsibility with spec-surface**: spec-surface defines external interaction contracts (API/CLI/event boundaries). spec-contract defines internal component-to-component handoff shapes. Both produce skeleton finding cards, but scan directions differ: spec-surface looks outward, spec-contract looks inward.

**Split of responsibility with spec-behavior**: spec-contract defines data handoff contracts and guards only; it does not define business semantics of user flows. User-action semantics belong to spec-behavior and should reference existing skeleton boundaries.

## Modes

Declare one mode at start:
- **Design mode (bootstrap)**: inventory all SYSTEM_MAP data-flow handoffs and set `status` and `order`.
- **Verify mode (incremental)**: audit one handoff on top of the existing baseline and sync row status + card state.

## Required Outputs

Common:
- [ ] Mode declared
- [ ] Index rows in report main table synced

Verify mode:
- [ ] `docs/contracts/contracts-report.md` baseline exists; otherwise stop and switch to Design mode
- [ ] Exactly one handoff audited per run
- [ ] Same report main table updated (no iteration append)
- [ ] WIP control applied (default WIP=1)
- [ ] Finding card created/updated on `ready -> in-progress`
- [ ] Finding card includes `type: skeleton` and `tests_path: tests/features/contracts/`

Design mode:
- [ ] Full handoff inventory from SYSTEM_MAP (not a single-point audit)
- [ ] Every handoff has `status` (`draft/ready/in-progress/done/blocked`)
- [ ] Every `ready` row has `order` and `order_reason`
- [ ] `docs/contracts/contracts-report.md` created/rewritten as baseline
- [ ] Items to execute are synced as `ready`; deferred items as `draft`
- [ ] No WIP limit in Design mode (goal is inventory, not execution)

## Verify Workflow

### Step 0: Handoff Selection

Prerequisite: `docs/contracts/contracts-report.md` must exist. If missing, stop Verify mode and run Design mode.

List auditable handoffs from the report main table and confirm this run's target with the user.

Principles:
- Do now + information complete -> `ready`
- Not now or information incomplete -> `draft`

Default WIP=1: if any row is already `in-progress`, only new `draft/ready` rows may be added.

Linked exception (allow 2-3 `in-progress` cards only when all apply):
- same handoff chain (`slice` shares chain prefix)
- shared acceptance target that must be verified together
- each card `Source` lists `linked findings`
- linked cards are closed together (no partial delivery)

### Step 1: Audit and Classification

For the selected handoff, classify each finding:
- **Gap**: no schema protection on the handoff (raw pass-through without type/format constraints)
- **Drift**: schema defined but implementation does not match
- **Risk**: schema exists but violates Dx anti-patterns (for example, missing idempotency key or weak boundary guard)

Infrastructure concern rule:
- If the issue is fundamentally an infrastructure decision gap (queue isolation, transport, capacity, worker topology), **do not open a finding card**
- Record as discovery conflict and escalate to Level 2 (Discovery Conflict Triage)
- Update SYSTEM_MAP Architecture Decision first, then return to this flow

### Step 2: Index Row Rules

| Field | Source and rule |
|---|---|
| `finding-id` | `contract-xxx`; never reuse after stabilized |
| `slice` | `handoff:{from-component}->{to-component}` |
| `serves` | map to goals.md Gx; no mapping -> `draft + needs-goal-clarification` |
| `related` | map to dominant-ops.md Dx/APx, ordered by criticality |
| `boundary` | from SYSTEM_MAP boundary name; if missing keep `draft` and request SYSTEM_MAP update |
| `status` | default `draft`; promote to `ready` only when `serves/related/boundary/done criteria seed` are complete |
| `order` | required only for `ready`; increment from 1 |
| `order_reason` | required only for `ready`; one sentence with ranking rationale |

Status lifecycle (always record transition rationale):
- `draft`: missing required information
- `ready`: required fields complete
- `in-progress`: selected for this run
- `done`: tests + docs fully synced (design-review + docs-governance done)
- `blocked`: unresolved dependency

`order_reason` ranking rules (high to low):
1. Dependency first: upstream handoffs before downstream
2. Failure cost: data pollution/irreversible writes/cross-boundary spread first
3. Coverage breadth: one schema protecting multiple flows first
4. Delivery readiness: complete info and directly testable first

Writing rule: `order_reason` must cite at least one rule, for example:
`R1 dependency first: Pipeline->StageRunner is prerequisite for three downstream handoffs.`

### Step 3: Card Sync (Execution Start)

On `ready -> in-progress`, create `docs/spec-backlog/{finding-id}.md`.

Required fields:
- Source (handoff source + report link)
- Serves (Gx), Related (Dx/APx), Boundary
- `type: skeleton`
- `tests_path: tests/features/contracts/`
- Behavior (SHALL/MUST for schema boundary guarantees)
- Error Handling Strategy
- Done Criteria (each maps to Then assertions; no vague wording)
- Out of Scope
- Integration Test Gaps (optional, later ledger fill)

### Step 4: Report Sync

Update `docs/contracts/contracts-report.md` main table following `./references/contracts-report-template.md`.
Do not append iteration logs.

## Design Workflow

Goal: produce a full handoff inventory + status + order as the skeleton implementation starting point.

### Step D0: Candidate Handoff Inventory

Scan SYSTEM_MAP data flows end-to-end and list all candidate handoffs (not a single sample).

### Step D1: Status Assignment

Assign each candidate handoff:
- `ready`: should be implemented now/soon and info is sufficient
- `draft`: deferred or missing information
- `blocked`: dependency unresolved

### Step D2: Ordering (`ready` only)

Fill `order` (1..n) and `order_reason` for `ready` rows using Verify Step 2 rules.

### Step D3: Contract Design + Main-Table Sync

Design schema contracts for `ready` rows and sync to `docs/contracts/contracts-report.md` main table.
`draft` rows keep placeholders only and do not enter execution.

### Step D4: Baseline Report Output

Create or rewrite `docs/contracts/contracts-report.md` baseline with at least:
- scan scope and source documents
- handoff main table (`status/order/order_reason/finding-id/card`)

Structure must match `./references/contracts-report-template.md` baseline section.

### Step D5: Guardrail

If infrastructure concerns appear during discussion, apply Verify Step 1 routing: do not open finding cards; update SYSTEM_MAP Architecture Decisions first.

## Completion Message

Report:
- Design mode: full scan result, status counts, `ready` ordering (1..n)
- Verify mode: audited handoff, status/order updates, and whether card was created/updated
