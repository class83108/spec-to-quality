---
name: spec-clarification
description: >
  Clarify the minimum specification needed for a feature slice before Gherkin or TDD. Starting
  from a shared feature work card, inspect which clarification views are missing — surface,
  contract, and/or behavior — and update the same work card plus any needed detailed spec files.
  Escalate to a targeted deep dive when structural uncertainty remains too high for honest test
  design.
  Requires goals.md, design-driver-discovery.md, SYSTEM_MAP.md, and a feature work card created by
  feature-slice.
---

# Spec Clarification

Read before proceeding:

- `../../references/architect-mindset.md`, focusing on:
  - keeping clarification at the right abstraction level
  - keeping slice detail traceable to ownership and seams
- `../../references/implementation-mindset.md`, focusing on:
  - clarifying work before testing it
  - treating the shared work card as the active execution anchor

This skill exists to answer one question:

**What is still unclear about this feature slice before we can safely write acceptance specs or start TDD?**

It does not assume every slice needs the same kind of clarification.

It inspects three views:

- **Surface** — external interaction and visible outcomes
- **Contract** — responsibility handoffs and seam semantics
- **Behavior** — business rules, state transitions, and correctness expectations

The skill may update one, two, or all three views depending on what the slice actually needs.

## Working Style

- **Clarify only what is missing.** Do not fill all three views by default.
- **Stay slice-scoped.** This skill works on one feature slice at a time, not the whole system.
- **Use the shared work card as the main coordination document.** Detailed spec files are optional outputs when the clarification needs durable detail.
- **Do not confuse exploration with formalization.** First make the slice understandable; only then write durable spec text where it helps.
- **Keep the abstraction level above code, but below discovery.** This is pre-TDD clarification, not project-wide architecture work.
- **Escalate when guessing begins.** If clarification cannot proceed without inventing ownership, consistency, or failure semantics, mark a deep dive instead of faking certainty.

## Shared Document

This skill MUST read and update:

- `docs/features/<feature-slug>/work-card.md`

This file is shared with:

- `feature-slice`
- `spec-clarification`
- `tdd-ready-check`

Detailed spec files may also be created or updated under:

- `docs/features/<feature-slug>/surface.md`
- `docs/features/<feature-slug>/contract.md`
- `docs/features/<feature-slug>/behavior.md`

Only create the detailed file(s) that are actually needed.

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Feature work card read and updated
- [ ] Clarification status refreshed for Surface / Contract / Behavior
- [ ] Missing questions resolved or explicitly left as blockers
- [ ] Earliest useful detailed spec file(s) created or updated when needed
- [ ] Recommendation recorded for next step: more clarification, targeted deep dive, `gherkin-extraction`, or `tdd-ready-check`

## Entry Preconditions

This skill expects:

- `goals.md`
- `design-driver-discovery.md`
- `SYSTEM_MAP.md`
- `docs/features/<feature-slug>/work-card.md`

If the work card does not exist, stop and route back to `feature-slice`.

## Workflow

### Phase 1: Read The Slice

Read the feature work card and extract:

- supported goal(s)
- top-level interaction and shared language
- relevant design driver(s)
- flow type and slice statement
- primary responsibility unit
- related seams
- current blockers

If the slice itself is still vague, stop and route back to `feature-slice`.

### Phase 2: Clarification Triage

Decide which view(s) are actually missing enough clarity to block progress.

#### Surface is missing when:

- the trigger from an external actor is unclear
- visible success / failure is unclear
- user-visible timing or feedback is unclear
- external entry/exit shape is unclear at a task level

#### Contract is missing when:

- a seam handoff is unclear
- responsibility ownership is clear but the handoff semantics are not
- payload / state handoff meaning is unstable
- downstream expectations depend on a stronger seam definition

#### Behavior is missing when:

- business rules are unclear
- state transition correctness is unclear
- rejection / exception behavior is unclear
- the slice needs decision logic or invariants pinned down

Do not over-clarify. Mark only the views that are truly blocking.

