# 具多代理編排的智慧問題解決

[延伸思考：此工作流程實施了一個複雜的除錯和解決管線，利用 AI 輔助除錯工具和可觀測性平台系統性地診斷和解決正式環境問題。智慧除錯策略結合了自動根本原因分析和人類專業知識，使用現代化 2024/2025 實踐，包括 AI 程式碼助手（GitHub Copilot、Claude Code）、可觀測性平台（Sentry、DataDog、OpenTelemetry）、用於回歸追蹤的 git bisect 自動化，以及正式環境安全的除錯技術，如分散式追蹤和結構化日誌記錄。該流程遵循嚴格的四階段方法：（1）問題分析階段 - error-detective 和 debugger 代理分析錯誤追蹤、日誌、重現步驟和可觀測性資料，以了解故障的完整情境，包括上游/下游影響，（2）根本原因調查階段 - debugger 和 code-reviewer 代理執行深度程式碼分析、自動 git bisect 識別引入的提交、相依性相容性檢查和狀態檢查以隔離確切的故障機制，（3）修復實施階段 - 領域特定代理（python-pro、typescript-pro、rust-expert 等）實施最小修復，並提供全面的測試覆蓋，包括單元測試、整合測試和邊緣案例測試，同時遵循正式環境安全實踐，（4）驗證階段 - test-automator 和 performance-engineer 代理執行回歸套件、效能基準測試、安全掃描，並驗證沒有引入新問題。跨多個系統的複雜問題需要專家代理（database-optimizer → performance-engineer → devops-troubleshooter）之間的協調配合，並明確傳遞情境和狀態共享。工作流程強調理解根本原因而非治療症狀、實施持久的架構改進、透過增強的監控和告警自動化檢測，以及透過型別系統增強、靜態分析規則和改進的錯誤處理模式防止未來發生。成功不僅透過問題解決來衡量，還透過減少平均恢復時間（MTTR）、預防類似問題和改善系統彈性來衡量。]

## 階段 1：問題分析 - 錯誤檢測和情境收集

使用 Task 工具，subagent_type="error-debugging::error-detective"，然後是 subagent_type="error-debugging::debugger"：

**首先：Error-Detective 分析**

**提示：**
```
分析錯誤追蹤、日誌和可觀測性資料：$ARGUMENTS

交付成果：
1. 錯誤簽名分析：異常類型、訊息模式、頻率、首次出現
2. 堆疊追蹤深度分析：故障位置、呼叫鏈、涉及的元件
3. 重現步驟：最小測試案例、環境要求、所需的資料夾具
4. 可觀測性情境：
   - Sentry/DataDog 錯誤群組和趨勢
   - 顯示請求流程的分散式追蹤（OpenTelemetry/Jaeger）
   - 結構化日誌（帶關聯 ID 的 JSON 日誌）
   - APM 指標：延遲尖峰、錯誤率、資源使用
5. 使用者影響評估：受影響的使用者區段、錯誤率、業務指標影響
6. 時間軸分析：何時開始、與部署/配置變更的關聯
7. 相關症狀：類似錯誤、串聯故障、上游/下游影響

要採用的現代化除錯技術：
- AI 輔助日誌分析（模式檢測、異常識別）
- 跨微服務的分散式追蹤關聯
- 正式環境安全的除錯（無程式碼變更，使用可觀測性資料）
- 用於去重和追蹤的錯誤指紋識別
```

**預期輸出：**
```
ERROR_SIGNATURE: {異常類型 + 關鍵訊息模式}
FREQUENCY: {計數、速率、趨勢}
FIRST_SEEN: {時間戳記或 git 提交}
STACK_TRACE: {突顯關鍵幀的格式化追蹤}
REPRODUCTION: {最小步驟 + 範例資料}
OBSERVABILITY_LINKS: [Sentry URL、DataDog 儀表板、追蹤 ID]
USER_IMPACT: {受影響的使用者、嚴重程度、業務影響}
TIMELINE: {何時開始、與變更的關聯}
RELATED_ISSUES: [類似錯誤、串聯故障]
```

**其次：Debugger 根本原因識別**

