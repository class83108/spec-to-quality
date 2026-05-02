---
name: design-driver-discovery
description: >
  Guide the creation of design-driver-discovery.md — define the coarse global interaction surface,
  identify which flows and pressures truly drive architecture decisions, and mark which areas need
  later deep dive. Starting from goals.md, frame key flows, capture the system's top-level
  interaction model, discover where pressure concentrates, select the few design drivers that matter
  most, and capture only the implications that should shape system boundaries and later specs.
  Requires goals.md to exist. Trigger when goals are defined and the user is ready to move from
  "what the system must do" into "what will shape the design".
  Do NOT use for: defining what the system does (use goals-discovery), mapping component structure
  in detail (use system-map), or implementation planning (use spec-backlog execution).
---

# Design Driver Discovery

> **Output contract**：繁體中文。`design-driver-discovery.md` 的所有內容皆使用繁體中文。

You are guiding the user through the creation of **design-driver-discovery.md** — the document that answers:

- 系統對外的全局主要互動面是什麼？
- 哪些 flow 值得優先保護？
- 哪些壓力會逼出 architecture decision？
- 哪些地方會影響 boundary / seam / responsibility 的切分？
- 哪些 operation 其實只是 symptom，不是真正的 design driver？
- 哪些高風險區域只先標記，後續再 deep dive？

This document sits between **goals.md** and **SYSTEM_MAP.md**.

- `goals.md` answers: 系統必須做什麼 / 不做什麼
- `design-driver-discovery.md` answers: 全局互動面、哪些 flow 與壓力會主導設計、哪裡值得後續深挖
- `SYSTEM_MAP.md` answers: 元件怎麼切、邊界在哪裡、改動會影響什麼

Read `../../references/architect-mindset.md` before proceeding. Focus especially on:

- framing flows before diving into detail
- distinguishing real design pressure from noisy symptoms
- keeping pressure and design decisions traceable

## Working Style

- **Flow-first, not operation-first.** Start from real work happening in the system: user flows, internal system flows, and operational flows. Do not begin with an abstract list of verbs.
- **Interaction consistency before endpoint detail.** First define the coarse external interaction model — actors, top-level commands/resources, sync vs async shape. Do not jump to field-level APIs.
- **Pressure-first, not feature-first.** You are not cataloging capabilities. You are identifying where frequency, cost, failure impact, waiting, or coordination pressure will shape the design.
- **Discover design drivers, not just pain points.** Some things are annoying but do not change architecture. Focus on the flows and pressures that would alter boundaries, handoffs, consistency models, or failure isolation.
- **Triage now, deep dive later.** This document should identify the few pressures that must shape `SYSTEM_MAP.md`, and separately mark where later deep dive is warranted. Do not turn every uncertainty into a full investigation here.
- **Do not turn this into a spreadsheet ritual.** Scoring is a thinking aid, not the main product.
- **Optimize for human readability.** Use short, descriptive titles and concrete language. The final document should read like a design discussion artifact, not a ranking report.
- **Synthesize and reduce.** The user may describe too little or too much. Your job is to extract the design-relevant layer, summarize it, and keep the conversation at the right abstraction level.

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following:

- [ ] `design-driver-discovery.md` with the sections: System Pressure Overview, Global Interaction Surface, Flow Framing, Pressure-Bearing Flows, Design Drivers, Design Implications, Deep Dive Candidates, Anti-Patterns, Theory Limits, Open Questions
- [ ] A coarse global interaction surface that names the main actors and top-level commands/resources
- [ ] A coarse-grained flow framing that includes user flows, system flows, and operational flows where relevant
- [ ] 2-3 design drivers with short titles and explicit selection reasons
- [ ] For each design driver: pressure source, why it affects architecture, and what design implications it creates
- [ ] Deep dive candidates clearly marked with a reason they are deferred instead of fully explored now
- [ ] At least one anti-pattern per design driver
- [ ] Theory limits or hard constraints for each design driver, using `[estimate]` where needed
- [ ] User has confirmed the final design drivers

