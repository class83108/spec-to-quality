---
name: feature-slice
description: >
  Entry point when the user has a concrete feature idea and needs to slice it before spec work or
  TDD. Creates or updates a shared feature work card, identifies the current flow slice, checks
  whether the work belongs to the current goals/design drivers/system map, and routes to the next
  missing clarification layer or to an upstream deep dive when the structure is still unstable.
  Requires goals.md, design-driver-discovery.md, and SYSTEM_MAP.md to exist.
---

# Feature Slice

Read `../../references/architect-mindset.md` before proceeding. Focus especially on:

- slicing work before discussing implementation detail
- keeping the slice traceable to goals, pressures, and seams

This skill is the intake gate between discovery and implementation.

Its job is not to fully specify the feature. Its job is to answer:

- 這次要做哪條 flow 的哪一段？
- 這段 slice 屬於哪個 goal / design driver / responsibility unit？
- 這段 slice 對應全局 interaction surface 的哪個互動？
- 目前缺的是 surface、contract、還是 behavior 清晰度？
- 是否其實還卡在需要 deep dive 的結構問題？
- 是否已經接近可進 TDD？

This skill should create or update a **shared feature work card** that later skills continue editing.

## Working Style

- **Slice first.** Do not start by asking which spec skill to run. First define the feature slice.
- **Use the current system shape.** Map the request onto existing goals, interaction surface, design drivers, responsibility units, and seams before deciding anything else.
- **Do not assume all features need all spec layers.** The next step depends on what is unclear for this slice.
- **Escalate structural uncertainty early.** If ownership, boundary, or shared interaction language is unstable, route upstream instead of forcing the slice forward.
- **Keep the work card live.** This card is the shared coordination document across feature-slice, spec clarification, and TDD readiness.

## Shared Document

Create or update:

- `docs/features/<feature-slug>/work-card.md`

This file is the shared feature-level coordination document used by:

- `feature-slice`
- `spec-clarification`
- `tdd-ready-check`

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] Feature request summarized in plain language
- [ ] Decomposition decision recorded: `single-slice` or `multi-slice` (with planned slice list and `flow-plan.md` if more than two slices)
- [ ] Current flow slice identified (the chosen first slice if multi-slice)
- [ ] Flow id assigned, and sibling slices on the same flow listed (or explicitly noted as none)
- [ ] Mapping to existing goal(s) or explicit note that a new goal may be needed
- [ ] Mapping to the global interaction surface or explicit note that the interaction model may need revisiting
- [ ] Mapping to relevant design driver(s) or explicit note that design-driver-discovery may need revisiting
- [ ] Primary responsibility unit and likely seams identified from SYSTEM_MAP
- [ ] `docs/features/<feature-slug>/work-card.md` created or updated
- [ ] Next missing clarification layer selected (`spec-clarification`, `tdd-ready-check`, or back to discovery/system-map/deep dive)

## Routing Principle

Do not route by habit.

Route by the **earliest missing layer for this feature slice**:

1. goals
2. global interaction surface
3. design drivers
4. system map
5. feature slice clarity
6. spec clarification (`surface` / `contract` / `behavior`)
7. TDD readiness

Stop at the first missing layer.

## Workflow

### Phase 1: Feature Intake

Summarize the feature request in plain language.

Ask:

- What user, system, or operational flow is this trying to change?
- What useful outcome should exist after this feature works?
- What is explicitly not part of this slice?

If the request is still too broad, reduce it before routing.

### Phase 1B: Flow Decomposition Check

Before cutting a single slice, decide whether the request is **one slice or many**. This is the most common place where a "feature" silently turns into 3-4 slices and the work loses cross-slice visibility.

A request is **multi-slice** when:

- it spans multiple distinct start/end pairs in the flow
- each part has independent risk worth protecting separately
- delivering the parts incrementally would still produce useful intermediate states

A request is **single-slice** when:

- one start, one end, one main risk
- splitting would create artificial dependencies without delivering intermediate value

#### If multi-slice

1. Assign a shared `Flow id` (e.g. `flow-checkout`, `flow-transcription-resume`)
2. List the planned slices, each with: slice slug, one-line slice statement, ordering rationale
3. Let the user pick the first slice to work on
4. When more than two slices are planned, write `docs/features/<flow-id>-flow-plan.md` with the slice list, so cross-slice visibility exists before each work card does. Keep it small — slug, statement, status (`planned` / `in-progress` / `done`)
5. Continue Phase 2 with the chosen first slice only — do not pre-build work cards for slices not yet started

#### If single-slice

Continue to Phase 2 directly.

#### Subsequent slices

Each later slice in a multi-slice flow re-enters this skill as a fresh invocation:

