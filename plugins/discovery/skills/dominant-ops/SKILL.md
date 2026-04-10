---
name: ec:dominant-ops
description: >
  Guide the creation of dominant-ops.md — identify where the system's pressure lies by analyzing
  operations through the lens of frequency, cost, and failure impact. Produces a ranked list of
  dominant operations, anti-patterns, and theory limits that directly shape downstream design.
  Requires goals.md to exist. Trigger when goals are defined and the user is ready to analyze
  operational pressure.
  Do NOT use for: defining what the system does (use ec:goals-discovery), mapping system structure
  (use ec:system-map), or implementation planning (use OpenSpec).
---

# Dominant Operations Analysis

You are guiding the user through the creation of **dominant-ops.md** — the document that answers "where does the pressure lie?" in this system. While goals.md defines what the system must do, dominant-ops.md identifies which operations carry the most weight and therefore deserve the most design attention.

Read `shared/references/architect-mindset.md` before proceeding, especially the Dominant Operations Thinking and Traceability sections.

## Working Style

- **Pressure-first, not feature-first.** You are not listing features — you are identifying where time, money, and risk concentrate. An operation that runs once a month but costs $500 and corrupts data on failure may matter more than one that runs 1000 times a day harmlessly.
- **Force ranking.** The user must pick a Top 3. If they say "everything is important", push back — that is the same as saying nothing is important.
- **Demand real numbers.** "It takes a while" is not acceptable. "Cold start is ~45s, processing is ~3min per request" is. If real numbers do not exist yet, mark them as estimates and flag for measurement.
- **Failure impact is the hidden dimension.** Users naturally think about frequency and cost. Your job is to keep asking "what happens if this fails silently?" — that is where the critical insights hide.

## Prerequisites

- **goals.md must exist** and be reviewed by the user
- If goals.md does not exist, stop and redirect to `ec:goals-discovery`
- Read goals.md before starting — you need the goal IDs (Gx) and constraints (Cx) for traceability

## Workflow

### Phase 1: Operations Inventory

List every significant operation the system performs. An "operation" is a unit of work that has a trigger, consumes resources, and produces an observable result.

For each operation:
1. **Name it** with a verb phrase (e.g., "process payment", "generate report", "sync external data")
2. **Link it to goals**: Which Gx does this operation serve?
3. **Estimate frequency**: How often does this run? (per request, per user, per day)
4. **Estimate cost**: Time, compute, money per execution
5. **Estimate failure impact**: What is the worst case if this fails silently?

**Guiding questions**:
- "Walk me through a single end-to-end run. What happens at each step?"
- "Which step do you dread the most when something goes wrong?"
- "Where do you spend the most time waiting?"
- "What would cause a complete re-run from scratch?"

**Merging rule**: Operations on the same causal chain can be merged into a single dominant operation. For example, "user submits form", "system validates input", and "system writes to DB" are part of the same chain and can be grouped as one Dx if they share the same pressure profile.

### Phase 2: Criticality Scoring

Apply the formula to each operation:

```
Criticality = Frequency x Cost x Failure Impact
```

Scoring guide (use relative scale, not absolute numbers):

| Dimension | Low (1) | Medium (2) | High (3) |
|---|---|---|---|
| Frequency | Weekly or less | Daily / per-user | Per-request / per-second |
| Cost | <1 min, <$0.01 | 1-10 min, $0.01-1 | >10 min, >$1, GPU/external API required |
| Failure Impact | Visible error, easy retry | Silent degradation, manual fix needed | Data corruption, cascade failure, re-run entire workflow |

This is a thinking tool, not a scoring system. The numbers help prioritize, but judgment matters more than arithmetic.

### Phase 3: Top 3 Selection

From the scored inventory, select the Top 3 dominant operations:

1. **D1**: The most critical operation — typically the one with the highest failure impact
2. **D2**: The second most critical — often the most frequent expensive operation
3. **D3**: The third most critical — often a human-in-the-loop or coordination operation

For each Dx, document:
- **What it does** (1-2 sentences)
- **Why it dominates** (which dimension drives its criticality)
- **Design implications** (what this means for architecture — boundaries, contracts, error handling)
- **Goal traceability** (which Gx it serves)

**Hard rule**: No more than 3. If the user insists on 4+, ask: "If you had to sacrifice design attention on one of these to improve another, which would you drop?" That is your D4.

### Phase 4: Anti-Patterns

Anti-patterns are design mistakes that would hurt dominant operations. They are not general best practices — they are specific protections for D1/D2/D3.

For each anti-pattern:
1. **Name it**: AP1, AP2, AP3...
2. **State what NOT to do**
3. **Reference which Dx it protects** — if an anti-pattern does not protect a specific dominant operation, it does not belong here
4. **Explain the consequence** of violating it

**The most important anti-pattern often comes from the user, not from you.** Ask: "What mistakes have you seen (or made) in similar systems?" Their experience-driven anti-patterns are more valuable than textbook ones.

### Phase 5: Theory Limits

For each dominant operation, establish the theoretical lower bound:

- **Best-case time**: If everything goes perfectly and there are zero bugs, how fast could this operation possibly run?
- **Binding constraint**: What physical or external limit prevents going faster? (network latency, API rate limit, DB write throughput, model inference time)
- **Current gap**: How far is the current (or estimated) performance from the theoretical limit?

Theory limits serve two purposes:
1. **Know when to stop optimizing**: If you are within 2x of theory limit, further optimization has diminishing returns
2. **Identify the real bottleneck**: The binding constraint tells you what to design around

**Demand real measurements where possible.** If the system exists, ask the user to run benchmarks. If it does not exist yet, mark estimates clearly and flag them for validation.

### Phase 6: Review and Cross-Check

Before finalizing:

1. **Goal coverage**: Does every goal with architectural implications have at least one operation touching it? Uncovered goals may indicate missing operations.
2. **Constraint compatibility**: Do the theory limits conflict with any constraints? (e.g., "D1 needs 10 API calls but C3 limits the rate to 5/min" — this is a fundamental tension that must be resolved)
3. **Anti-pattern completeness**: For each Dx, is there at least one anti-pattern protecting it?
4. **No orphan operations**: Operations in the inventory that do not map to any goal should be questioned — why does this operation exist?

## Output Shape

```markdown
# Dominant Operations — [System Name]

## Operations Inventory

| # | Operation | Goals | Freq | Cost | Failure Impact | Criticality |
|---|---|---|---|---|---|---|
| 1 | [verb phrase] | G1,G2 | [estimate] | [estimate] | [worst case] | [H/M/L] |
| ... | | | | | | |

## Top 3 Dominant Operations

### D1: [Name] (serves Gx, Gy)
- **What**: [1-2 sentences]
- **Why it dominates**: [which dimension, with numbers]
- **Design implication**: [what this means for architecture]

### D2: [Name] (serves Gx)
...

### D3: [Name] (serves Gx, Gz)
...

## Anti-Patterns

- **AP1** (protects D1): [what not to do] — [consequence]
- **AP2** (protects D1, D2, D3): [what not to do] — [consequence]
- ...

## Theory Limits

| Dominant Op | Theory Best | Binding Constraint | Current/Estimated | Gap |
|---|---|---|---|---|
| D1 | [time] | [constraint] | [current] | [ratio] |
| D2 | ... | ... | ... | ... |
| D3 | ... | ... | ... | ... |

## Open Questions
- [Uncertainties that could shift the ranking]
```

## Design Checks

Revisit this analysis if:
- A new goal is added to goals.md that introduces a fundamentally different operation
- Real measurements significantly differ from estimates in theory limits (>3x gap)
- A production incident reveals a failure mode not captured in failure impact scoring
- The team wants to add a 4th dominant operation — force a re-ranking instead

## Examples

### Example 1: Discovering Hidden Failure Impact

An e-commerce system lists "validate order before checkout" as low criticality (runs per-order, fast, cheap).

Push back: "What happens if validation passes but the inventory count was stale — the item was actually out of stock? Who catches it?" User realizes: nobody, until the warehouse tries to fulfill. This silent failure means cancelled orders, refunds, and customer churn. Failure impact jumps from Low to High, and the operation may enter the Top 3.

### Example 2: User Provides the Best Anti-Pattern

You draft AP2 as "Do not cache intermediate results in memory between processing steps."

User corrects: "It is broader than that. The database must be the single source of truth between pipeline stages — do not trust in-memory state OR return values. We learned this the hard way when a background worker died mid-task and we lost intermediate results that only existed in memory."

The user's version is better. Use it. This is why anti-patterns should be elicited, not prescribed.

### Example 3: Merging Operations

User lists 5 separate operations for a content moderation system: "moderator opens review queue", "moderator reads flagged content", "moderator selects action", "moderator writes reason", "system applies moderation".

These are one operation chain with one pressure profile: D2 (Human Moderation Cycle). Merge them. The frequency, cost, and failure impact apply to the entire cycle, not to each click.

### Example 4: Theory Limits Reveal a Contradiction

A reporting system's D1 (generate monthly analytics report) has a theory limit of 45 minutes due to the volume of data aggregation. Constraint C3 says "reports must be available within 10 minutes of request." 45 min > 10 min.

This is not an optimization problem — it is a constraint violation. Flag it immediately: "D1's theory limit conflicts with C3. Either the latency constraint needs to relax, or the approach needs to change fundamentally (e.g., pre-aggregated materialized views, incremental computation, or async report generation with notification). This must be resolved before proceeding to system mapping."

## Key Rules

- **goals.md is the input, not your imagination.** Every operation must trace to a goal. If you invent an operation that serves no goal, the goal is missing, not the operation.
- **Top 3 is a hard cap.** Three dominant operations force focus. Four or more dilute attention.
- **Anti-patterns without Dx references are platitudes.** "Do not use global state" is generic advice. "Do not use global state (protects D1) because worker processes are forked independently and shared memory causes race conditions" is a real anti-pattern.
- **Theory limits require numbers.** "It could be faster" is not a theory limit. "DB query takes minimum 1.2s for full table scan (measured), network round-trip adds ~50ms, so theory best is ~1.25s" is.
- **Do not skip the inventory.** Going straight to "what are the top 3" without listing all operations first guarantees blind spots.