**提示：**
```
使用 error-detective 輸出執行根本原因調查：

來自 Error-Detective 的情境：
- 錯誤簽名：{ERROR_SIGNATURE}
- 堆疊追蹤：{STACK_TRACE}
- 重現：{REPRODUCTION}
- 可觀測性：{OBSERVABILITY_LINKS}

交付成果：
1. 帶支持證據的根本原因假設
2. 程式碼級別分析：變數狀態、控制流程、時序問題
3. Git bisect 分析：識別引入的提交（使用 git bisect run 自動化）
4. 相依性分析：版本衝突、API 變更、配置漂移
5. 狀態檢查：資料庫狀態、快取狀態、外部 API 回應
6. 故障機制：為什麼程式碼在這些特定條件下失敗
7. 修復策略選項與權衡（快速修復 vs. 適當修復）

下一階段所需的情境：
- 需要變更的確切檔案路徑和行號
- 受影響的資料結構或 API 合約
- 可能需要更新的相依性
- 驗證修復的測試場景
- 要維護的效能特性
```

**預期輸出：**
```
ROOT_CAUSE: {帶證據的技術解釋}
INTRODUCING_COMMIT: {git SHA + 摘要（如果透過 bisect 找到）}
AFFECTED_FILES: [帶特定行號的檔案路徑]
FAILURE_MECHANISM: {為什麼失敗 - 競爭條件、null 檢查、型別不匹配等}
DEPENDENCIES: [相關系統、程式庫、外部 API]
FIX_STRATEGY: {帶推理的建議方法}
QUICK_FIX_OPTION: {如適用的臨時緩解}
PROPER_FIX_OPTION: {長期解決方案}
TESTING_REQUIREMENTS: [必須涵蓋的場景]
```

## 階段 2：根本原因調查 - 深度程式碼分析

使用 Task 工具，subagent_type="error-debugging::debugger" 和 subagent_type="comprehensive-review::code-reviewer" 進行系統性調查：

**首先：Debugger 程式碼分析**

**提示：**
```
執行深度程式碼分析和 bisect 調查：

來自階段 1 的情境：
- 根本原因：{ROOT_CAUSE}
- 受影響的檔案：{AFFECTED_FILES}
- 故障機制：{FAILURE_MECHANISM}
- 引入的提交：{INTRODUCING_COMMIT}

交付成果：
1. 程式碼路徑分析：從入口點追蹤執行到故障
2. 變數狀態追蹤：關鍵決策點的值
3. 控制流程分析：採取的分支、迴圈、非同步操作
4. Git bisect 自動化：建立 bisect 腳本以識別確切的中斷提交
   ```bash
   git bisect start HEAD v1.2.3
   git bisect run ./test_reproduction.sh
   ```
5. 相依性相容性矩陣：可運作/失敗的版本組合
6. 配置分析：環境變數、功能旗標、部署配置
7. 時序和競爭條件分析：非同步操作、事件排序、鎖定
8. 記憶體和資源分析：洩漏、耗盡、競爭

現代化調查技術：
- AI 輔助程式碼解釋（Claude/Copilot 理解複雜邏輯）
- 使用重現測試的自動 git bisect
- 相依性圖分析（npm ls、go mod graph、pip show）
- 配置漂移檢測（比較 staging vs production）
- 使用正式環境追蹤進行時間旅行除錯
```

**預期輸出：**
```
CODE_PATH: {入口 → ... → 帶關鍵變數的故障位置}
STATE_AT_FAILURE: {變數值、物件狀態、資料庫狀態}
BISECT_RESULT: {引入錯誤的確切提交 + diff}
DEPENDENCY_ISSUES: [版本衝突、中斷性變更、CVE]
CONFIGURATION_DRIFT: {環境之間的差異}
RACE_CONDITIONS: {非同步問題、事件排序問題}
ISOLATION_VERIFICATION: {確認單一根本原因 vs. 多個問題}
```

**其次：Code-Reviewer 深入分析**

**提示：**
```
審查程式碼邏輯並識別設計問題：

來自 Debugger 的情境：
- 程式碼路徑：{CODE_PATH}
- 故障時的狀態：{STATE_AT_FAILURE}
- Bisect 結果：{BISECT_RESULT}

交付成果：
1. 邏輯缺陷分析：不正確的假設、遺漏的邊緣案例、錯誤的演算法
2. 型別安全缺口：更強的型別可以防止問題的地方
3. 錯誤處理審查：遺漏的 try-catch、未處理的 promise、panic 場景
4. 合約驗證：輸入驗證缺口、未滿足的輸出保證
5. 架構問題：緊密耦合、遺漏的抽象、分層違規
6. 類似模式：其他具有相同漏洞的程式碼位置
7. 修復設計：最小變更 vs. 重構 vs. 架構改進

審查檢查清單：
- null/undefined 值是否正確處理？
- 非同步操作是否正確等待/鏈接？
- 錯誤案例是否明確處理？
- 型別斷言是否安全？
- API 合約是否受到尊重？
- 副作用是否隔離？
```

