---
name: tdd-ready-check
description: >
  Decide whether a feature slice is clear enough to move into `gherkin-extraction` or TDD. Starting
  from the shared feature work card, verify that the slice, responsibility, seams, clarification
  status, and test intent are sufficiently clear, then record readiness and the recommended next
  execution path.
  Requires goals.md, design-driver-discovery.md, SYSTEM_MAP.md, and a feature work card.
---

# TDD Ready Check

Read before proceeding:

- `../../references/implementation-mindset.md`, focusing on:
  - testing the main risk
  - choosing the lowest honest test layer
- `../../references/architect-mindset.md`, focusing on traceability

This skill exists to answer one question:

**Can this slice safely move into `gherkin-extraction` or TDD now?**

It does not generate the full implementation plan. It validates whether enough clarity already exists.

## Working Style

- **Do not confuse momentum with readiness.** Wanting to start coding is not evidence that the slice is ready.
- **Check the slice, not the whole project.** TDD readiness is determined one feature slice at a time.
- **Use the shared work card as the main readiness record.**
- **Push work upward when clarity is missing.** If the slice is unclear, route back to `feature-slice`, `spec-clarification`, `system-map`, or earlier discovery instead of guessing.
- **Test level must fit the work.** Not every task needs Gherkin or feature tests. Helper-level work may be ready for direct unit TDD.

## Shared Document

This skill MUST read and update:

- `docs/features/<feature-slug>/work-card.md`

This file is shared with:

- `feature-slice`
- `spec-clarification`
- `tdd-ready-check`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Work card read and updated
- [ ] Readiness assessed across goal context, slice clarity, responsibility/seam context, clarification status, and test intent
- [ ] Test strategy confirmed (primary protection layer, supporting layers, manual validation, and deferred coverage)
- [ ] Recommended next step recorded (`gherkin-extraction`, `direct TDD`, or `return for clarification`)
- [ ] Explicit blockers recorded if the slice is not ready

## Entry Preconditions

This skill expects:

- `goals.md`
- `design-driver-discovery.md`
- `SYSTEM_MAP.md`
- `docs/features/<feature-slug>/work-card.md`

If the work card does not exist, stop and route back to `feature-slice`.

## Workflow

### Phase 1: Read Current Slice State

From the work card, extract:

- supported goal(s)
- relevant design driver(s)
- slice statement
- primary responsibility unit
- related seams
- current clarification status
- current blockers

If any of these are missing in a way that prevents reasoning, mark the slice as not ready and route back appropriately.

### Phase 2: Readiness Dimensions

Check the following five dimensions.

#### 1. Goal Context

Ask:

- Is it clear which goal this slice supports?
- Is the slice's success condition meaningful relative to that goal?
- Is out-of-scope work explicit enough to prevent drift?

If not, route back to `feature-slice` or `goals-discovery`.

#### 2. Slice Clarity

Ask:

- Is the slice narrow enough to implement and test?
- Are the start and end points clear?
- Is the main risk of this slice known?

If not, route back to `feature-slice`.

#### 3. Responsibility And Seam Context

Ask:

- Is the main responsibility unit known?
- Are the touched seams known?
- Is it clear whether this work stays inside one box or crosses a handoff?

If not, route back to `system-map` or `spec-clarification`.

#### 4. Clarification Status

Ask:

- Is the missing view still surface, contract, or behavior?
- Are the necessary unknowns already clarified enough?
- Would writing tests now force the team to guess on visible outcome, seam semantics, or business rule?

If guessing would still be required, route back to `spec-clarification`.

#### 5. Test Intent

Ask:

- What risk are the tests trying to protect?
- Is this primarily a helper-level concern, a seam/handoff concern, or a slice-level behavior concern?
- Is the chosen test level appropriate?

If the team cannot state what the tests are protecting, the slice is not ready.

### Phase 3: Choose Test Strategy

Determine the candidate execution path and the testing mix that protects the slice.

Test strategy should be chosen by the **main risk being protected**, not by defaulting to the highest test level.

