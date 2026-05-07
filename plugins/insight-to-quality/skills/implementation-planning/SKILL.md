---
name: implementation-planning
description: >
  Create a minimal project-level implementation plan after discovery.md, system-design.md, and
  system-map.md are stable enough. Turn the upstream system understanding into a practical
  development map for the whole project: define implementation stages, order them, explain their
  dependencies, and state concrete exit criteria so the team knows what must be true before moving
  to the next stage.
  Requires discovery.md, system-design.md, and system-map.md to exist.
---

# Implementation Planning

> **Output contract:** The skill's conversational output must be in Traditional Chinese. Any newly written planning content produced by this skill must also be in Traditional Chinese unless a downstream file explicitly requires another language.

This skill exists to answer one question:

**Given the current upstream documents, what implementation stages does this project need, in what order, and what must be true before the team moves from one stage to the next?**

This skill is not another round of discovery, design, or system mapping.
It starts after those layers are stable enough and turns them into a practical development map for the whole project.

## Purpose

Use this skill to turn:

- `discovery.md`
- `system-design.md`
- `system-map.md`

into a small, execution-facing implementation plan.

This skill is responsible for:

- defining the planning scope
- anchoring the work in the current upstream documents
- deciding whether the project should begin with `enablement-first` or `behavior-first`
- defining the implementation stages
- ordering the stages
- stating the purpose and exit criteria of each stage
- identifying what each stage depends on and what it unlocks
- identifying the current stage or current implementation focus
- recording dependencies, blockers, and next-step focus

This skill is not responsible for:

- rewriting discovery goals or system requirements
- re-arguing major design decisions
- re-cutting the system into new responsibilities or seams
- producing full feature specs
- producing full implementation tickets or low-level coding tasks
- listing every implementation-sized change that may happen inside a stage

## Shared Document

Create or update:

- `docs/implementation-plan.md`

This document is the global development map for the project.

It is not required to enumerate every future feature brief in advance.
Use this plan to manage:

- stage ordering
- stage intent
- exit criteria
- blockers
- current-stage focus

Downstream feature briefs may still be opened during implementation, as long as they stay within the current stage assumptions.

## Required Outputs

Before finishing, you must produce:

- [ ] a clear planning scope
- [ ] a short upstream anchor to discovery / system-design / system-map
- [ ] a delivery strategy: `enablement-first` or `behavior-first`
- [ ] an ordered implementation stage list
- [ ] a stated goal and exit criteria for each stage
- [ ] a defined current stage or current implementation focus
- [ ] concrete exit criteria for the current stage
- [ ] noted dependencies / blockers
- [ ] noted what each stage unlocks
- [ ] a created or updated `docs/implementation-plan.md`

## Definitions

### `enablement`

Use `enablement` when the work mainly creates the foundation that later behavior depends on.

Examples:

- persistence scaffolding
- worker / queue / adapter plumbing
- minimum artifact path foundation
- minimum seam or contract skeleton

### `behavior`

Use `behavior` when the work mainly delivers externally observable system behavior.

Examples:

- a command now works
- a query now returns the expected state
- a user-visible or operator-visible flow step now behaves correctly

## Workflow

### Phase 1: Planning Scope

Define what this implementation plan is covering right now.

At minimum, answer:

- which feature area, flow, or system area this plan covers
- what is explicitly out of scope for this planning pass
- whether this is greenfield startup work or incremental work on an existing codebase

Keep scope bounded. This skill should produce a practical project implementation map, not a vague long-term roadmap.

### Phase 2: Upstream Anchor

Anchor the plan in the current upstream documents.

Capture only the minimum needed:

- relevant discovery goal(s)
- relevant top-level interaction(s)
- relevant baseline flow
- relevant system-design decisions
- relevant responsibility units, core data / key states, and seams from `system-map.md`

Do not restate the upstream documents in full.

### Phase 3: Choose Delivery Strategy

Decide whether the near-term delivery strategy is:

- `enablement-first`
- `behavior-first`

Explain why.

Use `enablement-first` when:

- later behavior work cannot be implemented honestly without first building structural support
- the main risk is missing foundation, not missing user-visible behavior

Use `behavior-first` when:

- the first behavior path is already structurally supportable
- the main risk is proving real external behavior, not creating plumbing

### Phase 4: Define And Order The Implementation Stages

