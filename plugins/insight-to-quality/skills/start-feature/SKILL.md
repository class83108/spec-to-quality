---
name: start-feature
description: >
  Entry point when the user has a specific feature idea but does not know its scope
  relative to the existing system. Routes to the earliest missing layer, then into
  spec-xxx execution when ready.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md to exist (discovery complete).
---

# Start Feature

Scope the idea, identify change type, and route to the earliest missing layer.

## Core Decision Principle

Do not assume every new request starts from skeleton.
Route by **what is missing for this specific feature slice**:

- if existing skeleton boundaries already cover the slice -> go directly to `spec-behavior`
- if behavior depends on missing internal contract boundary -> go to `spec-contract` first
- if behavior depends on missing external interface boundary -> go to `spec-surface` first
- if the request introduces a new goal (new Gx) -> go back to discovery layer first

## Routing Order (Earliest Missing Layer)

1. goals
2. dominant-ops
3. SYSTEM_MAP
4. skeleton coverage (`spec-contract` / `spec-surface`)
5. behavior coverage (`spec-behavior`)

Stop at the first missing layer.

## Feature Triage (mandatory)

For each incoming feature idea, classify it into one of these buckets:

1. **Behavior-only extension under existing boundaries**
- signal:
  - maps to existing Gx
  - SYSTEM_MAP boundary/component unchanged
  - required contract/surface shapes already exist
- route: `spec-behavior`

2. **Same goal, but missing boundary protection**
- signal:
  - maps to existing Gx
  - behavior needs a new or changed handoff schema -> internal gap
  - or behavior needs a new/changed external API/CLI/event shape -> surface gap
- route:
  - internal gap first -> `spec-contract`
  - external gap first -> `spec-surface`
  - after skeleton ready -> `spec-behavior`

3. **New goal / scope expansion beyond current goals**
- signal:
  - cannot map to existing Gx without stretching meaning
  - changes project intent, not only implementation details
- route:
  - `goals-discovery` first
  - then re-evaluate dominant-ops / SYSTEM_MAP deltas
  - then enter `spec-contract` / `spec-surface` / `spec-behavior`

## Key Rules

- No Gx mapping -> route to `goals-discovery`
- Existing Gx but unknown pressure shift -> route to `dominant-ops`
- Existing Gx but unknown boundary/component ownership -> route to `system-map`
- Boundary known but contract/surface gaps exist:
  - internal handoff gap -> `spec-contract`
  - external interface gap -> `spec-surface`
- If skeleton coverage is sufficient for the slice -> route directly to `spec-behavior`
- Never route by habit to skeleton first; route by missing layer for the slice

## Read Set for Decision

Minimum files to inspect before routing:
- `goals.md` (Gx mapping)
- `dominant-ops.md` (pressure shift check)
- `SYSTEM_MAP.md` (boundary/component ownership)
- `docs/contracts/contracts-report.md` (internal skeleton coverage)
- `docs/surface/surface-report.md` (external skeleton coverage)
- `docs/behaviors/behavior-report.md` (existing behavior coverage)

## Handoff Format

When routing, provide:
- feature summary
- Gx mapping result (`existing Gx` or `new Gx needed`)
- affected boundary/components
- missing layer rationale (why this route is earliest)
- next skill and expected output

Examples:
- `route -> spec-behavior`: "Extends an existing G2 flow; contract/surface coverage already exists, only behavior semantics slice is missing."
- `route -> spec-contract`: "Existing G3 flow needs a new Pipeline->Evaluator handoff schema; add internal skeleton coverage first."
- `route -> goals-discovery`: "The request cannot map to existing Gx; this is a new goal."

## Guardrail

Do not route to legacy align/index flows.
Use report-row driven workflow (`contracts/surface/behaviors` reports + finding cards).