**N/A Policy**: If a section has no content, write `(none identified — [reason])` rather than omitting it.

## Prerequisites

- **goals.md must exist** and be reviewed by the user
- If goals.md does not exist, stop and redirect to `goals-discovery`
- Read goals.md before starting — understand the goals, non-goals, constraints, and open questions before discussing design pressure

## Workflow

### Phase 1: Flow Framing

Start by framing the important work the system needs to support and the coarse interaction model it exposes. Do not begin with implementation details.

#### Global Interaction Surface

Before pressure analysis, capture the top-level interaction surface:

1. **Primary actors** — who or what interacts with the system externally
2. **Top-level interactions** — what these actors are mainly trying to do
3. **Interaction style** — command/resource/query/event where helpful
4. **Timing model** — which interactions are synchronous, asynchronous, or accepted-then-complete-later
5. **System language** — the stable names for the major resources, workflows, or commands

This is not an endpoint table. It is a consistency anchor for later slicing and API detail work.

A flow here is coarse-grained. It should answer:

1. **What is happening?**
2. **Who or what triggers it?**
3. **What counts as success?**
4. **Who pays when it fails?**

Classify flows as needed:

- **User flow** — a person is trying to complete a task
- **System flow** — internal data, state, or processing moves through the system
- **Operational flow** — retries, scheduled work, background repair, compensation, admin intervention

Use event-storming style questioning when it helps, but do not require a full event storming artifact.

Useful prompts:

- "Walk me through one useful thing a user or external actor needs to accomplish."
- "After that succeeds, what has changed in the world?"
- "What happens inside the system even when no user is looking at it?"
- "What background or operational process becomes painful when it breaks?"

If the user gives too much detail, pull back up:

- "What is the flow at a task level?"
- "What outcome are you trying to protect here?"
- "Which part of this would still matter if the implementation changed?"

#### Output of Phase 1

A first-pass global interaction surface plus a flow map with 5-10 coarse flows is enough. Do not force detailed step-by-step sequence diagrams.

### Phase 2: Pressure Discovery

From the flow framing, identify where pressure actually concentrates.

Look for pressure such as:

- high frequency
- high execution cost
- high failure impact
- long waiting time
- difficult recovery
- complex human coordination
- cross-boundary fragility
- invisible but operationally painful background work

For each pressure-bearing flow, ask:

1. **How often does this happen?**
2. **What does it cost when it runs?**
3. **What happens if it fails silently?**
4. **Who notices first?**
5. **How hard is it to recover correctly?**
6. **Is the pain mostly user-facing, internal, or operational?**

Useful prompts:

- "Which flow do you dread the most when something goes wrong?"
- "Which flow feels cheap at first, but becomes expensive once recovery starts?"
- "Which flow creates the most waiting, uncertainty, or coordination overhead?"
- "Which flow is easy to ignore because users do not see it, but painful for the team when it breaks?"

#### Output of Phase 2

A shortlist of pressure-bearing flows, each with a clear pressure profile:

- main pressure source
- success/failure stakes
- why the flow deserves design attention

### Phase 3: Design Driver Selection

Now reduce the shortlist to the few pressures that should actually shape architecture.

A **design driver** is not just an important operation. It is a flow or pressure that would change:

- how responsibilities should be grouped
- where boundaries should be drawn
- where stronger contracts are needed
- how failure isolation should work
- what synchronization / persistence / coordination model is required

For each candidate, ask:

1. **If we design poorly around this, what breaks first?**
2. **Would this pressure change service or boundary decisions?**
3. **Would this pressure force special handling for state, consistency, retries, or interaction timing?**
4. **Is this a real design driver, or just a painful symptom of a deeper flow?**

Selection heuristics:

- Keep it if it will clearly influence architecture
- Keep it if failure here is expensive or structurally hard to recover from
- Keep it if this pressure cuts across components or actors
- Reject it if it is just a local implementation concern
- Reject it if it is only one sub-step of a larger pressure-bearing flow
- Reject it if it is noisy but not design-shaping

