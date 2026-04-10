---
name: ec:align-surface
description: >
  Design or verify surface-layer alignment — user-facing interfaces and infrastructure shaped by
  dominant-ops pressure and constraints. Interfaces include APIs, web UI, CLI, admin panels, SSE
  endpoints — whatever the system exposes to its users. Infrastructure covers deployment topology,
  worker configuration, and capacity planning.
  In design mode (no existing code), guide interface and infrastructure design from dominant-ops
  user journeys. In verify mode (existing code), audit coverage and capacity.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md to exist.
  Do NOT use for: defining goals (use ec:goals-discovery), analyzing pressure (use ec:dominant-ops),
  mapping system structure (use ec:system-map), or designing contracts/schemas (use ec:align-internals).
---

# Align Surface (Interfaces and Infrastructure)

You are guiding the user through **surface-layer alignment** — ensuring that user-facing interfaces and infrastructure choices are shaped by dominant-ops pressure, not by framework defaults or habit.

This skill covers two related concerns:
1. **Interfaces**: APIs, web UI, CLI, admin panels, webhooks, SSE/WebSocket endpoints — everything users or external systems interact with
2. **Infrastructure**: Deployment topology, worker/process configuration, capacity planning — the runtime environment that supports the interfaces

This skill operates in two modes:
- **Design mode**: No existing code. Guide the user to design interfaces and plan infrastructure from dominant-ops user journeys.
- **Verify mode**: Existing code. Audit whether interfaces cover all dominant-ops user journeys and whether infrastructure can handle the pressure.

Determine the mode by asking: "Is there existing interface code (routes, views, endpoints), or are we designing from scratch?"

Read `shared/references/architect-mindset.md` before proceeding, especially Dominant Operations Thinking and Design for the Hardest Constraint.

## Working Style

- **User journeys, not endpoint lists.** Do not start by listing endpoints. Start by asking: "What does the user need to do for D1? D2? D3?" Each journey produces interface requirements naturally.
- **Non-goals eliminate options.** Non-goals from goals.md are more useful than goals for interface design. NG "no mobile app" eliminates responsive-first design. NG "no multi-tenant" eliminates tenant isolation concerns. Let non-goals shrink the design space before expanding it.
- **Constraints decide technology.** Do not choose technology based on preference. C1 says "Django" — then it is Django. C3 says "must support SSE" — then evaluate whether the web server supports long-lived connections. Technology questions are constraint-satisfaction problems, not taste problems.
- **Classify every endpoint.** Every interface element falls into one of three categories. The category determines its error handling pattern, and getting this wrong causes subtle production bugs.

## The Three Endpoint Categories

Every user-facing endpoint belongs to exactly one of these categories:

### Trigger (Write, Async)
- **Purpose**: Initiate a state change or background operation
- **Pattern**: Accept request, validate, enqueue work, return acknowledgment
- **Key concern**: Prevent duplicate submissions (idempotency keys, disable-on-click, redirect-after-POST)
- **Error handling**: Return validation errors synchronously; processing errors arrive later via status polling or push notification
- **Examples**: "Start processing", "Submit order", "Trigger report generation"

### Query (Read)
- **Purpose**: Retrieve current state or computed data
- **Pattern**: Accept request, read from DB/cache, return data
- **Key concern**: Safe to retry, safe to cache, safe to call multiple times
- **Error handling**: Return error directly, caller can retry
- **Examples**: "Get order status", "List recent items", "View dashboard"

### Push (Server-initiated, Real-time)
- **Purpose**: Stream updates from server to client
- **Pattern**: Client opens long-lived connection, server pushes events
- **Key concern**: Reconnection strategy — what happens when the connection drops? Client must handle reconnect and replay missed events.
- **Error handling**: Connection-level (heartbeat, timeout, reconnect), not request-level
- **Examples**: "Progress bar via SSE", "Live notification stream", "Real-time collaboration updates"

**Guide the user to classify every endpoint.** Once classified, the error handling pattern follows naturally. Mixing categories (e.g., a trigger that also streams results) creates complexity — split it into two endpoints instead.

