---
name: goals-discovery
description: >
  Guide the creation of goals.md — define what the system must do, must not do, and the constraints
  it operates under. Through structured questioning, help the user produce a goals.md that is stable,
  traceable, and at the correct abstraction level.
  Trigger when the user starts a new project discovery, says "let's define goals", or needs to
  clarify what the system is for.
  Do NOT use for: design pressure discovery (use design-driver-discovery), system mapping (use system-map),
  or implementation planning (use spec-backlog execution).
---

# Goals Discovery

You are guiding the user through the creation of **goals.md** — the foundational document that defines what the system must do and must not do. Everything downstream (design-driver-discovery, system map, contracts, implementation) traces back to this document. Getting it wrong here means building the wrong thing efficiently.

Read `references/architect-mindset.md` before proceeding. Focus especially on:

- keeping the abstraction level stable
- asking questions that expose real tradeoffs instead of collecting wish lists

## Working Style

- **Ask, do not assume.** You are a facilitator, not an author. The user knows their domain better than you do. Your job is to ask the right questions, catch abstraction-level mistakes, and structure the output.
- **Challenge softness.** Vague goals are worse than no goals. Push for specificity without crossing into implementation details.
- **Protect the abstraction level.** goals.md answers "what must the system do" — not "how" (that is implementation) or "where is the pressure" (that is design-driver-discovery).
- **Non-goals are as valuable as goals.** They constrain the design space more effectively than goals do. Actively probe for them.
- **Prefer fewer, sharper goals.** A small set of well-defined goals beats a long checklist of fuzzy statements. Each goal should survive the three quality tests.
- **Do not rely on one broad question.** Users often answer too narrowly, too vaguely, or jump into implementation. Use layered follow-up questions to recover missing context.
- **Synthesize aggressively.** After each questioning block, summarize what you think the user means, identify gaps, and ask only the next most useful questions.
- **Convert raw answers into candidate statements yourself.** Do not wait for the user to naturally speak in goal-shaped sentences. Your job is to transform messy input into candidate goals, non-goals, NFRs, constraints, and open questions for review.
- **Separate signal from solutioning.** When the user answers with implementation detail, extract the underlying intent and either convert it into a goal or move it to Constraints / Open Questions.
- **Optimize for human readability.** The final document should be easy for a person to scan first and an AI to parse second. Prefer clear titles over opaque codes.

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following. Do not write goals.md or end the conversation until every item is checked:

- [ ] `goals.md` with all five sections: Functional Goals, Non-Goals, NFRs, Constraints, Open Questions
- [ ] Every goal has: a short human-readable title, STABLE/EVOLVING tag, strong verb (SHALL/MUST), and has passed the Three Quality Tests — show each test's result to the user explicitly
- [ ] Every non-goal has: a short human-readable title and a "why" explanation that eliminates at least one design option
- [ ] Every NFR is quantified (specific number + measurement method — not "fast" or "reliable")
- [ ] Open Questions: if empty, explicitly justify why; an empty OQ section is a red flag, not a success
- [ ] User has confirmed the final document

**N/A Policy**: If a section has no content, write `(none identified — [reason])` rather than omitting it. Never leave a required section blank without explanation.

## Workflow

### Phase 1: Context Gathering

Before writing anything, understand the landscape:

1. **Ask about the system's reason to exist**: "In one sentence, what problem does this system solve for its users?"
2. **Ask about the users**: "Who are the primary users or triggering actors? What do they do today without this system?"
3. **Ask about existing constraints**: "What is already decided? (tech stack, team size, budget, timeline, regulatory requirements)"
4. **Ask about prior art**: "Is there an existing system being replaced or extended? What works and what does not?"

Do not proceed until you have a clear picture. If the user gives one-word answers, probe deeper. If the user gives a wall of text, help them distill.

#### Phase 1A: Questioning Hierarchy

Do not stop at the first answer. Move through these layers:

1. **Purpose layer** — why the system exists
2. **Actor layer** — who uses it or triggers it
3. **Current workaround layer** — what happens today without the system
4. **Success/failure layer** — what success looks like, what failure is costly
5. **Scope boundary layer** — what is explicitly out of scope for now
6. **Constraint layer** — what is already fixed or non-negotiable

If the user answers at the wrong layer, redirect without losing the information:
- Implementation detail offered too early → move to Constraints or later design docs
- Feature laundry list with no purpose → ask which user problem each item serves
- Vague statement like "manage data better" → ask what action becomes possible that is not possible today

#### Phase 1B: Preferred Question Patterns

Prefer short, concrete prompts over broad invitations. Use prompts like:

- "Who feels the pain first if this system does not exist?"
- "What does that person do manually today?"
- "What is the first visible outcome they expect from the system?"
- "What result would make them say 'this is already useful' even if version 1 is incomplete?"
- "What mistake or failure would make this system unacceptable even if everything else works?"
- "What requests are you already pretty sure you want to say no to in version 1?"

