---
name: ec:feature-coverage
description: >
  Coverage analysis before writing .feature files. Enforces analysis of all 6 scenario
  categories (Applicable / Not applicable with reason) and waits for user confirmation
  before any Gherkin scenario is written.
  Trigger when the user wants to start writing .feature files, define test scenarios,
  plan scenarios, or says "coverage analysis".
  Do NOT use for: when .feature files already exist and implementation should begin
  (use ec:tdd-workflow), during bug fixes, or when simply discussing requirements.
---

# Feature Coverage Analysis

You are entering the pre-writing step before any .feature file is created. You must complete coverage analysis and get user confirmation before writing any Gherkin scenario.

**Before starting, read `references/implementation-mindset.md` Part 3** — it contains definitions for all 6 scenario categories, analysis entry points, and overlap rules.

## Workflow (follow strictly)

### Step 0: Read the Feature Plan

Read `docs/feature-plans/{feature-name}.md` (same name as the OpenSpec change).

If no matching feature plan is found:
→ Stop and inform: "There is no feature plan for this change. Run ec:feature-planning to create one before proceeding with coverage analysis."

Extract and record the following — these are the analysis entry points for categories 1, 2, and 3:

- **Serves**: List of Gx IDs and their goal descriptions
- **Error Handling Strategy**: Catch boundary, Domain errors, Recovery strategy
- **Anti-patterns**: List with IDs (AP1, AP2...) and descriptions
- **Boundary Rules**: Which boundaries are touched and their constraints

Only proceed to Step 1 after completing Step 0.

### Step 1: Read OpenSpec Implementation Details

Read the specific behavioral specifications from the relevant OpenSpec change — these are the analysis entry points for categories 4, 5, and 6:

- SHALL / MUST statements from spec.md (source for category 5 State mutation)
- Conditional logic in Requirements (source for category 4 Business rules)
- Schema definitions (source for category 6 Output contract)

Also read:
- The "Feature Scenario Mapping Table" in the project CLAUDE.md (if present)
- 1–2 existing .feature files of the same type as a coverage reference baseline

### Step 2: Produce Coverage Analysis Table

For each of the 6 scenario categories, make a judgment and output in this fixed table format. See `implementation-mindset.md` Part 3 for category definitions and entry points.

| # | Category | Applicable | Specific Scenario Direction | Gx | Reason if Not Applicable |
|---|----------|-----------|-----------------------------|----|--------------------------|
| 1 | Happy path — normal end-to-end flow | | | | |
| 2 | Error / Failure paths — anti-pattern triggers, domain error returns, external dependency failures | | | | |
| 3 | Boundary & Edge cases — theory limits, null values, oversized inputs, minimum input | | | | |
| 4 | Business rules — conditional logic, multi-condition combinations, rule exceptions | | | | |
| 5 | State mutation — DB/file write correctness, idempotency, re-run cleanup | | | | |
| 6 | Output contract — return format matches schema, shape match at boundary crossings | | | | |

**Applicable column**: fill "Yes" or "No". "No" must include a reason — never leave it blank.

After completing the table, **proceed immediately to Step 3** — do not skip.

### Step 3: Gx Completeness Check

Confirm that every Gx from the feature plan's Serves field appears at least once in the Gx column of the analysis table.

If a Gx does not appear anywhere → flag it and ask the user: is this an oversight in coverage, or does this goal genuinely require no verification in this change (with reason)?

### Step 4: Cross-Reference with Existing Features

If the project has structurally similar .feature files (e.g., other modules at the same level), list:
- Which categories those features cover
- Differences from this analysis
- Whether the differences are intentional or gaps

### Step 5: Wait for Confirmation

Present the analysis to the user and explicitly ask:

1. Are there any missing categories or scenarios?
2. Are the "Not applicable" judgments reasonable?
3. Is the Gx mapping complete?
4. Are differences from existing features acceptable?

**Do not start writing any .feature file before the user explicitly confirms.**

The final sentence of your response must clearly state: "Once confirmed, I will use `ec:gherkin` to write the .feature file. After writing, we will proceed to the Verification Ledger (Mock Boundary Review) before writing any tests."

### Step 6: Enter Gherkin Writing

After confirmation, use the ec:gherkin skill to write the .feature file. Ensure every "Applicable" category in the analysis table has at least one corresponding Scenario.

## Examples

### Example 1: Normal Flow

User says: "Start coverage analysis"

**Step 0**: Read `docs/feature-plans/task-queue.md`, extract:
- Serves: G2 (system must support up to 5 concurrent tasks)
- Error Handling Strategy: catch boundary = boundary only; domain errors = TaskNotFound, QueueFull; infrastructure errors = fail fast
- AP1 (D1): do not accept new tasks while one is incomplete; AP2 (D2): state must be persisted to DB
- Boundary Rules: data may only flow from TaskQueue to Worker, not in reverse

**Step 1**: Read OpenSpec spec.md, extract SHALL statements (task state must be written to DB) and conditional logic (reject if queue is full)

**Step 2** analysis table:

| # | Category | Applicable | Specific Scenario Direction | Gx | Reason if Not Applicable |
|---|----------|-----------|-----------------------------|----|--------------------------|
| 1 | Happy path | Yes | 5 tasks run in parallel, all succeed | G2 | |
| 2 | Error / Failure paths | Yes | AP1 triggered (QueueFull, returns structured domain error); AP2 validated (state persisted after worker crash); fail fast on DB connection failure | G2 | |
| 3 | Boundary & Edge cases | Yes | Exactly 5 concurrent (theory limit upper bound); only 1 task (minimum value) | G2 | |
| 4 | Business rules | No | | | spec.md Requirements has no conditional branch logic |
| 5 | State mutation | Yes | Task state written to DB and queryable; re-triggering does not create duplicate tasks | G2 | |
| 6 | Output contract | Yes | Schema match at TaskQueue ↔ Worker boundary | G2 | |

**Step 3**: G2 appears in all applicable categories — complete.

### Example 2: Feature Plan Not Found

Step 0 cannot find `docs/feature-plans/task-queue.md`.

Correct behavior: "There is no feature plan for this change. Run ec:feature-planning to create one before proceeding with coverage analysis." Do not continue.

### Example 3: User Wants to Skip Analysis and Write Feature Directly

User says: "Just write the .feature for me, skip the analysis."

Correct behavior: Explain the value of coverage analysis, then let the user decide:

1. User insists on skipping → respect the decision, hand off directly to `ec:gherkin`
2. User accepts the suggestion → begin Step 0, follow the full workflow

## Notes

- "Not applicable" must include a reason — never leave it blank
- If unsure whether a category applies, mark as "Needs discussion" rather than No
- Coverage analysis ensures consistency, not 100% coverage — a deliberate skip is better than an unconscious gap
- The Gx column may contain multiple IDs (one scenario serves multiple goals), separated by commas
- Overlap rules are in `implementation-mindset.md` Part 3
