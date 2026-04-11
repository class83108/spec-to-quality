# Implementation Mindset

This is the shared thinking framework for implementation-phase skills. Skills that make or verify
code-level decisions should reference this document: **tdd-workflow** (when declaring design decisions
in the Verification Ledger) and **design-review** (when verifying those decisions were honored).

*Scope: code-level decisions within component boundaries already defined by discovery.
If component boundaries are not yet established, complete discovery first.*

## Core Principle

**Implicit decisions are the source of most design debt.** When an implementation choice is never
stated, it cannot be reviewed, questioned, or consciously changed later. This mindset covers two
layers of design quality:

- **Part 1 — Declared Decisions**: choices that must be stated explicitly before writing code.
  These are verified at design-review time by comparing implementation to declaration.
- **Part 2 — Structural Checks**: properties that should hold regardless of declared choices.
  These are evaluated by questioning, not by checking against a declaration.

---

## Part 1: Declared Decisions

### Error Handling Strategy

The most consequential implementation decision that discovery does not prescribe. Declare this
in OpenSpec design decisions before writing Gherkin scenarios or step definitions, because it
directly shapes both error scenarios and Verification Ledger mock boundaries.

#### Why it must be declared early

- Error scenarios in `.feature` files should reflect the declared strategy, not the implementer's instinct
- Mock boundaries in the Verification Ledger are placed differently depending on where errors are caught
- design-review cannot verify a strategy that was never stated

#### The Three Decisions

Answer all three before writing tests. Record them in OpenSpec design decisions.

**Decision 1 — Catch Boundary**

Where do exceptions stop propagating within this component?

| Option | When appropriate |
|--------|-----------------|
| Boundary only | Simple components; errors are the caller's concern |
| Per-layer with re-wrapping | When each layer has a distinct error vocabulary (e.g., `DBError` → `RepositoryError` → `ServiceError`) |
| Selective | Catch only specific expected exceptions; let all others propagate |

**Decision 2 — Error Taxonomy**

Classify the errors this feature can encounter:

- *Domain errors*: expected business failures that are part of the API contract (not found,
  validation failed, duplicate, quota exceeded). Callers are expected to handle these.
- *Infrastructure errors*: failures of external dependencies (DB connection lost, API timeout,
  disk full). These indicate the system cannot fulfill the request right now.
- *Programming errors*: conditions that should never occur in correct code (assertion failures,
  unexpected `None`, type mismatches). These indicate bugs.

**Decision 3 — Recovery Strategy**

What happens when each error type is encountered?

| Error type | Typical strategy | Never do |
|------------|-----------------|----------|
| Domain error | Return structured error to caller | Retry — the result won't change |
| Infrastructure error | Fail fast / retry with backoff / degrade gracefully | Silently swallow |
| Programming error | Fail fast always, surface loudly | Catch and continue |

#### Declaration Format

In OpenSpec design decisions (spec.md body), add one block per component this change touches:

```
**Error handling — [component name]:**
- Catch boundary: [boundary only / per-layer / selective: which exceptions]
- Domain errors: [list, e.g. "TaskNotFound, DuplicateTask"]
- Infrastructure errors: [fail fast / retry N times / degrade to X]
```

#### Anti-Patterns

**Silent swallow** — `except Exception: pass`
Hides all failures. The system appears to work while silently corrupting state or dropping data.

**Over-catching** — catching `Exception` when only `ValueError` is expected
Masks unexpected failures that should surface as bugs.

**Wrong-layer catch** — catching infrastructure errors inside domain logic
Couples domain to infrastructure. Domain logic should not know about DB connection errors.

**Exception as control flow** — using `KeyError` to check dict membership in non-iterator contexts
Makes code harder to read and can interfere with unrelated error handling up the call stack.

**Undeclared strategy** — writing error handling without a stated strategy
Each layer ends up with its own ad-hoc approach, producing inconsistent behavior.

#### Relationship to Discovery

Discovery defines error handling at the *system* level via `dominant-ops.md` anti-patterns
(e.g., "state must persist across restarts"). This document covers the *implementation* of that
intent within a single component. When they conflict, the dominant-ops anti-pattern takes precedence.

---

## Part 2: Structural Checks

These properties do not require a prior declaration — they are evaluated during design-review
by questioning, not by checking against a stated intent. Use the questioning approach: surface
the issue and let the user decide, rather than issuing directives.

### Single Responsibility

A class or function should have one reason to change.

- Does this function do both IO (DB/API call) and business logic in the same body?
- If a requirement changes, how many places need to be updated?
- Could you describe what this does in one sentence without using "and"?

### Dependency Direction

Inner layers (domain/business logic) must not depend on outer layers (framework/infrastructure).

