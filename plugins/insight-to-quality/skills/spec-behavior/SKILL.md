---
name: spec-behavior
description: >
  Define feature behavior finding cards to close business-logic gaps after skeleton TDD.
  Design mode: scan all Gx user flows, identify slices without feature findings, and set status/order.
  Verify mode: audit one Gx flow slice and produce a feature finding card.
  Requires goals.md, dominant-ops.md, SYSTEM_MAP.md, and existing skeleton reports (contracts/surface).
---

# Spec Behavior (Feature Behavior Coverage)

> **Output contract**：繁體中文。Design mode 建立/重寫 baseline，Verify mode 增量更新同一份 `docs/behaviors/behavior-report.md`。

Read before starting:
- `../../references/architect-mindset.md` (Gx traceability, user-journey thinking)
- `../../references/implementation-mindset.md` Part 0/1 (derivation + error strategy)
- `./references/behavior-report-template.md` (behavior-report structure and field spec)

## Scan Unit

The scan unit for spec-behavior is: **user flows and business behavior under each goal (Gx)**.

User flow means a sequence of actions triggered by external actors (users/downstream systems) that leads to an observable system response. Business behavior means the correctness rules for how the system should respond after user actions (state transitions, calculations, decision logic).

**Split with spec-contract**: spec-contract protects internal schema handoff correctness; spec-behavior protects business correctness of "what user does -> whether system response is correct".

**Split with spec-surface**: spec-surface protects external shape correctness (input validity/output schema). spec-behavior defines business semantics behind those shapes.

**When to use**: do not wait for all skeleton work to finish. Design mode can run once skeleton inventory exists. Verify mode only requires skeleton dependencies listed in the selected slice (`skeleton_deps`) to be done.

## Modes

Declare one mode at start:
- **Design mode (bootstrap)**: scan all Gx in goals.md, compare behavior report and skeleton reports, identify missing behavior slices, set status/order.
- **Verify mode (incremental)**: select one slice, refine business behavior, reference existing skeleton boundaries, create/sync feature finding card.

## Required Outputs

Common:
- [ ] Mode declared
- [ ] Report main-table index rows synced

Verify mode:
- [ ] `docs/behaviors/behavior-report.md` baseline exists; otherwise run Design mode first
- [ ] Exactly one behavior slice audited per run
- [ ] Same report main table updated
- [ ] WIP control applied (default WIP=1)
- [ ] Finding card created/updated on `ready -> in-progress`
- [ ] Finding card includes `type: feature` and `tests_path: tests/features/behaviors/`

Design mode:
- [ ] Full scan of all Gx in goals.md
- [ ] Compare behavior report vs skeleton reports to identify covered/missing behavior slices
- [ ] Every candidate slice has `status` (`draft/ready/in-progress/done/blocked`)
- [ ] Every `ready` row has `order` and `order_reason`
- [ ] `docs/behaviors/behavior-report.md` created/rewritten as baseline
- [ ] To-do items set as `ready`, deferred items set as `draft`
- [ ] No WIP limit in Design mode

## Verify Workflow

### Step 0: Scope Selection

Prerequisite: `docs/behaviors/behavior-report.md` must exist. If missing, stop and run Design mode.

List `draft/ready` slices from the report main table and confirm target slice with the user.

Default WIP=1: if any row is `in-progress`, only new `draft/ready` rows may be added.

Linked exception (allow 2-3 in-progress cards only when all apply):
- same user-flow chain (`slice` shares Gx prefix and must be accepted together)
- shared acceptance target that must be tested together
- each card `Source` includes `linked findings`
- linked cards are closed together

### Step 1: Audit and Classification

For selected slice, classify findings:
- **Gap**: behavior not defined in any finding card
- **Drift**: card exists but behavior description deviates from Gx intent
- **Risk**: card exists but business rules are ambiguous/conflicting with Gx constraints

Schema concern rule:
- If root cause is missing schema shape (input/output format, contract gap), **do not open in spec-behavior**
- Record dependency block and return to spec-surface/spec-contract for skeleton first
- Record in report `notes`: `dependency block: skeleton not done`
- behavior cards must reference existing skeleton boundaries instead of redefining schema

Infrastructure concern rule:
- If root cause is infrastructure decision gaps (transport, queue isolation, worker topology), **do not open finding cards**
- Record discovery conflict and escalate to Level 2
- Record in report `notes`: `escalate to SYSTEM_MAP`
- Update SYSTEM_MAP Architecture Decisions first, then return

### Step 2: Index Row Rules

