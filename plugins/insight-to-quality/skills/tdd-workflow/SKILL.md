---
name: tdd-workflow
description: >
  Execute Red -> Green -> Refactor for the current feature brief. Start from
  docs/features/<feature-slug>/brief.md, perform a lightweight readiness check, then run the TDD
  path using the most honest test layer for the current risk.
  Requires docs/features/<feature-slug>/brief.md to exist.
---

# TDD Workflow

Read before proceeding:

- `../../references/implementation-mindset.md`, focusing on:
  - choosing Red at the layer where the real uncertainty lives
  - keeping Green minimal
  - using Refactor to remove drift and escalate structural problems upward

This skill exists to answer one question:

**Given the current feature brief, how should we execute Red -> Green -> Refactor now?**

This skill includes a lightweight readiness gate.
It does not assume that every slice needs a separate readiness document first.

## Working Style

- **Honor the brief.** Do not silently expand scope during implementation.
- **Start from the main risk.** Test strategy follows the risk, not habit.
- **Readiness is a gate, not a ceremony.** If the brief still forces guessing, stop early and route back upstream.
- **Red must prove uncertainty.** If the test passes immediately, either the test is wrong or the risk is already covered.
- **Refactor without hiding upstream problems.** If implementation exposes a planning, design, or structure problem, route back to the right layer instead of compensating in code.

## Inputs

This skill MUST read:

- `docs/features/<feature-slug>/brief.md`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Feature brief read
- [ ] Readiness checked for the current work
- [ ] Red phase completed with a failing test or scenario that proves the targeted uncertainty
- [ ] Green phase completed with minimal implementation
- [ ] Refactor phase completed with tests still green
- [ ] Feature brief updated with execution notes, unresolved gaps, and any upstream issues discovered

## Entry Preconditions

This skill expects:

- a feature brief exists

If the brief does not exist, stop and route back to `feature-brief`.

## Workflow

### Phase 0: Read The Brief

From the feature brief, extract:

- feature summary
- slice type
- main risk
- relevant responsibility units
- relevant core data / entities
- relevant seams
- validation intent:
  - main risk to protect
  - scenario bullets
  - suggested primary layer
- known / unknown execution notes

### Phase 1: Lightweight Readiness Gate

Before writing tests, check:

- Is the current work item narrow enough to execute?
- Is the main risk concrete enough to protect?
- Is the validation intent clear enough to choose a test layer?
- Do the remaining unknowns still force guessing on surface, contract, or behavior?

If the answer is no, stop and route back:

- missing project sequencing / stage clarity -> `implementation-planning`
- missing slice definition or execution note -> `feature-brief`
- missing design rationale -> `system-design`
- missing data / entity / seam clarity -> `system-map`

Do not continue into TDD while still guessing.

### Phase 2: Choose The Red Entry Point

The Red phase should start at the most honest layer for the current risk.

Choose from:

- `unit`
- `integration`
- `feature`
- `manual support`

#### Use `unit` when:

- the risk is local logic
- the behavior is deterministic
- the seam is not the main uncertainty

#### Use `integration` when:

- the risk is seam correctness
- the risk is persistence or coordination
- each part may work alone but the handoff might still be wrong

#### Use `feature` when:

- the risk is user-visible or acceptance-level behavior
- the scenario bullets are the right primary protection layer

#### Use `manual support` when:

- part of the remaining important risk depends on human judgment

Manual support does not replace automated Red.
It only records what still needs human validation.

### Phase 3: Red

Write or extend the minimal test(s) or scenario(s) that express the targeted uncertainty.

Rules:

- the failure must connect directly to the main risk being protected
- the test must fail for the right reason
- do not add speculative coverage outside the current work

If the test passes immediately:

- check whether the behavior is already covered
- check whether the assertion is too weak
- correct the test before continuing

### Phase 4: Green

Implement the smallest code change that makes the Red test pass.

Rules:

- stay inside the current brief boundary
- do not add future-facing abstractions unless forced by the current work
- do not absorb unrelated cleanup into Green

### Phase 5: Manual Validation Checkpoint

If the brief implies any manual support work, execute it after Green.

Examples:

- run a happy path through the CLI or UI
- inspect a user-visible message
- check content or media quality

Record:

- what was checked
- result: pass / fail / blocked
- any surprise that suggests the brief or upstream layers are wrong

### Phase 6: Refactor

Refactor only after:

- the primary automated protection is green
- any supporting automated checks are green
- required manual support has passed or is explicitly blocked

Refactor goals:

- reduce duplication
- improve naming
- align code with the relevant responsibility units and seams
- remove repeated local setup noise

Do not turn Refactor into architecture redesign.

If refactor reveals that:

- the brief boundary is wrong
- the data / seam structure is wrong
- the chosen test layer no longer matches the real risk

then stop treating it as local cleanup and route back upstream.

### Phase 7: Upstream Escalation Rules

If implementation reveals a problem, route it to the right layer:

- project stage or sequencing problem -> `implementation-planning`
- brief boundary or execution-note problem -> `feature-brief`
- new design decision or trade-off pressure -> `system-design`
- unclear data / entity / seam structure -> `system-map`

Do not compensate for an upstream problem by burying it in code.

### Phase 8: Sync The Brief

Update the feature brief with:

- execution status
- actual primary test layer used
- manual support result
- unresolved gaps
- upstream follow-up needed

Recommended addition:

```markdown
## Execution Notes
- Red started at: ...
- Supporting layers used: ...
- Manual support: ...
- Remaining gaps: ...
- Upstream follow-up: ...
```

## Key Rules

- **TDD starts from the chosen risk, not from habit.**
- **Readiness is checked here before coding starts.**
- **A helper slice may go straight to unit TDD.**
- **A seam slice may start at integration.**
- **A user-visible slice may still start at the `feature` layer when the scenario bullets are strong enough.**
- **When implementation discovers the brief was wrong, go back up the chain instead of forcing the code through.**