**預期輸出：**
```
LOGIC_FLAWS: [特定的不正確假設或演算法]
TYPE_SAFETY_GAPS: [型別可以防止問題的地方]
ERROR_HANDLING_GAPS: [未處理的錯誤路徑]
SIMILAR_VULNERABILITIES: [具有相同模式的其他程式碼]
FIX_DESIGN: {最小變更方法}
REFACTORING_OPPORTUNITIES: {如果需要更大的改進}
ARCHITECTURAL_CONCERNS: {如果存在系統性問題}
```

## 階段 3：修復實施 - 領域特定代理執行

根據階段 2 的輸出，使用 Task 工具路由到適當的領域代理：

**路由邏輯：**
- Python 問題 → subagent_type="python-development::python-pro"
- TypeScript/JavaScript → subagent_type="javascript-typescript::typescript-pro"
- Go → subagent_type="systems-programming::golang-pro"
- Rust → subagent_type="systems-programming::rust-pro"
- SQL/資料庫 → subagent_type="database-cloud-optimization::database-optimizer"
- 效能 → subagent_type="application-performance::performance-engineer"
- 安全性 → subagent_type="security-scanning::security-auditor"

**提示範本（針對語言調整）：**
```
實施具全面測試覆蓋的正式環境安全修復：

來自階段 2 的情境：
- 根本原因：{ROOT_CAUSE}
- 邏輯缺陷：{LOGIC_FLAWS}
- 修復設計：{FIX_DESIGN}
- 型別安全缺口：{TYPE_SAFETY_GAPS}
- 類似漏洞：{SIMILAR_VULNERABILITIES}

交付成果：
1. 解決根本原因（而非症狀）的最小修復實施
2. 單元測試：
   - 特定故障案例重現
   - 邊緣案例（邊界值、null/空、溢位）
   - 錯誤路徑覆蓋
3. 整合測試：
   - 帶真實相依性的端到端場景
   - 適當的外部 API 模擬
   - 資料庫狀態驗證
4. 回歸測試：
   - 類似漏洞的測試
   - 涵蓋相關程式碼路徑的測試
5. 效能驗證：
   - 顯示無降級的基準測試
   - 如適用的負載測試
6. 正式環境安全實踐：
   - 用於漸進式推出的功能旗標
   - 如果修復失敗則優雅降級
   - 用於修復驗證的監控鉤子
   - 用於除錯的結構化日誌記錄

現代化實施技術（2024/2025）：
- AI 配對程式設計（GitHub Copilot、Claude Code）用於測試生成
- 型別驅動開發（利用 TypeScript、mypy、clippy）
- 合約優先 API（OpenAPI、gRPC schema）
- 可觀測性優先（結構化日誌、指標、追蹤）
- 防禦性程式設計（明確的錯誤處理、驗證）

實施要求：
- 遵循現有的程式碼模式和慣例
- 添加策略性除錯日誌記錄（JSON 結構化日誌）
- 包含全面的型別註解
- 更新錯誤訊息為可操作的（包含情境、建議）
- 維護向後相容性（如果中斷則版本化 API）
- 添加用於分散式追蹤的 OpenTelemetry span
- 包含用於監控的指標計數器（成功/失敗率）
```

**預期輸出：**
```
FIX_SUMMARY: {變更內容和原因 - 根本原因 vs. 症狀}
CHANGED_FILES: [
  {path: "...", changes: "...", reasoning: "..."}
]
NEW_FILES: [{path: "...", purpose: "..."}]
TEST_COVERAGE: {
  unit: "X 場景",
  integration: "Y 場景",
  edge_cases: "Z 場景",
  regression: "W 場景"
}
TEST_RESULTS: {all_passed: true/false, details: "..."}
BREAKING_CHANGES: {無 | 帶遷移路徑的 API 變更}
OBSERVABILITY_ADDITIONS: [
  {type: "log", location: "...", purpose: "..."},
  {type: "metric", name: "...", purpose: "..."},
  {type: "trace", span: "...", purpose: "..."}
]
FEATURE_FLAGS: [{flag: "...", rollout_strategy: "..."}]
BACKWARD_COMPATIBILITY: {已維護 | 帶緩解的中斷}
```

