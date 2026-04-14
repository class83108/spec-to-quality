---
name: goals-discovery
description: >
  Guide the creation of goals.md — define what the system must do, must not do, and the constraints
  it operates under. Through structured questioning, help the user produce a goals.md that is stable,
  traceable, and at the correct abstraction level.
  Trigger when the user starts a new project discovery, says "let's define goals", or needs to
  clarify what the system is for.
  Do NOT use for: dominant operations analysis (use dominant-ops), system mapping (use system-map),
  or implementation planning (use spec-backlog execution).
---

# Goals Discovery

You are guiding the user through the creation of **goals.md** — the foundational document that defines what the system must do and must not do. Everything downstream (dominant-ops, system map, contracts, implementation) traces back to this document. Getting it wrong here means building the wrong thing efficiently.

Read `references/architect-mindset.md` before proceeding. The principles there — especially Document Level = Abstraction Level and the Questioning Hierarchy — are your primary tools in this skill.

## Working Style

- **Ask, do not assume.** You are a facilitator, not an author. The user knows their domain better than you do. Your job is to ask the right questions, catch abstraction-level mistakes, and structure the output.
- **Challenge softness.** Vague goals are worse than no goals. Push for specificity without crossing into implementation details.
- **Protect the abstraction level.** goals.md answers "what must the system do" — not "how" (that is implementation) or "where is the pressure" (that is dominant-ops).
- **Non-goals are as valuable as goals.** They constrain the design space more effectively than goals do. Actively probe for them.
- **Prefer fewer, sharper goals.** 8-12 well-defined goals beat 25 fuzzy ones. Each goal should survive the three quality tests.

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following. Do not write goals.md or end the conversation until every item is checked:

- [ ] `goals.md` with all five sections: Functional Goals, Non-Goals, NFRs, Constraints, Open Questions
- [ ] Every goal has: Gx ID, STABLE/EVOLVING tag, strong verb (SHALL/MUST), and has passed the Three Quality Tests — show each test's result to the user explicitly
- [ ] Every non-goal has: NGx ID and a "why" explanation that eliminates at least one design option
- [ ] Every NFR is quantified (specific number + measurement method — not "fast" or "reliable")
- [ ] Open Questions: if empty, explicitly justify why; an empty OQ section is a red flag, not a success
- [ ] User has confirmed the final document

**N/A Policy**: If a section has no content, write `(none identified — [reason])` rather than omitting it. Never leave a required section blank without explanation.

## Workflow

### Phase 1: Context Gathering

Before writing anything, understand the landscape:

1. **Ask about the system's reason to exist**: "In one sentence, what problem does this system solve for its users?"
2. **Ask about the users**: "Who are the primary users? What do they do today without this system?"
3. **Ask about existing constraints**: "What is already decided? (tech stack, team size, budget, timeline, regulatory requirements)"
4. **Ask about prior art**: "Is there an existing system being replaced or extended? What works and what does not?"

Do not proceed until you have a clear picture. If the user gives one-word answers, probe deeper. If the user gives a wall of text, help them distill.

### Phase 2: Goals Elicitation

Guide the user to articulate functional goals. For each candidate goal:

1. **State it with a strong verb**: The system SHALL / MUST — not "can", "may", "is able to"
2. **Apply the three quality tests**:
   - *Replacement test*: Swap out implementation details — does the sentence still hold? If not, it is too specific.
   - *Two-design test*: Can you imagine 2-4 different architectures that all satisfy this goal? If not, it is too specific. If you can imagine 20+, it is too vague.
   - *Six-month test*: Will this still be true in 6 months? If not, it belongs in a spec, not in goals.
3. **Assign an ID**: G1, G2, G3... — these IDs are used for traceability throughout all downstream documents.
4. **Tag stability**: `[STABLE]` for goals that change only if the system's fundamental purpose changes. `[EVOLVING]` for goals whose parameters (numbers, thresholds) are expected to shift.

**The dual-layer pattern**: When a goal describes a capability the system needs now but hopes to reduce reliance on over time, split it:
- Functional goal `[STABLE]`: The system must support X (architectural capability)
- NFR `[EVOLVING]`: Track X-rate and target reduction over time (quality metric)

This prevents a single goal from mixing stable architecture decisions with shifting quality targets.

### Phase 3: Non-Goals Elicitation

Non-goals are not "things we do not care about" — they are **explicit exclusions that constrain the design space**. For each non-goal:

1. **State what the system will NOT do** and briefly why
2. **Assign an ID**: NG1, NG2, NG3...
3. **Verify it actually constrains something**: A non-goal that does not eliminate at least one design option is not worth writing down

Probing questions:
- "What features will users ask for that you will say no to?"
- "What would a competitor build that you deliberately will not?"
- "What scope creep has happened in similar projects?"

**Non-goals often outnumber goals in a well-defined system.** If the user has 10 goals and 2 non-goals, push harder.

### Phase 4: Non-Functional Requirements (NFRs)

NFRs must be quantified. "Fast" and "reliable" are not NFRs — they are wishes.

For each NFR:
1. **Define the metric**: What exactly are you measuring?
2. **Define the target**: What number constitutes success?
3. **Define the measurement method**: How will you know if you hit the target?
4. **Tag as `[EVOLVING]`**: NFR numbers almost always change over time