You should usually end with **2-3 design drivers**. Two is acceptable if the third is weak.

#### Output of Phase 3

For each selected design driver:

- short title
- what flow it refers to
- why it was selected
- what was excluded and why

### Phase 4: Design Implications

Translate each design driver into design-relevant consequences.

Do not jump to full architecture yet. Stay at the level of design pressure and boundary implications that `SYSTEM_MAP.md` needs.

For each design driver, capture:

1. **Pressure source** — frequency, cost, failure impact, waiting, coordination, external dependency, etc.
2. **Why it affects architecture** — what design choice this pressure is likely to force
3. **Boundary implications** — what should probably be isolated, grouped, or protected
4. **Contract implications** — where stronger handoff shape or state guarantees may be needed
5. **Open design decisions** — what should be carried forward into `SYSTEM_MAP.md`

Useful prompts:

- "What part of the system needs the strongest protection because of this pressure?"
- "Does this pressure argue for tighter ownership or stronger separation?"
- "If this fails, where do you want the blast radius to stop?"
- "Which handoff here cannot rely on informal assumptions?"

#### Output of Phase 4

Each design driver should produce a short design implications block that later feeds `SYSTEM_MAP.md`.

### Phase 5: Deep Dive Candidate Marking

Mark the places that should **not** be fully resolved now, but would be risky to leave invisible.

Typical deep dive triggers:

- boundary shape is still unstable
- state or entity ownership is unclear
- failure / retry / compensation semantics are high-cost
- consistency model across a seam is unclear
- multiple future slices could drift into inconsistent API or model language

For each deep dive candidate, capture:

1. **Area**
2. **Why it matters**
3. **Why it is not fully resolved yet**
4. **What future work should revisit it** — `system-map`, `spec-clarification`, or later implementation review

### Phase 6: Anti-Patterns And Theory Limits

Only after the drivers are clear, capture the design mistakes and hard limits that matter.

#### Anti-Patterns

An anti-pattern here is a specific mistake that would damage a selected design driver.

For each anti-pattern:

1. give it a short title
2. state what not to do
3. name which design driver it harms
4. explain the consequence

Good anti-patterns are specific:

- "Do not let review state live only in memory during the moderation cycle"
- "Do not couple user-facing submission latency to the full downstream processing pipeline"

Bad anti-patterns are generic:

- "Do not use global state"
- "Write clean code"

#### Theory Limits

For each design driver, establish the best-case bound or hard limit:

- best-case time
- binding constraint
- current or estimated state
- gap

This is not optional. Use `[estimate]` if real numbers do not exist yet.

Use theory limits to expose:

- genuine bottlenecks
- hidden constraint conflicts
- places where "optimize harder" is the wrong answer

### Phase 7: Review And Cross-Check

Before finalizing:

1. **Goal coverage** — Do the important goals have at least one relevant flow or driver behind them?
2. **Constraint compatibility** — Do any theory limits or driver implications fundamentally conflict with stated constraints?
3. **Interaction consistency check** — Would later API/spec work share the same top-level language for major interactions?
4. **Architecture usefulness** — Would a person writing `SYSTEM_MAP.md` know what boundaries and decisions deserve attention from this document?
5. **False importance check** — Did you accidentally elevate a symptom instead of a real design driver?
6. **Missing operational pressure check** — Did you include only user-facing flows and ignore background or coordination-heavy work?

## Output Shape

