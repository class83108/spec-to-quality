# insight-to-quality — Agent Guide

Use this plugin as a layered flow, not as a bag of unrelated skills:

1. `discovery`
2. `system-design`
3. `system-map`
4. `implementation-planning`
5. `feature-brief`
6. direct `tdd-workflow`
8. `design-review`

Optional tools:

- `gherkin-extraction` when a slice truly benefits from acceptance-spec wording

Legacy / transitional tools:

- `feature-slice`
- `spec-clarification`
- `tdd-ready-check`

This is not a rigid waterfall.
Move upward when the uncertainty belongs to a higher layer.

## Shared Anchor

For project-level implementation ordering, the shared coordination document is:

- `docs/implementation-plan.md`

For current execution work, the shared coordination document is:

- `docs/features/<feature-slug>/brief.md`

Prefer updating these documents over scattering execution state into ad hoc notes.

## Routing Heuristic

When work is blocked or assumptions move, route to the layer where the problem actually lives:

- system purpose, scope, top-level interaction, or baseline shape changed -> `discovery`
- key design decision, alternative, trade-off, or deep-dive conclusion changed -> `system-design`
- ownership, responsibility split, or seam shape changed -> `system-map`
- project stage order, enablement-vs-behavior sequencing, or exit criteria changed -> `implementation-planning`
- the current work item is still too broad or the brief is not sharp enough -> `feature-brief`
- acceptance-spec wording would materially help a user-visible slice -> `gherkin-extraction`

Do not compensate for an upstream problem with local code.

## Reference Sources

The main reference documents are:

- `references/design-decision-mindset.md`
- `references/architect-mindset.md`
- `references/implementation-mindset.md`

Use the skill-specific `SKILL.md` as the primary workflow definition.
Use these reference documents as thinking support, not as scripts.
