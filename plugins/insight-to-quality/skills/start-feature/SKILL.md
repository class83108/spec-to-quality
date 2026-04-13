---
name: start-feature
description: >
  Entry point when the user has a specific feature idea but does not know its scope
  relative to the existing system. Maps the idea to goals, boundaries, and alignment
  data, then routes to the earliest skill that needs to run — which may be anywhere
  from goals-discovery back to feature-planning directly.
  Requires goals.md, dominant-ops.md, and SYSTEM_MAP.md to exist (discovery complete).
  Do NOT use for: picking the next feature from known SYSTEM_MAP gaps (use
  feature-planning directly), or continuing an in-flight spec (use feature-planning
  directly).
---

# Start Feature

You are the entry point for a user who has a specific feature idea but does not know
how it relates to the existing system. Your job is to **scope the idea, find the
earliest skill that needs to run, and route there** — you do not plan the feature
yourself.

The routing may point anywhere from discovery-level skills (goals-discovery,
dominant-ops, system-map) back to alignment skills, or directly into feature-planning
if everything is already in place.

## When NOT to use this skill

Tell the user to use a different skill if:

- "I want to see what's left and pick the next thing to implement" → **feature-planning**
  directly (it reads SYSTEM_MAP Gaps and presents candidates)
- "I want to continue the in-flight spec" → **feature-planning** directly
- Discovery is not yet complete → complete discovery first, then return here

## Prerequisites

- `goals.md`, `dominant-ops.md`, `SYSTEM_MAP.md` must all exist
- If any are missing, stop and name the missing skill

## Workflow

### Step 1: Understand the Feature Idea

Ask the user:

> "你想做什麼功能？用一兩句話描述它做什麼、誰使用、解決什麼問題。"

If the description is too vague ("improve performance", "make it better"), ask:

> "可以說得更具體嗎？這個功能完成後，使用者可以做到什麼事情是現在做不到的？"

Do not proceed until you have a concrete description.

### Step 2: Route — Work Top-Down

Check each layer in order. Stop at the **first layer that needs work** and route there.
Do not continue checking lower layers once a gap is found at a higher layer.

---

**Layer 1 — Goals**

Read `goals.md`. Does the feature map to any existing Gx?

- No match, and it is clearly a new system capability →
  > "這個功能目前沒有對應的系統目標。需要先更新 goals.md 才能繼續。建議觸發 `goals-discovery`。"
  Route to **goals-discovery**.

- Partially matches but a goal needs to be expanded → same, route to **goals-discovery**.

- Clear match exists → record the Gx IDs and continue to Layer 2.

---

**Layer 2 — Dominant Operations**

Read `dominant-ops.md`. Does this feature change the system's pressure distribution?

- The feature introduces a new high-frequency or high-cost operation not reflected in
  any Dx →
  > "這個功能會改變系統的壓力分佈，dominant-ops.md 需要更新後再繼續。建議觸發 `dominant-ops`。"
  Route to **dominant-ops**.

- The feature fits within existing Dx operations → continue to Layer 3.

---

**Layer 3 — System Map**

Read `SYSTEM_MAP.md` Boundary Map and Component Map. Does the feature cross any boundary
or touch any component that exists in the map?

- The feature requires a boundary or component not in the map →
  > "這個功能需要 SYSTEM_MAP 目前沒有記錄的邊界或元件，需要先更新地圖。建議觸發 `system-map`。"
  Route to **system-map**.

- The feature maps to existing boundaries → record the affected seams and continue to
  Layer 4.

---

**Layer 4 — Alignment**

For each affected seam, check alignment data sufficiency:

**Check A — SYSTEM_MAP Gaps**: Is there already a gap entry for this seam in
`Current State.Gaps`? If yes → sufficient for that seam.

**Check B — Archive**: Check `docs/alignment/archive/`. If a report covers this seam,
surface the date to the user:
> "找到 [YYYY-MM-DD] 的舊 alignment 報告涵蓋 [Seam]。若邊界未有大改動可沿用；否則建議重跑。"
Let the user decide whether to reuse or refresh.

If a seam has no coverage from either check → alignment is missing for that layer.

Produce a sufficiency table:

```
Seam A (internals): SYSTEM_MAP Gaps 已有紀錄 → 充足
Seam B (surface):   無資料 → 需要 align-surface（verify mode）
```

Route to the missing alignment skill(s). If both are missing, run align-internals
before align-surface (contracts must be stable before surface is audited).

If all seams are sufficient → continue to Layer 5.

---

**Layer 5 — Feature Planning**

All layers are in order. Hand off to feature-planning with a scoping summary:

```
功能：[description]
服務目標：Gx
觸及邊界：[Seam A] (internals), [Seam B] (surface)
相關 SYSTEM_MAP gaps：[list, or "由 alignment 報告提供"]
```

> "所有前置資料已就緒，進入 feature-planning。"

Trigger **feature-planning**.

---

### Step 3: Confirm Before Routing

Before triggering any skill, state the routing decision and wait for user confirmation:

> "根據分析，需要先執行 [skill]（原因：[one line reason]）。確認後繼續。"

Do not trigger skills without explicit user confirmation.

## Key Rules

- **Route to the earliest gap.** Do not skip layers — a goal gap is more fundamental
  than an alignment gap, and fixing the wrong layer wastes effort.
- **Do not plan the feature here.** Scope and route only. Feature decisions belong in
  feature-planning.
- **No Gx → no feature.** Block until the goal is established.
- **Unknown boundary → update SYSTEM_MAP first.** The map is the foundation for all
  downstream skills.
- **Archive age is a signal, not a decision.** Surface the date; let the user decide.
- **align-internals before align-surface.** Contracts must be stable before surface
  design is audited.
