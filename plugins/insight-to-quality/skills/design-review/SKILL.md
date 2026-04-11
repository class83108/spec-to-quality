---
name: ec:design-review
description: >
  Design quality verification after TDD green. Two parts: (1) verify that design decisions
  declared in OpenSpec were followed in implementation; (2) structural checks — separation of
  concerns, dependency direction, and other issues linters cannot catch, surfaced through
  questions rather than directives.
  Trigger when the user has completed TDD green, wants a code review, or says "review this"
  or "check the design".
  Do NOT use for: understanding what code does, learning syntax, before TDD green is complete
  (finish ec:tdd-workflow first), or during bug fixes.
---

# Design Review

You are entering the design review stage. **Before starting, read:**
- `references/architect-mindset.md` (view implementation through the traceability lens)
- `references/implementation-mindset.md` (verification standards and structural check criteria)

This review has two parts:

1. **Declared decisions verification**: Check whether the error handling strategy declared in OpenSpec and the discovery anti-patterns were followed in the implementation. If violations are found, point them out directly — no questions needed.
2. **Structural checks**: Separation of concerns, dependency direction, and other issues linters cannot catch — surface these through questions, not directives.

## Prerequisites

- Tests are green (all tests pass)
- Lint + type check pass (refer to the Commands section in the project CLAUDE.md)
- If either condition is not met, remind the user to complete these first

## Phase 0: Collect Declared Design Decisions

Before reviewing any code:

1. **Read `docs/feature-plans/{feature-name}.md`** to get:
   - Error Handling Strategy (Catch boundary, Domain errors, Recovery strategy)
   - Anti-patterns (with IDs)
   - Boundary Rules
   - If no feature plan is found → record as "undeclared"; mark all decisions as undeclared in Part 1

2. **Determine the review scope**:
   ```bash
   git diff --name-only HEAD~1  # or adjust to match the actual situation
   ```

## Part 1: Declared Decisions Verification

### Error Handling Strategy

Check against the three decisions declared in OpenSpec (see `implementation-mindset.md` Part 1):

| Declared Decision | Check Item |
|-------------------|-----------|
| Catch boundary | Is try/except only at the declared layer? Does it appear where it should not? |
| Domain errors | Are the declared domain errors raised correctly? Are there undeclared ones appearing silently? |
| Recovery strategy | Does the actual recovery behavior match the declaration? |

If undeclared → mark as "undeclared" and prompt to add the error handling strategy declaration to OpenSpec before continuing with Part 2.

### Discovery Alignment (when feature plan exists)

Starting from the feature plan read in Phase 0:
- Does the implementation violate any Anti-patterns from the feature plan?
- Does the implementation cross any Boundary Rules from the feature plan?
- If there is an Internals / Surface Alignment Report, were its flagged gaps accidentally introduced in this implementation?

Only raise findings if there are issues — mark OK if there are none.

## Part 2: Structural Checks

Review each of the five dimensions one by one. Detailed check questions are in `implementation-mindset.md` Part 2.

1. **Separation of concerns** — does each class/function do exactly one thing?
2. **Dependency direction** — does any inner layer depend on an outer layer?
3. **Naming semantics** — does the name accurately reflect the behavior?
4. **Testability** — is it easy to add tests? Are there hidden dependencies?
5. **Consistency** — is the style consistent with similar features in the project?

## Output Format

### Part 1: Declared Decisions Verification

| Declared Decision | Declared Content | Verification Status | Notes |
|-------------------|-----------------|---------------------|-------|
| Catch boundary | [from OpenSpec] | Complies / Violates / Undeclared | |
| Domain errors | [from OpenSpec] | Complies / Violates / Undeclared | |
| Recovery strategy | [from OpenSpec] | Complies / Violates / Undeclared | |
| Discovery alignment | anti-patterns + boundary | Complies / Violations found / N/A | |

If violations are found, list them below the table:

**Violation** `file:line` — description (declared X, but implementation does Y)

### Part 2: Structural Issues (if any)

**[Dimension] `file:line` — brief description**

> Current approach: (brief description of what exists)
>
> Direction to consider: (question, not directive)
>
> Why it's worth thinking about: (explanation of the design principle)

### Summary

| Dimension | Status | Notes |
|-----------|--------|-------|
| Error handling strategy | Complies / Violates / Undeclared | |
| Discovery alignment | Complies / Violations found / N/A | |
| Separation of concerns | OK / Issues found | |
| Dependency direction | OK / Issues found | |
| Naming semantics | OK / Issues found | |
| Testability | OK / Issues found | |
| Consistency | OK / Issues found | |

## Examples

### Example 1: Declaration Exists, Violation Found

OpenSpec declares: catch boundary = boundary only; infrastructure errors = fail fast

Review finds: `service.py:42` has `try/except Exception: logger.error(...)` without re-raise.

Correct behavior: Mark "Catch boundary: Violates" in the Part 1 table, then list below:
"**Violation** `service.py:42` — non-boundary layer catches Exception without re-raising, violating the declared boundary-only strategy."

### Example 2: No Declaration

Phase 0 reads OpenSpec but finds no error handling strategy declaration.

Correct behavior: Mark all three decisions as "Undeclared" in Part 1, and prompt: "Recommend adding an error handling strategy declaration to OpenSpec design decisions (see `implementation-mindset.md` Declaration Format)." Continue with Part 2.

### Example 3: Everything OK

Part 1 fully complies; Part 2 all five dimensions are OK.

Correct behavior: Present both tables directly. You may say "All declared decisions comply; design structure is clean."

### Example 4: Prerequisites Not Met

User says "review this" but tests have not been run yet.

Correct behavior: Remind the user — "Design review requires green tests + lint/type check passing. Recommend completing these first."

## Key Principles

- **Declaration violations → point out directly**: No question needed — just say "declared X, but implementation is Y"
- **Structural issues → questions over directives**: "This function both reads from DB and does transformation — do you think splitting it would help?"
- **Undeclared is not the same as wrong design**: Mark as "undeclared" to let the user know this decision needs to be captured
- **Don't nitpick**: Only raise issues with meaningful impact
- **Acknowledge trade-offs**: A deliberate trade-off the user consciously chose is not a review target, even if you disagree
- **Respect context**: Prototypes / spikes can be held to a lower standard; core modules should be held to a higher standard
