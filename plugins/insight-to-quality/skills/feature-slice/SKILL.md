---
name: feature-slice
description: >
  Entry point when the user has a concrete feature idea and needs to slice it before spec work or
  TDD. Creates or updates a shared feature work card, identifies the current flow slice, checks
  whether the work belongs to the current goals/design drivers/system map, and routes to the next
  missing clarification layer.
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
- 目前缺的是 surface、contract、還是 behavior 清晰度？
- 是否已經接近可進 TDD？

This skill should create or update a **shared feature work card** that later skills continue editing.

## Working Style

- **Slice first.** Do not start by asking which spec skill to run. First define the feature slice.
- **Use the current system shape.** Map the request onto existing goals, design drivers, responsibility units, and seams before deciding anything else.
- **Do not assume all features need all spec layers.** The next step depends on what is unclear for this slice.
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
- [ ] Current flow slice identified
- [ ] Mapping to existing goal(s) or explicit note that a new goal may be needed
- [ ] Mapping to relevant design driver(s) or explicit note that design-driver-discovery may need revisiting
- [ ] Primary responsibility unit and likely seams identified from SYSTEM_MAP
- [ ] `docs/features/<feature-slug>/work-card.md` created or updated
- [ ] Next missing clarification layer selected (`spec-clarification`, `tdd-ready-check`, or back to discovery/system-map)

## Routing Principle

Do not route by habit.

Route by the **earliest missing layer for this feature slice**:

1. goals
2. design drivers
3. system map
4. feature slice clarity
5. spec clarification (`surface` / `contract` / `behavior`)
6. TDD readiness

Stop at the first missing layer.

## Workflow

### Phase 1: Feature Intake

Summarize the feature request in plain language.

Ask:

- What user, system, or operational flow is this trying to change?
- What useful outcome should exist after this feature works?
- What is explicitly not part of this slice?

If the request is still too broad, reduce it before routing.

### Phase 2: Flow Slice

Define the slice at a coarse but implementable level.

Capture:

- **Flow** — user / system / operational
- **Slice statement** — what part of the flow this work covers
- **Start** — what event or state begins the slice
- **End** — what observable result or state ends the slice
- **Main risk** — what could go wrong that this slice should protect against

If the user gives implementation detail, pull back up to the slice statement.

### Phase 3: Upstream Mapping

Check whether this slice fits the current discovery artifacts.

#### Goals check

- Which existing goal(s) does this serve?
- If none fit without stretching meaning, route back to `goals-discovery`

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
- the pressure changes the design drivers
- the system map cannot say who owns the work

### Phase 5: Work Card Sync

Create or update `docs/features/<feature-slug>/work-card.md` with this structure:

```markdown
# Feature Work Card — [Feature Name]

## Goal Context
- Supports: [goal title(s)]
- Why now: [why this slice matters]
- Out of scope: [explicit exclusions]

## Design Driver Context
- Relevant drivers: [driver title(s)]
- Pressure to protect: [main pressure or risk]

## Feature Slice
- Flow type: user / system / operational
- Slice: [what part of the flow is being implemented]
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
- **No goal fit means stop and go back to goals.**
- **No design-driver fit means stop and revisit pressure discovery.**
- **No clear owner or seam means stop and revisit system-map.**
- **Do not multiply spec work by reflex.** Route to `spec-clarification`, then clarify only the missing view(s).