#### Deep dive is needed when:

- a boundary exists on paper but ownership is still unstable
- core state / entity lifecycle is unclear
- failure, retry, compensation, or consistency semantics would still be guesswork
- multiple slices are likely to drift into conflicting API/model language
- the missing clarity is no longer local to one view and points back to discovery or map structure

### Phase 3: Clarify The Missing View(s)

#### Surface clarification

Capture only what the slice needs:

- who or what triggers the interaction
- what visible success looks like
- what visible failure looks like
- what waiting / retry / confirmation behavior matters

#### Contract clarification

Capture only what the slice needs:

- which seam is involved
- what kind of thing is handed off
- what the receiving side may rely on
- what the main invalid or ambiguous cases are

#### Behavior clarification

Capture only what the slice needs:

- what rule must hold
- what transition is allowed or forbidden
- what the main edge or rejection cases are
- what observable correctness means for this slice

### Phase 4: Sync The Work Card

Update the `Clarification Status` section of the work card.

Each of Surface / Contract / Behavior must include:

- `Known`
- `Unknown`
- `Needed`

Also update:

- `Main risk to protect`
- `Test strategy`
- `Ready for TDD`
- `Blockers`
- `Deep dive needed`
- `Deep dive focus`

### Phase 5: Create Durable Spec Files Only If Needed

Detailed spec files are useful when:

- the clarification is non-trivial
- the seam or rule is likely to be revisited
- Gherkin or TDD would otherwise repeat reasoning
- multiple people or future agents will need the same clarification

If the clarification is simple and already captured in the work card, do not force a separate file.

When creating a detailed file:

- `surface.md` should focus on external interaction and visible outcomes
- `contract.md` should focus on seam handoff meaning, not raw schema dumps
- `behavior.md` should focus on rules, transitions, and business assertions

### Phase 6: Recommend The Next Step

Choose one:

- `stay in spec-clarification` — more than one blocking ambiguity still unresolved
- `targeted deep dive` — the blocking ambiguity is structural, not just slice-local
- `gherkin-extraction` — acceptance-worthy behavior is now clear
- `tdd-ready-check` — clarification is sufficient, validate readiness formally

## Work Card Expectations

The shared work card should already contain this structure:

```markdown
# Feature Work Card — [Feature Name]

## Goal Context
- Supports: ...
- Why now: ...
- Out of scope: ...

## Interaction Surface Context
- Top-level interaction: ...
- Shared language to preserve: ...
- Interaction drift risk: ...

## Design Driver Context
- Relevant drivers: ...
- Pressure to protect: ...

## Feature Slice
- Flow id: ...
- Flow type: ...
- Interaction anchor: ...
- Slice: ...
- Sibling slices: ...
- Start: ...
- End: ...
- Primary responsibility unit: ...
- Related seams: ...

## Clarification Status

### Surface
- Known: ...
- Unknown: ...
- Needed: yes / no

### Contract
- Known: ...
- Unknown: ...
- Needed: yes / no

### Behavior
- Known: ...
- Unknown: ...
- Needed: yes / no

## TDD Readiness
- Main risk to protect: ...
- Test strategy: [primary layer + supporting layers + manual/deferred coverage]
- Ready for TDD: yes / no
- Blockers: ...
- Deep dive needed: yes / no
- Deep dive focus: [ownership / failure semantics / consistency / shared language / none]

## Next Step
- Route to: ...
- Why: ...
```

## Key Rules

- **This skill is view-based, not document-count-based.**
- **Do not create `surface.md`, `contract.md`, and `behavior.md` by reflex.**
- **If the slice can move forward with just a work-card update, do that.**
- **If the slice still has upstream ambiguity, go back to `feature-slice`, `design-driver-discovery`, or `system-map` instead of guessing.**
- **If the ambiguity is structural rather than local, call it a deep dive and say exactly what needs to be resolved.**
- **The goal is readiness for Gherkin or TDD, not documentation volume.**
