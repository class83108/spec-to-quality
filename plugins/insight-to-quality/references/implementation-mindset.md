# Implementation Mindset

This document captures implementation-phase principles.

It is intentionally independent of any specific workflow, skill name, file path, or template.
Use it as a way to think about testing, execution, refactoring, and review.

---

## 1. Clarify Before You Test

Testing is not a substitute for clarity.

Before writing tests, be able to explain:

- what slice of work is being implemented
- what outcome matters
- what risk the test is supposed to protect
- where responsibility and seams are involved

If the team would still need to guess while writing tests, the work is not ready for implementation.

Testing unclear work does not create safety.
It only creates expensive confusion.

---

## 2. Test The Main Risk

Do not choose test type by habit.
Choose it by the main uncertainty that needs protection.

Ask:

- Is the risk local logic?
- Is the risk seam or handoff correctness?
- Is the risk user-visible behavior?
- Is the risk subjective or experiential quality?

The test strategy should protect the most important failure mode first.

If the chosen tests do not address the main risk, they are decoration, not protection.

---

## 3. Use The Lowest Honest Test Layer

Prefer the lowest layer that can honestly expose the uncertainty.

### Unit-level testing

Best for:

- local rules
- deterministic transformations
- helper behavior
- narrow in-process state decisions

### Integration-level testing

Best for:

- seams and handoffs
- persistence behavior
- adapters
- component coordination

### Acceptance-level testing

Best for:

- user-visible or externally visible behavior
- important cross-boundary outcomes
- high-value business rules with observable results

### Manual validation

Best for:

- perceived responsiveness
- UI clarity
- media or content quality
- temporary validation where automation is not yet economical

Using a higher test layer than necessary slows feedback.
Using a lower test layer than necessary hides the real risk.

---

## 4. Red Should Expose Real Uncertainty

Red is not "write any failing test."
Red should expose the uncertainty that matters.

That uncertainty may live at different layers:

- local logic
- seam behavior
- acceptance outcome

A good Red test fails for a reason that matters.

A weak Red test:

- fails for setup noise
- asserts something too small to matter
- avoids the actual risk because it is harder to test honestly

If the test passes immediately, ask whether:

- the behavior is already covered
- the assertion is too weak
- the test is pointed at the wrong layer

---

## 5. Green Should Be Minimal

Green means the smallest implementation change that resolves the targeted uncertainty.

It does not mean:

- expanding the feature
- performing broad cleanup
- adding speculative abstractions
- solving adjacent problems "while you are here"

If Green keeps growing, the slice is probably too large or the real risk was not isolated clearly enough.

---

## 6. Refactor Is About Removing Drift

Refactor is not cosmetic cleanup.
It is where you remove distortion introduced during Green.

Refactor should improve:

- duplication
- naming
- locality of knowledge
- responsibility alignment
- test clarity

Refactor is also where you check whether new implementation knowledge ended up in the wrong place.

---

## 7. Watch For Repeated Knowledge

Repeated knowledge is a strong signal of future drift.

Look for:

- duplicated helper logic
- duplicated constants or enums
- duplicated rule meaning in tests and implementation
- repeated payload or fixture shapes
- multiple places encoding the same seam assumption

The problem is not repetition by itself.
The problem is when multiple places now need to change together and nothing says which one is the source of truth.

---

## 8. Watch For Responsibility Overload

A function, helper, or module is overloaded when it carries multiple unrelated reasons to change.

Typical signals:

- validation, mapping, rule logic, and I/O mixed together
- helper code quietly holding business meaning
- tests that need large unrelated setup to reach a small behavior
- orchestration code also making domain decisions

When responsibility overload appears, local complexity rises fast and change safety drops.

---

## 9. Smells Are Signals, Not Verdicts

Code smells and test smells are not proofs of bad design.
They are signals that deserve inspection.

Useful code smell signals:

- long functions
- mixed abstraction levels
- hidden side effects
- temporal coupling
- primitive-heavy seams

Useful test smell signals:

- brittle setup
- over-mocking
- weak assertions
- scenario sprawl

The point is not to mechanically eliminate every smell.
The point is to ask what structural weakness the smell might be revealing.

