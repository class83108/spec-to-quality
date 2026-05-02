# insight-to-quality — Agent Guide

This plugin is built around one main idea:

**keep discovery, slice clarification, testing strategy, implementation, and review connected, so local coding decisions do not drift away from system intent.**

---

## Main Flow

Use this high-level chain:

1. `goals-discovery`
2. `design-driver-discovery`
3. `system-map`
4. `feature-slice`
5. `spec-clarification`
6. `tdd-ready-check`
7. `gherkin-extraction` or `direct TDD`
8. `tdd-workflow`
9. `design-review`

This is not a rigid waterfall.
Move upward when the problem belongs to a higher layer.

---

## What Each Stage Does

### `goals-discovery`

Clarifies:

- what the system must do
- what it will not do
- what constraints already exist

### `design-driver-discovery`

Clarifies:

- which flows deserve protection
- which pressures should shape the design
- which problems are real design drivers versus symptoms

### `system-map`

Clarifies:

- the main responsibility units
- the important seams
- what to inspect when change happens

### `feature-slice`

Turns a feature idea into:

- one concrete slice of one flow
- mapped to goals, design drivers, responsibility units, and seams

### `spec-clarification`

Fills only the missing slice-level clarity:

- surface
- contract
- behavior

### `tdd-ready-check`

Decides:

- whether the slice is clear enough for implementation
- what risk is being protected
- what test strategy to use

### `gherkin-extraction`

Used only when the slice deserves acceptance-style executable scenarios.

### `tdd-workflow`

Executes:

- Red
- Green
- Refactor

using the already chosen test strategy.

### `design-review`

Checks:

- whether the implementation stayed aligned with the slice
- whether seams or ownership drifted
- whether writeback is needed upstream

---

## Shared Execution Anchor

For feature implementation work, the shared coordination document is:

- `docs/features/<feature-slug>/work-card.md`

This is the active anchor for:

- `feature-slice`
- `spec-clarification`
- `tdd-ready-check`
- `gherkin-extraction`
- `tdd-workflow`
- `design-review`

Do not scatter slice execution state across unrelated notes if the work card can hold it.

---

## Core Rules

- **Do not skip discovery when upstream intent is unclear.**
- **Do not jump into code before the slice is clear.**
- **Do not default every slice to the same test level.**
- **Do not assume Gherkin is always needed.**
- **Do not hide structural problems inside local refactor.**
- **Do not let deferred risk disappear from view.**
- **When assumptions move, write back to the layer where the assumption lives.**

---

## Test Strategy Rules

Choose tests by risk, not by habit.

### Use `unit` when the main risk is:

- local logic
- helper behavior
- deterministic transformation

### Use `integration` when the main risk is:

- seam or handoff correctness
- persistence behavior
- adapter or coordination behavior

### Use `feature` when the main risk is:

- user-visible behavior
- acceptance-worthy business outcome

### Use `manual validation` when the main risk is:

- perceived responsiveness
- UI or message clarity
- media/content quality
- a temporary happy path not yet worth full automation

Mixed strategies are normal.

---

## Red / Green / Refactor Rules

### Red

Start Red at the layer where the real uncertainty lives.

Red does not have to begin with a unit test.

### Green

Implement the smallest change that resolves the targeted uncertainty.

### Refactor

Refactor should remove:

- duplicate knowledge
- overloaded responsibilities
- drift between tests and implementation
- weak naming and poor locality

If refactor reveals a structural problem, escalate upward.

---

## Review Rules

A green test run is not enough.

Review should check:

- slice alignment
- seam alignment
- test strategy alignment
- manual validation completion
- deferred coverage visibility
- upstream writeback needs

Findings come first when real risks remain.

---

## Upward Routing Heuristic

When implementation discovers a problem, route it to the right layer:

- system purpose changed -> `goals-discovery`
- real design pressure changed -> `design-driver-discovery`
- ownership or seam changed -> `system-map`
- slice meaning is unclear -> `feature-slice`
- surface / contract / behavior is unclear -> `spec-clarification`
- test strategy no longer fits -> `tdd-ready-check`

Do not compensate for an upstream problem with clever local code.

---

## Mindset Sources

Two reference documents define the underlying principles:

- `references/architect-mindset.md`
- `references/implementation-mindset.md`

Skills should use them as principle sources, not as workflow scripts.
