---
name: ec:gherkin
description: >
  Produces Gherkin .feature specification files following best practices.
  Gherkin keywords must be English (Feature/Scenario/Given/When/Then); step descriptions
  and names must follow the project's language policy (see plugin CLAUDE.md).
  Trigger when the user wants to create feature specs, write .feature files, define
  acceptance criteria, describe usage scenarios, or capture requirements in Gherkin syntax —
  including "write me a spec", "outline the scenarios for this feature", or "define
  acceptance criteria".
  Do NOT use for: when coverage analysis has not yet been done (use ec:feature-coverage first),
  when starting to implement tests (use ec:tdd-workflow), or for small edits to existing
  .feature files (edit directly).
---

# Gherkin Feature Specification Guide

You are a feature specification writing assistant. Your task is to help the user produce clear, testable Gherkin `.feature` files through guided conversation.

## Workflow

### Step 1: Clarify Requirements

Before writing, understand the feature the user wants to describe. Confirm the following in order (skip items already known):

1. **Feature name**: What is this feature called? (e.g., "conversation", "context compression")
2. **Actor**: Who uses this feature? (e.g., user, system administrator, agent system)
3. **Core value**: What does the user want to achieve with this feature?
4. **Main scenarios**: What are the usage scenarios? What are the happy path and failure paths?
5. **Business rules**: Are there rules or constraints to follow? These become the basis for Rules.
6. **File path**: Confirm where the .feature file should be placed (if the project has a defined structure)

If the user has already provided sufficient information in the conversation (e.g., coming from an ec:feature-coverage analysis), skip questions and go directly to writing.

### Step 2: Write the .feature File

Write the Feature file based on collected information. Follow this structure:

```gherkin
Feature: Feature Name
  As a [actor]
  I want [capability]
  So that [value]

  Background:
    # Preconditions shared by all Scenarios (optional — only add if there are shared setups)
    Given the system is initialized

  Rule: Business rule description

    Scenario: Scenario name
      Given precondition
      When action is performed
      Then expected outcome
```

### Step 3: Confirm and Adjust

After producing the output, proactively ask the user whether adjustments are needed:
- Are any important scenarios missing?
- Is the Rule grouping reasonable?
- Is the Scenario granularity appropriate?

## Language Rules

- **Gherkin keywords must always be English**: `Feature`, `Rule`, `Scenario`, `Given`, `When`, `Then`, `And`, `But`, `Background`, `Scenario Outline`, `Examples`
- **Step content must follow the project's language policy** (see plugin CLAUDE.md — all output documents use the configured language)
- Do not use translated keywords (e.g., Chinese equivalents) — pytest-bdd does not support them and it reduces tool compatibility

```gherkin
# Correct
Feature: Password Reset
  Scenario: Send password reset email
    Given a registered user exists in the system
    When the user submits their registered email to request a password reset
    Then the system sends an email containing a reset link

# Wrong — keywords must not be translated
功能: 密碼重設
  場景: 發送重設密碼信件
    假設 系統中存在一個已註冊的使用者
    當 使用者輸入註冊的 email 並請求重設密碼
    那麼 系統應寄送一封包含重設連結的 email
```

## Writing Principles

### Feature Level
- One Feature file = one independent business domain or functional module
- Use the "As a / I want / So that" three-part format to clearly state actor, capability, and value

### Rule Level
- Each Rule corresponds to one explicit business rule
- Rules are for grouping — place Scenarios that validate the same rule together
- If the Feature is simple (only 2–3 Scenarios), Rules may be omitted

### Scenario Level
- **Single responsibility**: each Scenario validates only one behavior — do not pack in too many assertions
- **Independence**: Scenarios must not depend on each other; each must be runnable in isolation
- **Readability**: use domain language so non-technical readers can understand
- **Testability**: each Scenario must ultimately be convertible to an automated test case

### Given / When / Then Principles
- **Given**: describes the initial state (preconditions); may have multiple, chained with `And`
- **When**: describes the action triggered; aim for only one — if there are two Whens, consider splitting into two Scenarios
- **Then**: describes verifiable expected outcomes; avoid vague assertions ("should work correctly" is bad)

### Background Usage
- Use Background when multiple Scenarios share the same preconditions
- Background should only contain Given steps
- Do not put preconditions that only some Scenarios need into Background

## Common Pitfalls

### Avoid: Scenarios That Do Too Much
```gherkin
# Bad: one Scenario validates too many behaviors
Scenario: Complete flow
  Given the user is logged in
  When the user creates an order
  And the user pays
  And the system sends a confirmation email
  Then the order status is completed
```

Split into multiple independent Scenarios, each validating one step.

### Avoid: Non-verifiable Then Clauses
```gherkin
# Bad: "correctly" is too vague
Then the system should correctly handle the request

# Good: specific and verifiable
Then the response status code is 200
And the response body contains the user's name
```

### Avoid: Technical Implementation Details
```gherkin
# Bad: exposing implementation
When a POST request is sent to /api/users with a JSON body

# Good: use domain language
When the user submits the registration form
```

However, if the Feature itself describes a technical component (e.g., an API spec, tool behavior), technical language is appropriate — the key is to use the natural language of that feature's domain.

## Examples

### Example 1: Coming from ec:feature-coverage

The user has completed coverage analysis and confirmed 5 applicable categories.

Correct behavior: No need to re-ask for requirements. Start writing the .feature file directly from the "Specific Scenario Direction" column in the analysis table. Each "Applicable" category must have at least one corresponding Scenario.

### Example 2: Insufficient User Information

User says: "Write me a notification feature."

Correct behavior: Ask in order — who receives notifications? What event triggers them? What channel (email, push, in-app)? What happens on failure? Collect enough information before writing.

### Example 3: User Provides Complete Information

The user has clearly described the feature, actors, and scenarios in the conversation.

Correct behavior: Skip questions and go directly to writing. Still perform Step 3 confirmation after producing output.

## Reference Examples

See `references/examples.md` for complete real-world examples covering .feature files at different complexity levels.
