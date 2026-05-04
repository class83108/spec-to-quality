# insight-to-quality — Agent Guide

Use this plugin as a layered flow, not as a bag of unrelated skills:

1. `discovery`
2. `system-design`
3. `system-map`
4. `feature-slice`
5. `spec-clarification`
6. `tdd-ready-check`
7. `gherkin-extraction` or direct `tdd-workflow`
8. `design-review`

This is not a rigid waterfall.
Move upward when the uncertainty belongs to a higher layer.

## Shared Anchor

For feature implementation work, the shared coordination document is:

- `docs/features/<feature-slug>/work-card.md`

Prefer updating the work card over scattering execution state into ad hoc notes.

## Routing Heuristic

When work is blocked or assumptions move, route to the layer where the problem actually lives:

- system purpose, scope, top-level interaction, or baseline shape changed -> `discovery`
- key design decision, alternative, trade-off, or deep-dive conclusion changed -> `system-design`
- ownership, responsibility split, or seam shape changed -> `system-map`
- the feature is still too broad or the first slice is wrong -> `feature-slice`
- surface / contract / behavior is still unclear -> `spec-clarification`
- test strategy or readiness is unclear -> `tdd-ready-check`

Do not compensate for an upstream problem with local code.

## Reference Sources

The main reference documents are:

- `references/design-decision-mindset.md`
- `references/architect-mindset.md`
- `references/implementation-mindset.md`

Use the skill-specific `SKILL.md` as the primary workflow definition.
Use these reference documents as thinking support, not as scripts.
