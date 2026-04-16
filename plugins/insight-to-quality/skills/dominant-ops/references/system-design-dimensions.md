# System Design Dimensions Reference

> Used in the **Design Implications** step of dominant-ops.
> For each Dx, the agent selects relevant dimensions from the list below and asks the user structured follow-up questions.
> Do not ask every dimension. Select based on Dx characteristics. User answers become the Design Implications section in dominant-ops.md and inputs for technology decisions in system-map.

---

## How to Use

1. Review each Dx's **characteristic tags** (high frequency, high failure cost, real-time, etc.)
2. Use the "Dx characteristic -> relevant dimensions" mapping table below to choose exploration dimensions
3. Ask structured questions dimension by dimension; the goal is to make the user think explicitly, not to let the agent decide
4. User answers can be either "choose X" or "not sure yet, decide in system-map"; both are valid
5. Record confirmed choices in Design Implications and carry them into the system-map session

---

## Dx Characteristic -> Relevant Dimensions Mapping

| Dx characteristic | Priority dimensions to explore |
|---------|--------------|
| High-frequency reads | Caching strategy, read scaling, connection pooling |
| High-frequency writes | Write scaling, batching strategy, consistency model |
| High failure cost | Reliability boundaries, idempotency, retry strategy, consistency model |
| Real-time / low latency | Communication protocol, caching strategy |
| Long-running tasks | Task queueing, checkpoint/resume, timeout strategy |
| Short + long tasks coexist | Queue isolation, worker pool separation |
| Large data volume | Storage type selection, read/write scaling, pagination/archiving |
| Human-in-the-loop workflows | State persistence, recovery from interruption, partial-completion semantics |
| Cross-boundary data flows | Consistency model, schema versioning |
| External dependencies (third-party API/services) | Adapter pattern, fault tolerance strategy, timeout |
| Special data types (geo/time-series/full-text search) | Storage type selection |
| Audit/compliance requirements | Observability, append-only log |

---

## Dimension Catalog

### Storage Type Selection

**When to discuss**: large data volume, special data types, cross-boundary data flows

Follow-up questions:
- Is the data relational (requires JOIN/foreign key guarantees), or document/key-value/graph based?
- Do you need geospatial queries? -> PostGIS / H3 / S2 vs traditional SQL
- Do you need time-series storage (metrics/logs/event streams)? -> InfluxDB / TimescaleDB vs SQL
- Do you need full-text search? -> Elasticsearch / Typesense vs SQL `LIKE`
- Do you need in-memory storage (session/cache/pub-sub broker)? -> Redis / Memcached

Decision record format:
```
storage: PostgreSQL + Redis (PostgreSQL as primary DB, Redis for session + cache)
rationale: Data is highly relational and needs FK guarantees; DOM-1 read frequency is high, so Redis reduces DB pressure
```

---

### Caching Strategy

**When to discuss**: high-frequency reads, real-time / low latency

Follow-up questions:
- Which data is read far more often than written? Is short-lived staleness acceptable?
- Do you need application-level cache (in-process) or distributed cache (Redis across processes/services)?
- Do static assets or API responses require CDN?
- Are there cache invalidation scenarios complex enough that caching is not worth it?

Decision record format:
```
caching: Redis (distributed cache, TTL 60s)
rationale: DOM-1 reads are 100x writes; brief staleness is acceptable; CDN not applicable (personalized data)
```

---

### Communication Protocol

**When to discuss**: real-time / low latency, human-in-the-loop workflows, cross-boundary data flows

Follow-up questions:
- Must callers wait for results (sync), or can results be asynchronous?
- Do you need real-time push? -> SSE vs WebSocket vs long-polling
  - SSE: one-way server -> client push, simple, good for progress updates
  - WebSocket: two-way communication, good for chat/realtime collaboration
  - Long-polling: fallback, higher latency but broad compatibility
- What are reconnect semantics? Do you need `Last-Event-ID` replay?
- Service-to-service protocol: REST / GraphQL / gRPC? (payload size, streaming requirements, client diversity)

Decision record format:
```
protocol: SSE (server -> client progress push) + REST (CRUD operations)
rationale: DOM-2 requires real-time progress; SSE is enough (one-way); reconnect needs Last-Event-ID replay
```

---

### Task Queueing and Queue Isolation

**When to discuss**: long-running tasks, short + long tasks coexist, high failure cost

