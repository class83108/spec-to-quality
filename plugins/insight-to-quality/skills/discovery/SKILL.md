---
name: discovery
description: >
  Guide early discovery — first clarify the problem framing and the system
  requirements that make later design meaningful, then define goals, non-goals, NFRs, constraints,
  and open questions. Also produce a top-level API / interaction sketch and a baseline flow /
  high-level design sketch so later design review starts from a visible, discussable initial system
  shape instead of abstract wish-list goals.
  Do NOT use for: early system design reasoning (use system-design), system mapping (use
  system-map), or implementation planning (use spec-backlog execution).
---

# Discovery

You are guiding the user through **discovery** that culminates in a grounded `discovery.md`.

Do not jump straight to goals. First make the system legible enough that the goals mean something.

For this skill, discovery happens in this order:

1. **Problem framing** — why the system exists, for whom, and what success means
2. **System requirements** — what first-version behavior the system must actually support
3. **Goals synthesis** — goals, non-goals, NFRs, constraints, and open questions

If the first two are weak, `discovery.md` will be either empty, misleading, or secretly dependent on unstated implementation assumptions.

Read `references/design-decision-mindset.md` before proceeding. Focus especially on:

- keeping abstraction levels stable
- grounding structure in real system shape
- asking questions that expose tradeoffs instead of collecting wish lists

## Purpose

Use this skill to make the first-version system shape visible enough that a grounded `discovery.md` can be written honestly.

This skill is responsible for:

- clarifying the problem being solved
- surfacing the first-version system requirements
- sketching the system's top-level interactions or API shape
- converging that sketch into a stable top-level command/query surface for downstream discussion
- sketching the baseline flow or high-level system behavior shape
- translating that understanding into a stable `discovery.md`

This skill is not responsible for:

- making the key system-design trade-off decisions
- explaining why one major design option should be chosen over another
- deciding responsibility boundaries or seams
- deciding which entities or records deserve separate names
- cutting implementation modules or component boundaries
- writing detailed specs or implementation plans

## Outputs

Before this skill is complete, you must produce all of the following:

- A concise **problem framing summary**:
  system purpose, primary actors, current workaround, success shape, obvious unknowns
- A reviewed **system requirements summary**:
  first-version inputs/outputs, core interaction or processing model, required decisions/control points, state that must be preserved, required recovery/change behavior
- A **Top-Level API / Interaction Sketch**:
  the major resources, commands, queries, or processing entry points the first version is likely to expose, plus which interactions are synchronous vs accepted-then-complete-later where that distinction matters
- A **Top-Level Command / Query Surface Summary**:
  a short normalized list of the stable command and query names the later skills should reuse, including which commands create or advance workflow truth and which queries are read-only projections
- A **Baseline Flow / High-Level Design Sketch**:
  the current best-effort view of how the main interactions connect and what the main processing path looks like, without turning into design rationale, responsibility slicing, or implementation structure
- `discovery.md` with these sections:
  Functional Goals, Non-Goals, Non-Functional Requirements, Constraints, Open Questions
- User confirmation that the resulting `discovery.md` is acceptable

Every functional goal must have:

- a short human-readable title
- a `STABLE` or `EVOLVING` tag
- a strong verb (`SHALL` or `MUST`)

Every non-goal must have:

- a short title
- a "why" explanation that removes at least one design option

Every NFR must have:

- a metric
- a target
- a measurement method

If a required section is empty, write:

- `(none identified — [reason])`

## Workflow

### Phase 1: Establish The Baseline Problem

Before talking about system behavior, make sure the team is aligned on the problem the first version is trying to solve.

This phase should establish:

- what job the system is supposed to get done
- who initiates or depends on that job
- what the current workaround or baseline process is
- what visible outcome makes version 1 worth building
- what kind of failure would make the first version unacceptable
- what important assumptions are still unverified

This is not a generic interview phase. Its purpose is to create enough shared context that the next phases can sketch a believable system shape.

Good prompts:

- "What job is this system supposed to get done?"
- "Who needs that job done, and how do they handle it today?"
- "What does a successful first-version outcome look like in the real world?"
- "If version 1 works poorly, what failure hurts first?"
- "What assumptions are we making about usage, scale, or workflow that might still be wrong?"

Before moving on, summarize:

- system purpose
- primary actors
- current workaround or baseline process
- first-version value
- major failure concerns
- obvious constraints
- obvious unknowns

### Phase 2: System Requirements

Do not write goals until the first-version system shape is visible.

System requirements here are not code-level implementation choices. They describe what the first version must actually support.

Cover these five areas:

1. **Inputs and outputs**
   What goes into the system, and what must come out?
2. **Core interaction or processing model**
   What are the main interactions, operations, state changes, or processing stages?
3. **Required decisions or control points**
   Where must a person or the system approve, validate, choose, dispatch, reconcile, or otherwise make a meaningful decision?
4. **State and persistence requirements**
   What data, state, or artifacts must be preserved for the system to still make sense?
5. **Recovery or change requirements**
   What retry, update, correction, rerun, undo, resume, or change behavior must version 1 support?

Examples of valid system requirements:

- "The system must support create -> review -> approve -> publish."
- "The system must support transcription -> translation -> TTS -> final render with human review before TTS."
- "The system must allow QR codes to be created, redirected, updated, soft-deleted, and expired with distinct error semantics."

Examples of things that are not system requirements by themselves:

- "Use Celery"
- "Use Whisper"
- "Use PostgreSQL"

Those are constraints or later design choices unless already fixed.

Good prompts:

- "What goes in, and what comes out?"
- "What are the core interactions or major processing steps?"
- "If this is workflow-shaped, what are the major steps? If not, what are the key actions and state changes?"
- "Where do important decisions or approvals happen?"
- "What must be preserved so the system remains understandable and operable?"
- "If something is wrong or changes later, what kind of recovery or correction must be possible?"

Before moving on, a new developer should be able to answer:

- What goes in?
- What comes out?
- How does the first version broadly behave?
- Where do important decisions happen?
- What state must be preserved?
- What recovery or change behavior matters?

#### Phase 2B: Top-Level API / Interaction Sketch

Once the system requirements are clear enough, sketch the first-version interaction surface.

This is not a detailed API spec. It is a shape sketch that makes the system easier to reason about.

Capture, where relevant:

- the main resources or workflow entry points
- the main commands
- the main queries or read models
- which interactions are synchronous
- which interactions are accepted-then-complete-later

Examples:

- For a CRUD-like service: create / update / delete / fetch / search operations
- For a workflow system: start run / review / approve / resume / rerun / inspect status
- For a processing pipeline: submission entry points, result retrieval, correction points, progress queries

Do not over-specify:

- field-level schemas
- full endpoint contracts
- exact event payloads
- internal service boundaries

The goal is that the next skill can ask "why this interaction shape?" instead of inventing the shape from scratch.

Before leaving this phase, normalize the interaction sketch into a reusable command/query surface:

- give each major command a stable, concise name
- give each major query or read model a stable, concise name
- note which commands create, advance, approve, rerun, or otherwise change workflow truth
- note which interactions are strictly read-only

Do not turn this into:

- endpoint design
- payload schema design
- event contract design

The goal is to prevent later skills from re-describing the same interaction shape with drifting vocabulary.
Later skills should reuse these stable names instead of replacing them with UI wording, temporary aliases, or implementation-specific labels.

#### Phase 2C: Baseline Flow / High-Level System Behavior Sketch

Once the interaction sketch is visible, produce a lightweight baseline behavior view.

This is not the final architecture and not a detailed component map. It is the current best-effort answer to:

- how the main flow probably runs end-to-end
- how the major interactions connect
- what major stages or external capabilities seem likely to be involved at a high level

Examples:

- For a QR system:
  create flow, redirect flow, image retrieval flow, storage lookup path
- For a media pipeline:
  ingestion, preprocessing, transformation, review points, output generation
