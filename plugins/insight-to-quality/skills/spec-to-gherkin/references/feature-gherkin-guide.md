# Feature Gherkin Guide

Gherkin writing reference for **feature** findings from **spec-behavior**.
Primary consumer: `spec-to-gherkin` skill (when `type: feature`).

---

## Anchor

> User completes an action → system responds correctly per business rules.

Every scenario revolves around **whether the system's business behavior is correct** — not whether the schema shape is valid (skeleton tests cover that). The system under test is the business logic that transforms valid input into the correct outcome.

---

## Relationship to Skeleton Tests

Feature tests assume skeleton tests already pass. This means:

- **Input shape is already validated** — feature scenarios start with valid input (schema validation is not re-tested)
- **Contract boundaries are already guarded** — feature scenarios reference existing skeleton contracts, do not redefine them
- **Error format is already enforced** — feature scenarios test business-level errors (rule violations, state conflicts), not format-level errors (missing field, wrong type)

If a feature scenario needs schema validation to make sense, the corresponding skeleton finding must be completed first (enforced by `skeleton_deps` in the finding card).

---

## Scenario Patterns by Coverage Category

All examples use English keywords + Traditional Chinese step text.

### 1. Happy Path — user achieves goal

```gherkin
Scenario: 使用者成功完成 {Gx action} 且系統正確回應
  Given 使用者處於 {precondition state}
  And 系統中存在 {required context data}
  When 使用者執行 {action}
  Then 系統狀態變更為 {expected state}
  And 使用者收到 {observable confirmation}
```

Focus: the **Gx objective** is achieved. The scenario name should trace to the finding card's `Serves` field.

### 2. Error / Failure — business rule violation

```gherkin
Scenario: 使用者違反 {business rule} 時系統拒絕操作
  Given 使用者處於 {precondition state}
  And {condition that violates rule}
  When 使用者嘗試執行 {action}
  Then 系統拒絕操作並回傳 {business_error}
  And 系統狀態維持不變

Scenario: 外部依賴失敗時系統執行降級策略
  Given 使用者執行 {action} 且系統依賴 {external_service}
  And {external_service} 回傳錯誤
  When 系統處理該請求
  Then 系統執行 {fallback_strategy}
  And 使用者收到 {degraded_but_usable_response}
```

Focus: business-level errors (not schema errors). Source from the finding card's `Error Handling Strategy`.

### 3. Boundary & Edge — business logic limits

```gherkin
Scenario: 使用者在 {edge condition} 下執行操作
  Given 系統處於 {boundary state}（例如：剛好達到上限、第一筆資料、最後一筆資料）
  When 使用者執行 {action}
  Then 系統正確處理邊界情境且結果為 {expected}

Scenario: 使用者在無資料狀態下執行操作
  Given 系統中尚無任何 {entity}
  When 使用者執行 {action that depends on entity}
  Then 系統回傳 {empty state response} 而非錯誤
```

Focus: edge cases in **business logic** (first/last item, empty collection, exactly-at-limit). Source from the finding card's `Behavior` constraints and linked dominant-op theory limits.

### 4. Business Rules — conditional logic and combinations

```gherkin
Scenario: {condition A} 且 {condition B} 時系統執行 {rule}
  Given {condition A} 成立
  And {condition B} 成立
  When 使用者執行 {action}
  Then 系統依照 {rule} 產出 {expected outcome}

Scenario: {rule exception} 時系統略過 {rule}
  Given {exception condition} 成立
  When 使用者執行 {action}
  Then 系統不套用 {rule} 且結果為 {alternative outcome}
```

Focus: conditional branches, multi-condition combinations, rule exceptions. Source from the finding card's `Behavior (SHALL/MUST)` conditional clauses.

### 5. State Mutation — transitions are correct and observable

```gherkin
Scenario: {action} 導致系統從 {state A} 轉移到 {state B}
  Given 系統處於 {state A}
  When 使用者執行 {action}
  Then 系統狀態為 {state B}
  And {state B} 可被後續查詢觀察到

Scenario: 相同操作重複執行時系統表現冪等
  Given 使用者已成功執行過 {action}
  When 使用者再次執行相同的 {action}
  Then 系統狀態與第一次執行後一致
  And 不產生重複的副作用
```

Focus: state transitions are correct, observable, and idempotent where required. Source from the finding card's `State Transitions` and write-related `Done Criteria`.

### 6. Output Contract — system response matches user expectation

```gherkin
Scenario: {action} 的回應包含使用者需要的所有資訊
  Given 使用者執行 {action} 且操作成功
  When 使用者檢視回應
  Then 回應包含 {required info for next user step}
  And 回應格式讓使用者能繼續後續操作
```

Focus: the system response is **useful to the user** — not just schema-valid, but containing the information needed for the next step. This is the bridge between skeleton (shape correct) and feature (semantics correct).

---

## Boundary — What NOT to Test Here

| Don't test | Belongs to | Why |
|-----------|-----------|-----|
| Input schema validation (missing field, wrong type → rejection) | surface-gherkin-guide | Skeleton tests already guard input shape |
| Internal handoff schema between components | contract-gherkin-guide | Feature tests business behavior, not data plumbing |
| Response schema structure (field presence, type correctness) | surface-gherkin-guide | Feature tests the value semantics, not the shape |
| Infrastructure behavior (queue retry, transport timeout) | SYSTEM_MAP Architecture Decision | Not a finding card |

**Rule of thumb**: if the test can pass with any schema-valid response (regardless of the actual values), it's a skeleton test, not a feature test. Feature tests care about **what the values mean**.

---

## Naming & Organization

- **File path**: `tests/features/behaviors/{finding-id}.feature` (read from finding card `tests_path`)
- **Feature title**: `Feature: {Gx short description} — {behavior slice}`
- **Scenario naming**: start with what the user does, then the expected system response
  - Good: `使用者提交有效認證資訊後系統建立 session`
  - Bad: `認證功能測試`
- **One finding → one `.feature` file**
- **Tag**: `@behavior @{finding-id}`

---

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| Re-testing schema validation in feature scenarios | Duplicates skeleton tests; maintenance burden doubles | Start with valid input; trust skeleton tests for shape |
| Testing implementation details instead of observable behavior | Fragile; breaks on refactor even when behavior is unchanged | Assert on user-observable outcomes (state, response), not internal method calls |
| Missing Given context (testing action in a vacuum) | Scenario is ambiguous; unclear what state the system starts in | Always establish precondition state explicitly |
| Asserting on database internals instead of user-observable state | Couples test to storage implementation | Assert via the same interface the user would use to observe the change |
| Writing scenarios without tracing to Gx | No traceability; unclear why the test exists | Every scenario should link to at least one Gx in the finding card's `Serves` |
| Combining multiple business rules in one scenario | Failure is ambiguous; unclear which rule broke | One scenario per rule or rule combination |
