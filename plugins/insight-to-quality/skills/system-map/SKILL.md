---
name: system-map
description: >
  Guide the creation of SYSTEM_MAP.md — a navigation map for development that synthesizes goals
  and dominant-ops into a Component Map, Boundary Map, and Change Protocol. This is the single
  living document that developers and AI agents consult to understand where things are, what
  depends on what, and what breaks when something changes.
  Requires goals.md and dominant-ops.md to exist.
  Do NOT use for: defining goals (use goals-discovery), analyzing pressure (use dominant-ops),
  defining internal data handoff contracts (use spec-contract), or defining external interface shapes (use spec-surface).
---

# System Map

> **Output contract**：繁體中文。SYSTEM_MAP.md 的所有內容（Overview、Component Map、Boundary Map、Architecture Decisions、Change Protocol）皆使用繁體中文。

You are guiding the user through the creation of **SYSTEM_MAP.md** — the living navigation document for development. Think of it as a department manager: it knows the big picture, points to details, tracks progress, and coordinates changes. It is NOT a design document — it is a map.

Read `../../references/architect-mindset.md` before proceeding, especially Document Level = Abstraction Level and The Abstraction Boundary Tests.

## Working Style

- **Map, not territory.** SYSTEM_MAP.md should fit in your head in 30 seconds. If you are writing paragraphs of explanation, you have crossed into design-doc territory. Write pointers, not prose.
- **Every cell is a link.** Components, boundaries, and contracts should all point to the actual files or documents where details live. The map tells you where to look, not what you will find.
- **This is a living document.** Unlike goals.md and dominant-ops.md (which are mostly write-once), SYSTEM_MAP.md is updated with every significant change. Design it for maintainability, not completeness.
- **Boundaries are the most valuable part.** Developers rarely ask "what components exist?" — they ask "if I change X, what else breaks?" The Boundary Map and Change Protocol answer this question directly.

## Required Outputs

Before declaring this skill complete, you MUST produce ALL of the following. Do not write SYSTEM_MAP.md before every item is checked:

- [ ] System Overview (3–5 lines)
- [ ] Tech Stack Decisions: every "TBD — carry to system-map" from dominant-ops resolved or explicitly deferred with rationale
- [ ] Component Map: table (with Tech column) + Mermaid diagram (with tech annotations) — **the Mermaid diagram is required, not optional**
- [ ] Boundary Map: at least 2 seams, each with Architecture Decision (driven by / decision / rationale / technology)
- [ ] Current State section (even if mostly gaps — write what is known; do not omit for new projects)
- [ ] Change Protocol covering all 4 types
- [ ] Phase 8 validation checklist completed (all three checks must pass before writing the document)

**N/A Policy**: Sections that don't yet have content (e.g., Lessons on first creation) must contain a comment explaining why: `<!-- Empty at initial creation — populated by design-review as features are completed -->`. Never silently omit a section.

## Prerequisites

- **goals.md must exist** — provides the Gx IDs and system purpose
- **dominant-ops.md must exist** — provides the Dx IDs, anti-patterns, and design implications
- If either is missing, redirect to the appropriate skill first

## Workflow

### Phase 1: System Overview

Write a 3-5 line overview that lets someone understand what this system does in 30 seconds. Include:

1. **Purpose**: One sentence from goals.md's System Purpose
2. **Key numbers**: From dominant-ops.md's theory limits (e.g., "processes ~N items/day", "serves ~N users")
3. **Tech stack summary**: From constraints (e.g., "Django + PostgreSQL + Celery + Redis")
4. **Current phase**: Where is the project in its lifecycle? (bootstrap, active development, production, maintenance)

### Phase 2: Tech Stack Decisions

Consume the Design Implications from dominant-ops.md. For each Dx, read:
- **Characteristic tags** — understand the pressure profile
- **Technical requirements** — understand what the pressure demands
- **Dimension decisions** — identify which are confirmed and which are "TBD — carry to system-map"

#### Step 1: Resolve TBD items

For each "TBD — carry to system-map" dimension, make a concrete technology decision. The agent presents the context (Dx tags + technical requirements + the question that was deferred), and the user decides.

Record each decision in this format:

```
[dimension] for Dx:
  decision:   [concrete technology choice]
  rationale:  [why this choice, referencing Dx pressure]
  source:     dominant-ops Design Implications → Dx
```

