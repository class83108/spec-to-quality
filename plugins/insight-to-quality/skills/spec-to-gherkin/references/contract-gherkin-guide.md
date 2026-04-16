# Contract Gherkin Guide

Gherkin writing reference for **skeleton** findings from **spec-contract**.
Primary consumer: `spec-to-gherkin` skill (when `type: skeleton` + `source: internals`).

---

## Anchor

> Valid data shape passes the handoff; invalid shape is rejected at the boundary.

Every scenario in a contract `.feature` revolves around **whether the schema boundary holds**.
The system under test is the handoff point between two components — not the business logic on either side.

---

## Scenario Patterns by Coverage Category

All examples use English keywords + Traditional Chinese step text (per language policy).

### 1. Happy Path — valid schema passes

```gherkin
Scenario: 有效的 {handoff name} 資料通過邊界
  Given 上游元件產出符合 {schema} 的資料
  When 資料經過 {boundary} handoff
  Then 下游元件收到完整資料且欄位型別正確
```

Focus: confirm the contract works end-to-end for normal input.

### 2. Error / Failure — invalid shape rejected

```gherkin
Scenario: 缺少必填欄位時 {boundary} 拒絕資料
  Given 上游元件產出的資料缺少 {required_field}
  When 資料經過 {boundary} handoff
  Then handoff 回傳 {domain_error_type} 且包含缺少欄位的說明

Scenario: 欄位型別錯誤時 {boundary} 拒絕資料
  Given 上游元件產出的 {field} 為字串而非預期的整數
  When 資料經過 {boundary} handoff
  Then handoff 回傳型別錯誤且指出錯誤欄位
```

Focus: each rejection produces a clear, structured domain error — not a silent swallow or raw exception.

### 3. Boundary & Edge — limit values

```gherkin
Scenario: 空值欄位在 {boundary} 的處理
  Given 上游元件產出的 {optional_field} 為 null
  When 資料經過 {boundary} handoff
  Then 下游元件收到資料且 {optional_field} 使用預設值

Scenario: 超長字串在 {boundary} 的處理
  Given 上游元件產出的 {field} 長度超過 {max_length}
  When 資料經過 {boundary} handoff
  Then handoff 回傳長度超限錯誤
```

Focus: null, empty, max-length, minimum input — the edges of valid schema space.

### 4. Business Rules — field constraints

```gherkin
Scenario: 不合法的列舉值在 {boundary} 被拒絕
  Given 上游元件產出的 {enum_field} 值為 "invalid_value"
  When 資料經過 {boundary} handoff
  Then handoff 回傳列舉值不合法錯誤且列出允許的值

Scenario: 跨欄位約束驗證
  Given 上游元件產出的 {field_a} 為 X 且 {field_b} 為 Y
  And X 與 Y 的組合違反 {constraint_rule}
  When 資料經過 {boundary} handoff
  Then handoff 回傳約束違反錯誤
```

Focus: enum values, format rules, cross-field validation — schema-level constraints, not business logic.

### 5. State Mutation — schema evolution

```gherkin
Scenario: 新增 optional 欄位後既有消費者不受影響
  Given 下游元件使用舊版 schema（不含 {new_field}）
  And 上游元件產出包含 {new_field} 的新版資料
  When 資料經過 {boundary} handoff
  Then 下游元件正常處理資料且忽略未知欄位
```

Focus: schema version changes do not break existing consumers. Only applicable when the finding card explicitly mentions schema evolution or backward compatibility.

Mark `Not Applicable` when: single-version system with no evolution concern.

### 6. Output Contract — error response shape

```gherkin
Scenario: schema 驗證失敗的錯誤回應符合 ErrorResponse 格式
  Given 上游元件產出不合法的資料
  When 資料經過 {boundary} handoff
  Then 錯誤回應包含 error_code 與 message 欄位
  And 錯誤回應結構符合 ErrorResponse schema
```

Focus: the error itself has a contract — downstream code can reliably parse rejection responses.

---

## Boundary — What NOT to Test Here

| Don't test | Belongs to | Why |
|-----------|-----------|-----|
| User behavior correctness (user does X → system responds Y) | feature-gherkin-guide | Contract tests verify shape, not business semantics |
| HTTP status codes, API routing, client-visible error format | surface-gherkin-guide | Those are external interface concerns, not internal handoff |
| Business logic behind the schema (calculation rules, state machines) | feature-gherkin-guide | Contract boundary holds shape; logic correctness is a separate concern |
| Infrastructure behavior (queue retry, transport timeout) | SYSTEM_MAP Architecture Decision | Not a finding card; update SYSTEM_MAP instead |

**Rule of thumb**: if removing the business logic and replacing it with a stub that returns valid schema would not change the test outcome, it belongs here. If the test requires real business logic to pass, it belongs in feature.

---

## Naming & Organization

- **File path**: `tests/features/contracts/{finding-id}.feature` (read from finding card `tests_path`)
- **Feature title**: `Feature: {boundary} 資料 handoff 契約`
- **Scenario naming**: start with the data condition, not the component name
  - Good: `缺少 speaker_id 時 Pipeline→StageRunner 拒絕資料`
  - Bad: `StageRunner 測試案例 1`
- **One finding → one `.feature` file**: do not merge multiple findings into one file
- **Tag**: `@contract @{finding-id}`

---

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| Testing business logic through schema validation | Conflates shape and semantics; schema test passes but behavior is wrong | Move behavior assertions to feature test |
| Asserting exact field values instead of shape | Brittle; breaks when valid default values change | Assert type, presence, and constraints — not literal values |
| Skipping error response shape assertion | Downstream code cannot reliably parse rejections | Always verify error response matches declared ErrorResponse schema |
| Writing one giant scenario with all validations | Unreadable; failure message unclear | One scenario per validation concern |
| Mocking the schema validator itself | Tests nothing | Mock the upstream data producer, not the validation boundary |
