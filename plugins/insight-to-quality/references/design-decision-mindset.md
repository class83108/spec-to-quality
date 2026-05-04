# Design Decision Mindset

This document captures how to reason about early design decisions before the work moves into
component mapping, ownership structure, or detailed specs.

It is written from the perspective of a **system design interviewer** evaluating whether a candidate
can move from requirements to a credible initial design, explain the important choices, and reason
about trade-offs without hiding behind vague architecture language.

Use it as a thinking framework, not as a rigid script.

It should be read alongside a practical system design interview flow:

1. clarify requirements
2. define the contract or API shape
3. sketch a high-level baseline design
4. deep dive into the few decisions or bottlenecks that matter most

This document is strongest at steps 3 and 4.

---

## 1. Start From A Baseline, Not From Abstractions

A strong design discussion does not begin with words like:

- scalability
- flexibility
- robustness
- clean architecture

It begins with a **baseline system shape**:

- what goes in
- what comes out
- what the main flow looks like
- what the top-level API or interaction model is

An interviewer wants to see whether you can first sketch a plausible system before defending it.

Useful check:

- If there is no baseline flow or API sketch, decision discussion is probably premature.

Important nuance:

- In many interviews, you do **not** need to fully resolve every important decision before drawing a
  high-level design.
- It is often stronger to sketch a plausible baseline first, then use deep dives to improve or
  revise it.

---

## 2. Good Early Design Work Surfaces Decisions

Requirements do not directly become architecture.
They first become a set of **important decisions**.

Examples:

- static vs dynamic redirect
- synchronous vs accepted-then-complete-later
- append-only history vs overwrite-in-place
- preprocess before recognition vs recognize directly
- generate on demand vs precompute

Interviewers look for whether the candidate can notice:

- where alternatives exist
- which alternatives materially change the system
- which choices are merely local details

If every choice is treated as equally important, design thinking is still shallow.

Interviewers often prefer to see:

- the top 2-4 decisions that define the system shape
- not a long list of every possible implementation choice

---

## 3. A Decision Is Only Real If Options Exist

Saying:

- "we use Redis"
- "we use 302"
- "we use a queue"

is not yet design reasoning.

A real decision includes:

- the question being answered
- the main options
- the current choice
- the reason for that choice

Useful check:

- If you cannot clearly say "instead of what?", you probably do not have a real decision yet.

---

## 4. Explain Why This Choice Fits *This* Problem

Interviewers are not mainly testing whether you know common patterns.
They are testing whether you can connect a choice to the current problem.

Weak answer:

- "302 is better because it is more flexible."

Stronger answer:

- "We need owners to update or delete the mapping after QR creation, so we prefer 302 because each
  scan should come back through our server and reflect current state."

The standard is not pattern recall.
The standard is **contextual justification**.

Useful prompts:

- What requirement does this choice protect?
- What would break if we chose the alternative?
- Why is this trade-off acceptable here?

---

## 5. Trade-Offs Matter More Than Correctness Theater

Interviewers do not expect perfect solutions.
They expect visible trade-off reasoning.

A credible answer should show:

- what the chosen path improves
- what it makes worse
- what risk it introduces
- why that risk is acceptable for now

Examples:

- choosing dynamic redirect improves mutability and analytics, but increases operational dependency
- choosing append-only history improves traceability, but makes projection and cleanup harder
- choosing preprocessing improves downstream quality, but adds latency and compute cost

If a design choice sounds like pure upside, it is probably under-examined.

In interviews, saying "I would start with this because it is simpler for version 1, and revisit if
X becomes dominant" is often stronger than pretending the first answer is universally optimal.

---

## 6. Not Every Important Topic Is A Structural Topic

Some decisions belong in early design reasoning.
Some belong later in structural mapping.

Early design decision topics usually sound like:

- why this processing order?
- why this API shape?
- why sync vs async?
- why this redirect behavior?
- why keep history?