If the user still cannot decide (e.g., needs prototyping first), record as "deferred — requires [specific action]" with a concrete next step, not an open-ended deferral.

#### Step 2: Cross-Dx consolidation

Multiple Dx dimensions may point to the same technology area (e.g., D1 needs caching, D2 needs caching). Consolidate into a unified tech stack list:

```
Tech Stack:
  Database:     [choice] — serves D1 (reliability), D2 (state persistence)
  Cache:        [choice] — serves D1 (read performance)
  Task Queue:   [choice] — serves D2 (isolation), D3 (batch processing)
  Protocol:     [choice] — serves D2 (real-time feedback)
  ...
```

Each entry must reference which Dx drives the choice. If a technology serves no Dx, question whether it is needed.

#### Step 3: Constraint compatibility check

Cross-check tech decisions against goals.md constraints (Cx). Flag conflicts immediately — e.g., "chose Redis but C4 says no additional infrastructure beyond PostgreSQL."

### Phase 3: Component Map

List every major component and visualize their relationships. A "component" is an independently deployable or independently changeable unit.

#### Component Table

For each component:

| Column | Content |
|---|---|
| Component | Name (linked to source directory or file) |
| Responsibility | One sentence — what it does |
| Owns | What data or state this component is the authority for |
| Tech | Key technology from Phase 2 decisions (e.g., "FastAPI", "Celery worker", "PostgreSQL") |
| Status | Active / Planned / Deprecated |

#### Component Diagram

Use a Mermaid diagram to visualize component relationships and data flow. This gives a visual overview that the table alone cannot provide.

```mermaid
graph TD
    subgraph "User-Facing Layer"
        A["Web UI (React)"] --> B["API Gateway (FastAPI)"]
    end
    subgraph "Business Logic"
        B --> C[Service A]
        B --> D[Service B]
        C --> E["Worker Queue (Celery)"]
    end
    subgraph "Data Layer"
        C --> F[("Database (PostgreSQL)")]
        D --> F
        E --> G["Cache (Redis)"]
    end
```

Adapt the diagram to the actual system. Add tech annotations from Phase 2 decisions in parentheses after component names — this makes technology choices visible at the diagram level. Keep it to one level of depth — nested subgraphs are fine for grouping, but do not diagram individual classes or methods. The diagram should answer: "What are the major moving parts, what technology powers them, and how do they connect?"

**Granularity guide**:
- Too coarse: "Backend" — this is meaningless, what specifically?
- Too fine: "UserSerializer" — this is a single class, not a component
- Right level: "Authentication module", "Order processing service", "Report generator"

The right granularity is: a unit that a developer can own, change, and deploy independently.

**Guiding question**: "If a new developer joins the team, what are the 8-15 'buckets' they need to understand?"

### Phase 4: Boundary Map

This is the most valuable section. Identify the key boundaries (seams) in the system — places where components interact through contracts.

For each boundary:

1. **Name the boundary** and label it (Seam A, Seam B, Seam C...)
2. **What connects**: Which components on each side?
3. **Contract type**: Schema, API endpoint, event, shared DB table, file format
4. **Architecture Decision**: Structured four-line format (see below)
5. **Change impact**: What breaks if this contract changes?
6. **Where to look**: Link to contract definition (schema file, API spec, interface definition)

#### Architecture Decision format

Every seam must include an Architecture Decision block that traces from pressure to technology:

```
driven by:  Dx（[pressure description]）
decision:   [architectural decision]
rationale:  [why this decision, referencing Dx pressure]
technology: [concrete technology choice, or "TBD — requires [action]"]
```

Example:
```
Seam C: Pipeline orchestration -> Stage execution
  driven by:  D2 (human coordination, high failure cost)
  decision:   Separate queues and workers for interactive vs long-running tasks
  rationale:  D2 response time must not be starved by D3 long-running work
  technology: Celery — dedicated `interactive` + `batch` queues
```

The `technology` field may be left as TBD during initial creation if the team needs prototyping first, but must include a concrete next step for resolution. A seam with no `driven by` reference should be questioned — if no Dx pressure drives this boundary, is it a real seam or an over-split?

Apply the three abstraction boundary tests to each seam:
- Independent Change: Can you change one side without the other?
- Change Reason: Do the two sides change for different reasons?
- Failure Isolation: Can one side fail without corrupting the other?

If a boundary fails any test, it may be drawn in the wrong place. Discuss with the user.

Optionally, visualize boundary relationships with a Mermaid diagram:

