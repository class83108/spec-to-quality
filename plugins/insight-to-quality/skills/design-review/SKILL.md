---
name: design-review
description: >
  Review completed implementation work against the current feature brief, implementation plan, and
  upstream documents. Verify that the code still matches the intended brief, responsibility units,
  seams, test strategy, and stage intent; then decide whether any upstream writeback is required
  before the work is considered done.
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

**Did this implementation stay aligned with the current brief and project plan, or did it discover changes that must be written back upstream?**

This is not just a style review.

It is the checkpoint between:

- local implementation success
- and project-level confidence that the work still fits the intended stage, brief, and structure

## Working Style

- **Review against the intended brief, not against a vague idea of cleanliness.**
- **Check whether refactor stayed local or escaped upward.**
- **Findings come first.** If there are meaningful risks or regressions, report them before any summary.
- **Use the feature brief as the main local review anchor.**
- **Use the implementation plan as the stage-level anchor.**
- **When implementation changed assumptions, push the change to the right upstream layer.**

## Inputs

This skill MUST read:

- `docs/implementation-plan.md`
- `docs/features/<feature-slug>/brief.md`

And may read when present:

- `discovery.md`
- `system-design.md`
- `system-map.md`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Implementation plan read
- [ ] Feature brief read
- [ ] Findings reported first when risks or misalignments exist
- [ ] Review of brief alignment, seam alignment, and test-strategy alignment completed
- [ ] Manual validation and unresolved gaps reviewed when present
- [ ] Upstream writeback decision made (`none`, `brief`, `implementation-planning`, `system-map`, `system-design`, or `discovery`)

## Entry Preconditions

This skill expects:

- a feature brief exists
- implementation for the current work has been attempted

If implementation has not started or the current work is not yet in execution, stop and route back to `tdd-workflow`.

## Workflow

### Phase 1: Reconstruct Intended Scope

From the implementation plan and feature brief, reconstruct:

- relevant discovery goal(s)
- relevant prior system-design decision(s)
- current stage
- current focus statement
- feature summary
- main risk to protect
- relevant responsibility units
- relevant seams
- suggested primary test layer
- manual support expectations
- known unresolved notes

This is the baseline you review against.

### Phase 2: Brief Alignment Review

Ask:

- Did the implementation stay inside the intended brief?
- Did it quietly absorb adjacent work?
- Did it solve the intended risk, or drift into a different problem?

If the code went beyond the brief, classify:

- **local expansion** — still inside the same responsibility units and seam assumptions
- **scope creep** — touches work that should have been a separate brief or a later stage

### Phase 3: Responsibility And Seam Review

Review whether the code still matches the system structure assumed in `system-map.md`.

Check:

- are the intended responsibility units still the real ones carrying the work?
- did the code create a new seam or bypass an existing seam?
- did responsibility leak across boundaries?
- did a helper or adapter quietly absorb business ownership it should not own?
- was the minimum data shape or seam contract violated in a meaningful way?

If the answer changes the system shape, the result is not just a local refactor. It needs writeback.

### Phase 4: Test Strategy Alignment Review

Review whether the actual test execution matches the brief's validation intent.

Check:

- did the actual primary layer protect the main risk?
- were the scenario bullets meaningfully covered?
- was manual support done where implied?
- are unresolved gaps still acceptable and explicitly tracked?

Typical misalignments:

- unit tests exist, but the real seam risk was never tested
- integration tests exist, but the user-visible or operator-visible outcome was never asserted
- scenario bullets existed, but the most important observable outcome was never covered
- manual support was implied but skipped silently

### Phase 5: Refactor Safety Review

Inspect the final code for refactor quality, especially the risks called out during `tdd-workflow`.

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

#### Brief writeback

Use when:

- the implementation stayed within the same stage
- no system structure changed
- but the local brief, scenario bullets, or execution notes are now stale

#### Implementation-planning writeback

Use when:

- the current stage exit criteria changed
- the stage order changed
- a new blocker changes the implementation sequence
- the work should really have been a different stage or a prior enablement stage

#### System-map writeback

Use when:

- responsibility ownership changed
- a seam moved or a new seam emerged
- change navigation assumptions are no longer correct
- the minimum data shape or seam contract is now wrong

#### System-design writeback

Use when:

- the real pressure turned out different from what was assumed
- an operational or seam risk became design-shaping

#### Discovery writeback

Use when:

- the work changed what the system is now expected to do
- or the implementation revealed the original goal framing was wrong

### Phase 7: Completion Gate

The work is ready to be treated as complete only when:

- no critical findings remain unresolved
- the implemented code still fits the intended brief
- the current stage assumptions still hold or have been written back
- the test strategy was actually honored or consciously adjusted
- manual support is either passed or explicitly blocked with reason
- unresolved gaps are visible, not forgotten
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

- brief alignment: aligned / drifted
- seam alignment: aligned / drifted
- test strategy alignment: aligned / drifted
- manual support: pass / fail / blocked / none
- unresolved gaps: tracked / untracked / none
- upstream writeback: none / brief / implementation-planning / system-map / system-design / discovery

## Key Rules

- **A green test run is not enough by itself.**
- **If refactor changed the brief or stage assumptions, it is not "just local cleanup".**
- **Manual support is part of the execution contract when implied.**
- **Unresolved gaps must stay visible.**
- **When system shape or stage sequence changed, write it back upstream instead of relying on memory.**