## 階段 4：驗證 - 自動化測試和效能驗證

使用 Task 工具，subagent_type="unit-testing::test-automator" 和 subagent_type="application-performance::performance-engineer"：

**首先：Test-Automator 回歸套件**

**提示：**
```
執行全面的回歸測試並驗證修復品質：

來自階段 3 的情境：
- 修復摘要：{FIX_SUMMARY}
- 變更的檔案：{CHANGED_FILES}
- 測試覆蓋：{TEST_COVERAGE}
- 測試結果：{TEST_RESULTS}

交付成果：
1. 完整測試套件執行：
   - 單元測試（所有現有 + 新）
   - 整合測試
   - 端到端測試
   - 合約測試（如果是微服務）
2. 回歸檢測：
   - 比較修復前/後的測試結果
   - 識別任何新失敗
   - 驗證所有邊緣案例都已涵蓋
3. 測試品質評估：
   - 程式碼覆蓋率指標（行、分支、條件）
   - 如適用的變異測試
   - 測試確定性（多次執行）
4. 跨環境測試：
   - 在 staging/QA 環境中測試
   - 使用類正式環境的資料量測試
   - 使用真實的網路條件測試
5. 安全性測試：
   - 身份驗證/授權檢查
   - 輸入驗證測試
   - SQL 注入、XSS 預防
   - 相依性漏洞掃描
6. 自動化回歸測試生成：
   - 使用 AI 生成額外的邊緣案例測試
   - 複雜邏輯的基於屬性的測試
   - 輸入驗證的模糊測試

現代化測試實踐（2024/2025）：
- AI 生成的測試案例（GitHub Copilot、Claude Code）
- UI/API 合約的快照測試
- 前端的視覺回歸測試
- 彈性測試的混沌工程
- 負載測試的正式環境流量重播
```

**預期輸出：**
```
TEST_RESULTS: {
  total: N,
  passed: X,
  failed: Y,
  skipped: Z,
  new_failures: [如有則列出],
  flaky_tests: [如有則列出]
}
CODE_COVERAGE: {
  line: "X%",
  branch: "Y%",
  function: "Z%",
  delta: "+/-W%"
}
REGRESSION_DETECTED: {是/否 + 如果是則提供詳細資訊}
CROSS_ENV_RESULTS: {staging: "...", qa: "..."}
SECURITY_SCAN: {
  vulnerabilities: [列出或「無」],
  static_analysis: "...",
  dependency_audit: "..."
}
TEST_QUALITY: {deterministic: true/false, coverage_adequate: true/false}
```

**其次：Performance-Engineer 驗證**

**提示：**
```
測量效能影響並驗證無回歸：

來自 Test-Automator 的情境：
- 測試結果：{TEST_RESULTS}
- 程式碼覆蓋率：{CODE_COVERAGE}
- 修復摘要：{FIX_SUMMARY}

交付成果：
1. 效能基準測試：
   - 回應時間（p50、p95、p99）
   - 吞吐量（請求/秒）
   - 資源使用率（CPU、記憶體、I/O）
   - 資料庫查詢效能
2. 與基準線的比較：
   - 前/後指標
   - 可接受的降級閾值
   - 效能改進機會
3. 負載測試：
   - 尖峰負載下的壓力測試
   - 記憶體洩漏的浸泡測試
   - 突發處理的尖峰測試
4. APM 分析：
   - 分散式追蹤分析
   - 慢查詢檢測
   - N+1 查詢模式
5. 資源效能分析：
   - CPU 火焰圖
   - 記憶體分配追蹤
   - Goroutine/執行緒洩漏
6. 正式環境準備就緒：
   - 容量規劃影響
   - 擴展特性
   - 成本影響（雲端資源）

現代化效能實踐：
- OpenTelemetry 檢測
- 持續效能分析（Pyroscope、pprof）
- 真實使用者監控（RUM）
- 合成監控
```