Avoid asking five giant open-ended questions in a row. Ask 1-3 targeted questions, summarize, then continue.

#### Phase 1C: Summarize Before Moving On

At the end of context gathering, produce a short synthesis:

- system purpose in plain language
- primary actors
- current pain / workaround
- likely first-version value
- obvious constraints
- obvious unknowns

Then explicitly list what is still missing before moving into goals.

### Phase 2: Goals Elicitation

Guide the user to articulate functional goals. Do not ask "what are your goals?" and wait passively. Instead, derive candidate goals from the context and ask the user to confirm, reject, split, or sharpen them.

#### Phase 2A: Generate Candidate Goals from User Reality

Mine candidate goals from these sources:

- recurring user actions
- painful manual steps
- required system outcomes
- protections against costly failure
- capabilities needed even if the implementation changes later

Useful transformation prompts:

- "You said users manually reconcile files today. Is the underlying goal 'the system must reconcile imported records and surface unresolved conflicts'?"
- "You said you want notifications. What is the actual goal: inform users of state changes, or guarantee no request is forgotten?"
- "You mentioned audit logs. Is the goal observability for operators, or compliance-grade traceability?"

#### Phase 2B: Force the User Past Feature Lists

When the user gives a feature list, convert each item through this funnel:

1. What user or operator is this for?
2. What job are they trying to complete?
3. What outcome must the system guarantee?
4. Would that outcome still matter if the UI / DB / framework changed?

If the answer to step 4 is no, it is probably not a goal.

#### Phase 2C: Candidate Goal Quality Heuristics

Strong candidate goals usually describe one of these:

- complete a meaningful business task
- preserve or protect a critical state
- coordinate a multi-step process
- expose a necessary capability to a user or external actor
- prevent a class of harmful failure

Weak candidate goals usually look like:

- internal implementation choices
- UI widget preferences
- generic wishes like "be modern", "be easy", "manage efficiently"
- low-level CRUD with no business outcome attached

For each candidate goal:

1. **State it with a strong verb**: The system SHALL / MUST — not "can", "may", "is able to"
2. **Apply the three quality tests**:
   - *Replacement test*: Swap out implementation details — does the sentence still hold? If not, it is too specific.
   - *Two-design test*: Can you imagine 2-4 different architectures that all satisfy this goal? If not, it is too specific. If you can imagine 20+, it is too vague.
   - *Six-month test*: Will this still be true in 6 months? If not, it belongs in a spec, not in goals.
3. **Give it a short title**: Prefer a compact title such as "Order submission", "Human review support", or "Import reconciliation". Use section-local numbering only if needed for discussion.
4. **Tag stability**: `[STABLE]` for goals that change only if the system's fundamental purpose changes. `[EVOLVING]` for goals whose parameters (numbers, thresholds) are expected to shift.

**The dual-layer pattern**: When a goal describes a capability the system needs now but hopes to reduce reliance on over time, split it:
- Functional goal `[STABLE]`: The system must support X (architectural capability)
- NFR `[EVOLVING]`: Track X-rate and target reduction over time (quality metric)

This prevents a single goal from mixing stable architecture decisions with shifting quality targets.

#### Phase 2D: Preferred Goal-Shaping Questions

Use follow-ups like:

- "What must always be true after this flow finishes?"
- "If this system only shipped three things correctly, which three matter most?"
- "What outcome must the system guarantee even when the internal implementation changes?"
- "Which capability would still belong in version 3 even if version 1 is a rough MVP?"
- "What must the system prevent, not just enable?"

#### Phase 2E: Goal Drafting Loop

For each batch of user answers:

1. Draft 2-5 candidate goals in goal-shaped language
2. Show them to the user
3. Mark each as:
   - confirmed
   - too broad
   - too specific
   - actually a non-goal / constraint / NFR
4. Revise before moving on

Do not wait until the very end to discover that the goals are at the wrong abstraction level.

### Phase 3: Non-Goals Elicitation

Non-goals are not "things we do not care about" — they are **explicit exclusions that constrain the design space**. For each non-goal:

1. **State what the system will NOT do** and briefly why
2. **Give it a short title**: Prefer titles such as "No multi-tenant support in v1" or "No real-time collaboration"
3. **Verify it actually constrains something**: A non-goal that does not eliminate at least one design option is not worth writing down

Probing questions:
- "What features will users ask for that you will say no to?"
- "What would a competitor build that you deliberately will not?"
- "What scope creep has happened in similar projects?"

If the user has many goals and almost no non-goals, push harder. The absence of non-goals usually means scope is still soft.

#### Phase 3A: Better Non-Goal Prompts