| Field | Source and rule |
|---|---|
| `finding-id` | `behavior-xxx`; never reuse after stabilized |
| `slice` | `behavior:{Gx}:{flow-name}` |
| `serves` | directly map to goals.md Gx; no mapping -> `draft + needs-goal-clarification` |
| `related` | map to dominant-ops.md Dx/APx; may be blank if purely Gx-driven |
| `boundary` | from SYSTEM_MAP component/boundary; missing boundary stays `draft` |
| `skeleton_deps` | required skeleton finding-id list, format `surface-001,contract-002`; use `-` when none |
| `priority` | score = user impact x frequency x goal criticality; map to D1/D2/D3 |
| `status` | default `draft`; promote to `ready` only when required fields complete and all `skeleton_deps` are `done` (unless `-`) |
| `order` | required only for `ready`; increment from 1 |
| `order_reason` | required only for `ready`; one-sentence ranking rationale |

Status lifecycle:
- `draft`: required info missing or skeleton deps not done
- `ready`: required fields complete and deps done (or `-`)
- `in-progress`: selected for this run
- `done`: tests + docs synced
- `blocked`: unresolved dependencies

`order_reason` rules (high to low):
1. Goal criticality (core Gx flows first)
2. User visibility (directly observable feedback first)
3. Dependency first (upstream behavior before downstream)
4. Delivery readiness (deps done and slice is fully specified)

Writing rule example:
`R1 goal criticality: the G1 authentication flow is prerequisite for all subsequent Gx flows.`

### Step 3: Card Sync (Execution Start)

On `ready -> in-progress`, create `docs/spec-backlog/{finding-id}.md`.

Required fields:
- Source (Gx source + report link)
- Serves (Gx), Related (Dx/APx, optional), Boundary
- `type: feature`
- `tests_path: tests/features/behaviors/`
- `skeleton_deps` (required skeleton finding ids when applicable)
- Behavior (SHALL/MUST for user action -> correct system response; do not redefine schema)
- State Transitions (before/after states when state machine applies)
- Error Handling Strategy (business rule violation handling)
- Done Criteria (each maps to Then assertions anchored to user-observable outcomes)
- Out of Scope
- Integration Test Gaps (optional)

Feature TDD note:
- validate business correctness for user behavior and resulting system state/response
- schema boundaries remain covered by existing skeleton tests
- implementation should reference existing skeleton contracts instead of remocking schema validation in feature tests

Done Criteria wording examples:
- Correct: "user submits valid credentials -> system creates session and marks it authenticated"
- Correct: "user accesses unauthorized resource -> request denied, attempt logged, permission-denied returned"
- Incorrect: "missing required input field -> return 400" (surface skeleton scope)

### Step 4: Report Sync

Update `docs/behaviors/behavior-report.md` main table (no iteration append), including backlog linkage (`finding-id` list).

## Design Workflow

Goal: produce full inventory + status + order for all Gx behavior slices as the Verify work queue.

### Step D0: Candidate Behavior Slice Inventory

Scan all goals in goals.md and cross-check with SYSTEM_MAP user-flow descriptions to identify candidate slices per Gx.

Also scan existing reports:
- existing `type: feature` slices in behavior report: preserve existing states (`in-progress/done`)
- existing skeleton findings in contracts/surface reports: record as candidate `skeleton_deps`, do not duplicate cards
- Gx flows with no feature slice coverage: add as candidate slices

Each candidate slice includes one-line `flow-note` (what the user tries to do and expected outcome). Do not define full business rules/state machine in Design mode.

### Step D1: Status Assignment

Assign each candidate slice:
- `ready`: do now, Gx is clear, and required skeleton deps are done
- `draft`: deferred, Gx unclear, or skeleton deps not done
- `in-progress`: existing ongoing item (preserve on rerun)
- `done`: existing completed item (preserve on rerun)
- `blocked`: unresolved dependency (skeleton incomplete or missing SYSTEM_MAP boundary)

### Step D2: Ordering (`ready` only)

Fill `order` (1..n) and `order_reason` for `ready` slices using Verify Step 2 rules.

### Step D3: Main-Table Sync

Sync decisions to `docs/behaviors/behavior-report.md` Behavior Slices Table:
- `ready`: fill `order/order_reason/skeleton_deps`
- `draft`: keep missing info and reason
- `in-progress/done/blocked`: preserve existing state

### Step D4: Baseline Output

Create or rewrite `docs/behaviors/behavior-report.md` baseline with template-compatible section structure and table headers.

### Step D5: Guardrail

If discussion reveals:
- **Schema concern**: do not open feature finding card; ensure required skeleton finding exists first
- add report note: `dependency block: skeleton not done`
- **Infrastructure concern**: do not open finding card; update SYSTEM_MAP Architecture Decision first
- add report note: `escalate to SYSTEM_MAP`

## Completion Message

Report:
- Design mode: full scan result (Gx count, missing behavior slices count), status counts, ready order (1..n), unresolved `skeleton_deps` list (if any)
- Verify mode: audited behavior slice, status/order updates, whether card was created/updated, and referenced skeleton findings