**預期輸出：**
```
PERFORMANCE_BASELINE: {
  response_time_p95: "Xms",
  throughput: "Y req/s",
  cpu_usage: "Z%",
  memory_usage: "W MB"
}
PERFORMANCE_AFTER_FIX: {
  response_time_p95: "Xms (delta)",
  throughput: "Y req/s (delta)",
  cpu_usage: "Z% (delta)",
  memory_usage: "W MB (delta)"
}
PERFORMANCE_IMPACT: {
  verdict: "改進|中性|降級",
  acceptable: true/false,
  reasoning: "..."
}
LOAD_TEST_RESULTS: {
  max_throughput: "...",
  breaking_point: "...",
  memory_leaks: "無|檢測到"
}
APM_INSIGHTS: [慢查詢、N+1 模式、瓶頸]
PRODUCTION_READY: {是/否 + 如果否則提供阻礙因素}
```

**第三：Code-Reviewer 最終批准**

**提示：**
```
執行最終程式碼審查並批准部署：

來自測試的情境：
- 測試結果：{TEST_RESULTS}
- 檢測到回歸：{REGRESSION_DETECTED}
- 效能影響：{PERFORMANCE_IMPACT}
- 安全掃描：{SECURITY_SCAN}

交付成果：
1. 程式碼品質審查：
   - 遵循專案慣例
   - 無程式碼異味或反模式
   - 適當的錯誤處理
   - 充足的日誌記錄和可觀測性
2. 架構審查：
   - 維護系統邊界
   - 未引入緊密耦合
   - 可擴展性考量
3. 安全性審查：
   - 無安全漏洞
   - 適當的輸入驗證
   - 身份驗證/授權正確
4. 文件審查：
   - 需要時的程式碼註解
   - 更新的 API 文件
   - 如有運營影響則更新操作手冊
5. 部署準備就緒：
   - 記錄回滾計畫
   - 定義功能旗標策略
   - 配置監控/告警
6. 風險評估：
   - 影響範圍估計
   - 推出策略建議
   - 定義成功指標

審查檢查清單：
- 所有測試通過
- 無效能回歸
- 安全漏洞已解決
- 中斷性變更已記錄
- 維護向後相容性
- 充足的可觀測性
- 清晰的部署計畫
```

**預期輸出：**
```
REVIEW_STATUS: {已批准|需要修訂|被阻擋}
CODE_QUALITY: {分數/評估}
ARCHITECTURE_CONCERNS: [列出或「無」]
SECURITY_CONCERNS: [列出或「無」]
DEPLOYMENT_RISK: {低|中|高}
ROLLBACK_PLAN: {
  steps: ["..."],
  estimated_time: "X 分鐘",
  data_recovery: "..."
}
ROLLOUT_STRATEGY: {
  approach: "canary|blue-green|rolling|big-bang",
  phases: ["..."],
  success_metrics: ["..."],
  abort_criteria: ["..."]
}
MONITORING_REQUIREMENTS: [
  {metric: "...", threshold: "...", action: "..."}
]
FINAL_VERDICT: {
  approved: true/false,
  blockers: [如未批准則列出],
  recommendations: ["..."]
}
```

## 階段 5：文件和預防 - 長期彈性

使用 Task 工具，subagent_type="comprehensive-review::code-reviewer" 進行預防策略：

**提示：**
```
記錄修復並實施預防策略以避免再次發生：

來自階段 4 的情境：
- 最終裁決：{FINAL_VERDICT}
- 審查狀態：{REVIEW_STATUS}
- 根本原因：{ROOT_CAUSE}
- 回滾計畫：{ROLLBACK_PLAN}
- 監控要求：{MONITORING_REQUIREMENTS}

交付成果：
1. 程式碼文件：
   - 非顯而易見邏輯的內聯註解（最少）
   - 函式/類別文件更新
   - API 合約文件
2. 運營文件：
   - 帶修復描述和版本的 CHANGELOG 條目
   - 利害關係人的發布說明
   - 待命工程師的操作手冊條目
   - 事後檢討文件（如果是高嚴重程度事件）
3. 透過靜態分析進行預防：
   - 添加 linting 規則（eslint、ruff、golangci-lint）
   - 配置更嚴格的編譯器/型別檢查器設定
   - 添加領域特定模式的自訂 lint 規則
   - 更新 pre-commit 鉤子
4. 型別系統增強：
   - 添加窮盡性檢查
   - 使用區別聯合/和類型
   - 添加 const/readonly 修飾符
   - 利用品牌型別進行驗證
5. 監控和告警：
   - 建立錯誤率告警（Sentry、DataDog）
   - 添加業務邏輯的自訂指標
   - 設定合成監控（Pingdom、Checkly）
   - 配置 SLO/SLI 儀表板
6. 架構改進：
   - 識別類似的漏洞模式
   - 提議重構以更好地隔離
   - 記錄設計決策
   - 如需要則更新架構圖
7. 測試改進：
   - 添加基於屬性的測試
   - 擴展整合測試場景
   - 添加混沌工程測試
   - 記錄測試策略缺口

現代化預防實踐（2024/2025）：
- AI 輔助程式碼審查規則（GitHub Copilot、Claude Code）
- 持續安全掃描（Snyk、Dependabot）
- 基礎設施即程式碼驗證（Terraform validate、CloudFormation Linter）
- API 的合約測試（Pact、OpenAPI 驗證）
- 可觀測性驅動開發（在部署前進行檢測）
```