Record:

- **Primary protection layer** — the layer carrying the main automated risk protection
- **Supporting automated layers** — any lower or adjacent layers that also matter
- **Manual validation** — any human-run happy path or subjective check still required
- **Deferred coverage** — important coverage intentionally postponed for now

#### Direct unit TDD

Use when the work is small and local, such as:

- pure helper function
- deterministic transformation
- parser / formatter / mapper without design-significant seam ownership

Conditions:

- input/output meaning is clear
- no unresolved business rule ambiguity
- no unresolved seam ambiguity

Main risk profile:

- local logic error
- deterministic transformation error
- helper-level behavior drift

#### Integration-focused TDD

Use when the work crosses a seam or exercises handoff behavior:

- boundary adapter
- state handoff
- internal contract reliance
- persistence or workflow coordination at slice scale

Conditions:

- seam semantics are clear enough
- responsibility ownership is known

Main risk profile:

- handoff mismatch
- persistence / coordination mismatch
- "each part works alone but the connection is wrong"

#### Feature / acceptance-driven TDD

Use when the slice represents user-visible or architecture-significant behavior:

- main user flow slice
- observable failure handling
- state transition with business meaning
- high-risk rule or design-driver-sensitive behavior

Conditions:

- surface / contract / behavior clarity is sufficient
- the acceptance-worthy outcome is clear enough to extract into Gherkin

Main risk profile:

- user-visible flow failure
- business behavior correctness failure
- architecture-significant outcome failure

#### Manual validation

Use when the slice still needs human judgment for part of its validation, for example:

- perceived latency or responsiveness
- UI clarity or error-message usefulness
- media / content quality
- temporary happy-path confirmation when full end-to-end automation is too costly

Main risk profile:

- subjective or experiential quality
- currently non-automated but still important validation

Manual validation can coexist with automated layers. It is not a failure state by itself.

### Phase 3A: Quick Selection Table

Use this table when choosing the primary protection layer:

| Main Risk | Primary Protection Layer |
|---|---|
| Small logic or helper error | `unit` |
| Seam / handoff / coordination error | `integration` |
| User-visible behavior or rule outcome error | `feature` |
| Subjective or experiential quality | `manual validation` |

If the slice has more than one meaningful risk, use a mixed strategy:

- `unit` for local rule protection
- `integration` for seam protection
- `feature` for acceptance-worthy flow protection
- `manual validation` for the parts automation does not yet cover well

### Phase 4: Gherkin Or Not

Decide whether Gherkin should exist before implementation.

Use Gherkin when:

- the slice protects user-visible behavior
- the slice protects a high-risk business rule
- the slice crosses an important seam and the observable outcome matters
- the slice is important enough to deserve acceptance-level regression protection

Do not require Gherkin when:

- the work is helper-level and local
- the work is a pure implementation refactor inside a stable behavior boundary
- lower-level tests are enough to protect the risk

### Phase 5: Sync The Work Card

Update the `TDD Readiness` section with:

- `Main risk to protect`
- `Test strategy`
- `Ready for TDD`
- `Blockers`

Also update `Next Step` with one of:

- `gherkin-extraction`
- `direct TDD`
- `return to feature-slice`
- `return to spec-clarification`
- `return to system-map`
- `return to discovery`

## Work Card Expectations

The work card should contain at least:

```markdown
## TDD Readiness
- Main risk to protect: ...
- Test strategy:
  - Primary protection: ...
  - Supporting layers: ...
  - Manual validation: ...
  - Deferred coverage: ...
- Ready for TDD: yes / no
- Blockers: ...

## Next Step
- Route to: ...
- Why: ...
```

## Key Rules

- **A slice is not ready if tests would still require guessing.**
- **Not every slice needs Gherkin.**
- **Small code units can go straight to unit TDD if their meaning is already clear.**
- **Business-rule helpers still need behavior clarity.**
- **Boundary adapters still need contract clarity.**
- **When readiness fails, push the problem back to the right layer instead of writing speculative tests.**
