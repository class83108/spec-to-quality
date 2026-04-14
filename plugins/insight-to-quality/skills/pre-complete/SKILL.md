---
name: pre-complete
description: >
  Legacy standalone verification checklist.
  In the current workflow, its checks are integrated into design-review release-gate.
  Use only when the user explicitly asks for a separate pre-complete step.
---

# Pre-Complete Verification (Legacy)

This skill is retained for backward compatibility.

## Default Policy

Use `design-review` for final verification. It already includes release-gate checks.

## If user explicitly requests standalone pre-complete

Run the same checks as design-review release-gate:
1. tests
2. lint/format
3. type check
4. git diff --stat + git status
5. finding card sync (`docs/spec-backlog/{finding-id}.md`)
6. discovery doc sync (SYSTEM_MAP/dominant-ops/goals)
7. integration gaps tracking

Present a verification table and never claim completion without evidence.