**預期輸出：**
```
DOCUMENTATION_UPDATES: [
  {file: "CHANGELOG.md", summary: "..."},
  {file: "docs/runbook.md", summary: "..."},
  {file: "docs/architecture.md", summary: "..."}
]
PREVENTION_MEASURES: {
  static_analysis: [
    {tool: "eslint", rule: "...", reason: "..."},
    {tool: "ruff", rule: "...", reason: "..."}
  ],
  type_system: [
    {enhancement: "...", location: "...", benefit: "..."}
  ],
  pre_commit_hooks: [
    {hook: "...", purpose: "..."}
  ]
}
MONITORING_ADDED: {
  alerts: [
    {name: "...", threshold: "...", channel: "..."}
  ],
  dashboards: [
    {name: "...", metrics: [...], url: "..."}
  ],
  slos: [
    {service: "...", sli: "...", target: "...", window: "..."}
  ]
}
ARCHITECTURAL_IMPROVEMENTS: [
  {improvement: "...", reasoning: "...", effort: "小|中|大"}
]
SIMILAR_VULNERABILITIES: {
  found: N,
  locations: [...],
  remediation_plan: "..."
}
FOLLOW_UP_TASKS: [
  {task: "...", priority: "高|中|低", owner: "..."}
]
POSTMORTEM: {
  created: true/false,
  location: "...",
  incident_severity: "SEV1|SEV2|SEV3|SEV4"
}
KNOWLEDGE_BASE_UPDATES: [
  {article: "...", summary: "..."}
]
```

## 複雜問題的多領域協調

對於跨多個領域的問題，透過明確的情境傳遞依序編排專業代理：

**範例 1：導致應用程式逾時的資料庫效能問題**

**順序：**
1. **階段 1-2**：error-detective + debugger 識別慢資料庫查詢
2. **階段 3a**：Task(subagent_type="database-cloud-optimization::database-optimizer")
   - 使用適當的索引最佳化查詢
   - 情境：「查詢執行需要 5 秒，user_id 欄位上缺少索引，檢測到 N+1 查詢模式」
3. **階段 3b**：Task(subagent_type="application-performance::performance-engineer")
   - 為頻繁存取的資料添加快取層
   - 情境：「透過在 user_id 欄位上添加索引，資料庫查詢從 5 秒最佳化到 50 毫秒。由於 N+1 查詢模式每個請求載入 100+ 使用者記錄，應用程式仍然經歷 2 秒的回應時間。為使用者設定檔添加 5 分鐘 TTL 的 Redis 快取。」
4. **階段 3c**：Task(subagent_type="incident-response::devops-troubleshooter")
   - 配置查詢效能和快取命中率的監控
   - 情境：「已使用 Redis 添加快取層。需要監控：查詢 p95 延遲（閾值：100ms）、快取命中率（閾值：>80%）、快取記憶體使用（在 80% 時告警）。」

**範例 2：正式環境中的前端 JavaScript 錯誤**

**順序：**
1. **階段 1**：error-detective 分析 Sentry 錯誤報告
   - 情境：「TypeError: Cannot read property 'map' of undefined，過去一小時 500+ 次出現，影響 iOS 14 上的 Safari 使用者」
2. **階段 2**：debugger + code-reviewer 調查
   - 情境：「當沒有結果時，API 回應有時返回 null 而不是空陣列。前端假定為陣列。」
