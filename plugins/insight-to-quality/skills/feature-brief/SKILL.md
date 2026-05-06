---
name: feature-brief
description: >
  Create a minimal execution-facing brief for one concrete feature or slice after
  implementation-planning has identified the current stage or current focus. Anchor the work in
  discovery.md, system-design.md, and system-map.md, define the slice briefly, record what must be
  protected, and leave the work ready for TDD-oriented execution.
  Requires discovery.md, system-design.md, system-map.md, and docs/implementation-plan.md to
  exist.
---

# Feature Brief

> **Output contract:** The skill's conversational output must be in Traditional Chinese. Any newly written brief content produced by this skill must also be in Traditional Chinese unless a downstream file explicitly requires another language.

This skill exists to answer one question:

**For the current implementation focus, what exactly are we building now, what matters most to protect, and what is the minimum shared brief the team needs before starting TDD-oriented work?**

This skill is not another round of implementation planning.
It takes the current project-level plan and turns one concrete unit of work into a short execution brief.

## Purpose

Use this skill to turn:

- `docs/implementation-plan.md`
- `discovery.md`
- `system-design.md`
- `system-map.md`

into a short, execution-facing brief for one feature or one implementation slice.

This skill is responsible for:

- restating the current work in plain language
- anchoring it in the upstream documents
- defining the slice boundary
- identifying the main risk to protect
- recording the minimum known / unknown execution notes
- recording the validation intent in a compact form

This skill is not responsible for:

- creating a new project-level implementation order
- rewriting upstream documents
- creating a full acceptance-spec document by default
- expanding into a long clarification dossier

## Shared Document

Create or update:

- `docs/features/<feature-slug>/brief.md`

This document is the short execution brief for the current work item.

## Required Outputs

Before finishing, you must produce:

- [ ] a plain-language feature summary
- [ ] upstream anchors to discovery / system-design / system-map / implementation-plan
- [ ] a clear slice definition
- [ ] the main risk to protect
- [ ] a short known / unknown execution note section
- [ ] a validation intent section
- [ ] a created or updated `docs/features/<feature-slug>/brief.md`

## Workflow

### Phase 1: Read The Current Focus

Read the current implementation focus from `docs/implementation-plan.md`.

Extract at minimum:

- current stage
- current focus statement
- touched responsibility units
- touched core data / entities
- touched seams
- current stage exit criteria

If the implementation plan does not yet make the current focus clear enough, route back to `implementation-planning`.

### Phase 2: Anchor Upstream

Anchor the work to:

- relevant discovery goal(s) and interaction(s)
- relevant system-design decisions or trade-offs
- relevant responsibility units, core data / key states, entities, interactions, and seams from `system-map.md`

Keep this short.
The brief is not another map or another design document.

### Phase 3: Define The Slice

Write the current work item in a compact execution form.

Capture at least:

- `Feature summary`
- `Slice type`: `enablement` / `behavior`
- `Start`
- `End`
- `In scope`
- `Out of scope`
- `Main risk`

This should be shorter and more concrete than the project-level implementation plan.

### Phase 4: Record Execution Notes

Record only what the current work still needs for honest execution.

Use three short buckets:

- `Known`
- `Unknown`
- `Needs to be pinned down now`

If too many unknowns remain, route back upstream instead of pretending the brief is ready.

### Phase 5: Record Validation Intent

Capture the minimum shared testing intent for this work.

At minimum include:

- `Main risk to protect`
- `Scenario bullets`
- `Suggested primary layer`: `unit` / `integration` / `feature` / `manual`

Use short scenario bullets by default.
Prefer a lightweight `Given / When / Then` style inside each bullet when that makes the behavior clearer.
Do not turn this into a full Gherkin document.

## Feature Brief Shape

Write or update `docs/features/<feature-slug>/brief.md` using this minimum shape:

```markdown
# Feature Brief — [Feature Name]

## Summary
- Feature summary: ...
- Slice type: enablement / behavior

## Upstream Anchor
- Discovery goals / interactions: ...
- Relevant system-design decisions: ...
- Relevant responsibility units: ...
- Relevant core data / entities: ...
- Relevant entity interaction path: ...
- Relevant seams: ...
- Implementation-plan stage: ...

## Slice Definition
- Start: ...
- End: ...
- In scope: ...
- Out of scope: ...
- Main risk: ...

## Execution Notes
- Known: ...
- Unknown: ...
- Needs to be pinned down now: ...

## Validation Intent
- Main risk to protect: ...
- Scenario bullets:
  - Given ...
    When ...
    Then ...
  - Given ...
    When ...
    Then ...
- Suggested primary layer: unit / integration / feature / manual
```

## Validation

### Brief check

Can a developer read this brief and know what to build now without reopening all upstream documents?

### Scope check

Is the slice boundary narrow enough to execute honestly?

### Risk check

Is the main risk concrete enough that later tests can target it directly?

### Escalation check

If important unknowns remain, is it clear whether they belong in:

- `implementation-planning`
- `system-design`
- `system-map`

If not, the brief is still too vague.

## Guardrails

- Do not rewrite the implementation plan.
- Do not turn this into a full spec packet.
- Do not restate all upstream reasoning in long form.
- Do not create a separate Gherkin file by default.
- Do not keep unknowns hidden; record them briefly or route back upstream.
