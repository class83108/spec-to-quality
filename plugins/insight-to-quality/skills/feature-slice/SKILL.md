---
name: feature-slice
description: >
  Entry point when the user has a concrete feature idea and needs to turn it into one implementable
  slice before spec work or TDD. Align the feature with discovery.md, system-design.md, and
  system_map.md, create or update a shared work card, and route to the earliest missing upstream or
  downstream clarification layer.
  Requires discovery.md, system-design.md, and system_map.md to exist.
---

# Feature Slice

> **Output contract:** The skill's conversational output must be in Traditional Chinese. Any newly written planning content or work-card content produced by this skill must also be in Traditional Chinese unless a downstream file explicitly requires another language.

Read `../../references/architect-mindset.md` before proceeding. Focus especially on:

- slicing work before implementation detail
- aligning change with existing responsibilities and seams
- routing uncertainty to the right upstream layer

This skill is the intake gate between discovery / system-design / system-map and implementation.

Its job is not to fully specify a feature.
Its job is to turn a feature idea into answers for:

- which interaction or flow in the existing system this feature belongs to
- which baseline system shape it depends on
- which prior system-design decisions constrain it
- which primary responsibility unit it lands in
- which core data or key states it changes or depends on
- which already-named entities or records it relies on
- where it enters the known entity interaction flow
- which seams it is likely to touch
- which already-named data, state, entity, or handoff objects it changes
- what the actual first implementable slice should be
- whether the earliest next step is upstream revision, spec clarification, or TDD readiness

## Working Style

- **Slice before spec.** Cut an implementable slice first, then decide which downstream skill should run next.
- **Anchor in the current system.** Align to `discovery.md`, `system-design.md`, and `system_map.md` before treating the feature as valid.
- **Escalate the earliest missing layer.** If goals, design decisions, core data / state responsibility, interactions, or seams are unstable, go back upstream instead of pushing forward.
- **Keep the work card live.** The work card is the shared coordination document for downstream skills.

## Shared Document

Create or update:

- `docs/features/<feature-slug>/work-card.md`

This file is shared by:

- `feature-slice`
- `spec-clarification`
- `tdd-ready-check`
- `gherkin-extraction`
- `tdd-workflow`
- `design-review`

## Required Outputs

Before finishing, you must produce:

- [ ] a plain-language summary of the feature request
- [ ] a recorded decomposition decision: `single-slice` or `multi-slice`
- [ ] the current first slice
- [ ] mapping to the relevant discovery context:
  goals, top-level interaction, baseline flow
- [ ] mapping to relevant prior system-design decisions, or an explicit note that `system-design` may need to be revisited
- [ ] the primary responsibility unit from `system_map.md`
- [ ] the related core data / key states from `system_map.md`
- [ ] the relevant already-named entities / records from `system_map.md`
- [ ] the relevant entity interaction path from `system_map.md`, or an explicit note that the interaction flow is still unclear
- [ ] the likely touched seams from `system_map.md`
- [ ] the touched data / state / handoff objects already recognized by `system_map.md`, or an explicit note that `system-map` must be revisited because they are still unnamed
- [ ] a created or updated `docs/features/<feature-slug>/work-card.md`
- [ ] the selected next step:
  `spec-clarification`, `tdd-ready-check`, or route back to `discovery` / `system-design` / `system-map`

## Routing Principle

Route by the **earliest missing layer** for this feature:

1. discovery fit
2. system-design fit
3. system-map fit
4. slice clarity
5. spec clarification
6. TDD readiness

Stop at the first missing layer.

## Workflow

### Phase 1: Feature Intake

First restate the feature request in plain language.

Answer:

- which user, system, or operational outcome this feature is trying to change
- what observable result in the world would be different after it is done
- what is explicitly out of scope for this work

If the request is too large, too vague, or really an entire roadmap, narrow it before moving on.

### Phase 2: Slice Decomposition

Decide whether the feature is:

- `single-slice`
- `multi-slice`

Heuristics:

- if it has one clear start/end, one primary risk, and no more natural delivery cut, it is usually `single-slice`
- if it spans multiple flow segments or contains independently deliverable risks, it is usually `multi-slice`

Also decide the slice type:

- `behavior slice`
- `enablement slice`

Use `enablement slice` when the first honest step is foundational work that must exist before the user-visible or workflow-visible behavior can be implemented safely, for example:

- baseline project wiring
- storage / queue / worker / job plumbing
- artifact path or serving foundation
- migration / persistence scaffolding
- adapter or contract skeleton needed before real flow behavior can be exercised

Do not use `enablement slice` as a dumping ground for vague setup work. It still needs a clear start, end, and risk.

If it is `multi-slice`:

1. assign a `Flow id`
2. list the planned slices
3. choose the first slice to work on
4. if there are more than 2 planned slices, create:
   `docs/features/<flow-id>-flow-plan.md`