- the Phase 2 sibling check links it to the previous slices via shared `Flow id`
- update `flow-plan.md` status when starting and finishing each slice
- if a later slice reveals the flow plan was wrong, edit `flow-plan.md` rather than silently re-cutting

### Phase 2: Flow Slice

Define the slice at a coarse but implementable level.

Capture:

- **Flow id** — a stable identifier shared by every slice of the same flow (e.g. `flow-checkout`, `flow-transcription-resume`)
- **Flow** — user / system / operational
- **Interaction anchor** — which top-level interaction from discovery this slice belongs to
- **Slice statement** — what part of the flow this work covers
- **Start** — what event or state begins the slice
- **End** — what observable result or state ends the slice
- **Main risk** — what could go wrong that this slice should protect against

If the user gives implementation detail, pull back up to the slice statement.

#### Sibling slice check

Before finalizing the slice, scan `docs/features/*/work-card.md` for any work card with the **same `Flow id`**. If sibling slices exist:

- list them in the work card under `Sibling slices`
- check whether the new slice's start/end is consistent with how the flow was already cut (no overlap, no contradiction)
- if the flow shape itself has shifted, route back to `system-map` instead of silently re-cutting the flow

This is the lightweight cross-slice protection. It is not a global registry — it is a grep across existing work cards.

### Phase 3: Upstream Mapping

Check whether this slice fits the current discovery artifacts.

#### Goals check

- Which existing goal(s) does this serve?
- If none fit without stretching meaning, route back to `goals-discovery`

#### Interaction-surface check

- Which top-level interaction does this slice refine?
- If none fit cleanly, or if naming/semantics conflict with the current interaction model, route back to `design-driver-discovery`

#### Design driver check

- Which existing design driver(s) matter here?
- If the pressure profile appears fundamentally new, route back to `design-driver-discovery`

#### System-map check

- Which responsibility unit is primary?
- Which seams are touched?
- If ownership or seam shape is unclear, route to `system-map`

### Phase 4: Clarification Need

Once the slice is anchored, decide what kind of clarity is missing.

#### Route to `spec-clarification` when:

- one or more of surface / contract / behavior is still unclear
- the slice is anchored but tests would still require guessing
- the seam or user-visible outcome needs clarification before Gherkin or TDD

Within `spec-clarification`, the actual missing view may be:

- surface
- contract
- behavior

#### Route back upstream when:

- the slice does not fit current goals
- the slice does not fit the current global interaction surface
- the pressure changes the design drivers
- the system map cannot say who owns the work
- ownership / failure semantics / consistency are still unstable enough to require deep dive

### Phase 5: Work Card Sync

Create or update `docs/features/<feature-slug>/work-card.md` with this structure:

```markdown
# Feature Work Card — [Feature Name]

## Goal Context
- Supports: [goal title(s)]
- Why now: [why this slice matters]
- Out of scope: [explicit exclusions]

## Interaction Surface Context
- Top-level interaction: [interaction from discovery]
- Shared language to preserve: [resource / command / workflow terms]
- Interaction drift risk: [none or what may be inconsistent]

## Design Driver Context
- Relevant drivers: [driver title(s)]
- Pressure to protect: [main pressure or risk]

## Feature Slice
- Flow id: [stable id shared across slices of the same flow]
- Flow type: user / system / operational
- Interaction anchor: [top-level interaction]
- Slice: [what part of the flow is being implemented]
- Sibling slices: [other work cards with the same Flow id, or `none`]
- Start: [entry condition]
- End: [exit condition]
- Primary responsibility unit: [from SYSTEM_MAP]
- Related seams: [seam titles]

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

## Scenarios To Write
<!-- Filled by tdd-ready-check. Required for both gherkin and direct-TDD paths. -->
| Scenario | Risk Protected | Observable Outcome | Primary Layer |
|---|---|---|---|

## Next Step
- Route to: [skill name]
- Why: [earliest missing layer]
```

### Phase 6: Handoff

When routing onward, report:

- feature summary
- slice summary
- goals mapping result
- design driver mapping result
- primary responsibility unit
- touched seams
- next skill and why it is the earliest missing layer

## Key Rules

- **This skill creates the shared feature work card.**
- **Do not jump into spec details before the slice is clear.**
- **A broad feature idea is not ready for TDD.** It must first become a slice.
- **A multi-slice flow is not one big slice.** Decompose first, work one slice at a time, keep cross-slice visibility via `Flow id` and (when planned > 2) a `flow-plan.md`.
- **Do not pre-build work cards for slices not yet started.** Plan visibility lives in `flow-plan.md`; per-slice context lives in each slice's work card when its turn comes.
- **No goal fit means stop and go back to goals.**
- **No design-driver fit means stop and revisit pressure discovery.**
- **No clear owner or seam means stop and revisit system-map.**
- **Do not multiply spec work by reflex.** Route to `spec-clarification`, then clarify only the missing view(s).
