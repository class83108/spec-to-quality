---
name: spec-surface
description: >
  Define and audit interaction contracts between external actors and the system before implementation.
  Design mode: coarse-grained inventory of all interface boundary areas and set status/order.
  Verify mode: refine one boundary area into an exact contract shape and produce a skeleton finding card.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md.
---

# Spec Surface (External Interface Contracts)

> **Output contract**：繁體中文。Design mode 建立/重寫 baseline，Verify mode 增量更新同一份 `docs/surface/surface-report.md`。

Read before starting:
- `../../references/architect-mindset.md` (dominant operations thinking, boundary abstraction tests)
- `../../references/implementation-mindset.md` Part 0/1 (derivation + error strategy)
- `./references/surface-report-template.md` (surface-report structure and field spec)

## Scan Unit

The scan unit for spec-surface is: **external actor to system interaction interfaces**.

External actors include users, downstream systems, and integrators. Interfaces are boundary contracts actors use to trigger/query/receive system behavior. Transport/protocol choices (HTTP, WebSocket, message bus) are decided in SYSTEM_MAP Architecture Decisions. spec-surface only focuses on **contract shape**: accepted input, produced output, and error signaling.

**Split with spec-contract**: spec-contract handles internal handoff schemas. spec-surface handles external boundary interaction contracts. Both produce skeleton findings, but scan directions differ.

**Split with spec-behavior**: spec-surface defines interface shapes and boundary guards only (input validity and output schema conformance). Business semantics after shape stabilization belongs to spec-behavior.

## Modes

Declare one mode at start:
- **Design mode (bootstrap)**: inventory all outward boundary areas in SYSTEM_MAP and record potential entry points; do not define route-level details or full schema yet.
- **Verify mode (incremental)**: pick one boundary area and refine exact contract shape (input/output schema + error contract), then sync findings/index.

## Required Outputs

Common:
- [ ] Mode declared
- [ ] Index rows in report main table synced

Verify mode:
- [ ] `docs/surface/surface-report.md` baseline exists; otherwise switch to Design mode
- [ ] Exactly one boundary area audited per run
- [ ] Same report main table updated
- [ ] WIP control applied (default WIP=1)
- [ ] Finding card created/updated on `ready -> in-progress`
- [ ] Finding card includes `type: skeleton` and `tests_path: tests/features/contracts/`

Design mode:
- [ ] Full boundary-area inventory from SYSTEM_MAP (not single-point)
- [ ] Every area has `status` (`draft/ready/in-progress/done/blocked`)
- [ ] Every `ready` row has `order` and `order_reason`
- [ ] `docs/surface/surface-report.md` created/rewritten as baseline
- [ ] Items to do are `ready`; deferred are `draft`
- [ ] No WIP limit in Design mode

## Verify Workflow

### Step 0: Scope Selection

Prerequisite: `docs/surface/surface-report.md` must exist. If missing, stop and run Design mode.

List `draft/ready` boundary areas from the report main table and confirm this run target with the user.

Default WIP=1: when any row is already `in-progress`, only new `draft/ready` rows can be added.

Linked exception (allow 2-3 `in-progress` rows only when all apply):
- same boundary chain (`slice` shares boundary-area prefix)
- shared acceptance target that must be delivered together
- each card `Source` includes `linked findings`
- linked cards are closed together

### Step 1: Audit and Classification

For the selected boundary area, classify findings:
- **Gap**: contract undefined (input/output schema not specified, error codes undefined)
- **Drift**: contract documented but implementation deviates (response schema mismatch)
- **Risk**: contract exists but violates Dx anti-patterns (for example, missing idempotency key)

Infrastructure concern rule:
- If root cause is an infrastructure decision gap (transport selection, queue isolation, worker topology, capacity), **do not open finding cards**
- Record as discovery conflict and escalate to Level 2
- Update SYSTEM_MAP Architecture Decisions first, then return

Tech-stack decision rule:
- Concrete tech choices (for example SSE vs WebSocket, framework versions) are **not** spec-surface scope
- Those decisions come from system-map consuming dominant-ops Design Implications
- If unresolved, record dependency block and do not open a finding card

