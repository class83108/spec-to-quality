# Architect Mindset

This document captures architecture and discovery principles.

It is intentionally independent of any specific workflow, skill name, file path, or template.
Use it as a thinking framework, not as a step-by-step procedure.

---

## 1. Design For Change

The quality of a design is measured by how well it absorbs change.

A design is not good because it fits today's requirements elegantly.
It is good because tomorrow's changes cause limited ripple, clear impact, and understandable tradeoffs.

When evaluating a design decision, ask:

- What kind of change is this structure trying to make easier?
- What kind of change does it make harder?
- If requirements shift, where will the pressure show up first?

If a design is optimized only for the current snapshot, it is probably too brittle.

---

## 2. Abstraction Exists To Protect Meaning

Abstraction is not about hiding detail for its own sake.
It is about protecting stable meaning from unstable detail.

A good abstraction:

- preserves what matters
- hides what changes often
- makes responsibility easier to reason about

A weak abstraction:

- hides important differences
- bundles unrelated concerns
- forces readers to guess where truth actually lives

Useful check:

- If removing a detail changes the meaning of the decision, that detail belongs at this level.
- If removing a detail does not change the meaning, it probably belongs lower.

---

## 3. Boundaries Should Follow Change And Failure

The point of a boundary is not visual neatness.
The point is to separate things that should change differently and fail differently.

Use these tests:

### Independent Change Test

Can one side change without forcing change on the other?

### Change Reason Test

Do the two sides change for different reasons?

### Failure Isolation Test

Can one side fail without corrupting the other side's state or meaning?

If a boundary fails these tests, it is probably drawn in the wrong place.

---

## 4. Responsibilities Matter More Than Containers

Architectural structure should first answer:

- who owns this responsibility?
- who owns this state?
- who decides this rule?
- who handles failure here?

Technical containers such as modules, services, layers, or directories are secondary.

Good structure is responsibility-first:

- ownership is visible
- handoffs are explainable
- change impact is easier to predict

Bad structure is container-first:

- everything looks organized
- but no one can explain where a change should begin

---

## 5. Pressure Should Shape Structure

Not every flow matters equally.
Not every pain point should influence architecture.

The flows that deserve design attention are the ones where pressure concentrates:

- high frequency
- high cost
- high failure impact
- high coordination burden
- slow or fragile recovery
- important user-visible waiting or uncertainty

The goal is not to rank activity for its own sake.
The goal is to identify which pressures should shape structure.

Ask:

- Which flows are worth protecting first?
- Which pressure would force an ownership or boundary decision?
- Which painful symptom is really caused by a deeper structural issue?

Architecture should be shaped by real pressure, not by aesthetic symmetry.

---

## 6. Distinguish Views From Ownership

Many useful ways of analyzing a system are not the same as ownership structure.

Examples of analysis views:

- input/output flow
- validation
- state transitions
- recovery behavior

These are useful for understanding the system.
But they do not automatically define the main architectural units.

Ownership structure should answer:

- who receives work
- who owns state
- who advances the workflow
- who handles recovery
- who delivers the result

Do not mistake a useful analysis lens for a stable ownership boundary.

---

## 7. Traceability Keeps Design Honest

A design decision should be explainable from upstream intent to downstream implementation.

At minimum, you should be able to answer:

- what purpose or goal this serves
- what pressure or risk justifies it
- what responsibility or boundary it affects
- what implementation choice follows from it

If a design element cannot be explained in these terms, question whether it is real structure or just accidental complexity.

Traceability is not bureaucracy.
It is a way to keep local decisions from drifting away from system intent.

---

## 8. Do Not Abstract Noise

Premature abstraction is a form of information loss.

Do not abstract details that:

- no one outside the local area cares about
- do not affect ownership or change impact
- are already well handled by existing mechanisms
- are only internal sub-steps of one stable responsibility

Good abstractions remove noise.
Bad abstractions hide reality.

If people still need the raw detail to do real work, the abstraction is probably too early or too weak.

---

## 9. Ask Better Questions Before Designing

Discovery quality depends more on question quality than answer quality.

Prefer questions like:

- What is the worst thing that happens if this fails?
- Who feels the pain first?
- What would still need to be true if the implementation changed completely?
- What would make this structurally harder to change later?
- If we remove this, does the system still make sense?

Avoid questions that invite vague wish lists without forcing tradeoffs.

Good questions pull hidden assumptions into the open.

---

## 10. Respect Earned Complexity

Some complexity is accidental and should be removed.
Some complexity is earned because the system is protecting something real.

Before simplifying, ask:

- Is this complexity protecting an important pressure?
- Is this seam preventing meaningful failure spread?
- Is this awkwardness compensating for a real external constraint?

Do not confuse unfamiliar complexity with unnecessary complexity.

At the same time, do not defend accidental mess by calling it architecture.
The burden is to explain what the complexity is buying.

---

## 11. Revisit The Architecture When Assumptions Move

Architectural thinking is not one-and-done.

Revisit the architecture when:

- the goal changes
- the real pressure shifts
- a seam no longer isolates what it should
- ownership becomes unclear
- implementation repeatedly leaks across the same boundary

Do not wait for a catastrophic rewrite moment.
Architecture should be adjusted when the assumptions behind it are no longer true.
