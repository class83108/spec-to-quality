---
name: feature-planning
description: >
  The bridge step between discovery completion and OpenSpec creation. Reads the SYSTEM_MAP
  Current State and gaps, cross-references existing feature plans and OpenSpec changes,
  decides which feature to implement next, guides error handling strategy decisions, produces
  docs/feature-plans/{name}.md, and updates SYSTEM_MAP Current State.
  Trigger when discovery is complete, the user wants to decide what to implement next, or
  says "ready to start planning implementation".
  Do NOT use for: when discovery is incomplete (complete align-internals and align-surface
  first), or when a feature plan already exists and implementation should begin directly
  (use feature-coverage).
---

# Feature Planning

You are entering the feature planning stage. This skill's responsibility is to decide what to implement from discovery gaps and lock down key design decisions before OpenSpec is created.

**Before starting, read:**
- `references/architect-mindset.md` (traceability: confirm the feature has a goal foundation)
- `references/implementation-mindset.md` (Part 1: the Three Decisions framework)

## Prerequisites

All of the following must exist before entering this skill:
- `goals.md`
- `dominant-ops.md`
- `SYSTEM_MAP.md`
- At least one align report (output from align-internals or align-surface)

If any are missing, stop and inform: "Feature planning requires complete discovery output. Starting implementation without discovery means every subsequent skill lacks a foundation. Please complete [missing skill] first."

## Workflow

### Step 1: Read Discovery Output

Read the following documents and extract key information:

**goals.md** → all Gx IDs and their descriptions

**dominant-ops.md** → all Dx IDs and their anti-patterns; record each anti-pattern's full ID (AP1, AP2...)

**SYSTEM_MAP.md** → focus on:
- `Current State.Gaps`: the known gap list (which gaps are already partially covered by feature plans)
- `Current State.In-flight`: work currently in progress
- `Boundary Map`: contract status for each boundary
- `Lessons`: pitfalls from completed features that may affect the candidates being planned — extract entries related to the boundaries or components this feature will touch and present them to the user during Step 2

**Align reports** → check whether `docs/alignment/internals-report.md` and `docs/alignment/surface-report.md` exist (active, unprocessed findings):
- Both exist → read both, merge their Gaps and Recommendations, deduplicate items that describe the same underlying problem. These findings supplement the SYSTEM_MAP Gaps.
- One or neither exists → proceed using SYSTEM_MAP Gaps only. SYSTEM_MAP is the single source of truth once reports have been processed and archived.

**Existing feature plans**: scan the `docs/feature-plans/` directory to understand which gaps are already covered or partially covered

### Step 2: Cross-Reference Covered Gaps and Build Candidate List

Cross-reference the SYSTEM_MAP Gaps with existing feature plans:

```
SYSTEM_MAP Gaps
    ↓ cross-reference
docs/feature-plans/ existing files
openspec/changes/ existing changes
    ↓
Not covered     → add directly to candidates
Partially covered → add to candidates, note "continuing [existing feature plan], feature N of M"
Fully covered   → exclude
```

Produce a candidate list, each item annotated with:
- The corresponding SYSTEM_MAP gap or align report deficiency
- Which Gx it serves
- Whether it continues an existing feature plan

```
Feature Candidates:
1. [feature-name] — closes [gap description], serves G1, G2
2. [feature-name] — continues task-queue.md (2 of 3), serves G1
...
```

If the SYSTEM_MAP Lessons section contains entries related to any candidate's boundaries or components, surface them:

```
Related lessons from previous features:
- Candidate 1 touches Seam A: [one-line summary from Lessons] (link to details)
```

Ask the user: "Which one should we plan first?"

### Step 3: Confirm Traceability and SYSTEM_MAP Update Need

After the user selects a feature, confirm two things:

**Traceability:**
- Which Gx does this feature serve? (must correspond to at least one goal)
- Which Dx is it related to? (determines which anti-patterns will be referenced)
- Which SYSTEM_MAP boundaries does it touch?

If the feature cannot be mapped to any Gx → stop: "This feature currently has no goal foundation. Confirm which goal it serves, or add it to goals.md before continuing."

**Whether SYSTEM_MAP needs updating:**

Ask the user: "Does this gap require multiple features because:
A) There is new understanding of system structure (boundary needs adjustment, component needs splitting) → update the SYSTEM_MAP Boundary Map or Component Map first, then continue
B) Engineering is staged (system structure is unchanged, just implemented in phases) → don't update structure; record the phased information in the feature plan and SYSTEM_MAP Current State"

If A → guide the user to update the relevant SYSTEM_MAP sections, then continue with Step 4.

### Step 4: Guide Error Handling Strategy

Read `implementation-mindset.md` Part 1 and guide through the Three Decisions one by one. Drive the conversation with discovery context — do not ask generic questions.

**Decision 1 — Catch Boundary**

Using SYSTEM_MAP boundary information:
"This feature touches [boundary A]. When something goes wrong, does the caller need to know about errors at this level, or should this component handle them internally before responding?"

**Decision 2 — Error Taxonomy**

Using dominant-ops anti-patterns:
"Based on [Dx]'s [APx], [failure scenario] is a risk to guard against. Is this a domain error the business expects (e.g., resource not found, duplicate creation), or an infrastructure failure from an external dependency (DB unreachable, API timeout)? Beyond what the anti-patterns mention, what other expected failure scenarios exist for this feature?"

**Decision 3 — Recovery Strategy**

"When domain errors occur, return a structured error to the caller? When infrastructure errors occur, fail fast and let the caller handle it, or is there a retry or degradation strategy?"

### Step 5: Extract Constraints

