---
name: feature-planning
description: >
  Alignment gap triage — reads internals-report and surface-report, prioritizes findings,
  routes each finding to OpenSpec pathway (large changes requiring TDD) or Fix pathway
  (small direct corrections), tracks progress in a 處理進度 section, updates SYSTEM_MAP
  when structure changes, and archives reports when all findings are resolved.
  Trigger when at least one alignment report exists and the user wants to start addressing
  gaps, or says "ready to start fixing alignment gaps".
  Do NOT use for: when no alignment reports exist (run align-internals or align-surface
  first), or when a feature plan already exists and implementation should begin directly
  (use feature-coverage).
---

# Feature Planning (Alignment Gap Triage)

You are entering the alignment gap triage stage. This skill reads alignment reports produced by `align-internals` and `align-surface`, decides which gap to address first, routes it to the correct fix pathway, and tracks progress until all findings are resolved.

**Before starting, read:**
- `references/architect-mindset.md` — traceability: every OpenSpec change must serve a Gx
- `references/implementation-mindset.md` Part 1 — Three Decisions framework (used in OpenSpec pathway only)

## Core Concept

**SYSTEM_MAP.md is the single source of truth for system structure.** Alignment reports are temporary — they record the gap between current code/docs and what the system should be. Each finding is a work item. This skill's job is to triage those work items, route each to the right fix pathway, update the ground truth after each fix, and eventually retire the reports.

```
alignment reports (findings)
        ↓
  feature-planning (triage)
        ↓
  ┌──────────────────────┬──────────────────────┐
  │     OpenSpec 路徑     │       Fix 路徑        │
  │                      │                      │
  │  新增行為 / 大結構改動  │  更正現有行為 / 小修正  │
  │                      │                      │
  │  開新 worktree        │  開 fix worktree      │
  │  → openspec change   │  → 直接修正            │
  │  → feature-coverage  │  → verify             │
  │  → gherkin → TDD     │  → 刪除 worktree      │
  │  → design-review     │                      │
  │  → pre-complete      │                      │
  └──────────────────────┴──────────────────────┘
           ↓ 每個 finding 完成後
  更新 alignment report 處理進度區塊
  必要時更新 SYSTEM_MAP
  所有 findings 都完成 → archive reports
```

## Prerequisites

All of the following must exist before entering this skill:
- `goals.md`
- `dominant-ops.md`
- `SYSTEM_MAP.md`
- At least one alignment report: `docs/alignment/internals-report.md` or `docs/alignment/surface-report.md`

If any are missing, stop and inform which prerequisite is missing and which skill produces it.

## Pathway Decision Criteria

**Check OpenSpec criteria first. Fix is the fallback when none of the OpenSpec criteria apply.**

**OpenSpec pathway — any one criterion is sufficient:**
- An entire seam is missing its contract implementation
- A complete endpoint or user journey is missing
- A new model or schema needs to be created
- System boundary or component structure needs adjustment
- The fix introduces new behavior (as opposed to correcting existing behavior to match original intent)
- Multiple components are affected by the same finding

**Fix pathway — all OpenSpec criteria must be absent:**
- Field or variable naming mismatch (Drift) — code does the right thing, just named wrong
- Documentation describes something that code already does correctly — update the doc
- A single validation rule is missing or wrong
- Import path or configuration reference is incorrect

**Drift special rule**: A doc/code mismatch defaults to Fix pathway, **unless** the mismatch reveals that the code is missing something that should exist — in that case, OpenSpec takes priority.

## Workflow

### Step 1: Read Current State

Read in order:

1. **SYSTEM_MAP.md** — understand system structure, boundaries, current state
2. **`docs/alignment/internals-report.md`** (if exists) — all sections including 處理進度
3. **`docs/alignment/surface-report.md`** (if exists) — all sections including 處理進度
4. **Current git branch** — run `git branch --show-current`; this is the base for all new worktrees
5. **`docs/feature-plans/`** (if exists) — understand what is already in-flight via OpenSpec

### Step 2: Build Prioritized Finding List

Extract all unresolved findings from both reports — findings not marked ✅ 已完成 in 處理進度.

For each finding:
1. Apply the pathway decision criteria → assign OpenSpec or Fix
2. Identify which Gx it serves and which Dx it relates to
3. Identify which SYSTEM_MAP boundary or component it touches

Display a prioritized list, ordered by Dx priority (D1 > D2 > D3 > others). Group by pathway:

```
未處理的 findings（共 N 項）：

OpenSpec 路徑：
1. [finding 摘要] — 來源: internals/surface [Seam/Scope 名稱], 影響: [SYSTEM_MAP 邊界], 服務: G1

Fix 路徑：
2. [finding 摘要] — 來源: internals/surface [Seam/Scope 名稱], 性質: Drift/缺口
```

