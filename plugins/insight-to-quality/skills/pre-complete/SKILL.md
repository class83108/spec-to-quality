---
name: ec:pre-complete
description: >
  Mandatory verification checklist before declaring completion. All verification commands
  must be run and outputs confirmed before saying "done", committing, or creating a PR.
  Prevents declaring completion without actually running verification.
  Trigger when a feature is nearly complete, the user wants to commit, create a PR, or
  says "we're about done".
  Do NOT use for: intermediate testing during development (that is part of ec:tdd-workflow),
  or simply wanting to check current test status.
---

# Pre-Complete Verification

You are about to declare work complete. Before saying "done", suggesting a commit, or creating a PR, you must complete the following verification.

## Core Principle

**Evidence before conclusions.** Do not say "should be fine" — run verification, see the output, then declare.

## Verification Commands

Refer to the Commands section of the project CLAUDE.md for the correct test, lint, and type check commands. Do not assume any specific package manager or command format.

## Checklist

Execute in order. Show actual output for every step.

### 1. Tests

Use the test command defined in the project CLAUDE.md.

- Must all PASS
- If any FAIL → stop; investigate and fix the failure

### 2. Lint & Format

Use the lint/format command defined in the project CLAUDE.md.

- Must produce zero errors
- If errors exist → fix them, then re-run tests to confirm still green

### 3. Type Check

Use the type check command defined in the project CLAUDE.md.

- Must produce zero errors (warnings are acceptable)
- If errors exist → fix them, then re-run tests to confirm still green

### 4. Change Confirmation

```bash
git diff --stat
git status
```

- List all changed files
- Confirm no files that should not be committed (.env, credentials, temp files)
- Confirm no files that should be committed are missing

### 5. OpenSpec Status (if used)

- Are all tasks in the current change marked complete?
- If there are incomplete tasks, are they intentionally left or an oversight?
- **Delta Spec Sync**: Have the current change's delta specs been synced to the `openspec/specs/` main spec?
  - If the change's specs directory has a modified spec.md, sync it to `openspec/specs/[spec-name]/spec.md`
  - The main spec should reflect the latest implementation decisions, not stay at the initial design
  - After syncing, confirm that the main spec content matches the implementation

### 5b. Discovery Document Sync (if using discovery skills)

Did implementation reveal any of the following:

- **SYSTEM_MAP needs updating**: discovered new components, incorrectly drawn boundaries, or component descriptions that no longer match implementation
- **dominant-ops needs supplementing**: hit an anti-pattern that is not documented
- **goals.md needs correction**: a goal description turned out to be inaccurate, or a non-goal needs adjustment

If yes → update the corresponding discovery document as part of this commit; mark as "updated" in the checklist.

If no → fill "no changes" and skip.

**Principle**: Discovery documents represent the team's understanding of the system; implementation validates that understanding. Lessons learned during implementation must be written back to the documents — understanding must not remain frozen at its pre-implementation state.

### 6. Integration Test Gaps

Read the `## Integration Test Gaps` section of `docs/feature-plans/{feature-name}.md` (filled in after ec:tdd-workflow Verification Ledger is complete).

#### 6a. Determine Whether Integration Tests Are Now Due

**Proactively recommend writing integration tests now** if any of the following is true:

- Integration Test Gaps has "Needs Integration Test" items, and this is the last spec for this feature (all related specs are complete)
- This change touches a junction between multiple components (e.g., A's output is consumed by B) and this data flow is marked as unverified in the Gaps
- This PR is about to be merged to main

If none of these conditions apply, proceed to 6b.

#### 6b. Confirm Gaps Are Trackable

- List all "Needs Integration Test" items in Integration Test Gaps that are not yet addressed
- Confirm each item has one of the following:
  - A corresponding integration test has been written
  - It is recorded in an issue tracker, TODO, or a future OpenSpec change
  - A manual testing checklist exists and has been run at least once
- If any item has neither integration test nor tracking record → remind the user, but do not block the commit
- If `## Integration Test Gaps` is empty → mark as "no gaps" and complete

## Output

After all checks pass, present results in a table:

| Verification Item | Status | Notes |
|-------------------|--------|-------|
| Tests | PASS (N tests) | |
| Lint | PASS | |
| Format | PASS | |
| Type check | PASS | N warnings (if any) |
| Changed files | N files | Confirmed no sensitive files |
| OpenSpec | All complete / N incomplete | |
| Delta spec sync | Synced / N/A | |
| Discovery documents | Updated / No changes / N/A | |
| Integration test gaps | N items tracked / No gaps | |

Only then may you suggest a commit or creating a PR.

## Examples

### Example 1: Normal Completion Flow

User says: "Should be about done, ready to commit."

1. Run tests → 25 tests passed → show output
2. Run lint → 0 errors → show output
3. Run type check → 0 errors, 2 warnings → show output
4. git diff --stat → list 5 files → confirm no sensitive files
5. OpenSpec → 3/3 tasks complete, delta spec synced
6. Verification Ledger → 1 tier 3 item (WebRTC signaling), tracked in issue #42 → trackable
7. Present summary table → "All checks pass, ready to commit"

### Example 2: Integration Test Due

User completes the last spec for the conversation module; the ledger has 2 tier 2 items (Gemini session setup, transcript cross-component transfer).

Correct behavior: "This is the last spec for the conversation module, and the ledger has 2 cross-component items that remain unverified. Recommend writing integration tests now: 1) FakeGeminiSession smoke test 2) Manager → Handler transcript data flow contract test. Write them now or record to an issue?"

### Example 3: Failure Found During Verification

Lint finds 2 errors.

Correct behavior: Fix → re-run tests (not just lint) → re-run lint → re-run type check → run the full checklist from the top. A fix may introduce new issues.

### Example 4: User Wants to Skip Verification

User says: "Skip it, just commit."

Correct behavior: Remind the user of the risks of skipping verification, but respect the decision. If the user insists, execute the commit but do not claim "all tests pass" in the commit message.

## If Verification Fails

- Fix the issue
- **Re-run the entire checklist from the top** (not just the failed item)
- Because a fix may introduce new issues
