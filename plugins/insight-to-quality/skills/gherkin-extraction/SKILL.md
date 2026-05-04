---
name: gherkin-extraction
description: >
  Extract acceptance-worthy behavior from a feature slice into a `.feature` file. Starting from the
  shared feature work card and any supporting clarification docs, decide whether Gherkin is
  warranted, identify the scenarios worth protecting, map them to the intended test strategy, and
  write the feature file.
  Requires a feature work card and sufficient readiness from tdd-ready-check.
---

# Gherkin Extraction

Read before proceeding:

- `../../references/implementation-mindset.md`, focusing on:
  - deciding whether acceptance-level executable scenarios are actually warranted
  - extracting only the scenarios that protect meaningful observable risk

This skill exists to answer one question:

**Which parts of this slice deserve acceptance-style executable scenarios, and how should they be written?**

Gherkin is not the default output for every slice.

Use it when:

- the slice protects user-visible behavior
- the slice protects a high-risk business rule
- the slice crosses an important seam and the observable outcome matters
- the slice deserves acceptance-level regression protection

Do not force it when:

- the work is only helper-level
- lower-level tests are enough
- there is no acceptance-worthy observable outcome

## Working Style

- **Extract, do not invent.** Gherkin should come from an already-clarified slice, not from guesswork.
- **Use acceptance value as the filter.** Not every detail belongs in a `.feature` file.
- **Respect the chosen test strategy.** Gherkin may describe the acceptance intent even when some lower layers carry most automated protection.
- **Prefer a small, sharp feature file.** Do not turn Gherkin into a full specification dump.

## Inputs

This skill MUST read:

- `docs/features/<feature-slug>/work-card.md`

And may read when present:

- `docs/features/<feature-slug>/surface.md`
- `docs/features/<feature-slug>/contract.md`
- `docs/features/<feature-slug>/behavior.md`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Work card read
- [ ] Determination made: Gherkin needed or not needed
- [ ] If needed, acceptance-worthy scenarios identified
- [ ] Scenario-to-risk mapping made explicit
- [ ] `.feature` file written under `docs/features/<feature-slug>/` or project test path
- [ ] Work card updated with Gherkin status and target file path

## Entry Preconditions

This skill expects:

- a feature work card exists
- `tdd-ready-check` has already marked the slice ready for either `gherkin extraction` or `direct TDD`

If the work card says the slice should go to direct TDD, stop unless the user explicitly asks to create Gherkin anyway.

## Workflow

### Phase 1: Read Acceptance Context

From the work card, extract:

- slice statement
- main risk to protect
- test strategy
- relevant goal(s)
- relevant prior system-design decision(s)
- current clarification status

If the work card still says `Ready for TDD: no`, stop and route back to `tdd-ready-check` or `spec-clarification`.

### Phase 2: Decide Whether Gherkin Is Warranted

Ask:

1. Is there a user-visible or externally observable outcome?
2. Is there a business rule important enough to protect at acceptance level?
3. Is there a cross-boundary result whose correctness matters beyond a local function?
4. Would the team benefit from a durable executable statement of expected behavior?

If the answer is mostly no, record:

- `Gherkin: not needed`
- reason
- recommend direct TDD

### Phase 3: Extract Scenarios

If Gherkin is warranted, identify only the highest-value scenarios.

Use these categories as prompts:

- happy path
- rule violation / rejection
- important state transition
- important seam outcome
- important recovery / retry-visible outcome

Do not try to cover every tiny edge in Gherkin. Lower-level tests may still carry some of that load.

For each planned scenario, capture:

1. **Why this scenario exists**
2. **What risk it protects**
3. **What observable outcome proves success or failure**
4. **Which test strategy layer owns it primarily**

### Phase 4: Scenario Mapping

Start from the `Scenarios To Write` table already populated by `tdd-ready-check`. Do not re-derive scenarios from scratch — filter and refine that table for acceptance-level value:

| Scenario | Risk Protected | Observable Outcome | Primary Layer |
|---|---|---|---|
| [scenario name] | [risk] | [what user/system can observe] | feature / integration / manual-anchor |

Rules:

- Drop rows whose primary layer is `unit` only — those belong in unit tests, not Gherkin
- Keep rows whose observable outcome is acceptance-worthy (user-visible, cross-seam, or rule-significant)
- Every kept scenario must protect a real risk
- Every kept scenario must have an observable outcome
- Avoid scenarios that only restate internal implementation
- If you find new acceptance-worthy scenarios while filtering, add them back to the work card's `Scenarios To Write` table so the source of truth stays consistent

### Phase 5: Write Gherkin

Write one `.feature` file for the slice when warranted.

Suggested default path:

- `docs/features/<feature-slug>/<feature-slug>.feature`

If the repo already has a canonical test feature path, use that instead and record it in the work card.

Rules:

- Gherkin keywords stay in English
- Step text follows project language policy
- Keep the file small and scenario-focused
- Tag scenarios by intended primary layer when useful
- Prefer meaningful scenario names over generic wording

### Phase 6: Sync The Work Card

Update the work card with:

- `Gherkin needed: yes / no`
- `Gherkin file: [path]` when created
- `Scenarios extracted: [count or names]`
- `Why Gherkin: [brief reason]`

## Recommended Work Card Additions

The work card should include a section like:

```markdown
## Gherkin Extraction
- Gherkin needed: yes / no
- Why: ...
- Gherkin file: ...
- Scenarios extracted: ...
```

## Key Rules

- **Gherkin is for acceptance-worthy behavior, not all behavior.**
- **If there is no observable acceptance outcome, direct TDD may be better.**
- **A small, high-signal `.feature` file is better than exhaustive scenario sprawl.**
- **Do not use Gherkin to compensate for unclear slice or unclear behavior.**