Ask the user: "這次要先處理哪一項？"

### Step 3: Route to Pathway

After the user selects a finding, route to the appropriate pathway.

---

#### OpenSpec Pathway

**3a. Confirm traceability**

- Which Gx does this change serve? (must map to at least one goal)
- Which SYSTEM_MAP boundary does it touch?

If no Gx mapping → stop: "This finding currently has no goal foundation. Confirm which goal it serves, or add it to goals.md before continuing."

**3b. Guide Error Handling Strategy (Three Decisions)**

Using SYSTEM_MAP boundary information and dominant-ops anti-patterns, drive the conversation — do not ask generic questions:

*Decision 1 — Catch Boundary*: "This change touches [boundary]. When something goes wrong, does the caller need to know, or should this component handle errors internally?"

*Decision 2 — Error Taxonomy*: "Based on [Dx]'s [APx], [failure scenario] is a risk. Is this a domain error (e.g., resource not found, duplicate) or an infrastructure failure (e.g., DB unreachable, timeout)?"

*Decision 3 — Recovery Strategy*: "When domain errors occur, return structured error to caller? When infrastructure errors occur, fail fast or retry/degrade?"

**3c. Extract Constraints**

- **Anti-patterns**: From dominant-ops.md, extract relevant APx entries and explain how each limits this change's behavior
- **Boundary rules**: From SYSTEM_MAP.md Boundary Map, extract rules this change must respect

**3d. Produce Feature Plan**

Create `docs/feature-plans/{feature-name}.md` (kebab-case, same name as the subsequent OpenSpec change):

```markdown
# Feature Plan — {feature-name}

## Source
- Serves: G1, G3
- Alignment finding: [brief description]
- Report: internals-report / surface-report — [Seam/Scope name]

## Error Handling Strategy
- Catch boundary: [boundary only / per-layer / selective: which exceptions]
- Domain errors: [list, e.g. TaskNotFound, DuplicateTask]
- Infrastructure errors: [fail fast / retry N times / degrade to X]

## Constraints

### Anti-patterns (from dominant-ops)
- AP1 (D1): [description] — impact on this change: [explanation]

### Boundary Rules (from SYSTEM_MAP)
- [rule]

## Integration Test Gaps
<!-- Filled in after tdd-workflow Verification Ledger is complete; initially empty -->
```

**3e. Create Worktree and Hand Off**

Using the current branch from Step 1, create a new worktree:

```bash
git worktree add -b {current-branch}-{feature-name} ../{project-dir}-{feature-name} {current-branch}
```

Then update 處理進度 in the source alignment report (status: 🔄 進行中).

Inform the user:
"Worktree 已建立：`../{project-dir}-{feature-name}`（從 `{current-branch}` 開出）。

接下來：
1. 在新 worktree 中執行 `opsx apply` 建立 OpenSpec change；在 spec.md body 最上方加入 `**Serves:** Gx`
2. 參考 feature plan 的 Error Handling Strategy 填寫 design.md 的 Decisions 區段
3. 建立後進入 `feature-coverage`"

After OpenSpec pre-complete is done → proceed to **Step 4**.

---

#### Fix Pathway

**3f. Describe the fix**

Confirm with the user:
- What exactly needs to change — code, doc, or both?
- Which file(s) are involved?
- Is the fix purely corrective (the original intent was already correct, the implementation drifted)?

**3g. Create Fix Worktree**

Using the current branch from Step 1:

```bash
git worktree add -b fix/{fix-name} ../{project-dir}-fix-{fix-name} {current-branch}
```

Update 處理進度 in the source alignment report (status: 🔄 進行中).

**3h. Apply the Fix**

Make the change in the worktree:
- Code fix: update the code
- Doc fix: update the document to match what code actually does
- Both: update both

**3i. Verify**

Run the project's test, lint, and type-check commands (from the project's CLAUDE.md Commands section). All checks must pass before continuing.

**3j. Commit, Delete Worktree**

```bash
# in the fix worktree:
git add {relevant files}
git commit -m "fix: {description}"

# back in the main worktree:
git worktree remove ../{project-dir}-fix-{fix-name}
# then merge or open PR as appropriate
```

Proceed to **Step 4**.

---

### Step 4: Update Reports and SYSTEM_MAP

After each finding is resolved (via either pathway):

**Update 處理進度 in the source alignment report** — mark the finding as ✅ 已完成 with the branch or PR reference.

**Update SYSTEM_MAP if the fix changed system structure:**

Ask: "這個修正是否改變了系統結構（邊界定義、元件職責、合約狀態）？"
- Yes → update the relevant SYSTEM_MAP section (Boundary Map, Component Map, or Current State)
- No → no SYSTEM_MAP update needed

