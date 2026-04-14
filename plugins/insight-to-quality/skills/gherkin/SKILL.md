---
name: gherkin
description: >
  Direct Gherkin writing helper.
  Use when the user explicitly asks for .feature writing/editing outside the standard
  spec-backlog flow, or for local refinements to an existing .feature file.
  Do NOT use as the default path for new findings (use feature-coverage first).
---

# Gherkin Feature Specification Guide

This skill writes or refines `.feature` files.

## Usage Boundary

- Default new-finding path: use `feature-coverage` (spec-to-gherkin) first
- Use this skill directly only when:
  - user explicitly requests direct gherkin writing, or
  - editing existing `.feature` files without running full coverage workflow

## Core Rules

- Gherkin keywords are always English
- Step content follows project language policy
- Scenarios should be independent and testable
- Then clauses must be verifiable and specific

## Output Pattern

```gherkin
Feature: ...
  Rule: ...

    Scenario: ...
      Given ...
      When ...
      Then ...
```

After writing, ask user whether any missing scenarios or rule grouping changes are needed.
