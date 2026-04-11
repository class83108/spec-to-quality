---
name: ec:tdd-workflow
description: >
  TDD red-green-refactor workflow for Python projects using Gherkin + pytest-bdd.
  Enforces strict Red → Green → Refactor order — no step may be skipped or merged.
  Trigger when the user wants to start implementing a feature, enter the TDD implementation
  phase, or says "start implementation".
  Do NOT use for: writing simple scripts, fixing typos, editing config files, when no
  .feature file exists yet (use ec:feature-coverage first), or during bug fixes.
---

# TDD Workflow (Gherkin + pytest-bdd)

You are entering the TDD development workflow. This workflow is rigid — steps must be followed in strict order; no step may be skipped or merged.

## Prerequisites

- A .feature file must exist and have been through ec:feature-coverage coverage confirmation
- If no .feature file exists, stop and remind the user to complete ec:feature-coverage analysis first

## Test and Quality Commands

Refer to the Commands section of the project CLAUDE.md for the correct test, lint, and type check commands. Do not assume any specific package manager or command format.

## Workflow

### Phase 0: Verification Ledger (Mock Boundary Review)

Before writing any tests, map every SHALL statement from the spec to its verification owner.

#### Step 0: Read the Feature Plan

Read `docs/feature-plans/{feature-name}.md` and extract:
- **Error Handling Strategy** (Catch boundary, Domain errors, Recovery strategy) — the basis for mock boundary cut points
- **Anti-patterns** (AP1, AP2...) — which failure modes need explicit simulation
- **Boundary Rules** — data flow direction, affects Checkpoint B judgments

This information is used directly in Step 2 for mock boundary planning. No need to re-read the original discovery documents.

#### Step 1: Extract the SHALL List

From the OpenSpec spec.md, extract all SHALL / MUST statements. If there is no OpenSpec, infer the behaviors to verify from the Then steps in the .feature file.

#### Step 2: Plan Mock Boundaries

Start from the Anti-patterns in the feature plan — these indicate which failure modes need explicit simulation. Mock boundaries should make these modes testable rather than letting them be absorbed invisibly. Mock cut points must align with the Catch boundary declared in the feature plan.

For each SHALL, determine:

1. **Can it be verified directly with a unit test?** → Mark as unit test, plan mock boundary
2. **Does it require an external service to verify?** → Handle by tier:
   - Tier 1 (runnable locally): testcontainers / in-memory replacement → unit test
   - Tier 2 (not runnable locally, predictable output): write a Fake (not MagicMock) returning a fixed response → unit test + mark
   - Tier 3 (not runnable locally, unpredictable behavior): mark as "needs integration test"

#### Step 3: Mock Boundary Checks (Checkpoint A + B)

For each planned mock:
- **Checkpoint A**: Does this mock conceal "external service behavior" or "your own logic"? If it conceals your own logic → adjust the mock boundary; mocks should cut at the outermost layer of external dependencies
- **Checkpoint B**: For cross-component data flow (A produces → B consumes), does A's output shape equal B's expected input shape? Is there a test verifying this?
- If using `MagicMock`, a `spec=` parameter must be added to constrain the interface; otherwise prefer a custom Fake

#### Step 4: Produce the Verification Ledger

Present the complete ledger (four sections) to the user and confirm whether the content is correct (mock boundaries, tier assignments).

Conversation display format:

```markdown
# Verification Ledger — [Feature Name]

## Error Handling Strategy (from feature plan)
- Catch boundary: [boundary only / per-layer / selective]
- Domain errors: [TaskNotFound, QueueFull]
- Infrastructure errors: [fail fast / retry N / degrade to X]

## Unit Test Coverage
- SHALL xxx → mock boundary: mock [external dependency], verify [own logic]

## Needs Integration Test
- SHALL xxx
  - Reason: [why unit test cannot verify this]
  - Minimum verification: [specific actionable verification method]

## Explicitly Not Tested (with reason)
- SHALL xxx — [reason, e.g. out of scope or deferred to v2]
```

After user confirmation, if "Needs Integration Test" or "Explicitly Not Tested" sections have content, write the gaps into `docs/feature-plans/{feature-name}.md` under `## Integration Test Gaps`:

```markdown
## Integration Test Gaps

### Needs Integration Test
- SHALL xxx
  - Reason: [why unit test cannot verify this]
  - Minimum verification: [specific actionable verification method]

### Explicitly Not Tested (with reason)
- SHALL xxx — [reason, e.g. out of scope or deferred to v2]
```

If both gap sections are empty, leave `## Integration Test Gaps` empty — no content needed.

**Only proceed to Phase 1 after the user confirms the ledger and the feature plan has been updated.**

