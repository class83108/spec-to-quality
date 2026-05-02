---
name: tdd-workflow
description: >
  Run the implementation workflow once a feature slice is ready. Starting from the shared feature
  work card and, when present, the slice's `.feature` file, execute Red -> Green -> Refactor using
  the chosen test strategy rather than a one-size-fits-all test level.
  Requires a feature work card and prior readiness from `tdd-ready-check`.
---

# TDD Workflow

Read before proceeding:

- `../../references/implementation-mindset.md`, focusing on:
  - choosing Red at the layer where the real uncertainty lives
  - keeping Green minimal
  - using Refactor to remove drift and escalate structural problems upward

This skill exists to answer one question:

**How should we execute Red -> Green -> Refactor for this slice, given the test strategy already chosen?**

It does not decide whether the slice is ready. That belongs to `tdd-ready-check`.

It does not decide whether Gherkin is needed. That belongs to `gherkin-extraction`.

## Working Style

- **Honor the chosen slice.** Do not silently expand scope during implementation.
- **Honor the chosen test strategy.** Use the primary protection layer and supporting layers from the work card; do not default everything to one style.
- **Red must prove uncertainty.** If the test passes immediately, either the test is wrong or the slice was already covered.
- **Refactor without redesigning discovery.** If implementation exposes a discovery problem, route back to the right layer instead of compensating in code.
- **Keep work card and code in sync.** The work card is not just planning; it is the active execution anchor.

## Inputs

This skill MUST read:

- `docs/features/<feature-slug>/work-card.md`

It may also read when present:

- `docs/features/<feature-slug>/<feature-slug>.feature`
- `docs/features/<feature-slug>/surface.md`
- `docs/features/<feature-slug>/contract.md`
- `docs/features/<feature-slug>/behavior.md`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Work card read
- [ ] Execution path identified from `Test strategy`
- [ ] Red phase completed with a failing test or scenario that proves the targeted uncertainty
- [ ] Green phase completed with minimal implementation
- [ ] Refactor phase completed with tests still green
- [ ] Work card updated with execution notes, unresolved gaps, and any upstream issues discovered

## Entry Preconditions

This skill expects:

- a feature work card exists
- the work card says `Ready for TDD: yes`

If `Ready for TDD: no`, stop and route back to `tdd-ready-check`.

If the work card says the next step is `gherkin-extraction`, stop unless a `.feature` file now exists or the user explicitly wants direct TDD instead.

## Workflow

### Phase 0: Read Execution Context

From the work card, extract:

- slice statement
- main risk to protect
- primary responsibility unit
- related seams
- test strategy:
  - primary protection
  - supporting layers
  - manual validation
  - deferred coverage

Also detect whether a `.feature` file exists for the slice.

### Phase 1: Choose The Red Entry Point

The Red phase should start at the primary protection layer named in the work card.

Red does **not** always mean "start with a unit test".

Choose the first failing test at the layer where the main uncertainty actually lives:

- `unit` when the risk is local logic, helper behavior, or deterministic transformation
- `integration` when the risk is seam correctness, handoff semantics, persistence, or coordination
- `feature` when the risk is acceptance-level, user-visible, or architecture-significant behavior

#### If primary protection is `unit`

Start with:

- local rule tests
- helper tests
- deterministic transformation tests

Use this path when the main uncertainty is local logic.

Red usually looks like:

- a small failing test around a rule or transformation
- a helper or local function returning the wrong value
- a pure in-process state decision not yet encoded correctly

#### If primary protection is `integration`

Start with:

- seam / handoff tests
- repository / persistence tests
- adapter or coordination tests

Use this path when the main uncertainty is cross-boundary correctness.

Red usually looks like:

- a handoff passes the wrong meaning across a seam
- persistence / retrieval does not preserve the expected state
- an adapter or coordinator connects components incorrectly
- each part may work alone, but the collaboration fails

#### If primary protection is `feature`

Start with:

- `.feature` scenarios when they exist
- otherwise stop and route back to `gherkin-extraction`

Use this path when the main uncertainty is acceptance-level or user-visible behavior.

Red usually looks like:

- a `.feature` scenario failing because the end-to-end behavior is absent or wrong
- the system reaches the wrong visible state
- a business rule is violated in a way the user or external actor can observe

#### If manual validation is the main unresolved risk

This is not a pure TDD-style Red, but it still matters.

Use this path when the slice depends on:

- perceived responsiveness
- UI/message clarity
- media/content quality
- other human-judged outcomes

In this case:

- still write the strongest useful automated Red you can
- explicitly record the remaining human-judged risk in the work card
- do not pretend full automation already covers the slice

### Phase 2: Red

Write or extend the minimal test(s) that express the targeted uncertainty.

Rules:

- the test must fail for the right reason
- the failure must connect to the main risk being protected
- do not add speculative test coverage outside the slice

If the test passes immediately:

- check whether the behavior is already covered
- check whether the assertion is too weak
- if needed, stop and correct the test instead of continuing

Red-phase guardrails:

- do not start at a lower layer if the real uncertainty is at a seam or acceptance level
- do not start at a higher layer if the uncertainty is purely local and deterministic
- do not add large setup just to preserve the habit of unit-first TDD
- do not write a Red test that cannot explain which risk it is protecting

### Phase 3: Green

Implement the smallest code change that makes the Red test pass.

Rules:

- stay inside the slice boundary
- do not add future-facing abstractions unless forced by the slice
- do not absorb unrelated cleanup into Green

Use supporting layers only as needed:

- add supporting unit coverage if the primary layer is integration
- add supporting integration coverage if the primary layer is feature and a seam needs confidence

### Phase 4: Manual Validation Checkpoint

If the work card lists any manual validation, execute it after Green.

Examples:

- run a happy path through the UI or CLI
- inspect a user-visible error message
- validate media or content output quality

Record:

- what was checked
- result: pass / fail / blocked
- any surprise that suggests upstream clarification is still wrong

If manual validation reveals a spec or slice problem, route back upstream before continuing.

### Phase 5: Refactor

Refactor only after:

- the primary protection layer is green
- supporting automated layers are green
- required manual validation has passed or is explicitly blocked

Refactor goals:

- reduce duplication
- improve naming
- align code with the responsibility unit and seams already defined
- remove repeated local test/setup noise
- consolidate duplicated knowledge into the right owner

Do not turn Refactor into architecture rework.

#### Refactor Checks

During Refactor, actively check for:

##### Repetition and drift

- existing helper / utility already exists, but equivalent logic was redefined
- existing constant / enum / config value exists, but was copied or hardcoded again
- tests and implementation each define the same knowledge separately
- fixtures, payload shapes, or mapping rules are duplicated in multiple places

##### Responsibility overload

- one function mixes validation, mapping, state change, and side effects
- one function both decides business rules and performs I/O
- one helper quietly carries business-rule meaning or seam semantics
- one test requires a large unrelated setup just to exercise a small behavior

##### Common code smells

- long functions that need mental segmentation to read
- mixed abstraction levels in the same block
- excessive parameter passing where a higher-level concept is missing
- hidden side effects inside seemingly simple helpers
- temporal coupling where call order is fragile but implicit
- primitive-heavy seam handling where meaning is too easy to lose

##### Common test smells

- over-mocked tests that protect a fake world instead of the real seam
- brittle setup that suggests the slice or ownership is wrong
- assertions too weak to protect the stated risk
- scenario sprawl with many low-value cases

#### Refactor Escalation Rule

If refactor reveals a **local** issue, fix it here.

If refactor reveals that:

- the responsibility unit is wrong
- the seam is wrong
- the slice boundary is wrong
- the clarification is wrong
- the chosen test strategy no longer matches the real risk

then stop treating it as a local refactor and route back upstream.

### Phase 6: Upstream Escalation Rules

If implementation reveals a problem, route it to the right layer:

- goal mismatch -> `goals-discovery`
- new design pressure -> `design-driver-discovery`
- unclear ownership or broken seam -> `system-map`
- missing surface / contract / behavior clarity -> `spec-clarification`
- test strategy no longer makes sense -> `tdd-ready-check`
- missing acceptance scenarios -> `gherkin-extraction`

Do not compensate for an upstream problem by burying it in code.

### Phase 7: Sync The Work Card

Update the work card with:

- execution status
- what automated layers were actually used
- manual validation result
- deferred coverage still outstanding
- blockers or upstream follow-up needed

Recommended addition:

```markdown
## Execution Notes
- Red started at: ...
- Supporting layers used: ...
- Manual validation: ...
- Deferred coverage still open: ...
- Upstream follow-up: ...
```

## Key Rules

- **TDD starts from the chosen risk, not from habit.**
- **A helper slice may go straight to unit TDD.**
- **A seam slice may start at integration.**
- **A user-visible slice should not skip Gherkin if Gherkin was explicitly chosen as the next step.**
- **Manual validation is part of the strategy when declared; it is not optional decoration.**
- **When implementation discovers the slice was wrong, go back up the chain instead of forcing the code through.**