```mermaid
graph LR
    A[Component X] -- "Seam A: schema contract" --> B[Component Y]
    B -- "Seam B: protocol/adapter" --> C[External Service]
    D[Orchestrator] -- "Seam C: command" --> A
    D -- "Seam C: command" --> B
```

**Common boundary patterns**:
- Between processing stages (input/output schemas)
- Between internal logic and external services (protocol/adapter)
- Between orchestration and execution (command/worker)
- Between user-facing layer and business logic (controller/service)

### Phase 5: Current State

Track project progress at a high level:

1. **Phase progress**: What major milestones are done, in progress, and planned?
2. **In-flight work**: What is currently being developed? (link to specs or branches)
3. **Known gaps**: What is missing compared to goals.md? (link to issues or specs)

This section is updated frequently — keep it scannable.

### Phase 6: Lessons

A lightweight section that captures implementation pitfalls relevant to future work touching the same boundary or component. This section is **not written during initial SYSTEM_MAP creation** — it is populated incrementally by design-review's Lessons Capture step as features are completed.

Each entry is a one-line summary with a link to the detailed record in the spec-backlog finding card. SYSTEM_MAP is the navigator; the spec-backlog finding is the source of truth.

**What belongs here**: Mid-size pitfalls that would affect someone working on the same boundary or component in the future. Examples: a contract needing unexpected nullable handling, an anti-pattern that was harder to obey than expected, a technical limitation that forced a design compromise.

**What does NOT belong here**: Small pitfalls local to a single change (stay in the finding card), or large discoveries that trigger a discovery revision (handled by the Discovery Conflict Triage in implementation-mindset.md).

### Phase 7: Change Protocol

This is the section developers and AI agents consult most. Define what to do when something changes, organized by impact radius.

#### Type 1: Goal Change (largest impact)

When a goal in goals.md changes or a new goal is added:
1. Review dominant-ops.md — does the pressure ranking change?
2. Review SYSTEM_MAP boundaries — do any seams need to move?
3. Create a spec-backlog finding for the implementation work
4. Update SYSTEM_MAP after implementation

#### Type 2: Contract/Boundary Change (medium impact)

When a schema, API contract, or interface changes:
1. Identify all components on both sides of the boundary
2. Update producer and consumer simultaneously (or version the contract)
3. Update tests on both sides
4. Update SYSTEM_MAP boundary entry

#### Type 3: Internal Component Change (smallest impact)

When changing logic inside a single component without affecting its contract:
1. Verify the output contract is unchanged
2. Update internal tests
3. No SYSTEM_MAP update needed (unless status changes)

#### Type 4: New Component Addition

When adding a new component to the system:
1. Define its contracts with existing components (what seams does it touch?)
2. Add to Component Map (table and diagram)
3. Add new boundaries to Boundary Map
4. Update orchestration if applicable

**For each type, the key question is: "What else do I need to touch?"** The Change Protocol answers this before the developer starts coding, preventing "changed A, forgot to update B" problems.

## Infrastructure Escalation Intake

When spec-contract, spec-surface, or spec-behavior discover an infrastructure concern during their workflow, they do **not** open a finding card. Instead, they escalate back to system-map with a note: `escalate to SYSTEM_MAP`.

When receiving an escalation:

1. **Read the escalation context**: Which spec skill flagged it? What infrastructure decision is missing? Which Dx/Gx is affected?
2. **Determine the right Boundary Map seam**: Does this concern belong to an existing seam, or does it require a new one?
3. **Add or update the Architecture Decision**: Fill in the four-line format (driven by / decision / rationale / technology) for the affected seam.
4. **Update Component Map if needed**: If the decision introduces a new component (e.g., a message queue, a cache layer), add it to the table and diagram.
5. **Return to the spec skill**: After the Architecture Decision is recorded, the spec skill can resume its workflow.

This is the single mechanism for infrastructure decisions. Finding cards are for contract and behavior gaps — infrastructure gaps are resolved in SYSTEM_MAP.

### Phase 8: Review and Validate

**You MUST complete all three checks before writing SYSTEM_MAP.md.** Present each result to the user before proceeding.

