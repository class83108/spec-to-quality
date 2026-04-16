# Implementation Mindset

Shared thinking framework for implementation-phase skills.
Primary consumers: `spec-to-gherkin`, `tdd-workflow`, `design-review`.

Scope: code-level decisions within boundaries already defined by discovery docs.

## Core Principle

Implicit decisions create long-term design debt. Every important choice must be:
1. declared
2. traceable
3. verifiable

---

## Part 0: Derivation Rules (Spec Backlog)

This part defines how to convert discovery/alignment information into executable implementation specs.

### 0.1 Index Row Field Derivation

For `docs/spec-backlog/index.md` fields:

- `serves`:
  - source: `goals.md`
  - method: map finding outcome to concrete Gx objective
  - no direct mapping => keep `draft` and mark `needs-goal-clarification`

- `related`:
  - source: `dominant-ops.md`
  - method: map finding impact chain to Dx/APx
  - order by criticality `D1 > D2 > D3`

- `boundary`:
  - source: `SYSTEM_MAP.md` boundary/component names
  - missing name => update SYSTEM_MAP first; do not invent

- `priority`:
  - score = failure impact x frequency x cost
  - convert to dominant-op bucket (`D1/D2/D3`) for queue ordering

- `type`:
  - source: producing skill
  - `skeleton` — from `spec-contract` or `spec-surface`; covers contract/schema/API shape gaps
  - `feature` — from `spec-behavior`; covers functional behavior/user journey gaps
  - determines Gherkin anchor and test path (see Part 3)

- `tests_path`:
  - derive from `type`
  - `skeleton` → `tests/features/contracts/`
  - `feature` → `tests/features/behaviors/`

- `status`:
  - `draft` default
  - `ready` only if `serves/related/boundary/done-criteria-seed/type` all present
  - `in-progress` only when WIP=1 condition passes

### 0.2 Finding Card Field Derivation

For `docs/spec-backlog/{finding-id}.md`:

- `type` and `tests_path`:
  - copy directly from index row (see 0.1)
  - do not invent or override; if missing, resolve in index first

- `Behavior (SHALL/MUST)`:
  - source priority: alignment finding statement -> boundary constraints -> discovered implementation gaps
  - must be observable and testable

- `Done Criteria`:
  - every item must map to at least one scenario and one Then assertion
  - ban vague wording ("better", "optimized", "properly handled")
  - for `skeleton` type: done criteria anchors on schema validity and contract enforcement
  - for `feature` type: done criteria anchors on user-observable behavior and system response

- `Error Handling Strategy`:
  - must declare catch boundary, domain errors, infrastructure recovery before test writing

### 0.3 WIP Discipline

- multiple `draft/ready` allowed
- at most one `in-progress`
- newly discovered gaps during execution go to `draft`; do not interrupt active item unless user cancels it

---

## Part 1: Declared Decisions

### Error Handling Strategy

Declare before writing test steps.

#### Three Decisions

1. Catch Boundary
- boundary only / per-layer wrapping / selective catch

2. Error Taxonomy
- domain errors
- infrastructure errors
- programming errors

3. Recovery Strategy
- domain: structured return
- infrastructure: fail fast / retry with backoff / degrade
- programming: fail fast

#### Declaration Format (in finding card)

```markdown
## Error Handling Strategy
- Catch boundary: ...
- Domain errors: ...
- Infrastructure recovery: ...
```

#### Anti-Patterns

- Silent swallow (`except Exception: pass`)
- Over-catching (`except Exception` for expected narrow failures)
- Wrong-layer catch (domain layer handling infra exceptions directly)
- Exception-driven control flow without intent
- Undeclared strategy

#### Relationship to Discovery

System-level anti-patterns in `dominant-ops.md` override local implementation convenience.

---

## Discovery Conflict Triage

When code conflicts with plan, classify impact level first.

| Level | Signal | Fix starting point | Example |
|---|---|---|---|
| 0 — Code only | No test expectation changes | code only | rename/refactor with same behavior |
| 1 — Finding spec implementation | Behavior/details change, boundaries unchanged | finding card -> TDD from Red | algorithm change, internal data structure change |
| 2 — Contract or boundary | Data crossing shape/boundary changes | SYSTEM_MAP -> index/card -> TDD | seam split/merge, producer-consumer mismatch |
| 3 — Goal or dominant-op | Purpose/pressure ranking changes | goals/dominant-ops -> downstream cascade | missing goal, dominant-op reprioritization |

### Cascade Rule

- Level 3: goals/dominant-ops -> SYSTEM_MAP -> spec-backlog -> TDD
- Level 2: SYSTEM_MAP -> spec-backlog -> TDD
- Level 1: finding card -> TDD
- Level 0: code only

Do not skip intermediate documents.

---

## Part 2: Structural Checks

Used by design-review.

1. Single responsibility
2. Dependency direction
3. Naming semantics
4. Testability
5. Consistency

Use questions to surface tradeoffs; use direct findings only for clear violations.

---

## Part 3: Feature Coverage Analysis

Read finding card `type` first — it determines the Gherkin anchor and scenario focus.

### Skeleton type (契約/schema finding)

Happy path anchor: "valid input shape → contract passes"

| # | Category | Analysis starting point |
|---|---|---|
| 1 | Happy path | valid schema passes boundary without rejection |
| 2 | Error / Failure paths | invalid shape, missing required field, wrong type |
| 3 | Boundary & Edge cases | empty value, max-length field, optional field absent |
| 4 | Business rules | field constraint rules (enum values, format rules) |
| 5 | State mutation | schema version change does not break existing consumers |
| 6 | Output contract | error response shape matches declared contract |

### Feature type (功能行為 finding)

Happy path anchor: "使用者完成行為 → 系統正確回應"

| # | Category | Analysis starting point |
|---|---|---|
| 1 | Happy path | finding card `Serves` (Gx) + end-to-end success behavior |
| 2 | Error / Failure paths | finding card `Error Handling Strategy` + `Related` APx |
| 3 | Boundary & Edge cases | finding card limits/constraints + dominant-op theory limits (if linked) |
| 4 | Business rules | finding card `Behavior` conditional clauses |
| 5 | State mutation | finding card write-related Behavior + Done Criteria |
| 6 | Output contract | finding card boundary/output expectations |

### Overlap Rules

- error + boundary => classify as Error/Failure
- theory limit cases => Boundary/Edge
- pure branch logic => Business rules
- when ambiguous, classify by primary test intent

---

## How Skills Use This Document

- `spec-contract` / `spec-surface`: Part 0 field derivation and WIP discipline; produce `skeleton` type findings
- `spec-behavior`: Part 0 field derivation; produce `feature` type findings
- `docs-governance`: Part 0 priority/dependency derivation for scoped sync/route/archive checks
- `spec-to-gherkin`: Part 0 + Part 3; reads finding card `type` to select skeleton or feature coverage table
- `tdd-workflow`: Part 1 declarations + triage levels; test file goes to `tests_path` from finding card
- `design-review`: Part 1 compliance + Part 2 structure + release gate evidence
