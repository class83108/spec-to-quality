---
name: design-review
description: >
  Review a completed feature slice after implementation. Starting from the shared feature work
  card, verify that the implemented code still matches the intended slice, responsibility units,
  seams, test strategy, and manual validation expectations; then decide whether any upstream
  writeback is required before the work is considered done.
---

# Design Review

Read before proceeding:

- `../../references/architect-mindset.md`, focusing on:
  - boundary discipline
  - traceability back to design intent
- `../../references/implementation-mindset.md`, focusing on:
  - refactor as drift removal
  - deciding when a local implementation result requires upstream writeback

This skill exists to answer one question:

**Did this implementation stay aligned with the intended slice and system design, or did it discover changes that must be written back upstream?**

This is not just a style review.

It is the checkpoint between:

- local implementation success
- and system-level confidence that the slice still fits the design

## Working Style

- **Review against the intended slice, not against a vague idea of cleanliness.**
- **Check whether refactor stayed local or escaped upward.**
- **Findings come first.** If there are meaningful risks or regressions, report them before any summary.
- **Use the work card as the main review anchor.**
- **When implementation changed assumptions, push the change to the right upstream layer.**

## Inputs

This skill MUST read:

- `docs/features/<feature-slug>/work-card.md`

And may read when present:

- `docs/features/<feature-slug>/<feature-slug>.feature`
- `docs/features/<feature-slug>/surface.md`
- `docs/features/<feature-slug>/contract.md`
- `docs/features/<feature-slug>/behavior.md`
- `goals.md`
- `design-driver-discovery.md`
- `SYSTEM_MAP.md`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Work card read
- [ ] Findings reported first when risks or misalignments exist
- [ ] Review of slice alignment, seam alignment, and test-strategy alignment completed
- [ ] Manual validation and deferred coverage reviewed when present
- [ ] Upstream writeback decision made (`none`, `work-card only`, `spec`, `system-map`, `design-driver-discovery`, or `goals`)

## Entry Preconditions

This skill expects:

- a feature work card exists
- implementation for the slice has been attempted

If implementation has not started or the slice is not yet in execution, stop and route back to `tdd-workflow`.

## Workflow

### Phase 1: Reconstruct Intended Scope

From the work card, reconstruct:

- supported goal(s)
- relevant design driver(s)
- slice statement
- primary responsibility unit
- related seams
- main risk to protect
- test strategy
- manual validation expectations
- deferred coverage

This is the baseline you review against.

### Phase 2: Slice Alignment Review

Ask:

- Did the implementation stay inside the intended slice?
- Did it quietly absorb adjacent work?
- Did it solve the intended risk, or drift into a different problem?

If the code went beyond the slice, classify:

- **local expansion** — still inside the same responsibility unit and seam assumptions
- **scope creep** — touches work that should have been a separate slice

### Phase 3: Responsibility And Seam Review

Review whether the code still matches the system structure assumed in `SYSTEM_MAP.md`.

Check:

- is the intended primary responsibility unit still the real owner?
- did the code create a new seam or bypass an existing seam?
- did responsibility leak across boundaries?
- did a helper or adapter quietly absorb business ownership it should not own?

If the answer changes the system shape, the result is not just a local refactor. It needs writeback.

### Phase 4: Test Strategy Alignment Review

Review whether the actual test execution matches the chosen strategy in the work card.

Check:

- did the primary protection layer actually protect the main risk?
- were supporting layers used where needed?
- was manual validation done where declared?
- is deferred coverage still acceptable and explicitly tracked?

Typical misalignments:

- unit tests exist, but the real seam risk was never tested
- integration tests exist, but business behavior was never asserted
- Gherkin exists, but the most important observable outcome is missing
- manual validation was declared but skipped silently

### Phase 5: Refactor Safety Review

Inspect the final code for refactor quality, especially the risks we called out during `tdd-workflow`.

#### Repetition and drift

Check for:

- duplicate helper logic
- duplicate constants / enums / fixture knowledge
- duplicated seam or rule semantics in both tests and implementation

#### Responsibility overload

Check for:

- functions mixing rule logic, mapping, and I/O
- helpers that now carry business ownership
- tests with setup weight far larger than the behavior being protected

#### Common smells

Check for:

- long functions
- mixed abstraction levels
- hidden side effects
- fragile call order / temporal coupling
- primitive-heavy seam handling
- over-mocked tests protecting a fake world

### Phase 6: Upstream Writeback Decision

Decide whether the results stay local or need upstream updates.

#### Work-card only

Use when:

- implementation stayed within the slice
- no system structure changed
- only execution notes / manual validation / deferred coverage need sync

#### Spec writeback

Use when:

- surface / contract / behavior text is now stale
- implementation exposed a clarification gap that should be captured durably

#### System-map writeback

Use when:

- responsibility ownership changed
- a seam moved or a new seam emerged
- change navigation assumptions are no longer correct

#### Design-driver writeback

Use when:

- the real pressure turned out different from what was assumed
- an operational or seam risk became design-shaping

#### Goals writeback

Use when:

- the feature changed what the system is now expected to do
- or the implementation revealed the original goal framing was wrong

### Phase 7: Completion Gate

The slice is ready to be treated as complete only when:

- no critical findings remain unresolved
- the implemented code still fits the intended slice
- the test strategy was actually honored or consciously adjusted
- manual validation is either passed or explicitly blocked with reason
- deferred coverage is visible, not forgotten
- upstream writeback has either been applied or explicitly opened as follow-up

## Findings Format

Present findings first, ordered by severity.

Use this format:

| Severity | File:Line | Category | Risk | Suggested Fix |
|---|---|---|---|---|
| P1 | `path/to/file.py:120` | seam ownership | logic bypasses intended boundary and weakens failure isolation | move boundary logic back to the owning unit or update the seam intentionally |

If no findings are discovered, say so explicitly, then list residual risks or coverage gaps.

## Review Summary

After findings, summarize:

- slice alignment: aligned / drifted
- seam alignment: aligned / drifted
- test strategy alignment: aligned / drifted
- manual validation: pass / fail / blocked / none
- deferred coverage: tracked / untracked / none
- upstream writeback: none / work-card / spec / system-map / design-driver-discovery / goals

## Key Rules

- **A green test run is not enough by itself.**
- **If refactor changed the slice assumptions, it is not "just local cleanup".**
- **Manual validation is part of the design contract when declared.**
- **Deferred coverage must stay visible.**
- **When system shape changed, write it back upstream instead of relying on memory.**