List the implementation stages in order.

For each stage, record:

- `Stage name`
- `Stage type`: `enablement` / `behavior` / `mixed`
- `Stage goal`
- `Why now`
- `Depends on`
- `Unlocks`
- `Exit criteria`

Do not decompose into low-level coding subtasks.
This is a development-order map, not a ticket breakdown.

Stage examples may include:

- runtime / service skeleton
- SDK or core loop skeleton
- persistence foundation
- first real behavior path
- hardening / observability

The stage list should stay close to the service itself.
Do not get lost in company-specific environment detail unless it directly blocks service implementation.

### Phase 5: Define The Current Stage / Current Implementation Focus

Pick the current stage and define the current implementation focus clearly enough that downstream work can focus.

Capture at least:

- `Stage name`
- `Stage type`
- `Current focus statement`
- `Start`
- `End`
- `In scope`
- `Out of scope`
- `Main risk`
- `Touched responsibility units`
- `Touched core data / entities`
- `Touched seams`

The current focus must be:

- small enough to finish honestly
- important enough to unlock real progress
- clear enough that a future feature brief can be short

### Phase 6: Write Exit Criteria

For each stage, and especially the current stage, state what counts as done.

Exit criteria should be:

- concrete
- observable
- as small as possible while still unlocking the next step

Examples:

- a record can now be created and linked correctly
- a worker path can now run a minimal proof
- a seam can now carry the minimum typed handoff
- a command now returns a stable observable result

Do not define exit criteria as vague phrases like:

- "basic support exists"
- "foundation is ready"
- "plumbing is in place"

Say what can now be exercised or observed.

### Phase 7: Note Dependencies, Blockers, And Stage Unlocks

Record:

- what each stage depends on
- what may block the current stage
- which upstream layer to revisit if blocked
- what later stages or feature work become possible once it is done

Use upstream routing explicitly when needed:

- discovery mismatch -> `discovery`
- design choice mismatch -> `system-design`
- unclear data / seam / entity structure -> `system-map`

## Implementation Plan Shape

Write or update `docs/implementation-plan.md` using this minimum shape:

```markdown
# Implementation Plan

## Planning Scope
- Covers: ...
- Out of scope: ...
- Context: greenfield / existing codebase / mixed

## Upstream Anchor
- Discovery goals: ...
- Top-level interactions: ...
- Baseline flow: ...
- Relevant system-design decisions: ...
- Relevant responsibility units: ...
- Relevant core data / key states: ...
- Relevant seams: ...

## Delivery Strategy
- Strategy: enablement-first / behavior-first
- Why: ...

## Implementation Stages
| Stage | Type | Goal | Why now | Depends on | Unlocks | Exit criteria |
|---|---|---|---|---|---|---|
| ... | enablement / behavior / mixed | ... | ... | ... | ... | ... |

## Current Stage
- Stage name: ...
- Stage type: enablement / behavior / mixed
- Current focus statement: ...
- Start: ...
- End: ...
- In scope: ...
- Out of scope: ...
- Main risk: ...
- Touched responsibility units: ...
- Touched core data / entities: ...
- Touched seams: ...

## Current Stage Exit Criteria
- ...
- ...

## Dependencies And Blockers
- Depends on: ...
- Possible blockers: ...
- Route back if blocked: ...

## Stage Unlocks
- ...
- ...
```

## Validation

### Ordering check

Does the stage order actually reflect dependencies and unlocks, rather than personal preference?

### Stage check

Is the current stage and focus small, meaningful, and honest?

If it still sounds like a vague theme, it is not ready.

### Exit-criteria check

Would two different developers agree whether the current stage is done?

If not, the exit criteria are still too vague.

### Upstream traceability check

Can the current stage be traced back to:

- a discovery goal or interaction
- a system-design decision
- a responsibility unit / core data / seam in `system-map.md`

If not, the plan is probably drifting away from the existing structure.

## Guardrails

- Do not rewrite the upstream documents.
- Do not expand this into a full implementation spec.
- Do not decompose into low-level coding tasks.
- Do not use `enablement` as a dumping ground for vague setup work.
- Do not choose stage order without explaining dependencies and unlocks.
- Do not define exit criteria with fuzzy language.
- Do not let environment setup detail overshadow service-level implementation planning.
- If the current stage cannot be anchored upstream, stop and route back instead of guessing.