## Prerequisites

- **goals.md** — for non-goals (design space constraints) and NFRs (quality requirements)
- **dominant-ops.md** — for D1/D2/D3 user journeys and theory limits
- **SYSTEM_MAP.md** — for component boundaries and seams
- If any are missing, redirect to the appropriate skill

## Workflow — Design Mode

### Phase 1: User Journey Mapping

For each dominant operation (D1, D2, D3), map the user's journey:

1. **Who is the user?** (operator, admin, external system, automated scheduler)
2. **What do they need to do?** (trigger a process, review output, check progress, configure settings)
3. **What interface do they need?** (button, form, API call, dashboard, real-time feed)
4. **What is the expected response time?** (immediate, seconds, minutes — drives sync vs async)

**Do not invent journeys.** If a user action does not serve a dominant operation, it is either low-priority (design it minimally) or it should not exist (check against non-goals).

### Phase 2: Interface Design

From the user journeys, derive the interface requirements:

1. **List required endpoints/screens** grouped by journey
2. **Classify each** as Trigger, Query, or Push
3. **Define the interaction pattern**: What does the user see? What do they click? What feedback do they get?
4. **Apply the abstraction level test**: "If we swapped the frontend framework (e.g., HTMX to React, or CLI to web), would this description still work?" If yes, you are at the right level. If no, you are too specific.

### Phase 3: Infrastructure Planning

Based on interface requirements and dominant-ops pressure:

1. **Process topology**: What processes need to run? (web server, background workers, scheduler, cache)
2. **Concurrency model**: How many concurrent users/requests must the system handle? (derive from D1/D2/D3 frequency)
3. **Long-lived connections**: Does the system need SSE, WebSocket, or similar? If yes, the web server must support async connections — evaluate compatibility with the tech stack constraint.
4. **Worker sizing**: How many background workers are needed? (derive from D1 processing time x expected throughput)

**Capacity estimation template**:
```
D1 runs N times/day, takes ~M minutes each.
With W workers, max concurrent D1 operations = W.
Max throughput = W x (60/M) per hour = X/hour.
To handle N/day, need at least ceil(N / (X * active_hours)) workers.
```

This is a rough estimate — flag it for validation with real load testing.

### Phase 4: Deployment Configuration

Based on process topology:

1. **What services need to run?** (web server, worker pool, database, cache, reverse proxy)
2. **How do they communicate?** (HTTP, message queue, shared DB, in-memory)
3. **What are the scaling axes?** (add more workers? add more web processes? scale the database?)
4. **What monitoring is needed?** (which metrics would tell you D1 is degrading before users notice?)

### Phase 5: Validate Against Constraints

Cross-check the design against goals.md constraints and NFRs:

1. **Budget constraint**: Does the infrastructure cost fit within the budget? (C-x)
2. **Performance NFR**: Can the designed infrastructure meet the latency/throughput targets? (NFR-x)
3. **Tech stack constraint**: Does the chosen deployment model work with the mandated technology? (C-x)
4. **Security**: Do all trigger and query endpoints have appropriate authentication and authorization?

## Workflow — Verify Mode

### Phase 1: Inventory Existing Interfaces

Read the codebase to catalog all user-facing interfaces:
- URL routes and view functions
- API endpoints and their HTTP methods
- Admin panel registrations
- WebSocket/SSE endpoints
- CLI commands
- Scheduled tasks (cron, periodic tasks)

### Phase 2: Journey Coverage Audit

For each D1/D2/D3 user journey from dominant-ops.md:
- Can the user complete this journey using existing interfaces?
- Are there missing steps? (e.g., user can trigger D1 but cannot monitor its progress)
- Are there unnecessary steps? (e.g., user must visit 3 pages to do what should take 1)

### Phase 3: Endpoint Classification Audit

For each existing endpoint:
- Is it clearly a Trigger, Query, or Push?
- Does it have the correct error handling pattern for its category?
- Are Trigger endpoints protected against duplicate submissions?
- Do Push endpoints handle reconnection?