Common NFR categories to probe:
- Performance (latency, throughput)
- Reliability (uptime, data durability)
- Cost (per-operation budget, total budget)
- Scalability (concurrent users, data volume)
- Security (authentication, authorization, data protection)

### Phase 5: Constraints

Constraints are non-negotiable facts that bound the solution space. They are not goals — they are given.

Categories:
- **Technical constraints** (C-prefix): Mandated tech stack, existing infrastructure, API compatibility
- **Business constraints**: Budget, timeline, team size, regulatory requirements
- **Operational constraints**: Deployment environment, monitoring requirements

### Phase 6: Open Questions

Capture uncertainties that could change goals if resolved. An empty Open Questions section is a red flag — it usually means the user has not thought deeply enough, not that everything is clear.

Probing questions:
- "What assumptions are you least confident about?"
- "What would make you reconsider the most important goal?"
- "What information are you waiting on that could change the plan?"

### Phase 7: Review and Validate

**You MUST complete this checklist before writing goals.md.** Do not write the document before each item below is verified:

1. **Traceability check** — Can every goal be independently verified? Can every non-goal be traced to a design decision it eliminates? If not → revise before proceeding.
2. **Completeness check** — Are there obvious system capabilities not covered by any goal? If yes → add a goal or explicitly add a non-goal explaining why it's excluded.
3. **Conflict check** — Do any goals contradict each other? Do any goals contradict constraints? If yes → resolve the contradiction before proceeding.
4. **Level check** — Re-apply all Three Quality Tests (Replacement, Two-Design, Six-Month) to every goal. Present each test's result to the user — do not do this silently. If any goal fails a test → revise it.

If any item fails → address it first. Do not write goals.md until all four pass.

## Output Shape

The final goals.md should follow this structure:

```markdown
# Goals — [System Name]

## System Purpose
[1-2 sentences: what problem this system solves and for whom]

## Functional Goals
- **G1** [STABLE]: [strong verb] [what the system must do]
- **G2** [STABLE]: ...
- **G3** [EVOLVING]: ...

## Non-Goals
- **NG1**: [what the system will NOT do] — [why]
- **NG2**: ...

## Non-Functional Requirements
- **NFR1** [EVOLVING]: [metric] [target] [measurement method]
- ...

## Constraints
- **C1**: [non-negotiable fact]
- ...

## Open Questions
- **OQ1**: [uncertainty that could change goals if resolved]
- ...
```

## Design Checks

Revisit this skill if:
- During dominant-ops analysis, you discover the system's actual pressure does not align with stated goals
- During implementation, a goal turns out to be unachievable within constraints
- A stakeholder questions why a feature exists or does not exist, and goals.md cannot answer
- More than 2 Open Questions have been resolved — time to update goals.md

## Examples

### Example 1: New Project Discovery

User says: "I'm building a video dubbing pipeline"

1. Phase 1 questions: What problem does it solve? Who uses it? What is the current workflow? What tech is already decided?
2. User describes: translating videos from one language to another, currently done manually, team of 3 operators
3. You help structure: G1 (end-to-end pipeline), G2 (quality control with human review), NG1 (not real-time), NG2 (not multi-tenant SaaS)
4. Each goal passes the three quality tests. NFRs quantify processing time, cost per video, quality thresholds.

### Example 2: Goal at Wrong Abstraction Level

User proposes: "G5: Use Celery for background job processing"

This is an implementation detail, not a goal. The correct response:
- "Celery is a specific technology choice. The goal behind it might be something like 'G5: Process pipeline stages asynchronously so that long-running GPU tasks do not block the user interface.' Would that capture your intent? Celery would then appear in Constraints if it is already decided, or as a design decision later."

### Example 3: Dual-Layer Goal

User says: "The system must support human review of translations, but we want to reduce how often humans need to intervene"

Split into:
- **G4** [STABLE]: The pipeline must support pause-edit-resume at the translation stage, allowing human reviewers to modify outputs before proceeding.
- **NFR3** [EVOLVING]: Track human intervention rate per video. Target: reduce from 100% to <30% within 6 months as model quality improves. Measured by: `(videos requiring edits / total videos) per month`.

### Example 4: Weak Non-Goals Section

User provides 10 goals and 1 non-goal ("NG1: Not a mobile app").

Push back: "NG1 is fine but does not constrain much — you probably were not going to build a mobile app anyway. What about: Will the system handle multiple target languages simultaneously? Will it support live/streaming content? Will it integrate with external translation services? Saying 'no' to any of these would meaningfully narrow the design space."

## Key Rules

- **Never write goals.md yourself.** You guide — the user decides. Present drafts for their review and iterate.
- **Never proceed to dominant-ops with unresolved contradictions.** If G3 and C2 conflict, resolve it here.
- **Every goal must have an ID.** Downstream documents reference these IDs. An un-numbered goal is an untrackable goal.
- **Treat "everything is a goal" as a red flag.** If the user cannot say no to anything, they have not thought hard enough about priorities. Push for non-goals.
- **Open Questions are not failures.** They are honest acknowledgments of uncertainty. An empty OQ section is more suspicious than a full one.