3. **階段 3a**：Task(subagent_type="javascript-typescript::typescript-pro")
   - 使用適當的 null 檢查修復前端
   - 添加型別保護
   - 情境：「後端 API /api/users 端點在沒有結果時返回 null 而不是 []。修復前端以處理兩者。添加 TypeScript 嚴格 null 檢查。」
4. **階段 3b**：Task(subagent_type="backend-development::backend-architect")
   - 修復後端以始終返回陣列
   - 更新 API 合約
   - 情境：「前端現在處理 null，但 API 應該遵循合約並返回 [] 而不是 null。更新 OpenAPI 規範以記錄這一點。」
5. **階段 4**：test-automator 執行跨瀏覽器測試
6. **階段 5**：code-reviewer 記錄 API 合約變更

**範例 3：身份驗證中的安全漏洞**

**順序：**
1. **階段 1**：error-detective 審查安全掃描報告
   - 情境：「登入端點中的 SQL 注入漏洞，Snyk 嚴重程度：高」
2. **階段 2**：debugger + security-auditor 調查
   - 情境：「SQL WHERE 子句中的使用者輸入未經清理，允許身份驗證繞過」
3. **階段 3**：Task(subagent_type="security-scanning::security-auditor")
   - 實施參數化查詢
   - 添加輸入驗證
   - 添加速率限制
   - 情境：「用準備語句替換字串串聯。添加電子郵件格式的輸入驗證。實施速率限制（每 15 分鐘 5 次嘗試）。」
4. **階段 4a**：test-automator 添加安全性測試
   - SQL 注入嘗試
   - 暴力破解場景
5. **階段 4b**：security-auditor 執行滲透測試
6. **階段 5**：code-reviewer 記錄安全性改進並建立事後檢討

**情境傳遞範本：**
```
{next_agent} 的情境：

由 {previous_agent} 完成：
- {工作摘要}
- {關鍵發現}
- {所做的變更}

剩餘工作：
- {下一個代理的特定任務}
- {要修改的檔案}
- {要遵循的限制}

相依性：
- {受影響的系統或元件}
- {所需的資料}
- {整合點}

成功標準：
- {可衡量的結果}
- {驗證步驟}
```

## 配置選項

透過在呼叫時設定優先順序來自訂工作流程行為：

**VERIFICATION_LEVEL**：控制測試和驗證的深度
- **minimal**：帶基本測試的快速修復，跳過效能基準測試
  - 用於：低風險錯誤、外觀問題、文件修復
  - 階段：1-2-3（跳過詳細的階段 4）
  - 時間軸：約 30 分鐘
- **standard**：完整測試覆蓋 + 程式碼審查（預設）
  - 用於：大多數正式環境錯誤、功能問題、資料錯誤
  - 階段：1-2-3-4（所有驗證）
  - 時間軸：約 2-4 小時
- **comprehensive**：標準 + 安全稽核 + 效能基準測試 + 混沌測試
  - 用於：安全問題、效能問題、資料損毀、高流量系統
  - 階段：1-2-3-4-5（包括長期預防）
  - 時間軸：約 1-2 天

**PREVENTION_FOCUS**：控制對未來預防的投資
- **none**：僅修復，無預防工作
  - 用於：一次性問題、正在淘汰的舊程式碼、外部程式庫錯誤
  - 輸出：程式碼修復 + 測試僅
- **immediate**：添加測試和基本 linting（預設）
  - 用於：常見錯誤、重複模式、團隊程式碼庫
  - 輸出：修復 + 測試 + linting 規則 + 最少監控
- **comprehensive**：帶監控、架構改進的完整預防套件
  - 用於：高嚴重程度事件、系統性問題、架構問題
  - 輸出：修復 + 測試 + linting + 監控 + 架構文件 + 事後檢討

**ROLLOUT_STRATEGY**：控制部署方法
- **immediate**：直接部署到正式環境（用於熱修復、低風險變更）
- **canary**：漸進式推出到流量子集（中風險的預設）
- **blue-green**：帶即時回滾能力的完整環境切換
- **feature-flag**：部署程式碼但透過功能旗標控制啟動（高風險變更）

**OBSERVABILITY_LEVEL**：控制檢測深度
- **minimal**：僅基本錯誤日誌記錄
- **standard**：結構化日誌 + 關鍵指標（預設）
- **comprehensive**：完整分散式追蹤 + 自訂儀表板 + SLO