Structural mapping topics usually sound like:

- who owns this state?
- where does truth live?
- where should the seam be?
- what is the blast radius of failure?

Do not jump into ownership decomposition before the major design choices are visible.

This is especially important in interviews:

- if you decompose components too early, you may consume time before showing that the baseline
  design even satisfies the core requirements
- if you deep dive too early, you may never complete the high-level picture

---

## 7. Deep Dive Means Expanding Consequences, Not Just Deferring Confusion

A common mistake is to label every unresolved thought as a "deep dive."

In strong system design reasoning, a deep dive usually means:

- a decision already appears important
- its second-order consequences deserve focused expansion

In interviews, deep dives are often triggered by:

- the most important non-functional requirements
- the likely bottleneck in the baseline design
- a place where scale, latency, consistency, or availability pressure will force refinement

Examples:

- after choosing 302, deep dive into caching, CDN, and redirect latency
- after choosing append-only history, deep dive into projections and cleanup
- after choosing preprocessing, deep dive into cost, accuracy, and fallback behavior
- after sketching a feed system, deep dive into fanout-on-write vs fanout-on-read
- after sketching a read-heavy service, deep dive into cache invalidation and replica strategy

This is different from:

- "we have not thought about it yet"

Unclear or missing thinking should usually be recorded as an open question, not upgraded into a deep dive.

---

## 8. The Goal Is A Defensible Baseline, Not A Final Answer

In early design discussion, the job is not to prove that the first architecture is perfect.
The job is to produce a baseline that is:

- plausible
- discussable
- revisable

Interviewers reward candidates who can say:

- "Here is the baseline I would start with."
- "Here are the 2-3 decisions that define it."
- "Here are the trade-offs."
- "Here are the areas I would deep dive next."

That is stronger than pretending the first draft is already complete.

In other words:

- high-level design is often the first draft
- deep dive is where pressure tests happen
- revision is a sign of maturity, not weakness

---

## 9. Show Judgment About What Deserves Attention

One of the clearest signs of maturity is selectivity.

A good candidate does not deep dive every topic.
They identify:

- which few decisions define the system's shape
- which trade-offs are dangerous enough to surface now
- which details can wait

Useful check:

- If the discussion has ten decisions and all seem equally important, prioritization is missing.

Interviewers are looking for:

- signal detection
- prioritization
- scope control

They are also watching whether you can keep the conversation moving:

- requirements -> API / contract -> baseline design -> targeted deep dives

without getting trapped in a single local problem too early.

---

## 10. A Good Decision Memo Makes System Mapping Easier

The output of early design reasoning should make later structure work easier.

After the important decisions are clear, the next layer can more confidently ask:

- what responsibilities exist because of these choices?
- what state must now be owned carefully?
- what seams must protect the chosen trade-offs?

If later structural mapping still has to guess what the key design choices were, early design work was incomplete.

But note:

- in interview settings, you may not explicitly reach a formal "system map"
- the same principle still applies: later structural explanation should not have to guess what the
  baseline design and major decisions were

---

## 11. Questions An Interviewer Hopes You Can Answer

If your design discussion is strong, you should be able to answer questions like:

- What is the baseline design you are proposing?
- What are the 2-4 decisions that matter most?
- What alternatives did you consider?
- Why did you reject them?
- What trade-offs are you consciously accepting?
- Which topic would you deep dive next, and why?
- Which non-functional requirement is most likely to force your baseline design to change?

If these answers are visible, the design conversation is usually on solid ground.

---

## 12. Red Flags

Interviewers are cautious when they hear:

- abstract quality words with no baseline design
- chosen technologies with no alternatives discussion
- decisions presented as pure upside
- deep dives used as a label for missing thought
- premature component decomposition before the system shape is clear
- confident answers that do not mention risk
- spending too long on one deep dive before a baseline design exists
- listing many requirements or many decisions without prioritization

These are usually signs that the reasoning is not yet mature enough.