### Step 2: Index Row Rules

| Field | Source and rule |
|---|---|
| `finding-id` | `surface-xxx`; never reuse after stabilized |
| `slice` | `interface:{boundary-area}:{operation}` |
| `serves` | map to goals.md Gx; no mapping -> `draft + needs-goal-clarification` |
| `related` | map to dominant-ops.md Dx/APx, ordered by criticality |
| `boundary` | from SYSTEM_MAP component/boundary names; missing boundary stays `draft` |
| `priority` | score = failure impact x frequency x cost; map to D1/D2/D3 bucket |
| `status` | default `draft`; promote to `ready` when `serves/related/boundary/done criteria seed` complete |
| `order` | required only for `ready`; increment from 1 |
| `order_reason` | required only for `ready`; one-sentence ranking rationale |

Status lifecycle:
- `draft`: required info missing
- `ready`: required fields complete
- `in-progress`: selected for this run
- `done`: tests + docs synced
- `blocked`: dependency unresolved

`order_reason` rules (high to low):
1. Dependency first (upstream interfaces first)
2. Failure cost (irreversible state or user-visible failures first)
3. Coverage breadth (one contract shape protecting multiple flows first)
4. Delivery readiness (fully specified and directly testable first)

Writing rule example:
`R1 dependency first: session creation is prerequisite for all G2 follow-up operations.`

### Step 3: Card Sync (Execution Start)

On `ready -> in-progress`, create `docs/spec-backlog/{finding-id}.md`.

Required fields:
- Source (boundary area + report link)
- Serves (Gx), Related (Dx/APx), Boundary
- `type: skeleton`
- `tests_path: tests/features/contracts/`
- Behavior (SHALL/MUST for contract shape guarantees)
- Error Handling Strategy (error codes, boundary guard, client-visible error format)
- Done Criteria (each maps to Then assertions, including user-visible confirmation such as status code/response schema/event format)
- Out of Scope
- Integration Test Gaps (optional)

Skeleton TDD note:
- validate input schema guarding and output schema shape only
- implementation may use stubs returning schema-conformant mock outputs
- business behavior belongs to spec-behavior and should not be opened here

Done Criteria wording examples:
- Correct: "missing required input field -> return 4xx with ErrorResponse schema"
- Correct: "valid request -> return 2xx with SuccessResponse schema"
- Incorrect: "user can complete authentication" (this is behavior scope)

### Step 4: Report Sync

Update `docs/surface/surface-report.md` main table per template (no iteration append), including backlog linkage (`finding-id` list).

## Design Workflow

Goal: produce coarse-grained inventory + status + order for all external boundary areas as the Verify work queue.

### Step D0: Candidate Boundary Areas

Scan SYSTEM_MAP Component Map end-to-end and list all boundary areas where external actors interact with the system.

Each area should include a one-line `scope-note` describing what the actor is likely trying to do. Do not define exact operations/routes/schema here.

### Step D1: Status Assignment

Assign each candidate area:
- `ready`: do now, Gx/Dx mapping clear
- `draft`: not now or mapping unclear
- `in-progress`: existing ongoing item (preserve on rerun)
- `done`: existing completed item (preserve on rerun)
- `blocked`: unresolved dependency (for example missing SYSTEM_MAP boundary)

### Step D2: Ordering (`ready` only)

Fill `order` (1..n) and `order_reason` for `ready` rows using Verify Step 2 rules.

### Step D3: Main-Table Sync

Sync this run's decisions into `docs/surface/surface-report.md` Interface Boundaries Table:
- `ready`: fill `order/order_reason`
- `draft`: keep missing info and reason
- `in-progress/done/blocked`: preserve existing state

### Step D4: Baseline Output

Create or rewrite `docs/surface/surface-report.md` baseline with template-compatible section structure and table headers.

### Step D5: Guardrail

If discussion reveals:
- **Infrastructure concern**: do not open finding card; update SYSTEM_MAP Architecture Decision first
- **Tech-stack concern**: record dependency block and return after system-map decision

## Completion Message

Report:
- Design mode: full scan result, status counts, `ready` order (1..n)
- Verify mode: audited boundary area, status/order updates, and card create/update result