**Check archive condition** for each alignment report:
- If all findings in the report are ✅ 已完成 → archive it:

```bash
mv docs/alignment/internals-report.md docs/alignment/archive/$(date +%Y-%m-%d)-internals-report.md
mv docs/alignment/surface-report.md docs/alignment/archive/$(date +%Y-%m-%d)-surface-report.md
```

### Step 5: Continue or Done

**If unresolved findings remain** → return to Step 2. Re-display the updated prioritized list and ask which to handle next.

**If all findings are resolved and reports are archived** → inform:
"所有 alignment findings 已處理完成，報告已 archive。SYSTEM_MAP 是目前的最新 ground truth。如需再次審查，重新執行 `align-internals` 或 `align-surface`。"

## 處理進度 Section Format

This section is **appended to the bottom of each alignment report** the first time feature-planning processes a finding from that report. If the section already exists, update it in place.

```markdown
## 處理進度
| Finding | 來源 | 路徑 | 狀態 | Branch / PR |
|---|---|---|---|---|
| [finding 摘要] | internals — Seam A | OpenSpec | ✅ 已完成 | feature/seam-a-contract |
| [finding 摘要] | surface — D1 | Fix | 🔄 進行中 | fix/d1-query-endpoint |
| [finding 摘要] | internals — Seam B | Fix | ⬜ 待處理 | — |
```

Status values:
- ⬜ 待處理 — identified, not yet started
- 🔄 進行中 — worktree created, work in progress
- ✅ 已完成 — fix merged or OpenSpec pre-complete done

## Examples

### Example 1: OpenSpec Pathway — Missing Contract

internals-report shows: "Seam A — 缺口: TaskQueue contract 未實作 — 建議: 實作合約介面"

→ Pathway: OpenSpec (entire seam missing contract — new behavior must be added)
→ Confirm Serves G1, G3; touches SYSTEM_MAP Seam A
→ Guide Three Decisions; extract AP1, AP2 constraints
→ Produce `docs/feature-plans/task-queue-contract.md`
→ Current branch: `main` → `git worktree add -b main-task-queue-contract ../project-task-queue-contract main`
→ Update 處理進度: 🔄 進行中
→ After pre-complete: update 處理進度 ✅, update SYSTEM_MAP Seam A contract status → "defined"

### Example 2: Fix Pathway — Naming Drift

internals-report shows: "Seam B — Drift: Task model 欄位 `created_at` 在 code 中是 `createdAt`"

→ Pathway: Fix (naming mismatch; behavior is correct)
→ Current branch: `main` → `git worktree add -b fix/task-created-at-naming ../project-fix-task-created-at main`
→ Rename field in model and all references; verify tests pass
→ Commit, delete worktree, update 處理進度 ✅
→ No SYSTEM_MAP update needed (structure unchanged)

### Example 3: Drift Upgraded to OpenSpec

surface-report shows: "D1 — Drift: doc 描述有 Query endpoint 供進度查詢，但 code 中不存在"

→ Check OpenSpec criteria: endpoint is entirely missing → matches OpenSpec criterion
→ Pathway: OpenSpec (not Fix — the code is missing something that should exist)

### Example 4: Multiple Findings, Prioritized

internals-report has 3 findings, surface-report has 2 findings.

Displayed list:
```
OpenSpec 路徑：
1. Seam A contract 未實作 — 來源: internals — Seam A, 服務: G1 (D1 路徑，最高壓力)
2. D1 Query endpoint 缺失 — 來源: surface — D1, 服務: G1 (使用者無法完成 journey)

Fix 路徑：
3. Seam B 欄位命名 drift — 來源: internals — Seam B (低影響)
4. infrastructure doc 過時 — 來源: surface — Infrastructure (低影響)
5. D2 endpoint doc 不符 — 來源: surface — D2 (低壓力)
```

User picks item 1 → route to OpenSpec pathway. After completion, re-display (4 remaining items).

## Key Rules

- **OpenSpec criteria are checked first.** Fix is the fallback — never default to Fix when any OpenSpec criterion applies.
- **One finding at a time.** Do not start the next finding until the current one is resolved, 處理進度 is updated, and the worktree is deleted (Fix) or pre-complete is done (OpenSpec).
- **SYSTEM_MAP is the source of truth.** Update it when structure changes after a fix. Reports are temporary; SYSTEM_MAP is permanent.
- **Archive only when all findings are resolved.** Do not archive a report with ⬜ or 🔄 items remaining.
- **Worktrees branch from current branch, not from main.** Always check `git branch --show-current` first.
- **Fix worktrees are ephemeral.** Delete after merge. Each fix gets a fresh worktree.
- **Traceability is mandatory for OpenSpec pathway.** Every OpenSpec change must serve at least one Gx.
