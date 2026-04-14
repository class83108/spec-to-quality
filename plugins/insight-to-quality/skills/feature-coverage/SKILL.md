---
name: feature-coverage
description: >
  Spec-to-Gherkin conversion for one finding card.
  Merges coverage analysis and Gherkin writing:
  validate coverage completeness first, then write the .feature file.
---

# Spec-to-Gherkin (Coverage + Writing)

Two mandatory phases:
1) coverage gate
2) gherkin writing

Before starting, read:
- `references/implementation-mindset.md` Part 0 (criteria derivation)
- `references/implementation-mindset.md` Part 3 (6 category definitions)

## Prerequisites

- `docs/spec-backlog/{finding-id}.md` exists
- index row for this finding is `in-progress`

If either condition fails, stop.

## Required Outputs

- [ ] Extract card fields: Serves, Related, Boundary, Behavior, Error Strategy, Done Criteria
- [ ] 6-category coverage table (`Yes` or `No + reason`)
- [ ] Done Criteria -> scenario mapping table
- [ ] User confirmation before writing `.feature`
- [ ] `.feature` file written
- [ ] Post-check: all `Applicable=Yes` categories and all Done Criteria covered

## Derivation Rules (How to Fill Coverage)

1. Category 1/2/3 source priority
- first from finding card: Serves + Related + Error Strategy
- then from dominant-ops constraints (if referenced in card)

2. Category 4/5/6 source priority
- first from Behavior (SHALL/MUST)
- then from Boundary + Done Criteria

3. Done Criteria mapping
- each Done Criteria must map to >=1 Scenario and >=1 Then assertion
- if a criterion cannot map, mark `spec gap` and request card fix before writing

## Workflow

### Step 0: Read Inputs

Read:
- `docs/spec-backlog/{finding-id}.md`
- project CLAUDE.md Feature Scenario Concrete Mapping Table (if present)
- 1-2 similar existing `.feature` files (if present)

### Step 1: Coverage Table

| # | Category | Applicable | Specific Scenario Direction | Gx | Reason if Not Applicable |
|---|----------|-----------|-----------------------------|----|--------------------------|
| 1 | Happy path — normal end-to-end flow | | | | |
| 2 | Error / Failure paths — anti-pattern triggers, domain error returns, external dependency failures | | | | |
| 3 | Boundary & Edge cases — theory limits, null values, oversized inputs, minimum input | | | | |
| 4 | Business rules — conditional logic, multi-condition combinations, rule exceptions | | | | |
| 5 | State mutation — DB/file write correctness, idempotency, re-run cleanup | | | | |
| 6 | Output contract — return format matches schema, shape match at boundary crossings | | | | |

Rules:
- Applicable must be `Yes` or `No`
- `No` requires reason
- every Gx in `Serves` appears at least once

### Step 2: Done Criteria Mapping

| Done Criteria | Planned Scenario(s) | Then assertion target |
|---|---|---|
| DC-1 | Scenario A, Scenario B | Then ... |

If any Done Criteria has no mapping, revise before writing.

### Step 3: Confirmation Gate

Ask for explicit confirmation. Do not write `.feature` before confirmation.

### Step 4: Write Gherkin

- one finding -> one `.feature` file
- keywords in English
- step text in project language policy

### Step 5: Post-check and Sync

- ensure every `Applicable=Yes` category has scenario coverage
- ensure every Done Criteria has Then coverage
- if gaps exist, patch `.feature` immediately
- write short coverage summary back into finding card notes