**Anti-patterns**: From dominant-ops.md, extract anti-patterns from the relevant Dx that apply to this feature, with full IDs. For each, explain "how this constraint limits this feature's behavior."

**Boundary Rules**: From SYSTEM_MAP.md's Boundary Map, extract boundary rules this feature touches. Explain "which boundary this feature must not cross" or "the direction data must flow."

### Step 6: Produce Feature Plan

Create `{feature-name}.md` under `docs/feature-plans/`, using kebab-case — the same name as the subsequent OpenSpec change.

```markdown
# Feature Plan — {feature-name}

## Source
- Serves: G1, G3
- SYSTEM_MAP gap: [gap description]
- Coverage: Full coverage / Partial coverage (feature N of M, continues [previous feature-plan])

## Error Handling Strategy
- Catch boundary: [boundary only / per-layer / selective: which exceptions]
- Domain errors: [list, e.g. TaskNotFound, DuplicateTask]
- Infrastructure errors: [fail fast / retry N times / degrade to X]

## Constraints

### Anti-patterns (from dominant-ops)
- AP1 (D1): [anti-pattern description] — impact on this feature: [explanation]
- AP2 (D2): [anti-pattern description] — impact on this feature: [explanation]

### Boundary Rules (from SYSTEM_MAP)
- [This feature must not cross boundary X, because: ...]
- [Data may only flow from A to B, not in reverse]

## Integration Test Gaps
<!-- Filled in after tdd-workflow Verification Ledger is complete; initially empty -->
<!-- Records gaps that need integration tests but are deferred; pre-complete reads this section -->
```

### Step 7: Update SYSTEM_MAP Current State and Archive Reports

**First — write all curated findings to SYSTEM_MAP Gaps** (not only the selected feature). For every finding from the alignment reports that was judged worth tracking, add it to `Current State.Gaps`. Findings not worth tracking (quick fixes, doc corrections) are handled inline and do not appear here. This makes SYSTEM_MAP the complete source of truth going forward.

```markdown
## Current State
- **In-flight**: [feature-name] (docs/feature-plans/{feature-name}.md)
- **Gaps**:
  - [gap from alignment or prior SYSTEM_MAP]: not started / planned (docs/feature-plans/...) / partially covered (feature N of M, in-flight)
  - [additional curated findings from alignment reports, if any]
```

**Then — conditionally archive the alignment reports** (only if fully covered). Check the Coverage Status table in each report:

- If **all scopes show `audited`**: move the report to `docs/alignment/archive/` with a date prefix — it is complete and no more appending is needed.
- If **any scope is still `pending`**: leave the report in place. The next align session will append to it; `feature-planning` will re-read and incorporate the new findings next time.

```
# Only move when fully audited:
docs/alignment/internals-report.md  →  docs/alignment/archive/YYYY-MM-DD-internals-report.md
docs/alignment/surface-report.md    →  docs/alignment/archive/YYYY-MM-DD-surface-report.md
```

After a fully-covered report is archived, the active `docs/alignment/` directory contains no report for that layer. Re-run `align-internals` or `align-surface` whenever a fresh audit is needed.

### Step 8: Confirm and Hand Off

Present the feature plan to the user for confirmation:
1. Traceability is correct (Gx mapping, gap description)
2. Error handling strategy matches intent
3. Constraints do not miss any important anti-patterns or boundary rules

After confirmation, inform:
"Feature plan created at `docs/feature-plans/{feature-name}.md`, SYSTEM_MAP Current State updated.

Next steps:
1. Use `opsx:apply` to create the OpenSpec change; add `**Serves:** G1, G3` to the top of spec.md body
2. Reference this feature plan's Error Handling Strategy in the Decisions section of design.md
3. After OpenSpec is created, proceed to `feature-coverage`"

## Examples

### Example 1: Normal Flow

User says: "Discovery is complete, ready to start planning implementation."

1. Read discovery documents, find 3 gaps: TaskQueue contract not implemented, Worker boundary undefined, API surface missing status query
2. Scan `docs/feature-plans/` — currently empty
3. List 3 candidates → user selects "TaskQueue implementation"
4. Confirm Serves G1, G3; ask if SYSTEM_MAP update needed → staged engineering, no structural update
5. Guide Three Decisions:
   - Catch boundary: boundary only
   - Domain errors: TaskNotFound, QueueFull
   - Infrastructure errors: fail fast
6. Extract AP1 (do not accept tasks while one is in progress), AP2 (state must be persisted)
7. Produce `docs/feature-plans/task-queue.md`
8. Update SYSTEM_MAP Current State
9. Inform next steps

### Example 2: Continuing an Existing Feature Plan

SYSTEM_MAP Gaps shows "TaskQueue implementation" is partially covered (feature 1 of 3 complete).

Correct behavior: The candidate list shows "task-queue-worker (continues task-queue.md, 2 of 3)"; the Coverage field in the feature plan notes "Partial coverage (feature 2 of 3, continues task-queue.md)".

### Example 3: SYSTEM_MAP Structure Update Required

After selecting a feature, the team realizes the original SYSTEM_MAP placed A and B in the same boundary, but implementation planning reveals they need to be split into two independent boundaries.

Correct behavior: Stop and inform — "This finding reflects new understanding of system structure. Recommend updating the SYSTEM_MAP Boundary Map (split Seam A into Seam A1 and Seam A2) before continuing feature planning. This ensures subsequent feature plans and OpenSpec have the correct boundary foundation."

### Example 4: Incomplete Discovery

Only goals.md exists, no SYSTEM_MAP.

Correct behavior: "Feature planning requires complete discovery output. SYSTEM_MAP.md is missing — please complete system-map, align-internals, and align-surface before returning to feature planning."