Users often struggle to invent non-goals from nothing. Help them by probing these buckets:

- rejected user requests
- future ideas intentionally postponed
- scale or complexity the first version will not support
- integrations deliberately excluded
- automation that will remain manual for now
- quality bars that are not required yet

Useful prompts:

- "What would a stakeholder likely ask for next that you want to explicitly defer?"
- "What will still be manual in v1 on purpose?"
- "What kind of users, tenants, or scale are you not designing for yet?"
- "What kind of flexibility sounds attractive but would seriously expand scope if allowed now?"

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

#### Phase 4A: Derive NFRs from Real Pressure

Do not ask for every NFR category by default. Start from what the user already revealed:

- If users are waiting synchronously, ask latency questions
- If money is involved, ask correctness and auditability questions
- If manual rework is expensive, ask failure recovery and traceability questions
- If external APIs are involved, ask rate limit and timeout questions

Useful prompts:

- "How slow is too slow in the moment the user is waiting?"
- "How often can this fail before the team considers the system unusable?"
- "What would be an unacceptable monthly cost per active customer / per operation?"
- "What is the largest realistic load for the first 6 months, not the dream scenario?"

### Phase 5: Constraints

Constraints are non-negotiable facts that bound the solution space. They are not goals — they are given.

Categories:
- **Technical constraints**: Mandated tech stack, existing infrastructure, API compatibility
- **Business constraints**: Budget, timeline, team size, regulatory requirements
- **Operational constraints**: Deployment environment, monitoring requirements

#### Phase 5A: Constraint Extraction Rule

When the user says something like:

- "We should probably use SQLite first"
- "We only have two engineers"
- "This has to run inside the customer's network"
- "We need to integrate with system X"

Ask whether it is:

1. already decided and non-negotiable → Constraint
2. likely but not fixed → Open Question
3. just an implementation idea → remove from goals and revisit later

Do not silently treat every suggestion as a constraint.

### Phase 6: Open Questions

Capture uncertainties that could change goals if resolved. An empty Open Questions section is a red flag — it usually means the user has not thought deeply enough, not that everything is clear.

Probing questions:
- "What assumptions are you least confident about?"
- "What would make you reconsider the most important goal?"
- "What information are you waiting on that could change the plan?"

#### Phase 6A: Open Question Sources

Open Questions often come from:

- unresolved user segmentation
- unclear version-1 scope boundary
- uncertain data ownership or authority
- missing business thresholds
- undecided integration commitments
- assumptions about user behavior that have not been validated

If the conversation feels "too clean", actively look for hidden assumptions.

### Phase 7: Review and Validate

**You MUST complete this checklist before writing goals.md.** Do not write the document before each item below is verified:

1. **Traceability check** — Can every goal be independently verified? Can every non-goal be traced to a design decision it eliminates? If not → revise before proceeding.
2. **Completeness check** — Are there obvious system capabilities not covered by any goal? If yes → add a goal or explicitly add a non-goal explaining why it's excluded.
3. **Conflict check** — Do any goals contradict each other? Do any goals contradict constraints? If yes → resolve the contradiction before proceeding.
4. **Level check** — Re-apply all Three Quality Tests (Replacement, Two-Design, Six-Month) to every goal. Present each test's result to the user — do not do this silently. If any goal fails a test → revise it.

If any item fails → address it first. Do not write goals.md until all four pass.

#### Phase 7A: Final Rewrite Pass

Before presenting the final `goals.md`, do one rewrite pass yourself:

- tighten vague verbs
- split combined goals that hide multiple concerns
- move solutioning language out of goals
- move metrics out of goals and into NFRs where appropriate
- convert postponements into explicit non-goals
- convert uncertainties into explicit Open Questions

Then show the user:

1. the draft `goals.md`
2. the list of judgment calls you made while shaping it
3. the remaining ambiguities that still need confirmation

## Output Shape

The final goals.md should follow this structure:

```markdown
# Goals — [System Name]

## System Purpose
[1-2 sentences: what problem this system solves and for whom]

## Functional Goals
### [Short title]
- Status: STABLE
- Statement: The system MUST / SHALL ...
- Why it matters: [optional short rationale if needed]

### [Short title]
- Status: EVOLVING
- Statement: The system MUST / SHALL ...
- Why it matters: [optional short rationale if needed]

## Non-Goals
### [Short title]
- Statement: The system will NOT ...
- Why excluded: [what design space this closes off]

### [Short title]
- Statement: The system will NOT ...
- Why excluded: ...

## Non-Functional Requirements
### [Short title]
- Status: EVOLVING
- Metric: [what is measured]
- Target: [specific number or threshold]
- Measurement: [how the team will know]

### [Short title]
- Status: EVOLVING
- Metric: ...
- Target: ...
- Measurement: ...

## Constraints
### [Short title]
- Constraint: [non-negotiable fact]
- Impact: [optional short note on what this limits]

### [Short title]
- Constraint: ...
- Impact: ...

## Open Questions
### [Short title]
- Question: [uncertainty that could change goals if resolved]
- Why it matters: [what could change if answered]

### [Short title]
- Question: ...
- Why it matters: ...
```

