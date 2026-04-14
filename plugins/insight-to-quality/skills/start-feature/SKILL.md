---
name: start-feature
description: >
  Entry point when the user has a specific feature idea but does not know its scope
  relative to the existing system. Routes to the earliest missing layer, then into
  spec-backlog execution when ready.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md to exist (discovery complete).
---

# Start Feature

Scope the idea, find the earliest missing layer, and route there.

## Routing Order

1. goals
2. dominant-ops
3. SYSTEM_MAP
4. alignment coverage (internals/surface)
5. spec-backlog execution

Stop at the first missing layer.

## Key Rules

- No Gx mapping -> route to goals-discovery
- Unknown pressure shift -> route to dominant-ops
- Unknown boundary/component -> route to system-map
- Missing seam/journey alignment -> route to align-internals/align-surface
- If all ready -> route directly to spec-backlog (`index.md`) and select one `ready` item

## Handoff Format

When routing to execution, provide:
- feature summary
- Gx mapping
- affected boundary/seams
- candidate backlog row(s) in `ready`

Do not route to legacy spec flows.
