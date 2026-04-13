---
name: align-internals
description: >
  Design or verify data-layer alignment — contracts and persistence shaped by discovery documents.
  In design mode (no existing code), guide the user through contract and schema design driven by
  goals, dominant-ops, and system map boundaries. In verify mode (existing code), audit whether
  contracts and persistence align with discovery documents and surface gaps.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md to exist.
  Do NOT use for: defining goals (use goals-discovery), analyzing pressure (use dominant-ops),
  mapping system structure (use system-map), or designing user-facing interfaces (use align-surface).
---

# Align Internals (Contracts and Persistence)

> **Output contract** — All responses and documents must be in **Traditional Chinese (繁體中文)**. In verify mode, the gap report must be saved to **`docs/alignment/internals-report.md`** before this skill is declared complete. Do not ask the user where to save it — the path is fixed.

You are guiding the user through **data-layer alignment** — ensuring that the system's contracts (schemas, interfaces, protocols) and persistence (database models, storage) are shaped by the discovery documents, not by accident or convention.

This skill operates in two modes:
- **Design mode**: No existing code. Guide the user to design contracts and schemas from scratch, driven by goals and dominant-ops.
- **Verify mode**: Existing code. Audit whether the current contracts and persistence align with discovery documents. Surface gaps and misalignments.

Determine the mode by asking: "Is there existing code for contracts and/or database models, or are we designing from scratch?"

Read `references/architect-mindset.md` before proceeding, especially Traceability and What NOT to Abstract.

## Working Style

- **Contracts serve boundaries.** Every contract exists because SYSTEM_MAP.md identified a boundary (seam). If a contract does not map to a seam, question why it exists. If a seam has no contract, that is a gap.
- **Persistence serves goals.** Every table/model exists because a goal requires storing something. If a table does not trace to a Gx, question why it exists. If a goal requires data that is not persisted, that is a gap.
- **Dominant-ops shape the design.** D1/D2/D3 determine where to invest in schema quality, indexing, validation, and error handling. Non-dominant operations get minimal persistence design.
- **Do not over-design.** If the framework provides good defaults (e.g., Django ORM, SQLAlchemy), use them. Do not add repository patterns, DAOs, or abstraction layers unless a boundary test justifies them.

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following. Do not write contracts or end the session until every item is checked:

- [ ] Mode declared at the start (design mode or verify mode)
- [ ] Every seam from SYSTEM_MAP's Boundary Map explicitly assessed — do not skip or summarize
- [ ] Contract status stated for each seam: `exists / defined / gap`
- [ ] Storage seam contract assessed — what does the Domain layer expect from storage?
- [ ] **Open Questions link**: If goals.md has Open Questions (OQ1, OQ2...) that affect contract design, explicitly discuss how each OQ constrains or defers the contract decision — do not leave OQs disconnected from the contracts
- [ ] User confirmation of contract designs or gap report
- [ ] **[Verify mode only]** Gap report saved to `docs/alignment/internals-report.md` with the following structure — do not declare complete until the file exists:

  ```markdown
  # Internals Alignment Report — [System Name]

  ## Summary
  [one paragraph: overall alignment status]

  ## Aligned
  - [list of elements that match]

  ## Gaps (code is missing)
  - [seam/goal] — [what is missing] — [recommended action]

  ## Drift (docs vs code mismatch)
  - [element] — [what differs] — [which is correct?]

  ## Recommendations
  - [prioritized list, Dx-aligned items first]
  ```

**N/A Policy**: If a seam has no contract yet (e.g., early design phase), write `gap — [reason why contract is deferred]`. Never silently omit a seam from the assessment.

## Prerequisites

- **goals.md** — for Gx traceability and NFR requirements
- **dominant-ops.md** — for Dx priorities, anti-patterns, and theory limits
- **SYSTEM_MAP.md** — for boundary/seam definitions
- If any are missing, redirect to the appropriate skill

## Workflow — Design Mode

Use this workflow when there is no existing code and you are helping the user design contracts and schemas from scratch.

### Phase 1: Boundary Inventory

Start from SYSTEM_MAP.md's Boundary Map. For each seam:

1. **List what crosses the boundary**: What data flows from one side to the other?
2. **Determine direction**: Is it one-way (producer -> consumer) or bidirectional?
3. **Identify the contract type**: Schema (Pydantic, JSON Schema), API endpoint, event payload, shared DB table, file format
4. **Link to Dx**: Which dominant operation uses this boundary most heavily?

### Phase 2: Contract Design

For each boundary that needs a contract:

1. **Define the schema shape**: What fields are required? What types? What constraints?
2. **Apply the Dx lens**: If D1 uses this boundary, the contract needs extra attention — strict validation, clear error messages, versioning strategy
3. **Define error contracts**: What does the producer return when it fails? How does the consumer know something went wrong?
4. **Idempotency**: Can the consumer safely receive the same data twice? If this boundary supports retry/re-run (common in D1-D3), idempotency is not optional.

**Design principle**: The contract should be defined by the consumer's needs, not the producer's convenience. Ask: "What does the downstream component need to do its job?"

### Phase 3: Persistence Design

For each goal that requires storing data:

1. **Identify what needs to persist**: State, configuration, history, snapshots, metrics
2. **Determine the lifecycle**: Created once? Updated frequently? Append-only? Soft-delete?
3. **Determine the query patterns**: How will this data be read? By primary key? By filter? By time range? This drives indexing decisions.
4. **Link to anti-patterns**: Does AP-x from dominant-ops.md constrain how this data should be stored?

