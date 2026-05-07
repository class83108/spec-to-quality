# insight-to-quality — Agent Guide

Use this plugin as a layered flow, not as a bag of unrelated skills:

1. `discovery`
2. `system-design`
3. `system-map`
4. `implementation-planning`
5. `feature-brief`
6. direct `tdd-workflow`
7. `design-review`

This is not a rigid waterfall.
Move upward when the uncertainty belongs to a higher layer.

## Shared Anchor

For project-level implementation ordering, the shared coordination document is:

- `docs/implementation-plan.md`

For current execution work, the shared coordination document is:

- `docs/features/<feature-slug>/brief.md`

Prefer updating these documents over scattering execution state into ad hoc notes.

## Stage Loop

Treat implementation as a stage loop:

1. `implementation-planning` chooses the current stage
2. `feature-brief` defines the current work inside that stage
3. `tdd-workflow` executes Red -> Green -> Refactor
4. `design-review` decides whether the work is done locally or needs upstream writeback

Return to `implementation-planning` when:

- a stage is complete
- the current stage's exit criteria need to change
- the stage order changes
- a new blocker changes the implementation sequence
- work that looked like `behavior` actually needs a prior `enablement` stage

Do not return to `implementation-planning` for every local code change inside the same stage.

## Routing Heuristic

When work is blocked or assumptions move, route to the layer where the problem actually lives:

- system purpose, scope, top-level interaction, or baseline shape changed -> `discovery`
- key design decision, alternative, trade-off, or deep-dive conclusion changed -> `system-design`
- ownership, responsibility split, or seam shape changed -> `system-map`
- project stage order, enablement-vs-behavior sequencing, or exit criteria changed -> `implementation-planning`
- the current work item is still too broad or the brief is not sharp enough -> `feature-brief`

Use these entry rules for new work:

- new project or new system idea -> `discovery`
- existing project, but implementation order / next stage is unclear -> `implementation-planning`
- existing stage, and you need to define the current work item -> `feature-brief`
- current brief is clear enough and you are ready to implement -> `tdd-workflow`
- implemented work needs completion / drift review -> `design-review`

Do not compensate for an upstream problem with local code.

## Reference Sources

The main reference documents are:

- `references/design-decision-mindset.md`
- `references/architect-mindset.md`
- `references/implementation-mindset.md`

Use the skill-specific `SKILL.md` as the primary workflow definition.
Use these reference documents as thinking support, not as scripts.
