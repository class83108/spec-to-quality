---
name: system-map
description: >
  Guide the creation of system_map.md — an implementation-facing system cut that turns
  discovery.md and system-design.md into responsibility units, core data/state boundaries,
  entity-selection rationale, interaction flows between core entities, candidate implementation
  cuts, and a practical change protocol for day-to-day development.
  Requires discovery.md and system-design.md to exist. Use when the system shape and key design
  decisions are clear enough that the team now needs a stable structural map for implementation and
  modification work.
  Do NOT use for: defining what the system does (use discovery), doing early system design
  reasoning (use system-design), clarifying slice-level specs (use spec-clarification), or
  implementation planning (use spec-backlog execution).
---

# System Map

> **Output contract:** The skill's conversational output must be in Traditional Chinese, and every section written into `system_map.md` must also be in Traditional Chinese.

You are guiding the user through the creation of **system_map.md**.

This document is not another round of requirements analysis, and it is not another round of system design.

Its job is to expand:

- `discovery.md`
- `system-design.md`

into an **implementation-facing system cut** that developers can use to implement, maintain, and
safely modify the system.

This map should help later contributors answer:

- What are the major responsibility units in the system?
- Which core data or key states does the system actually manage?
- Which part of the system is the main place responsible for each core data item or key state?
- How do the core entities or records affect one another?
- Which seams, handoffs, or risk boundaries matter most?
- Which entities or records need stable names before feature slicing gets detailed?
- Which components or modules should probably be cut apart before implementation starts?
- If something needs to change, where should they look first?

It should not reopen questions like:

- whether the requirements were defined correctly
- why `301` was chosen instead of `302`
- why one major processing step happens before another
- the technical trade-off inside a deep-dive topic

Those questions should already have been handled upstream.

Read `../../references/architect-mindset.md` before proceeding. Focus especially on:

- responsibilities over containers
- boundaries around change and failure
- explicit responsibility for core data and state
- structural traceability back to upstream intent

Prefer plain implementation-facing language over architecture jargon.
When possible, use terms like:

- `核心資料` or `關鍵狀態` instead of `truth`
- `主責管理` or `以這裡為準` instead of `owner`
- `顯示用資料`, `對照資料`, or `衍生結果` instead of `projection`

## Purpose

Use this skill to turn a known system shape and known design decisions into a structure map that is actionable for implementation and maintenance.

This skill is responsible for:

- defining the major responsibility units
- making core data / state responsibility explicit
- checking whether any high-pressure state, entity, or workflow data still lacks a stable name at map level
- identifying important seams and handoffs
- explaining why certain entities or records must be named separately before detailed implementation
- describing how the core entities or records interact
- deriving a candidate component / module cut from the responsibility and core data / state map
- establishing a practical change protocol
- adding a high-level structure diagram when it materially improves clarity

This skill is not responsible for:

- redefining discovery intent
- re-comparing system-design alternatives
- producing low-level API contracts
- writing feature-level specs

## Outputs

Before finishing, you must produce:

- `system_map.md`, with at least these sections:
  `Overview`, `Responsibility Map`, `Core Data / State Map`, `Entity Selection Notes`, `Entity Interaction Map`, `Boundary / Seam Map`, `Implementation Cut`, `Current Focus`, `Change Protocol`
- a clear responsibility map
- a clear core data / state map that says what the system manages, who is the main place responsible, what other parts only read or display it, and what currently counts as the source of record
- a short entity-selection rationale for the items that truly need stable names before implementation
- an entity interaction map that describes how key entities or records create, update, select, replace, or reference one another
- an implementation cut that identifies likely components / modules and their dependency directions
- an explicit note of any important core data item or key state that is still missing a stable map-level name, or a statement that none are missing
- at least 2 meaningful seams, each with an explanation of what it protects
- a **high-level structure diagram** when helpful
  - show only responsibility units, ownership, top-level handoffs, and facade-to-owner relationships
  - do not show detailed sequence, deployment topology, or field-level schema
- user confirmation that the abstraction level is appropriate

If a high-level structure diagram is included, it must follow these rules:

- nodes are responsibility units, not files or classes
- edges are handoffs, ownership relationships, or facade-to-owner relationships
- the diagram is for navigation, not for dumping runtime detail

## Prerequisites

- `discovery.md` must exist and must already be confirmed by the user
- `system-design.md` must exist and must already be confirmed by the user

If the upstream documents are still unsettled around:

- the system shape
- the API or interaction sketch
- key design decisions
- deep-dive conclusions

route back to the appropriate upstream skill instead of forcing a structure map here.

## Workflow

### Phase 1: Re-anchor The Upstream Intent Briefly

Briefly restate the upstream baseline for this map.

Summarize:

- system purpose
- first-version scope
- top-level interactions
- key system-design decisions

Keep this section short. The point is not to rewrite the upstream documents. The point is only to confirm which system, baseline, and decision set this map is serving before structural cutting begins.

### Phase 2: Build The Responsibility Map

Identify the main responsibility units.

A responsibility unit should:

- have a clear job
- change for a recognizable reason
- hold a coherent set of rules, state, or coordination authority
- participate in explainable handoffs with other units

Do not turn this map into:

- a file list
- a class diagram
- a framework-layer inventory

Good example granularity:

- Intake / API facade
- Workflow control
- Review decision handling
- Execution tracking
- Asset lifecycle management
- External compute adapter

### Phase 3: Build The Core Data / State Map

Identify the important core data and key states that the system cannot afford to handle ambiguously.

For each important item, capture:

- what it is
- which responsibility unit is the main place responsible for managing it
- where the source of record lives
- which other parts only read it, display it, or derive data from it
- which handoff is most likely to fail

Keep these distinctions sharp:

- readable does not mean `主責管理`
- queryable does not mean writable
- displayed data does not mean source of record

Also check whether any high-pressure data / state is currently described only as:

- UI behavior
- workflow prose
- a seam risk without a named object
- a vague "state" spread across multiple units

If so, do not jump to feature-level schema design. Instead:

- name the data or state at map level
- identify which unit should main-manage it
- state whether the current map is sufficient or must be revised before feature slicing

### Phase 4: Write Entity Selection Notes

Do not start by listing every possible entity.

Instead, explain why certain items need to exist as separately named entities or records before implementation details begin.

Use criteria such as:

- it represents long-lived core data
- it has its own identity
- it has its own lifecycle
- it is the main anchor for a key rule set or state transition
- the system would become ambiguous if this were left as just a field, temporary payload, or display-only structure

For each selected item, capture at least:

- `項目`
- `為什麼要獨立命名`
- `如果不獨立命名會失真什麼`

Keep this section selective. It should justify the key entities or records, not re-list every data structure in the system.

### Phase 5: Write The Entity Interaction Map

Before writing seams, describe how the key entities or records affect one another.

Focus on interactions such as:

- one entity creates another
- one record selects the currently effective version of another
- one workflow step updates or replaces another piece of core data
- one execution record produces artifacts, telemetry, or history records
- one decision turns current data into historical comparison data

This section is not a schema and not a sequence diagram.
Its job is to explain how core data / state moves, changes, and references other core data / state.

### Phase 6: Build The Boundary / Seam Map

Keep only the seams that actually matter.

Each seam should answer at least:

- `Between`: which responsibility units it separates
- `Carries`: what is handed off across it
- `Why it exists`: what it protects
- `What can go wrong`: the main risk
- `Change impact`: what else must be checked when it changes

Strong seams often come from:

- handoff between different main-managing units
- workflow authority handoff
- translation between current effective data and historical comparison data
- a gate before an expensive or irreversible downstream action

When a seam remains hard to explain, ask whether the real problem is a missing named data / state object rather than a missing unit boundary.

### Phase 7: Derive The Implementation Cut

This phase is where the map turns into likely code boundaries.

Starting from the responsibility map, core data / state map, and entity interactions, identify the likely components or modules that implementation will revolve around.

For each candidate component or module, capture at least:

- `Component / Module`
- `Primary responsibility`
- `Main-manages or coordinates`
- `Depends on`
- `Must not absorb`
- `Why it should stay separate`

At this level, "component / module" may mean:

- a domain area
- an application service boundary
- a workflow/orchestration module
- an integration adapter
- a presentation/admin facade

The point is not to predict the exact folder tree. The point is to expose the likely implementation seams before detailed coding begins.

### Phase 8: Add A High-Level Structure Diagram When Helpful

If the text alone is not clear enough, add a high-level structure diagram.

Typical cases where a diagram helps:

- there are more than 5 responsibility units
- facades and owners are easy to confuse
- multiple handoffs cross in ways that are hard to read in prose

The diagram only needs to help the reader build a structural mental model quickly.

Do not draw:

- detailed runtime sequences
- deployment topology
- infrastructure provisioning
- a full database schema

### Phase 9: Write The Change Protocol

One of the most important values of this document is helping people know where to look first when they need to change something.

At minimum, distinguish:

- discovery-level change
- system-design-level change
- boundary or ownership change
- behavior-only change
- implementation-only change

The goal is to help developers and AI agents know when they must go back upstream instead of patching over structural problems in local code.

## Modeling Boundary

At `system_map` level, the job is to decide whether an important data/state/handoff object must be explicitly named for the structure to stay legible and implementable.

It is not the job to fully design:

- table schemas
- field sets
- endpoint contracts
- feature-level state machines

Use this rule:

- if the system cannot discuss who main-manages a core data item or key state without naming the object, name it now
- if implementation cannot be cleanly cut without naming the object or boundary now, name it now
- if the object is already named clearly and only needs detailed fields or rules, leave that for `feature-slice` and downstream clarification

## Validation

### Structure check

Does this document actually describe:

- responsibility
- core data / state management
- why certain entities or records need separate names
- how the key entities or records interact
- implementation boundaries
- seams

instead of reopening requirements or technology-selection arguments?

### Data / state check

Does every important core data item, key state, or workflow record have a clear main-managing unit?

If not, the map is still too weak to be useful.

### Seam check

Does each seam protect a real change risk or failure risk?

If a seam looks decorative rather than protective, it is probably the wrong seam.

### Upstream traceability check

Can the main responsibility splits and important seams be traced back to:

- the system shape in `discovery.md`
- the prior decisions or trade-offs in `system-design.md`

If not, the structure is probably being cut by intuition instead of intent.

### Navigation check

After reading the map, can a new developer tell:

- which responsibility unit to inspect first for a given interaction change
- which core data or key state is managed where
- why certain entities or records deserve separate names
- how the key entities or records affect one another
- which seams require special caution
- which modules probably need to be separated before implementation starts

### Diagram check

If a diagram exists, does it genuinely improve structural understanding?

If it only redraws the text or smuggles runtime detail into the map, leave it out.

## Guardrails

- Do not write this document as an architecture essay.
- Do not reopen upstream decisions.
- Do not reverse-engineer responsibility units from the source tree.
- Do not confuse read access with main-management responsibility.
- Do not promote every handoff into a seam.
- Do not list entities without explaining why they deserve separate names.
- Do not write the entity interaction section as a schema dump or sequence diagram.
- Do not draw the diagram as a deployment diagram or sequence diagram.
- If upstream decisions are still unsettled, do not sneak system design work into this skill.
- The core value of this document is helping changers understand how the system is cut, which data and entities matter, how they interact, which modules are likely to exist, and where modification risk lives.