**Common persistence patterns to consider**:
- **Source of truth**: Which component owns this data? Others must read through the owner, not query directly.
- **Snapshot vs. current**: Does the system need to remember what data looked like at a point in time, or only the current state?
- **Cleanup strategy**: When data becomes orphaned (e.g., after a re-run), how is it cleaned up? Immediately? After review? Never?

### Phase 4: Cross-Contract Validation

Check the contracts work together:

1. **Shape match**: For each boundary, does the producer's output schema match the consumer's input schema? Field names, types, optional/required — all must align.
2. **State machine coherence**: If entities move through states (e.g., pending -> processing -> completed), is the state machine consistent across all contracts that touch it?
3. **Referential integrity**: If Contract A references an ID from Contract B, is that ID guaranteed to exist?

### Phase 5: Output

Produce contract definitions and schema designs. The exact format depends on the tech stack, but should include:
- Schema definitions (Pydantic models, JSON Schema, Protobuf, TypeScript interfaces, etc.)
- Database model specifications (tables, columns, types, indexes, constraints)
- Relationship map (which models reference which)
- Open questions about schema decisions

## Workflow — Verify Mode

Use this workflow when code exists and you are auditing alignment.

### Phase 1: Read Discovery Documents

Read goals.md, dominant-ops.md, and SYSTEM_MAP.md. Build a mental checklist:
- Every Gx that implies persistence
- Every Dx that implies a contract
- Every seam in the Boundary Map

### Phase 2: Read Existing Code

Read the actual contract definitions and database models:
- Schema files (Pydantic models, serializers, type definitions)
- Database models (ORM definitions, migrations)
- API contracts (endpoint signatures, request/response types)

### Phase 3: Alignment Audit

For each discovery element, check whether code aligns:

| Check | Question | Finding |
|---|---|---|
| Boundary coverage | Does every seam in SYSTEM_MAP have a contract in code? | gap / aligned |
| Goal persistence | Does every Gx that implies storage have a corresponding model? | gap / aligned |
| Dx contract quality | Do D1/D2/D3 boundaries have strict validation and error contracts? | weak / aligned |
| Anti-pattern adherence | Does the code avoid the anti-patterns listed in dominant-ops.md? | violation / aligned |
| Schema consistency | Do producer output and consumer input schemas match? | mismatch / aligned |

### Phase 4: Document Drift

Check whether the discovery documents match the code:
- **Code has something docs do not describe**: Either the code is speculative (remove it) or the docs are incomplete (update them)
- **Docs describe something code does not implement**: Either it is planned (check status) or it was dropped (update docs)

### Phase 5: Gap Report

Produce a structured report and save it to `docs/alignment/internals-report.md` (create the directory if it does not exist). Do not skip the save step — `feature-planning` reads this file by path.

```markdown
# Internals Alignment Report — [System Name]

## Summary
[one paragraph: overall alignment status]

## Aligned
- [list of elements that match]

## Gaps (code is missing)
- [seam/goal] — [what is missing] — [recommended action]

## Drift (docs vs code mismatch)
- [element] — [what differs] — [which is correct?]

## Recommendations
- [prioritized list, Dx-aligned items first]
```

After saving, inform the user: "`docs/alignment/internals-report.md` 已儲存。若 `align-surface` 尚未執行，建議接著執行以取得完整的 alignment 視角，再進入 `feature-planning`。"

## Design Checks

Revisit this skill if:
- A new seam is added to SYSTEM_MAP.md — it needs a contract
- A dominant operation changes — contracts on its path may need strengthening
- A production bug traces to a schema mismatch between components
- A migration fails — check whether the schema matches the contract definition

## Examples

### Example 1: Design Mode — Missing Error Contract

User designs a contract between a task queue and a worker: `{ "task_id": str, "payload": dict }`.

Push back: "What does the worker return when it fails? Currently there is no error contract. If the worker crashes, the queue does not know whether to retry or mark as failed. Consider adding a result schema: `{ "task_id": str, "status": "success" | "failed" | "retry", "error": str | null }`."

### Example 2: Verify Mode — Persistence Without Goal

Audit finds a `NotificationPreference` model in the database, but goals.md has `NG4: The system will not send notifications`.

Flag it: "NotificationPreference exists in the database but NG4 explicitly excludes notifications. Is this leftover from a previous iteration? Should it be removed, or does NG4 need updating?"

### Example 3: Anti-Pattern Violation

dominant-ops.md has `AP2: DB is the single source of truth between stages`. Code review finds a function passing results between processing steps via return values, not writing to DB first.

Flag it: "This function passes intermediate results in memory between steps. AP2 requires DB as the single source of truth between stages. If the process crashes after step 1 but before step 2, the intermediate result is lost. Consider writing to DB after each step."

### Example 4: Schema Shape Mismatch

Contract A outputs `{ "user_id": int, "name": str }`. Contract B expects `{ "userId": int, "fullName": str }`.

Flag it: "Field naming mismatch between Contract A output and Contract B input: 'user_id' vs 'userId', 'name' vs 'fullName'. These will fail silently if passed directly. Either standardize naming or add an explicit mapping layer at the boundary."

## Key Rules

- **Every contract maps to a seam.** Orphan contracts (no seam in SYSTEM_MAP) are either missing from the map or should not exist.
- **Every persisted model maps to a goal.** Orphan models (no Gx) are either missing from goals or are dead code.
- **Dominant operations get strict contracts.** D1/D2/D3 boundaries deserve validation, error handling, and idempotency. Non-dominant boundaries can be simpler.
- **Do not add abstraction layers without a boundary test.** If the three abstraction boundary tests do not justify a new layer, the framework's built-in patterns are sufficient.
- **In verify mode, read before judging.** Understand why the code is the way it is before declaring misalignment. Earned complexity is different from accidental complexity.