This skill only handles the current first slice. Do not create work cards for slices that have not started yet.

### Phase 3: Anchor The Slice In Discovery

Align the slice to `discovery.md`.

At minimum, answer:

- which goal this slice supports
- which top-level interaction it belongs to
- which baseline flow it sits on
- whether forcing it into the current discovery framing would distort the system meaning

If it does not fit, route back to `discovery`. Do not invent a new system shape here.

### Phase 4: Anchor The Slice In System Design

Align the slice to `system-design.md`.

At minimum, answer:

- which prior system-design decisions affect this feature
- whether the feature depends on a known trade-off
- whether it overturns any current choice
- whether it reveals a new deep-dive topic

If the feature's core pressure no longer fits the current `system-design.md`, route back to `system-design`.

### Phase 5: Anchor The Slice In System Map

Align the slice to `system_map.md`.

At minimum, answer:

- which primary responsibility unit owns the work
- which core data or key states this slice changes, reads, or depends on
- which already-named entities or records this slice depends on
- where this slice enters the known entity interaction flow
- which seams are likely to be touched
- which data, state, entity, or workflow record will change
- whether those items already have stable map-level names
- whether the current main-management model is clear enough

If `system_map.md` cannot clearly point to a main-managing unit, a relevant data/state area, a named entity/record, or a seam, route back to `system-map`.

### Phase 6: Define The Current Slice

Record the current slice with:

- `Flow id`
- `Flow type`: user / system / operational
- `Slice type`: behavior / enablement
- `Interaction anchor`
- `Slice statement`
- `Start`
- `End`
- `Main risk`
- `Primary responsibility unit`
- `Related seams`

The goal here is:

- coarse but implementable
- not yet a full spec
- grounded in already-recognized map-level data, entities, interactions, and seams instead of inventing brand-new structural objects casually

For `enablement slice`, make the end condition concrete in system terms, for example:

- a required dependency can now be instantiated
- a persistence boundary exists and can be exercised
- a worker / adapter path can run a minimal end-to-end proof
- downstream behavior slices can now attach to a stable seam

If the start, end, or main risk still cannot be stated clearly, the slice is not stable enough.

### Phase 7: Decide The Next Layer

After anchoring, choose the earliest next missing layer.

#### Route to `spec-clarification` when:

- the slice is aligned upstream
- the relevant data / entities / records are already named at map level
- but surface, contract, or behavior still contains important unknowns

#### Route to `tdd-ready-check` when:

- the slice is aligned upstream
- surface, contract, and behavior are already clear enough
- it is now meaningful to discuss test risk and strategy

#### Route back upstream when:

- the feature no longer fits `discovery.md`
- the feature no longer fits `system-design.md`
- `system_map.md` cannot support the responsibility, core data/state, interaction, seam, or named-object explanation

### Phase 8: Work Card Sync

Create or update `docs/features/<feature-slug>/work-card.md` with:

```markdown
# Feature Work Card — [Feature Name]

## Discovery Context
- Supports goals: [...]
- Top-level interaction: [...]
- Baseline flow: [...]
- Out of scope: [...]

## System Design Context
- Relevant prior decisions: [...]
- Relevant trade-offs / risks: [...]
- Upstream drift risk: none / [explain]

## System Map Context
- Primary responsibility unit: [...]
- Related core data / key states: [...]
- Related entities / records: [...]
- Relevant entity interaction path: [...]
- Related seams: [...]
- Touched main-management areas: [...]
- Touched named data / states / entities / handoff objects: [...]
- Missing map-level naming?: no / [explain]

## Feature Slice
- Flow id: [...]
- Flow type: user / system / operational
- Slice type: behavior / enablement
- Slice statement: [...]
- Sibling slices: [...]
- Start: [...]
- End: [...]
- Main risk: [...]

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
- Test strategy: ...
- Ready for TDD: yes / no
- Blockers: ...

## Next Step
- Route to: [...]
- Why: [...]
```

### Phase 9: Handoff

When handing off to a downstream skill, report at least:

- feature summary
- slice summary
- discovery alignment result
- system-design alignment result
- primary responsibility unit
- touched seams
- next skill and why it is the earliest missing layer

## Key Rules

- **This skill creates the shared feature work card.**
- **Do not jump into detailed spec content before the slice is stable.**
- **A feature idea is not automatically one slice.**
- **No discovery fit means stop and go back to `discovery`.**
- **No system-design fit means stop and go back to `system-design`.**
- **No clear main-managing unit or seam means stop and go back to `system-map`.**
- **No clear core data / state area, entity interaction path, or seam means stop and go back to `system-map`.**
- **Only route to `spec-clarification` after the slice is anchored upstream.**
- **The goal is not to finish the design here; the goal is to cut the right first slice.**
