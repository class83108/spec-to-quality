---
name: system-design
description: >
  Guide the creation of system-design.md — starting from discovery.md, system requirements,
  and the top-level interaction sketch, identify the few key design decisions that define the
  system's shape, explain why those choices are being made, and capture the main trade-offs, risks,
  and follow-up deep-dive topics before SYSTEM_MAP work. Treat the current baseline design as
  revisable: deep-dive findings may refine or change the baseline before structure is flattened in
  SYSTEM_MAP.
  Do NOT use for: defining what the system does (use discovery), mapping component structure
  in detail (use system-map), or implementation planning (use spec-backlog execution).
---

# System Design

> **Output contract:** The skill's conversational output must be in Traditional Chinese, and every section written into `system-design.md` must also be in Traditional Chinese.

You are guiding the user to produce **system-design.md**.

This document is not a second pass at goals, and it is not a place to hide behind abstract pressure labels.

It must answer:

- Given the current baseline design, how is the system expected to work at a high level?
- Under that current system shape, what are the **few key design decisions** that matter most?
- **Why was this option chosen instead of the alternatives?**
- What **trade-offs, risks, and downstream pressure** follow from those choices?
- Which topics deserve a dedicated **deep dive** next?

If the document does not make the decision points visible, it tends to collapse into:

- abstract architecture commentary that does not help a new developer
- a restatement of `discovery.md` or system requirements
- premature ownership or seam discussion that belongs in `SYSTEM_MAP`

Read `../../references/design-decision-mindset.md` before proceeding.

Focus especially on:

- starting from a visible baseline design
- comparing real alternatives instead of naming vague concerns
- tying deep dives to important non-functional pressure or obvious bottlenecks
- keeping every major design choice traceable to system requirements

## Purpose

Use this skill to turn "what the system currently plans to do" into "why it is designed this way."

This skill is responsible for:

- reviewing the baseline flow or high-level design sketch created upstream
- restating the current system shape and top-level interactions
- identifying the few **key decisions** that truly shape the system
- documenting, for each decision, the options, current choice, reasoning, trade-offs, and risks
- marking the topics that deserve a deeper follow-up investigation
- updating the baseline design when alternative analysis or deep-dive findings change it

This skill is not responsible for:

- redefining goals or system requirements
- cutting the system into responsibility units
- producing low-level specs or an implementation plan

## Outputs

Before finishing, you must produce:

- a confirmed **current system shape summary**
- a confirmed **key decisions shortlist**
- `system-design.md`, with at least these sections:
  `Current System Shape`, `Key Decisions`, `Trade-offs and Risks`, `Deep Dive Topics`, `Open Questions`
- user confirmation that the key decisions and their rationale are reasonable

Each key decision should:

- trace back to known requirements, goals, or interaction shapes
- include real alternatives that can be compared
- state the current choice
- explain why that choice is being made

Each decision write-up must include at least:

- `Requirement context`
- `Options`
- `Current choice`
- `Why this choice`

## Prerequisites

- `discovery.md` must exist and must already be confirmed by the user
- the discovery output must provide usable:
  `problem framing`, `system requirements`, `top-level API / interaction sketch`, and `baseline flow / high-level design sketch`

If the project only has vague goals and no clear enough system shape yet, route back to `discovery`.

## Workflow

### Phase 1: Re-state Current System Shape

Do not jump into rationale yet. First make sure everyone shares the same understanding of the current system shape.

Review and summarize:

- system purpose
- first-version inputs and outputs
- the baseline flow or processing sequence
- the top-level API or interaction sketch
- the currently assumed major technical path

The goal here is not to rewrite the previous document. The goal is to create a shared baseline so the later decision discussion has something concrete to stand on.

Before moving on, a new developer should already be able to answer:

- How does the first version of this system work?
- How does the outside world interact with it?
- What is the current plan at a high level?

If the baseline is internally inconsistent, too vague, or mismatched with the requirements, fix the baseline first.

### Phase 2: Identify Key Decisions

From the current system shape, identify the few decision points that matter most.

A **key decision** is a choice where a different answer would materially change the system's behavior, capability, cost profile, risk profile, or architectural pressure.

Common categories:

- branching in the processing model
- synchronous vs asynchronous handling
- static vs dynamic behavior
- overwrite vs preserve history
- ordering of major steps
- whether a capability is delegated to an external system