- For an internal workflow service:
  request intake, validation, queueing, worker processing, review, completion

Good output shapes:

- a numbered flow
- a short bullet sequence
- a minimal block diagram in prose

Do not over-specify:

- exact component boundaries
- detailed data schemas
- internal class/module structure
- final infrastructure topology
- responsibility ownership
- why one design option is better than another

The goal is to create a baseline system shape that the next skill can challenge, justify, and revise.

### Phase 3: Goals Synthesis

Only now derive the goals.

Every goal should be traceable to either:

- the problem framing
- the system requirements

If a candidate goal cannot be traced back to one of those, it is probably too abstract or accidental.

Produce:

- **Functional Goals**:
  what the system must do or preserve
- **Non-Goals**:
  what the system will explicitly not do
- **Non-Functional Requirements**:
  quantified quality targets
- **Constraints**:
  already-decided, non-negotiable facts
- **Open Questions**:
  uncertainties that may still change the system shape or goals

Functional goal heuristics:

- Strong goals describe meaningful outcomes, protections, coordination responsibilities, or system guarantees.
- Weak goals are usually tool choices, UI preferences, vague aspirations, or low-level CRUD with no business meaning.

When drafting each functional goal:

1. State it with `SHALL` or `MUST`
2. Give it a short title
3. Tag it `STABLE` or `EVOLVING`
4. Check whether it still matters if implementation details change

Useful prompts:

- "What must always be true after this system finishes its work?"
- "Which system requirement must remain true even if the tools change?"
- "If version 1 only got three things right, which three matter most?"
- "What must the system prevent, not just enable?"
- "What will users ask for that you deliberately want to say no to?"

For NFRs, never accept words like "fast", "reliable", or "cheap" without:

- a number
- a measurement method

For constraints, distinguish carefully between:

- already decided and non-negotiable
- likely but not fixed
- merely suggested implementation ideas

For open questions, do not treat an empty section as success. It usually means hidden assumptions are being skipped.

## Validation

Before finalizing `discovery.md`, run these checks:

### Grounding check

Do we have both:

- a clear problem framing summary
- a clear system requirements summary
- a visible top-level API / interaction sketch
- a visible baseline flow / high-level design sketch

If not, stop and complete them first.

### Abstraction-level check

Are the goals actually goals, rather than:

- implementation choices
- design questions
- detailed specs

If not, rewrite them.

### Conflict check

Do any goals contradict:

- other goals
- non-goals
- constraints

If yes, resolve the contradiction before finalizing.

### Measurability check

Are all NFRs measurable?

If not, they are not ready.

### Newcomer check

After reading the problem framing and system requirements, could a new developer explain:

- what the first version is supposed to do
- what shape the system broadly has
- what the main interaction surface probably looks like
- what the current baseline flow or high-level design roughly is
- what is in scope and out of scope

If not, the discovery is still too abstract.

### Goal quality check

For each functional goal, explicitly test:

- **Replacement test**:
  if implementation details change, does the goal still hold?
- **Two-design test**:
  can you imagine a small number of different architectures that would still satisfy it?
- **Six-month test**:
  is this still likely to be true in six months, or does it belong in a lower-level spec?

Show these results to the user. Do not do them silently.

## Guardrails

- Do not write goals before the first-version system shape is visible.
- Do not mistake system requirements for tool or vendor choices.
- Do not let discovery.md carry all discovery work by itself.
- Do not skip the interaction sketch when the first version clearly exposes commands, queries, resources, or workflow entry points.
- Do not stop at isolated interactions if the first version already implies a baseline end-to-end flow.
- Do not restate the project title as if it were a goal.
- Do not move design-driver analysis into this skill.
- Do not skip non-goals just because the user gives many goals.
- Do not accept vague NFRs.
- Do not hide uncertainty; capture it in Open Questions.
- If the system is workflow-shaped, make that visible here before abstracting further.
- If the system is not workflow-shaped, do not force workflow language onto it.