1. **Navigation test** — Pick a random goal from goals.md. Trace it through the Component Map and Boundary Map to the relevant source files. Can you do this within 3 steps? If not → identify and fill the gap before writing.
2. **Change simulation** — Pick a likely future change (e.g., "swap storage implementation", "add a new CLI command"). Walk through the Change Protocol. Does it tell the developer everything they need to touch? If not → refine the protocol before writing.
3. **Newcomer test** — Could a developer who has never seen this codebase use SYSTEM_MAP.md to orient themselves in under 5 minutes? If not → simplify or restructure before writing.

If any check reveals a problem → fix it first. Do not write SYSTEM_MAP.md until all three pass.

## Output Shape

```markdown
# SYSTEM_MAP — [System Name]

## Overview
[3-5 lines: purpose, key numbers, tech stack, current phase]

## Tech Stack

| Technology | Role | Driven by |
|---|---|---|
| [e.g., PostgreSQL] | [e.g., primary database] | D1, D2 |
| [e.g., Redis] | [e.g., cache + task broker] | D1, D2 |
| ... | | |

## Component Map

| Component | Responsibility | Owns | Tech | Status |
|---|---|---|---|---|
| [linked name] | [one sentence] | [data/state] | [technology] | Active |
| ... | | | | |

[Mermaid diagram showing component relationships with tech annotations]

## Boundary Map

### Seam A: [Name]
- **Connects**: [Component X] <-> [Component Y]
- **Contract**: [type + link to definition]
- **Architecture Decision**:
  - driven by: Dx（[pressure description]）
  - decision: [architectural decision]
  - rationale: [why]
  - technology: [concrete choice or "TBD — requires [action]"]
- **Change impact**: [what breaks]

### Seam B: [Name]
...

[Optional: Mermaid diagram showing boundary relationships]

## Current State
- **Phase**: [current phase description]
- **In-flight**: [linked list of active work]
- **Gaps**: [linked list of known gaps]

## Lessons
<!-- Populated by design-review Lessons Capture; empty at initial creation -->
- [Seam/Component]: [one-line summary] ([docs/spec-backlog/FINDING-ID.md])

## Change Protocol
[Type 1-4 as described above]
```

## Design Checks

Revisit this document if:
- A new boundary is discovered during implementation that is not on the map
- The Change Protocol fails to predict a ripple effect during a real change
- A developer asks "where is X?" and the map cannot answer
- Component count grows beyond 20 — consider grouping into subsystems

## Examples

### Example 1: Boundary Fails the Three Tests

User proposes a boundary between "API handlers" and "database queries."

Apply the tests:
- Independent Change: Can you change the API handler without changing the DB query? Often no — they are tightly coupled through the data shape.
- This boundary may be drawn at the wrong level. A better boundary might be between "request handling" and "business logic", where the business logic defines its own data interface.

### Example 2: Map Becomes Territory

User starts writing detailed class diagrams and method signatures in SYSTEM_MAP.md.

Stop them: "This level of detail belongs in the code itself or in a design doc. SYSTEM_MAP should point to these files, not reproduce them. If someone needs to know the method signatures, they can follow the link. The map's job is to tell them which file to open."

### Example 3: Change Protocol Catches a Gap

During a real change (adding a caching layer), the developer walks through Type 4 (New Component). The protocol asks "what seams does it touch?" — the developer realizes the cache invalidation strategy affects the contract between the data layer and the API layer. Without the protocol, they might have added the cache and discovered stale data bugs in production.

### Example 4: Component Granularity Negotiation

User lists 25 components for a mid-size web application.

Push back: "25 components means no one can hold the map in their head. Can we group related components? For example, 'user auth', 'user profile', 'user preferences', and 'user notifications' might all be 'User Management' at the map level, with a link to a sub-map if needed. Aim for 8-15 top-level components."

## Key Rules

- **Pointers, not prose.** Every entry in the map should be 1-2 lines maximum. If you need to explain something at length, write it in a separate document and link to it.
- **Boundaries are the answer to "what breaks?"** If the Boundary Map cannot answer "I changed X, what else is affected?", it is incomplete.
- **Change Protocol is the most-used section.** Invest the most effort here. The rest of the map is context for the protocol.
- **Update discipline matters more than initial quality.** A perfect map that goes stale is worse than a rough map that stays current. Design for easy updates.
- **Do not duplicate goals.md or dominant-ops.md.** Reference them by ID (Gx, Dx). If you find yourself re-explaining a goal, you are at the wrong abstraction level.
- **Mermaid diagrams are navigation aids.** Keep them simple — one level of depth, labeled edges, grouped subgraphs. If the diagram needs a legend, it is too complex.