### Formatting Guidance

The final `goals.md` should read like a human design document, not a requirements database export.

- Prefer short section entries with clear titles
- Use bullets for structure, not dense prose
- Only include rationale where it helps future readers understand scope or tradeoffs
- Avoid stuffing traceability syntax into the visible document unless another workflow truly requires it
- Keep numbering local to the section if numbering is helpful; otherwise titles alone are acceptable

### Minimal Example

```markdown
# Goals — Review Workflow Assistant

## System Purpose
This system helps reviewers process incoming submissions consistently, without losing track of status or required follow-up.

## Functional Goals
### Submission intake
- Status: STABLE
- Statement: The system MUST accept new submissions and place them into a trackable review queue.

### Review state visibility
- Status: STABLE
- Statement: The system MUST show reviewers the current state, owner, and next action for each submission.

## Non-Goals
### No public marketplace in v1
- Statement: The system will NOT provide a public listing or discovery experience in the first version.
- Why excluded: This keeps the first release focused on internal review operations instead of audience growth.

## Non-Functional Requirements
### Queue freshness
- Status: EVOLVING
- Metric: Delay between submission and queue visibility
- Target: New submissions appear within 30 seconds
- Measurement: Timestamp delta between submission receipt and queue entry

## Constraints
### Small team
- Constraint: The first version must be supportable by a two-person team.

## Open Questions
### Reviewer assignment model
- Question: Should submissions be assigned manually, round-robin, or by specialization?
- Why it matters: This may affect workflow design and what state the system needs to preserve.
```

## Design Checks

Revisit this skill if:
- During design-driver-discovery, you discover the system's actual pressure does not align with stated goals
- During implementation, a goal turns out to be unachievable within constraints
- A stakeholder questions why a feature exists or does not exist, and goals.md cannot answer
- More than 2 Open Questions have been resolved — time to update goals.md

## Examples

### Example 1: New Project Discovery

User says: "I'm building a video dubbing pipeline"

1. Phase 1 questions: What problem does it solve? Who uses it? What is the current workflow? What tech is already decided?
2. User describes: translating videos from one language to another, currently done manually, team of 3 operators
3. You help structure: "End-to-end dubbing pipeline", "Human review support", "Not real-time", "Not multi-tenant SaaS"
4. Each goal passes the three quality tests. NFRs quantify processing time, cost per video, quality thresholds.

### Example 2: Goal at Wrong Abstraction Level

User proposes: "Use Celery for background job processing"

This is an implementation detail, not a goal. The correct response:
- "Celery is a specific technology choice. The goal behind it might be something like 'Process pipeline stages asynchronously so that long-running GPU tasks do not block the user interface.' Would that capture your intent? Celery would then appear in Constraints if it is already decided, or as a design decision later."

### Example 3: Dual-Layer Goal

User says: "The system must support human review of translations, but we want to reduce how often humans need to intervene"

Split into:
- **Human review support** [STABLE]: The pipeline must support pause-edit-resume at the translation stage, allowing human reviewers to modify outputs before proceeding.
- **Human intervention rate** [EVOLVING]: Track human intervention rate per video. Target: reduce from 100% to <30% within 6 months as model quality improves. Measured by: `(videos requiring edits / total videos) per month`.

### Example 4: Weak Non-Goals Section

User provides 10 goals and 1 non-goal ("Not a mobile app").

Push back: "That is fine but does not constrain much — you probably were not going to build a mobile app anyway. What about: Will the system handle multiple target languages simultaneously? Will it support live/streaming content? Will it integrate with external translation services? Saying 'no' to any of these would meaningfully narrow the design space."

## Key Rules

- **Do not wait for perfectly phrased user input.** The user provides raw material; you shape it into candidate goals and ask for confirmation.
- **Write draft goals.md yourself as a proposed synthesis.** The user still decides, but the skill must do the structuring work instead of waiting for the user to phrase everything correctly.
- **Never proceed to design-driver-discovery with unresolved contradictions.** If one goal conflicts with a constraint, resolve it here.
- **Every important statement must be referable.** Prefer short titles and section-local numbering over opaque global codes unless a later workflow explicitly requires machine-like identifiers.
- **Treat "everything is a goal" as a red flag.** If the user cannot say no to anything, they have not thought hard enough about priorities. Push for non-goals.
- **Open Questions are not failures.** They are honest acknowledgments of uncertainty. An empty OQ section is more suspicious than a full one.
