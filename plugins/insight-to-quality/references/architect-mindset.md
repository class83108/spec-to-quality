# Architect Mindset

This is the shared thinking framework for all discovery skills. Every skill references this document — internalize these principles before guiding the user.

## Core Belief

**Design quality = stability under change.** A good design is not one that handles today's requirements elegantly — it's one that absorbs tomorrow's requirement changes with minimal ripple. Every design decision should be evaluated through this lens.

## The Abstraction Boundary Tests

When drawing a boundary (between modules, services, stages, layers), apply all three tests. If any test fails, the boundary is likely wrong.

1. **Independent Change Test**: Can you change one side of the boundary without changing the other? If yes, boundary is correct.
2. **Change Reason Test**: Do the two sides change for *different* reasons? If yes, boundary is correct.
3. **Failure Isolation Test**: When one side fails, can the other maintain a valid state? If yes, boundary is correct.

**When to use**: splitting skills, splitting modules, defining stage boundaries, deciding what belongs in which document.

## Document Level = Abstraction Level

Each discovery document answers exactly one question. A detail belongs to the document whose answer would change if you removed that detail.

| Document | Question it answers | Change signal |
|---|---|---|
| goals.md | What must the system do / not do? | Changing it means the system's fundamental capabilities shift |
| dominant-ops.md | Where does the pressure lie? | Changing it means the optimization target shifts |
| SYSTEM_MAP.md | Where is everything, and what breaks if I change it? | Changing it means component boundaries or contracts shift |
| Implementation code | How does a single component work internally? | Changing it affects only that component |

**The "remove the number" test**: Take a sentence from goals.md and remove the numbers. If the sentence collapses, it is a goal-level statement (numbers are parameters, not the point). If the sentence still stands without numbers, the numbers belong in dominant-ops.md or NFRs.

## Dominant Operations Thinking

Not all operations are equal. Design effort should be concentrated on the operations that matter most:

```
Criticality = Frequency x Cost x Failure Impact
```

- **Frequency**: How often does this operation run?
- **Cost**: How much time/compute/money does each run consume?
- **Failure Impact**: What is the worst-case consequence of this operation failing silently?

Failure impact is the most commonly underestimated factor. A low-frequency operation with silent data corruption can be far more critical than a high-frequency, harmless operation.

**Top 3 rule**: Identify at most 3 dominant operations. If everything is dominant, nothing is — push the user to make hard choices.

## Traceability

Every design decision must trace back to a source:

```
Goal (Gx) --> Dominant Op (Dx) --> Boundary/Contract --> Implementation
```

If a design element cannot be traced to a goal or dominant op, question its existence. If a goal has no downstream design element, it is either aspirational (move to Open Questions) or missing implementation (flag as a gap).

## Design for the Hardest Constraint

Let the hardest constraint shape the abstraction boundary — do not spread abstraction evenly across all concerns. If GPU cold-start time is the binding constraint, the entire stage boundary design should revolve around that, not around aesthetic symmetry.

## What NOT to Abstract

Premature abstraction is as harmful as premature optimization. Do not abstract:

- **Internal sub-steps** of a single operation (e.g., TTS batch, verify, retry-single is an internal concern)
- **Persistence details** when the framework already handles them well (e.g., Django ORM does not need a repository pattern wrapper)
- **Admin/management UI** when the framework provides it (e.g., Django Admin does not need a ViewModel layer)

The test: "Is anyone outside this boundary asking about this detail?" If no, do not abstract it.

## Anti-Pattern Discipline

When documenting anti-patterns, every anti-pattern must reference a specific dominant operation it protects:

- Good: `AP2 (protects D1, D2, D3): DB is the single source of truth between stages`
- Bad: `AP2: Don't use in-memory state` (too vague, no traceability)

## The Questioning Hierarchy

When guiding users through discovery, the quality of questions matters more than the quality of answers:

1. **"What is the worst that happens if this fails?"** — better than "Is this important?"
2. **"Can you think of 2-4 different ways to achieve this?"** — tests whether the goal is at the right abstraction level
3. **"If we remove this, does the system still make sense?"** — separates must-haves from nice-to-haves
4. **"Will this still be true in 6 months?"** — filters out ephemeral concerns from stable goals

## Working with Existing Code (Verify Mode)

When the system already has code, discovery is not greenfield design — it is alignment verification:

- **Read before judging**: Understand what exists and why before declaring misalignment
- **Trace, do not redesign**: Check whether existing code traces back to goals and dominant-ops, do not propose a new architecture
- **Gaps over rewrites**: Surface what is missing or misaligned, do not suggest rewriting what works
- **Respect earned complexity**: If code is complex, check whether a dominant-op justifies that complexity before simplifying
