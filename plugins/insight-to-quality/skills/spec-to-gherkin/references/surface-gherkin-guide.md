# Surface Gherkin Guide

Gherkin writing reference for **skeleton** findings from **spec-surface**.
Primary consumer: `spec-to-gherkin` skill (when `type: skeleton` + `source: surface`).

---

## Anchor

> External interface accepts valid input and produces output matching declared schema;
> rejects invalid input with the correct error format.

Every scenario revolves around **whether the external contract shape holds** — not whether the business logic behind it is correct. The system under test is the boundary where an external actor (user, downstream system, integrator) interacts with the system.

---

## Interface Types

spec-surface covers all external-facing interfaces. The Gherkin patterns vary by interface type, but the testing intent is the same: **validate the contract shape at the system boundary**.

| Interface type | External actor | What to verify |
|---------------|---------------|---------------|
| **API (HTTP)** | Client / frontend / integrator | Request validation, response schema, error codes |
| **CLI** | User / script / CI | Argument parsing, exit code, stdout/stderr format |
| **Published Event** | Downstream subscriber | Event payload schema, event type/routing key |
| **SSE / WebSocket** | Connected client | Message format, connection lifecycle contract |
| **File I/O** | User / external system | Input format validation, output file structure |

Start with the type relevant to the finding card. Most projects begin with API.

---

## Scenario Patterns by Coverage Category

All examples use English keywords + Traditional Chinese step text.

### 1. Happy Path — valid input accepted

**API:**
```gherkin
Scenario: 合法請求回傳成功且 body 符合 SuccessResponse schema
  Given 客戶端準備符合 {endpoint} 入參 schema 的請求
  When 客戶端發送 {method} 到 {endpoint}
  Then 回應狀態碼為 2xx
  And 回應 body 欄位與 SuccessResponse schema 一致
```

**CLI:**
```gherkin
Scenario: 合法參數執行成功且輸出符合預期格式
  Given 使用者準備合法的指令參數 {args}
  When 使用者執行 {command} {args}
  Then exit code 為 0
  And stdout 輸出符合 {output_format} 格式
```

**Published Event:**
```gherkin
Scenario: 狀態變更後發布的事件 payload 符合 schema
  Given 系統內部觸發 {state_change}
  When 事件被發布
  Then 事件 payload 包含所有必要欄位
  And payload 結構符合 {EventSchema}
```

### 2. Error / Failure — invalid input rejected

**API:**
```gherkin
Scenario: 缺少必填欄位時回傳 4xx 且 body 符合 ErrorResponse schema
  Given 客戶端準備的請求缺少 {required_field}
  When 客戶端發送 {method} 到 {endpoint}
  Then 回應狀態碼為 {4xx}
  And 回應 body 包含 error_code 與 message 欄位
  And 回應結構符合 ErrorResponse schema

Scenario: 請求 content-type 錯誤時回傳 415
  Given 客戶端發送 content-type 為 text/plain 的請求
  When 客戶端發送 POST 到 {endpoint}
  Then 回應狀態碼為 415
```

**CLI:**
```gherkin
Scenario: 缺少必要參數時顯示錯誤訊息並回傳非零 exit code
  Given 使用者未提供必要參數 {required_arg}
  When 使用者執行 {command}
  Then exit code 為非零
  And stderr 包含缺少參數的說明
```

### 3. Boundary & Edge — limit values

```gherkin
Scenario: 空 body 請求被正確拒絕
  Given 客戶端發送空 body 的請求
  When 客戶端發送 POST 到 {endpoint}
  Then 回應狀態碼為 400
  And 錯誤訊息指出 body 不得為空

Scenario: 超過長度限制的欄位被拒絕
  Given 客戶端準備的 {field} 長度超過 {max_length}
  When 客戶端發送 {method} 到 {endpoint}
  Then 回應狀態碼為 400
  And 錯誤訊息指出欄位長度超限
```

Focus: empty body, oversized payload, missing optional headers, minimum valid input.

### 4. Business Rules — boundary-level constraints

```gherkin
Scenario: 不合法的列舉值被拒絕
  Given 客戶端準備的 {enum_field} 值為 "invalid_value"
  When 客戶端發送 {method} 到 {endpoint}
  Then 回應狀態碼為 422
  And 錯誤訊息列出允許的列舉值
```

Focus: field constraints enforced **at the boundary** (enum values, format rules, required combinations). These are shape rules, not business logic — if the validation happens before any domain code runs, it belongs here.

### 5. State Mutation — idempotency at the boundary

```gherkin
Scenario: 帶有相同 idempotency key 的重複請求回傳一致結果
  Given 客戶端已成功發送一次帶有 idempotency key {key} 的請求
  When 客戶端再次發送相同 idempotency key 的請求
  Then 回應狀態碼與第一次一致
  And 回應 body 與第一次一致
```

Only applicable when the finding card mentions idempotency or the dominant-op requires it. Mark `Not Applicable` otherwise.

### 6. Output Contract — error response shape consistency

```gherkin
Scenario: 所有錯誤回應格式一致
  Given 客戶端分別觸發 {error_case_1} 與 {error_case_2}
  When 收集兩次的錯誤回應
  Then 兩次回應都包含 error_code 與 message 欄位
  And 兩次回應結構均符合 ErrorResponse schema
```

Focus: error shape is **consistent** across all error cases for this endpoint — downstream clients can write one error parser, not many.

---

## Boundary — What NOT to Test Here

| Don't test | Belongs to | Why |
|-----------|-----------|-----|
| Internal handoff schema between components | contract-gherkin-guide | Surface tests the external boundary, not internal data flow |
| Business logic correctness (user does X → system responds Y) | feature-gherkin-guide | Surface tests shape, not semantics; a stub behind the endpoint should still pass |
| Transport/protocol choice (why SSE instead of WebSocket) | SYSTEM_MAP Architecture Decision | Not a testing concern; it's a design decision |
| Infrastructure behavior (retry, timeout, queue capacity) | SYSTEM_MAP Architecture Decision | Not a finding card |

**Rule of thumb**: if you replaced all domain logic behind the endpoint with a stub that returns hardcoded valid responses, the surface test should still pass. If the test requires real domain logic, it belongs in feature.

---

## Naming & Organization

- **File path**: `tests/features/contracts/{finding-id}.feature` (read from finding card `tests_path`)
- **Feature title**: `Feature: {endpoint/command/event} 介面契約`
- **Scenario naming**: start with the input condition, then the expected boundary response
  - Good: `缺少 session_id 時 POST /interviews 回傳 400`
  - Bad: `API 測試 1`
- **One finding → one `.feature` file**
- **Tag**: `@surface @{finding-id}`

---

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| Asserting business outcomes through API response | Conflates shape and behavior; test is fragile to logic changes | Assert status code + response schema shape, not computed values |
| Testing only happy path status code without body schema | Response shape can silently drift | Always assert body structure matches declared schema |
| Hardcoding response body values instead of shape | Brittle; breaks when valid data changes | Assert field presence, type, and constraints |
| Skipping error path testing | Clients get inconsistent error formats, integration breaks | Cover at least: missing required field, wrong type, invalid enum |
| Writing surface tests that require database state | Crossing into integration/feature territory | Surface test should work with stub/mock behind the boundary |
