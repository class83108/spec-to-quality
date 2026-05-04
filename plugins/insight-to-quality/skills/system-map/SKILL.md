---
name: system-map
description: >
  Guide the creation of SYSTEM_MAP.md — a software structure and change-navigation map that turns
  discovery.md and system-design.md into responsibility units, ownership boundaries, key seams, and
  a practical change protocol for day-to-day development.
  Requires discovery.md and system-design.md to exist. Use when the system shape and key design
  decisions are clear enough that the team now needs a stable structural map for implementation and
  modification work.
  Do NOT use for: defining what the system does (use discovery), doing early system design
  reasoning (use system-design), clarifying slice-level specs (use spec-clarification), or
  implementation planning (use spec-backlog execution).
---

# System Map

> **Output contract:** The skill's conversational output must be in Traditional Chinese, and every section written into `SYSTEM_MAP.md` must also be in Traditional Chinese.

You are guiding the user through the creation of **SYSTEM_MAP.md**.

This document is not another round of requirements analysis, and it is not another round of system design.

Its job is to expand:

- `discovery.md`
- `system-design.md`

into a **software structure map** that developers can use to implement, maintain, and safely modify the system.

This map should help later contributors answer:

- What are the major responsibility units in the system?
- Who owns each important state, entity, or workflow truth?
- Which seams, handoffs, or risk boundaries matter most?
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
- explicit ownership
- structural traceability back to upstream intent

## Purpose

Use this skill to turn a known system shape and known design decisions into a structure map that is actionable for implementation and maintenance.

This skill is responsible for:

- defining the major responsibility units
- making core ownership explicit
- identifying important seams and handoffs
- establishing a practical change protocol
- adding a high-level structure diagram when it materially improves clarity

This skill is not responsible for:

- redefining discovery intent
- re-comparing system-design alternatives
- producing low-level API contracts
- writing feature-level specs

## Outputs

Before finishing, you must produce:

- `SYSTEM_MAP.md`, with at least these sections:
  `Overview`, `Interaction-To-Responsibility Mapping`, `Responsibility Map`, `Core Ownership Map`, `Boundary / Seam Map`, `Decision Consequences In Structure`, `Current Focus`, `Change Protocol`
- a clear responsibility map
- a clear core ownership map
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

### Phase 1: Re-anchor The Upstream Intent

Briefly restate the upstream baseline for this map.

Summarize:

- system purpose
- first-version scope
- top-level interactions
- baseline flow
- key system-design decisions
- major trade-offs that affect structure

The point is not to rewrite the upstream documents. The point is to confirm which system, baseline, and decision set this map is serving.

### Phase 2: Map Interactions To Responsibility Units

Start from top-level interactions, not from the source tree.

For each important interaction, answer:

- who receives it first?
- who truly owns the state change or workflow authority it triggers?
- which parts are only facades?
- which parts are actual owners?

This avoids mixing UI, API entrypoints, workflow control, and data ownership into one blurred layer.

### Phase 3: Build The Responsibility Map

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

### Phase 4: Build The Core Ownership Map

Identify the important state, entity, or workflow truth that the system cannot afford to handle ambiguously.

For each important item, capture:

- what it is
- who the primary owner is
- where the source of truth lives
- who is only a consumer, projection, or derived view
- which handoff is most likely to fail

Keep these distinctions sharp:

- readable does not mean owned
- queryable does not mean writable
- projected does not mean source of truth

### Phase 5: Build The Boundary / Seam Map

Keep only the seams that actually matter.

Each seam should answer at least:

- `Between`: which responsibility units it separates
- `Carries`: what is handed off across it
- `Why it exists`: what it protects
- `What can go wrong`: the main risk
- `Change impact`: what else must be checked when it changes

Strong seams often come from:

- ownership transfer
- workflow authority handoff
- translation between current truth and historical truth
- a gate before an expensive or irreversible downstream action

### Phase 6: Capture Decision Consequences In Structure

This section continues from `system-design.md`, but it should not re-argue the decisions themselves.

For each important prior decision, answer:

- which responsibility units does this decision force into existence?
- which ownership boundaries must become explicit because of it?
- which seams become important because of it?

Examples:

- choosing dynamic redirect means the handoff around cache, lookup, and current mapping truth must be explicit
- choosing append-only history means execution truth and current projection must be separated
- choosing a human review gate means review-result ownership and resume authority must be unambiguous

### Phase 7: Add A High-Level Structure Diagram When Helpful

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

### Phase 8: Write The Change Protocol

One of the most important values of this document is helping people know where to look first when they need to change something.

At minimum, distinguish:

- discovery-level change
- system-design-level change
- boundary or ownership change
- behavior-only change
- implementation-only change

The goal is to help developers and AI agents know when they must go back upstream instead of patching over structural problems in local code.

## Validation

### Structure check

Does this document actually describe:

- responsibility
- ownership
- seams

instead of reopening requirements or technology-selection arguments?

### Ownership check

Does every important state, entity, or workflow truth have a clear owner?

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
- which state must not be modified across an ownership boundary
- which seams require special caution

### Diagram check

If a diagram exists, does it genuinely improve structural understanding?

If it only redraws the text or smuggles runtime detail into the map, leave it out.

## Guardrails

- Do not write this document as an architecture essay.
- Do not reopen upstream decisions.
- Do not reverse-engineer responsibility units from the source tree.
- Do not confuse read access with ownership.
- Do not promote every handoff into a seam.
- Do not draw the diagram as a deployment diagram or sequence diagram.
- If upstream decisions are still unsettled, do not sneak system design work into this skill.
- The core value of this document is helping changers understand how the system is cut and where modification risk lives.