Follow-up questions:
- Do you need background tasks? What triggers them (API call, schedule, event)?
- Do interactive tasks (fast, user waiting) and batch tasks (slow, background) coexist?
  - If yes -> isolate with separate queues + worker pools so long tasks do not starve interactive tasks
- How should failures be handled? Do you need a DLQ (Dead Letter Queue)?
- Do you need scheduled jobs (cron)?

Decision record format:
```
task-queue: Celery + Redis broker
queue-isolation: `interactive` queue (2 workers) + `batch` queue (1 worker)
rationale: DOM-2 (interactive) must not be starved by DOM-3 (batch); DLQ captures failed tasks for manual review
```

---

### Reliability Boundaries

**When to discuss**: high failure cost, cross-boundary data flow, external dependencies

Follow-up questions:
- Which operations need ACID guarantees (all-success or all-failure)? Where is each transaction boundary?
- If the same request runs twice, can duplicates be created? -> design idempotency keys
- What retry strategy is needed? Exponential backoff + max retries? DLQ?
- Are there failures that should not stop the whole flow? -> circuit breaker / graceful degradation

Decision record format:
```
transactions: Each pipeline stage completion write is an atomic transaction
idempotency: Stage execution ID used as idempotency key; retries do not create duplicates
retry: Up to 3 retries with exponential backoff; send to DLQ on 4th failure
```

---

### Consistency Model

**When to discuss**: high failure cost, cross-boundary data flow, high-frequency writes

Follow-up questions:
- Which data needs strong consistency (immediate read-after-write), and which can be eventually consistent?
- What is the source of truth? Is there risk of concurrent updates from multiple places?
- Do you need read-your-writes guarantees (user sees own update immediately)?
- Do schemas need versioning (consumers may read older formats)?

Decision record format:
```
consistency: Strong consistency (PostgreSQL; read-your-writes guaranteed within same DB transaction)
schema-versioning: Not needed (single service, no cross-service shared schema)
```

---

### Read/Write Scaling

**When to discuss**: high-frequency reads, high-frequency writes, large data volume

Follow-up questions:
- Is read pressure high enough to require read replicas?
- Is write pressure high enough to require sharding/partitioning?
- Are there batch-write opportunities to avoid per-row overhead?
- Does connection pool size conflict with DB connection limits?

Decision record format:
```
read-scaling: Read replicas not needed currently (DOM-1 read volume is within single-DB capacity per theory limits)
write-batching: Stage outputs are batch written to avoid per-row inserts
```

---

### State Persistence and Resume Recovery

**When to discuss**: human-in-the-loop workflows, long-running tasks, high failure cost

Follow-up questions:
- If a long operation fails midway, can it resume from checkpoint or must it restart?
- Which intermediate states must be persisted? DB only, or file/blob too?
- For partial completion (N steps total, fail at step M), should completed steps be rolled back or kept?
- On rerun, should completed steps rerun or be skipped?

Decision record format:
```
state: Persist output snapshot to DB immediately after each stage completes
partial-completion: Failed stages marked as failed; completed stages retained; partial rerun supported
```

---

### External Dependency Fault Tolerance

**When to discuss**: external dependencies (third-party APIs/services)

Follow-up questions:
- Which operations depend on external services? What are their SLA/availability profiles?
- When external services fail, should the system fail fast or degrade gracefully?
- Do you need an adapter layer to isolate external API changes?
- What timeout values are needed (per-call timeout vs global timeout)?
- Do you need a circuit breaker (pause calls after consecutive failures)?

Decision record format:
```
external: Anthropic API (LLM inference)
adapter: AnthropicAdapter isolates SDK details; domain code does not directly import anthropic
timeout: 60s timeout per LLM call; circuit breaker opens for 30s after 5 consecutive failures
```

---

### Observability

**When to discuss**: high failure cost, audit/compliance requirements, external dependencies

Follow-up questions:
- Which Dx paths need latency/error-rate metrics?
- Do you need structured logging? What should be logged at which boundaries?
- Is cross-service request tracing needed (distributed tracing)?
- Which conditions require alerts (wake people up) vs logs only?
- Do audit/compliance needs require append-only logs (tamper-resistant operation records)?

Decision record format:
```
observability: structured logging (start/end/error per stage); Sentry for exception tracking
alerting: DOM-1 error rate > 5% in 5min -> PagerDuty
audit-log: Not needed (non-compliance environment)
```
