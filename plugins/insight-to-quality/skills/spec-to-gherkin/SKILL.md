---
name: spec-to-gherkin
description: >
  Finding card to Gherkin conversion.
  Reads finding card type + finding-id prefix to route the correct gherkin guide reference,
  runs coverage analysis (6 categories), then writes the .feature file.
---

# Spec-to-Gherkin (Coverage + Writing)

> **Output contract**：繁體中文。Coverage table、Done Criteria mapping、確認訊息、finding card notes 更新皆使用繁體中文。Gherkin keywords 維持英文，step text 依專案語言政策。

Two mandatory phases:
1) coverage gate — validate what to test and at which level
2) gherkin writing — produce the `.feature` file

Before starting, read:
- `../../references/implementation-mindset.md` Part 0 (criteria derivation)
- `../../references/implementation-mindset.md` Part 3 (6 category definitions, skeleton vs feature tables)

## Prerequisites

- `docs/spec-backlog/{finding-id}.md` exists with `type` and `tests_path` fields
- The corresponding `spec-xxx` report main-table row status is `in-progress`
  - `contract-*` → `docs/contracts/contracts-report.md`
  - `surface-*` → `docs/surface/surface-report.md`
  - `behavior-*` → `docs/behaviors/behavior-report.md`

If either condition fails, stop.

## Type Routing

Read the finding card `type` and `finding-id` prefix to determine:

| `type` | `finding-id` prefix | Gherkin guide reference | Test path |
|--------|---------------------|------------------------|-----------|
| skeleton | `contract-` | `./references/contract-gherkin-guide.md` | `tests/features/contracts/` |
| skeleton | `surface-` | `./references/surface-gherkin-guide.md` | `tests/features/contracts/` |
| feature | `behavior-` | `./references/feature-gherkin-guide.md` | `tests/features/behaviors/` |

The guide reference determines scenario anchor, patterns, boundary rules, and anti-patterns.
The test path comes from the finding card `tests_path` field (do not hardcode).

Route validation rules:
- `contract-*` and `surface-*` must have `type: skeleton`
- `behavior-*` must have `type: feature`
- if prefix/type mismatch occurs, stop and request finding card correction before writing

## Required Outputs

- [ ] Finding card `type` and `finding-id` prefix identified
- [ ] Correct gherkin guide reference read
- [ ] Extract card fields: Serves, Related, Boundary, Behavior, Error Strategy, Done Criteria
- [ ] 6-category coverage table (`Yes` or `No + reason`) using the correct analysis starting point table
- [ ] Test level assignment for each applicable category (`unit` / `integration`)
- [ ] Done Criteria → scenario mapping table
- [ ] User confirmation before writing `.feature`
- [ ] `.feature` file written to `tests_path`
- [ ] Scenario level tags include `@unit` or `@integration` (derived from Owning Test Level)
- [ ] Post-check: all `Applicable=Yes` categories covered, all Done Criteria mapped

## Derivation Rules (How to Fill Coverage)

### Source priority by category

**Skeleton findings** (contract / surface):
- Category 1–3: finding card Boundary + Error Strategy → contract/surface shape constraints
- Category 4–6: finding card Behavior (SHALL/MUST) + Done Criteria → field constraints and error response shape

**Feature findings** (behavior):
- Category 1–3: finding card Serves (Gx) + Error Strategy + Related APx
- Category 4–6: finding card Behavior conditional clauses + Done Criteria + State Transitions

### Done Criteria mapping

- each Done Criteria must map to ≥1 Scenario and ≥1 Then assertion
- if a criterion cannot map, mark `spec gap` and request card fix before writing

---

## Workflow

### Step 0: Read Inputs

Read based on type routing:

**All types:**
- `docs/spec-backlog/{finding-id}.md`
- project CLAUDE.md Feature Scenario Concrete Mapping Table (if present)
- 1–2 similar existing `.feature` files in the same `tests_path` (if present)

**Skeleton (contract):**
- `./references/contract-gherkin-guide.md`