### Phase 1: Red (Write Tests, Confirm Failure)

Before writing any step definitions, read `references/implementation-mindset.md` Part 2 — specifically the **Naming** and **Testability** dimensions. Step names establish the vocabulary for this feature and will extend into production code. If a test feels hard to isolate or requires mocking too much, this is a signal of a design problem — pause and discuss before continuing.

1. **Scan existing step definitions**: Before creating any test file, scan the `step_defs/` directory and `conftest.py`:
   - List reusable step patterns (Given/When/Then sentences), fixtures, and helper functions
   - Mark which steps in the new scenarios can be reused directly and which need to be written fresh
   - **Decide placement**: Should the new step definitions go in a new `step_defs/{feature_name}_steps.py`, or be added to an existing file? Base the decision on functional cohesion — keep related steps together; avoid redefining steps across files
   - **Warning**: In pytest-bdd, if two step functions match the same pattern, the later definition silently overrides the earlier one — duplicate patterns must be avoided
2. **Create or select the target file**: Based on the decision above, create a new file or select the existing target file
3. **Write step definitions**:
   - Organize by scenario order (matching the .feature file structure) — do not group by step type
   - Keep Given/When/Then for each scenario together
   - For reused steps, write only the import — do not redefine
4. **Execute tests**: Use the test command defined in the project CLAUDE.md
5. **Confirm red**: All new tests must FAIL
   - If any test unexpectedly PASS → stop; tell the user which ones passed and discuss whether the scenario was written incorrectly or an implementation already exists
   - Show the complete failure output to the user
6. **Wait for user confirmation**: Explicitly say "Red confirmed. Ready to start implementation?"

**Do not write any production code before the user confirms the red state.**

### Phase 2: Green (Minimal Implementation to Pass Tests)

Before starting implementation, read `references/implementation-mindset.md` Part 2 — specifically the **Single Responsibility**, **Dependency Direction**, and **Consistency** dimensions. Making conscious choices during implementation is better than discovering issues in design review.

1. **Write only the minimal code to make tests pass** — no premature optimization, no extra features
2. **Run tests after completing each scenario's implementation** so the user can see progress
3. **Show the green state after all tests pass**:
   - Display the complete passing output
   - Explicitly say "Green — all N scenarios pass"

### Phase 3: Refactor (Quality Check)

1. **Lint & Format**: Use the lint/format commands defined in the project CLAUDE.md
2. **Type check**: Use the type check command defined in the project CLAUDE.md
3. **If there are errors**: Fix them and re-run tests to confirm still green
4. **Evaluate whether refactoring is needed**:
   - Does any function's cyclomatic complexity exceed 10?
   - Is there duplicated logic that can be extracted?
   - Is naming clear?
   - If refactoring is needed, re-run tests after each change to confirm still green

### Phase 4: Complete

- List all files added or modified in this session
- Confirm the final test state is green
- Remind the user that they can trigger ec:design-review for a design quality review

## Examples

### Example 1: Normal TDD Flow

User says: "Feature file confirmed, start implementing user registration."

1. Confirm `user_registration.feature` exists
2. **Verification Ledger**: Extract 6 SHALLs from spec, plan mock boundaries → 5 unit tests verify directly, 1 (actual email sending) marked as needing integration test → show ledger and ask "Content OK?" → user confirms
3. Create `test_user_registration.py`, write step definitions following the ledger's mock boundaries
4. Run tests → all FAIL → show red output → "Red confirmed. Ready to start implementation?"
5. User confirms → implement scenario by scenario, run tests after each
6. All PASS → show green → run lint + type check → fix issues → re-run tests to confirm green
7. Remind user that ec:design-review is available

### Example 2: A Test Unexpectedly Passes in Red Phase

Running tests shows 3 FAIL and 1 PASS.

Correct behavior: Stop and tell the user "Scenario X unexpectedly passed. Possible causes: 1) an existing implementation already covers this behavior, 2) the scenario is written too loosely. How should we handle this?"

### Example 3: Feature Needs Modification During Green Phase

During implementation, a Given condition in a scenario turns out to be unreasonable.

Correct behavior: Stop and tell the user "While implementing Scenario X, I found the Given condition may need adjustment — recommend going back to ec:feature-coverage to discuss." Do not silently modify the .feature file.

## Key Rules

- **Red-green-refactor order is irreversible**: Red → Green → Refactor — never skip
- **Do not write all production code at once** — implement scenario by scenario
- **Show every test result to the user** — do not just say "it passed"
- **If a .feature modification is needed during Green phase**, stop and tell the user; go back to ec:feature-coverage to discuss
