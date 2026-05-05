---
name: feature-slice
description: >
  Entry point when the user has a concrete feature idea and needs to turn it into one implementable
  slice before spec work or TDD. Align the feature with discovery.md, system-design.md, and
  SYSTEM_MAP.md, create or update a shared work card, and route to the earliest missing upstream or
  downstream clarification layer.
  Requires discovery.md, system-design.md, and SYSTEM_MAP.md to exist.
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
- which seams it is likely to touch
- which already-named truth, state, or handoff objects it changes
- what the actual first implementable slice should be
- whether the earliest next step is upstream revision, spec clarification, or TDD readiness

## Working Style

- **Slice before spec.** Cut an implementable slice first, then decide which downstream skill should run next.
- **Anchor in the current system.** Align to `discovery.md`, `system-design.md`, and `SYSTEM_MAP.md` before treating the feature as valid.
- **Escalate the earliest missing layer.** If goals, design decisions, ownership, or seams are unstable, go back upstream instead of pushing forward.
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
- [ ] the primary responsibility unit from `SYSTEM_MAP.md`
- [ ] the likely touched seams from `SYSTEM_MAP.md`
- [ ] the touched truth/state/handoff objects already recognized by `SYSTEM_MAP.md`, or an explicit note that `system-map` must be revisited because they are still unnamed
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

Align the slice to `SYSTEM_MAP.md`.

At minimum, answer:

- which primary responsibility unit owns the work
- which seams are likely to be touched
- which state, entity, or workflow truth will change
- whether those truths already have stable map-level names
- whether the current ownership model is clear enough

If `SYSTEM_MAP.md` cannot clearly point to an owner, seam, or named truth object, route back to `system-map`.

### Phase 6: Define The Current Slice

Record the current slice with:

- `Flow id`
- `Flow type`: user / system / operational
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
- grounded in already-recognized map-level truth instead of inventing brand-new structural objects casually

If the start, end, or main risk still cannot be stated clearly, the slice is not stable enough.

### Phase 7: Decide The Next Layer

After anchoring, choose the earliest next missing layer.

#### Route to `spec-clarification` when:

- the slice is aligned upstream
- the relevant truth objects are already named at map level
- but surface, contract, or behavior still contains important unknowns

#### Route to `tdd-ready-check` when:

- the slice is aligned upstream
- surface, contract, and behavior are already clear enough
- it is now meaningful to discuss test risk and strategy

#### Route back upstream when:

- the feature no longer fits `discovery.md`
- the feature no longer fits `system-design.md`
- `SYSTEM_MAP.md` cannot support the owner, seam, or named-truth explanation

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
- Related seams: [...]
- Touched ownership areas: [...]
- Touched named truths / states / handoff objects: [...]
- Missing map-level naming?: no / [explain]

## Feature Slice
- Flow id: [...]
- Flow type: user / system / operational
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
- **No clear owner or seam means stop and go back to `system-map`.**
- **Only route to `spec-clarification` after the slice is anchored upstream.**
- **The goal is not to finish the design here; the goal is to cut the right first slice.**