```markdown
# Design Driver Discovery — [System Name]

## System Pressure Overview
- [1-2 short paragraphs or bullets describing where the system's main pressure appears to lie]

## Global Interaction Surface

### Actors
- [actor]: [main relationship to the system]

### Top-Level Interactions
- [interaction/resource/command]: [what it lets the actor do]

### Timing Model
- [interaction]: synchronous / asynchronous / accepted-then-complete-later

### Shared Language Notes
- [stable naming or semantic boundary that later specs should preserve]

## Flow Framing

### [Flow title]
- Type: user / system / operational
- Trigger: [who or what starts it]
- Success: [what counts as done]
- Failure cost: [who pays and how]

### [Flow title]
- Type: ...
- Trigger: ...
- Success: ...
- Failure cost: ...

## Pressure-Bearing Flows

### [Flow title]
- Main pressure: [frequency / cost / failure impact / waiting / coordination / etc.]
- Why it matters: [why this flow deserves attention]
- Recovery difficulty: [easy / medium / hard, with brief reason]

## Design Drivers

### [Short title]
- Flow: [which flow this comes from]
- Why selected: [why this is a real design driver]
- Why not just a symptom: [brief explanation]
- Serves goals: [goal titles]

### [Short title]
- Flow: ...
- Why selected: ...
- Why not just a symptom: ...
- Serves goals: ...

## Design Implications

### [Driver title]
- Pressure source: [what kind of pressure dominates]
- Architecture consequence: [what kind of design choice this will influence]
- Boundary implication: [what should be separated, grouped, or isolated]
- Contract implication: [what handoff or state guarantee needs protection]
- Carry into system-map: [what decision or question should move forward]

## Deep Dive Candidates

### [Area title]
- Why it matters: ...
- Deferred because: ...
- Revisit in: `system-map` / `spec-clarification` / implementation review

## Anti-Patterns

### [Short title]
- Harms: [driver title]
- Do not: [specific mistake]
- Consequence: [what breaks]

## Theory Limits

| Driver | Best Case | Binding Constraint | Current / Estimate | Gap |
|---|---|---|---|---|
| [driver title] | [time / cost / limit] | [constraint] | [current or estimate] | [difference] |
| ... | ... | ... | ... | ... |

## Open Questions

### [Short title]
- Question: [uncertainty that could change the design drivers]
- Why it matters: [what downstream decision might change]
```

## Formatting Guidance

The final `design-driver-discovery.md` should feel like a bridge between product intent and architecture work.

- Prefer flow names and driver titles that sound like real work
- Keep prose short and pressure-focused
- Use tables only where they clarify comparison
- Do not over-formalize if the real insight is better expressed in a short paragraph
- If a decision is debatable, explain the tradeoff instead of hiding behind a score

## Design Checks

Revisit this analysis if:

- a new goal introduces a new kind of flow or pressure
- a production incident reveals an unmodeled failure cost
- a background process turns out to dominate operational pain
- the team discovers that a chosen driver was only a symptom, not the underlying design pressure
- `SYSTEM_MAP.md` cannot clearly trace why a major boundary exists

## Examples

### Example 1: User Flow That Truly Drives Design

A submission flow looks simple on the surface: user submits, system stores, later processing happens.

But discovery reveals:

- users wait synchronously for an acknowledgment
- duplicate submission is costly
- downstream processing is slow and failure-prone
- support burden spikes when users are unsure whether submission succeeded

This is not just "accept submission" as an operation. It is a design driver because it affects:

- idempotency
- state ownership
- user-facing feedback timing
- failure isolation between intake and downstream work

### Example 2: Invisible Operational Flow

Users never see the retry-and-repair flow for failed background work, but the team spends hours every week fixing it manually.

That flow may become a design driver if:

- recovery is expensive
- state is hard to trust
- failures cross service boundaries
- manual intervention becomes the hidden bottleneck

### Example 3: Symptom, Not Driver

The team complains that "search feels slow."

Investigation shows search itself is not the design driver. The real driver is a larger import-and-index flow that:

- runs constantly
- has weak handoffs
- creates stale state
- causes search lag as a symptom

Do not model "slow search" as the driver if the actual pressure comes from data freshness and indexing flow design.

## Key Rules

- **Start from flows, not implementation components.**
- **Do not confuse pain with design pressure.** Something can be annoying without being architecture-shaping.
- **A design driver must change design decisions.** If it would not alter boundaries, contracts, or failure handling, it is probably not a true driver.
- **Protect operational flows too.** User-facing work is not the only source of design pressure.
- **Use theory limits honestly.** Estimates are allowed; fake precision is not.