Good examples:

- `301 vs 302`
- `Static vs Dynamic QR`
- `Enhance -> diarize -> whisper` vs a different processing order
- `Append-only history vs in-place overwrite`
- `Accepted-then-complete-later vs synchronous completion`

Poor examples:

- which helper function to use
- which file a class should live in
- route naming style

Useful prompts:

- "Which parts of this system shape still have meaningful alternatives?"
- "Which choices would noticeably change the nature of the system if we picked differently?"
- "Which decisions best explain why this design is taking its current direction?"
- "If you had to explain the design to a new developer, which three choices would you highlight?"

Usually **2-4 key decisions** is enough.

### Phase 3: Evaluate Alternatives

For each decision, compare clearly:

- which requirement, flow, or interaction shape the decision is responding to
- what the main options are
- which option is currently preferred
- why that option is being chosen instead of the others

Do not settle for abstract value statements.

Try to state the reasoning in concrete terms:

- what do we gain with option A?
- what do we lose?
- why is that trade-off acceptable in this project context?

If a decision is still unresolved, say so honestly:

- the options are known
- the current choice is not settled
- more information is needed before choosing

If the alternatives analysis clearly invalidates part of the baseline, update the current system shape directly instead of carrying the contradiction into `SYSTEM_MAP`.

### Phase 4: Capture Trade-offs And Risks

Do not present any decision as a one-way win. Every important decision should include cost.

For each decision, capture at least:

- `Trade-offs`
- `Risks`
- `What to watch`

Examples:

- choosing `302` means the request may go back to origin each time, increasing latency or infrastructure cost
- choosing dynamic behavior strengthens server dependency, increasing availability risk
- choosing to enhance first increases preprocessing cost, but may improve downstream diarization or ASR quality
- choosing append-only history improves traceability, but raises projection or cleanup complexity

### Phase 5: Mark Deep Dive Topics

`Deep Dive Topics` are not just "things we have not figured out yet."

They are topics where:

- the decision is clearly important
- but the second-order consequences, scale pressure, failure modes, or consistency cost deserve focused follow-up work

Good examples:

- cache, CDN, or analytics consistency after `302`
- projection, cleanup, or lineage implications after `append-only history`
- the quality/cost impact of `enhance -> diarize -> whisper` under different input conditions

If a point is only missing information, put it in `Open Questions` instead.

If an early deep-dive conclusion is already strong enough to change the baseline design, write that change back into this document:

- what part of the baseline changes
- which decision's current choice changes with it

### Phase 6: Final Cross-Check

Before finalizing, verify:

- the key decisions genuinely grow out of the requirements or interaction shape
- each decision has explicit options and a current choice
- the document really answers "why this design"
- the trade-offs and risks are visible, not just the selected option
- a new developer could name the most important design decisions after reading it
- any baseline overturned by the analysis has already been updated

If the document mostly restates:

- goals
- system requirements
- vague "good/bad", "simple/complex", or "scalable/not scalable" commentary

then it is not done yet.

## Validation

### Traceability check

Every key decision should point back to at least one of:

- a system requirement
- a goal, non-goal, or constraint
- a top-level interaction shape

If it cannot be traced back, it usually is not a core decision.

### Alternatives check

Does each key decision have real alternatives that can be compared?

If there are no options, it is not a decision. It is only a description.

### Rationale check

Does each current choice actually answer:

- why this option?
- why not the other options?

If not, the document is still only describing outcomes.

### Newcomer check

After reading the document, can a new developer explain:

- how the system currently plans to work
- what the most important decisions are
- what the main costs and risks of those decisions are
- whether the baseline design was updated by the discussion

### Deep-dive check

Do the `Deep Dive Topics` represent real next-step exploration of decision consequences?

If a topic is simply unresolved, move it to `Open Questions`.

## Guardrails

- Do not force design-decision discussion before the system shape is clear enough.
- Do not treat implementation trivia as a key decision.
- Do not list only the current choice without the alternatives.
- Do not list only benefits while hiding trade-offs and risks.
- Do not move early into ownership or seam expansion that belongs in `SYSTEM_MAP`.
- Do not mix "not clarified yet" with "worth a deep dive."
- Do not treat the baseline design as fixed; updating it when needed is part of this skill.
- If the document does not help the reader understand "why this design," it is not finished.