- Does any domain-layer code import from infrastructure or framework layers?
- Is the Protocol or interface defined in the layer that owns the abstraction?
- Would swapping the DB or HTTP framework require changes inside domain logic?

### Naming

Names should accurately reflect behavior — neither too vague nor too implementation-specific.

- Does the name describe *what* it does, or *how* it does it?
- Are there over-generalised names (`process_data`, `handle_request`, `utils`) that hide intent?
- Is this name consistent with the vocabulary already established in the codebase?

### Testability

Code should be easy to test without extensive setup or wide mock surfaces.

- Are there hidden dependencies (`datetime.now()`, `random`, global state) that make tests fragile?
- Does testing this unit require mocking more than one or two collaborators?
- Could you add a new test scenario without changing the production code structure?

### Consistency

New code should follow the patterns already established in the project.

- Is there a similar feature elsewhere that handles the same concern differently?
- Does this introduce a new pattern where an existing one would have served?
- Would a new team member reading this alongside the rest of the codebase find it consistent?

---

---

## Part 3: Feature Coverage Analysis

The mapping from discovery and spec artifacts to the 6 scenario categories. Use this framework
in **ec:feature-coverage** to determine what each category should contain and where to look for
the analysis starting point.

### The 6 Categories

| # | Category | Analysis starting point |
|---|----------|------------------------|
| 1 | Happy path | feature plan → Gx goal description: what does "success" look like for this goal? |
| 2 | Error / Failure paths | feature plan → Anti-patterns (each AP maps to ≥1 scenario) + Domain errors list |
| 3 | Boundary & Edge cases | feature plan → Dx theory limits (e.g. max N concurrent) + Domain errors for invalid input |
| 4 | Business rules | spec.md → Requirements with conditional logic ("if A then B, else C"), use Scenario Outline |
| 5 | State mutation | spec.md → SHALL statements about writes + feature plan Boundary Rules for what to verify |
| 6 | Output contract | spec.md + feature plan → Boundary Rules, verify shape match at boundary crossings |

### Category Definitions

**1. Happy path**
The end-to-end flow when everything works. Driven by the Gx goal description — the scenario
should demonstrate that the goal is achieved. One scenario per distinct success path.

**2. Error / Failure paths**
Failures that the system must handle explicitly. Every anti-pattern in the feature plan maps
to at least one scenario. Domain errors from the Error Handling Strategy map to scenarios
verifying the structured error response. Infrastructure errors map to scenarios verifying
fail fast or recovery behavior.

**3. Boundary & Edge cases**
Inputs or states at the limits of what the system accepts. Sources:
- Dx theory limits (e.g. "D1: max 5 concurrent tasks" → test at exactly 5 and at 6)
- Domain errors for invalid input (e.g. "empty list", "oversized payload")
- Minimum viable input (e.g. single element, empty collection)
Do not duplicate scenarios already covered by Error / Failure paths.

**4. Business rules**
Conditional logic within the feature — "if A and B then C, if A but not B then D." Source is
spec.md Requirements. Each distinct condition branch is a candidate scenario. Use
`Scenario Outline + Examples` when the same rule applies to multiple input combinations.
This category is N/A when the spec has no conditional branching.

**5. State mutation**
Correctness of writes to DB, files, or external state. Source is spec.md SHALL statements
about persistence. Key questions: is the write idempotent? what is the state after a re-run?
does a failed write leave the system in a consistent state?

**6. Output contract**
The shape of what this feature returns or emits, verified at boundary crossings. Source is
feature plan Boundary Rules + spec.md schema definitions. Verify: does the output match what
the downstream consumer expects? This is especially important when the Verification Ledger
marks a cross-component data flow as unverified.

### Overlap Rules

- A scenario that tests both an error condition AND a boundary value → classify as **Error / Failure paths**
- A scenario that tests a limit from theory limits → **Boundary & Edge cases**, not **Business rules**
- A scenario that tests conditional logic with no boundary concern → **Business rules**
- When in doubt between two categories, pick the one that better describes the *intent* of the test

---

## How Skills Use This Document

**tdd-workflow (Verification Ledger)**

Before planning mock boundaries, check: has the error handling strategy been declared in
OpenSpec design decisions? If not, prompt the user to declare it. The declared strategy determines:
- Which error paths need explicit mock scenarios (domain errors → yes; programming errors → no)
- Where mock boundaries are placed (catch boundary = where the mock cuts)

**design-review**

Part 1 — Error Handling: read the declaration from OpenSpec, then verify the implementation matches.
A strategy that was consciously chosen and consistently applied is correct even if it is not the
approach you would have chosen. A strategy that was never stated cannot be verified — mark as
"undeclared" and prompt for a declaration, then continue to Part 2.

Part 2 — Structural Checks: use the questions above to surface concerns. Do not issue directives.
Explain the principle, ask the question, let the user decide.