### Phase 4: Infrastructure Capacity Audit

- **Worker count**: Are there enough workers to handle D1/D2/D3 at expected frequency?
- **Web server**: Can it handle the expected concurrent connections (especially if Push endpoints exist)?
- **Bottlenecks**: Does any single resource (DB connection pool, worker count, API rate limit) cap throughput below the required level?

### Phase 5: Gap Report

```markdown
# Surface Alignment Report — [System Name]

## Journey Coverage
| Journey (Dx) | Status | Gaps |
|---|---|---|
| D1: [name] | covered / partial / missing | [what is missing] |
| D2: [name] | ... | ... |
| D3: [name] | ... | ... |

## Endpoint Classification
| Endpoint | Category | Issues |
|---|---|---|
| [path] | Trigger/Query/Push | [missing idempotency / no reconnect / etc.] |

## Infrastructure
- Worker capacity: [sufficient / insufficient] — [details]
- Connection handling: [sufficient / insufficient] — [details]
- Monitoring: [present / missing] — [what to add]

## Recommendations
[prioritized list, Dx-aligned items first]
```

## Design Checks

Revisit this skill if:
- A new dominant operation is added — it needs a user journey and interface
- Users report friction in a journey that was designed here
- Infrastructure hits capacity limits during real usage
- A new constraint is added (e.g., "must support mobile") that changes interface requirements

## Examples

### Example 1: Design Mode — Journey-First Discovery

User says: "I need to design the API for my data processing system."

Do not start listing `/api/v1/...` endpoints. Instead:
- "Let's start with D1 (Process Incoming Data). What does the operator do? They upload a file, trigger processing, and wait for results. That gives us: a Trigger endpoint (upload + start), a Query endpoint (check status), and optionally a Push endpoint (live progress). Now let's do D2..."

### Example 2: Non-Goals Eliminate Technology

User asks: "Should we build a single-page application with React?"

Check non-goals: NG3 says "no complex frontend requiring dedicated frontend developers." This directly eliminates React/Vue/Angular SPAs. The answer is server-rendered pages with progressive enhancement (HTMX, Alpine.js, Turbo, etc.). Non-goals made the decision, not preference.

### Example 3: Verify Mode — Missing Journey Step

Audit finds: User can trigger D1 via `/api/start` (Trigger) and get final results via `/api/results` (Query). But there is no way to check progress between start and completion.

Flag it: "D1 takes ~8 minutes (from theory limits). The user triggers it and has no feedback until it finishes or fails. Consider adding either a Query endpoint for status polling or a Push endpoint for live progress. The choice depends on how many concurrent D1 operations are expected."

### Example 4: Infrastructure Contradiction

D1 requires SSE for live progress updates. The tech stack constraint says "Django." Django's default server (Gunicorn with sync workers) cannot handle long-lived SSE connections efficiently.

Flag it: "SSE requires long-lived connections, but Gunicorn sync workers block one worker per connection. Options: (1) Use an ASGI server like Daphne or Uvicorn for the entire app, (2) Route only SSE endpoints to ASGI via reverse proxy while keeping the rest on Gunicorn, (3) Use a separate service for SSE. Each has trade-offs in deployment complexity — which fits your constraints best?"

## Key Rules

- **Start from user journeys, not endpoint lists.** Endpoints are derived from journeys, not invented in isolation.
- **Classify every endpoint.** Trigger, Query, or Push. No exceptions. Mixed endpoints should be split.
- **Non-goals are your best design tool.** They eliminate options faster than goals add them.
- **Infrastructure is derived, not assumed.** Worker count, process topology, and capacity come from dominant-ops numbers, not from "best practices" or "what we usually do."
- **The abstraction level test applies to interfaces too.** "User triggers D1 and monitors progress" is the right level. "User clicks the blue button in the top-right corner" is too specific. "User interacts with the system" is too vague.
- **Do not design interfaces for non-dominant operations.** Give them the framework's defaults (e.g., Django Admin for CRUD). Invest design effort only where dominant-ops pressure demands it.