**Skeleton (surface):**
- `./references/surface-gherkin-guide.md`

**Feature:**
- `./references/feature-gherkin-guide.md`

### Step 1: Coverage Table

Use the analysis starting point table matching the finding `type` (from `implementation-mindset.md` Part 3).

**Skeleton type — analysis starting points:**

| # | Category | Analysis starting point |
|---|----------|------------------------|
| 1 | Happy path | valid schema passes boundary without rejection |
| 2 | Error / Failure paths | invalid shape, missing required field, wrong type |
| 3 | Boundary & Edge cases | empty value, max-length field, optional field absent |
| 4 | Business rules | field constraint rules (enum values, format rules) |
| 5 | State mutation | schema version change does not break existing consumers |
| 6 | Output contract | error response shape matches declared contract |

**Feature type — analysis starting points:**

| # | Category | Analysis starting point |
|---|----------|------------------------|
| 1 | Happy path | finding card `Serves` (Gx) + end-to-end success behavior |
| 2 | Error / Failure paths | finding card `Error Handling Strategy` + `Related` APx |
| 3 | Boundary & Edge cases | finding card limits/constraints + dominant-op theory limits (if linked) |
| 4 | Business rules | finding card `Behavior` conditional clauses |
| 5 | State mutation | finding card write-related Behavior + Done Criteria |
| 6 | Output contract | finding card boundary/output expectations |

**Coverage table format:**

| # | Category | Applicable | Test Level | Specific Scenario Direction | Gx | Reason if Not Applicable |
|---|----------|-----------|------------|-----------------------------|----|--------------------------|

Rules:
- Applicable must be `Yes` or `No`
- `No` requires reason
- if `Applicable=Yes`, `Test Level` is mandatory
- every Gx in `Serves` appears at least once

### Test level decision rules

| Level | When to use |
|-------|------------|
| `unit` | behavior can be verified inside one component with no real boundary crossing |
| `integration` | behavior requires crossing at least one real boundary (DB/file/queue/external adapter/API seam) |

**Skeleton findings** default to `unit` (schema validation is typically in-process).
Upgrade to `integration` when the handoff crosses a real I/O boundary (e.g., DB schema, file format, queue message).

**Feature findings** default to `integration` (business behavior usually crosses boundaries).
Downgrade to `unit` when the behavior is pure in-process logic with no I/O.

### Step 2: Done Criteria Mapping

| Done Criteria | Planned Scenario(s) | Then assertion target | Owning Test Level |
|---|---|---|---|
| DC-1 | Scenario A, Scenario B | Then ... | integration |

If any Done Criteria has no mapping, revise before writing.
If any Done Criteria has no `Owning Test Level`, revise before writing.

### Step 3: Confirmation Gate

Ask for explicit confirmation. Do not write `.feature` before confirmation.

Present:
1. Coverage table
2. Done Criteria mapping
3. Which gherkin guide reference is being used
4. Target file path (`tests_path/{finding-id}.feature`)

### Step 4: Write Gherkin

- one finding → one `.feature` file
- file location: finding card `tests_path` + `{finding-id}.feature`
- keywords in English
- step text in project language policy
- feature-level tag: `@contract @{finding-id}` (skeleton) or `@behavior @{finding-id}` (feature)
- scenario-level tag: each Scenario must include exactly one of `@unit` / `@integration`, matching Step 2 `Owning Test Level`
- follow the scenario patterns and anti-patterns from the routed gherkin guide reference

### Step 5: Post-check and Sync

- ensure every `Applicable=Yes` category has scenario coverage
- ensure every `Applicable=Yes` category has valid `Test Level`
- ensure every Done Criteria has Then coverage
- ensure every Done Criteria has `Owning Test Level` (`unit`/`integration`)
- ensure every Scenario has exactly one level tag (`@unit` or `@integration`)
- ensure Scenario level tags are consistent with Step 2 mapping
- if gaps exist, patch `.feature` immediately
- write short coverage summary back into finding card notes