**範例呼叫：**
```
問題：使用者在結帳頁面上遇到逾時錯誤（500+ 錯誤/小時）

配置：
- VERIFICATION_LEVEL: comprehensive（影響營收）
- PREVENTION_FOCUS: comprehensive（高業務影響）
- ROLLOUT_STRATEGY: canary（先在 5% 流量上測試）
- OBSERVABILITY_LEVEL: comprehensive（需要詳細監控）
```

## 現代化除錯工具整合

此工作流程利用現代化 2024/2025 工具：

**可觀測性平台：**
- Sentry（錯誤追蹤、發布追蹤、效能監控）
- DataDog（APM、日誌、追蹤、基礎設施監控）
- OpenTelemetry（供應商中立的分散式追蹤）
- Honeycomb（複雜分散式系統的可觀測性）
- New Relic（APM、合成監控）

**AI 輔助除錯：**
- GitHub Copilot（程式碼建議、測試生成、錯誤模式識別）
- Claude Code（全面的程式碼分析、架構審查）
- Sourcegraph Cody（程式碼庫搜尋和理解）
- Tabnine（帶錯誤預防的程式碼補全）

**Git 和版本控制：**
- 帶重現腳本的自動 git bisect
- 用於 bisect 提交自動化測試的 GitHub Actions
- 用於識別程式碼所有權的 Git blame 分析
- 用於理解變更的提交訊息分析

**測試框架：**
- Jest/Vitest（JavaScript/TypeScript 單元/整合測試）
- pytest（帶夾具和參數化的 Python 測試）
- Go testing + testify（Go 單元和表驅動測試）
- Playwright/Cypress（端到端瀏覽器測試）
- k6/Locust（負載和效能測試）

**靜態分析：**
- ESLint/Prettier（JavaScript/TypeScript linting 和格式化）
- Ruff/mypy（Python linting 和型別檢查）
- golangci-lint（Go 全面 linting）
- Clippy（Rust linting 和最佳實踐）
- SonarQube（企業級程式碼品質和安全性）

**效能分析：**
- Chrome DevTools（前端效能）
- pprof（Go 效能分析）
- py-spy（Python 效能分析）
- Pyroscope（持續效能分析）
- CPU/記憶體分析的火焰圖

**安全掃描：**
- Snyk（相依性漏洞掃描）
- Dependabot（自動化相依性更新）
- OWASP ZAP（安全性測試）
- Semgrep（自訂安全規則）
- npm audit / pip-audit / cargo audit

## 成功標準

當滿足以下所有條件時，修復被視為完成：

**根本原因理解：**
- 根本原因已識別並有支持證據
- 故障機制已清楚記錄
- 引入的提交已識別（如果適用，透過 git bisect）
- 類似漏洞已編目

**修復品質：**
- 修復解決根本原因，而非僅僅症狀
- 最小的程式碼變更（避免過度工程）
- 遵循專案慣例和模式
- 未引入程式碼異味或反模式
- 維護向後相容性（或記錄中斷性變更）

**測試驗證：**
- 所有現有測試通過（零回歸）
- 新測試涵蓋特定錯誤重現
- 測試邊緣案例和錯誤路徑
- 整合測試驗證端到端行為
- 測試覆蓋率提高（或維持在高水平）

**效能與安全性：**
- 無效能降級（p95 延遲在基準線的 5% 以內）
- 未引入安全漏洞
- 可接受的資源使用（記憶體、CPU、I/O）
- 高流量變更通過負載測試

**部署準備就緒：**
- 領域專家批准程式碼審查
- 記錄並測試回滾計畫
- 配置功能旗標（如適用）
- 配置監控和告警
- 使用故障排除步驟更新操作手冊

**預防措施：**
- 添加靜態分析規則（如適用）
- 實施型別系統改進（如適用）
- 更新文件（程式碼、API、操作手冊）
- 建立事後檢討（如果是高嚴重程度事件）
- 建立知識庫文章（如果是新問題）

**指標：**
- 平均恢復時間（MTTR）：SEV2+ < 4 小時
- 錯誤復發率：0%（相同根本原因不應再次發生）
- 測試覆蓋率：無減少，理想情況下增加
- 部署成功率：> 95%（回滾率 < 5%）

要解決的問題：$ARGUMENTS