---

## 10. Refactor Locally, Escalate Structurally

Some problems should be fixed in place.
Some problems are evidence that an upstream assumption is wrong.

Local problems:

- naming
- small duplication
- helper extraction
- setup simplification
- minor responsibility cleanup inside an already-correct owner

Structural problems:

- the slice was wrong
- the seam was wrong
- ownership is unclear
- the real pressure is different from what was assumed
- the chosen test strategy no longer matches the actual risk

Do not hide structural problems inside local refactor.
Escalate them upward.

---

## 11. Review Alignment, Not Just Green Tests

A green test run is necessary but not sufficient.

A real implementation review should ask:

- Did the code stay inside the intended slice?
- Did it preserve seam assumptions?
- Did the actual tests protect the intended risk?
- Was manual validation done where it mattered?
- Is deferred coverage still visible?
- Did implementation expose changes that need upstream writeback?

Review should confirm alignment between:

- intended work
- actual code
- actual test protection
- remaining risk

---

## 12. Keep Deferred Risk Visible

Not every important risk will be automated immediately.
That is acceptable.

What is not acceptable is letting deferred coverage disappear from view.

If a risk is deferred, it should remain explicit:

- what is not yet protected
- why it is not yet protected
- what kind of protection is expected later

Invisible risk is much more dangerous than acknowledged risk.

---

## 13. Write Back When Assumptions Move

Implementation is allowed to discover that the plan was incomplete.

When that happens, the right response is not to silently patch reality in code.
The right response is to update the layer where the assumption actually lives.

Ask:

- Did the intended behavior turn out to be unclear?
- Did the seam turn out to be different?
- Did the ownership model change?
- Did the real design pressure change?
- Did the system purpose change?

The answer tells you where to write back.

This is how implementation stays connected to architecture instead of gradually drifting away from it.

---

## 14. Worked Examples

These examples make two recurring anti-patterns concrete. Both come from real routing and risk-framing mistakes that look reasonable in the moment but create downstream drift.

### Example A — Goal Alignment Does Not Guarantee Routing

**Scenario.** A transcription system has a goal to keep transcription cost predictable, and a design driver about high cost-of-failure. The cache currently stores `{audio-id: full-transcript}` only — successful results, never partial state. A user requests: "When transcription fails mid-run, resume from the failure point instead of restarting."

**Tempting routing.** The request aligns with both the goal and the design driver. Treat it as a normal feature slice and proceed.

**Hidden problem.** The request quietly requires the cache (or a new component) to store something it does not currently store: partial results plus a progress marker. That is a seam-shape change, not a feature implementation.

**Better routing.** Enter `feature-slice` as the intake gate, but route up to `system-map` as soon as the slice surfaces the seam reshape. The work is structural before it is functional.

**Lesson.** Goal and design-driver alignment is necessary but not sufficient. Before continuing into spec or TDD, ask: *can the current system map hold this request without reshaping a seam?* If the answer is no, the work is structural before it is functional, and the routing should reflect that.

---

### Example B — Risk Is "What Bad Outcome", Not "What We Need"

**Scenario.** Same resume feature. Asked to state the main risk this slice should protect.

**Weak framing.** "We don't know where transcription got to, so we need to track progress."

**Why it is weak.** This is a solution shape disguised as a risk. It assumes "progress tracking" is the answer before naming the bad outcome. Tests written from this framing tend to verify that progress tracking exists, not that the resumed transcript is correct or that cost was actually saved.

**Better framing (any one of these is a real risk):**

- "Resumed transcript may contain duplicated or missing audio segments."
- "Resume may silently re-run already-transcribed audio, defeating the cost goal."
- "Repeated resume attempts may corrupt or accumulate partial state."

Each follows the shape *"under condition X, bad observable outcome Y can occur."* That shape lets the test scenarios target the outcome, not the chosen mechanism.

**Lesson.** Risk lives in the problem space. The moment a risk statement names a mechanism (`we need to track`, `we need to store`, `we need to add`), it has moved to the solution space and the test scenarios will inherit that bias. Keep risk framed as observable failure, and let mechanism choice happen later.
