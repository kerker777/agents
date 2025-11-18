協調從需求到正式環境部署的端到端功能開發：

[延伸思考：此工作流程透過完整的功能開發階段來協調專業代理，從探索和規劃到實作、測試和部署。每個階段都建立在先前的輸出之上，確保功能交付的一致性。此工作流程支援多種開發方法論（傳統式、TDD/BDD、DDD）、功能複雜度等級，以及現代化的部署策略，包括功能旗標、漸進式推出和觀測性優先的開發方式。代理會接收前一階段的詳細情境資訊，以在整個開發生命週期中保持一致性和品質。]

## 配置選項

### 開發方法論
- **traditional**: 實作後進行測試的循序開發方式
- **tdd**: 測試驅動開發，採用紅燈-綠燈-重構循環
- **bdd**: 行為驅動開發，採用情境式測試
- **ddd**: 領域驅動設計，採用限界上下文和聚合

### 功能複雜度
- **simple**: 單一服務，最少整合（1-2 天）
- **medium**: 多個服務，中等整合（3-5 天）
- **complex**: 跨領域，大量整合（1-2 週）
- **epic**: 重大架構變更，多個團隊（2 週以上）

### 部署策略
- **direct**: 立即向所有使用者推出
- **canary**: 從 5% 流量開始的漸進式推出
- **feature-flag**: 透過功能開關進行控制式啟用
- **blue-green**: 零停機部署，可立即回滾
- **a-b-test**: 分流流量進行實驗和指標追蹤

## 階段 1：探索與需求規劃

1. **業務分析與需求**
   - 使用 Task 工具，設定 subagent_type="business-analytics::business-analyst"
   - 提示：「分析功能需求：$ARGUMENTS。定義使用者故事、驗收標準、成功指標和業務價值。識別利害關係人、相依性和風險。建立具有明確範圍邊界的功能規格文件。」
   - 預期輸出：包含使用者故事、成功指標、風險評估的需求文件
   - 情境：初始功能請求和業務情境

2. **技術架構設計**
   - 使用 Task 工具，設定 subagent_type="comprehensive-review::architect-review"
   - 提示：「設計功能的技術架構：$ARGUMENTS。使用需求：[包含步驟 1 的業務分析]。定義服務邊界、API 合約、資料模型、整合點和技術堆疊。考慮可擴展性、效能和安全性需求。」
   - 預期輸出：包含架構圖、API 規格、資料模型的技術設計文件
   - 情境：業務需求、現有系統架構

3. **可行性與風險評估**
   - 使用 Task 工具，設定 subagent_type="security-scanning::security-auditor"
   - 提示：「評估功能的安全性影響和風險：$ARGUMENTS。檢視架構：[包含步驟 2 的技術設計]。識別安全性需求、合規需求、資料隱私疑慮和潛在漏洞。」
   - 預期輸出：包含風險矩陣、合規檢查清單、緩解策略的安全性評估
   - 情境：技術設計、法規要求

## 階段 2：實作與開發

4. **後端服務實作**
   - 使用 Task 工具，設定 subagent_type="backend-architect"
   - 提示：「實作後端服務：$ARGUMENTS。遵循技術設計：[包含步驟 2 的架構]。建構 RESTful/GraphQL API，實作業務邏輯，整合資料層，加入韌性模式（斷路器、重試機制），實作快取策略。包含用於漸進式推出的功能旗標。」
   - 預期輸出：包含 API、業務邏輯、資料庫整合、功能旗標的後端服務
   - 情境：技術設計、API 合約、資料模型

5. **前端實作**
   - 使用 Task 工具，設定 subagent_type="frontend-mobile-development::frontend-developer"
   - 提示：「建構前端元件：$ARGUMENTS。整合後端 API：[包含步驟 4 的 API 端點]。實作響應式 UI、狀態管理、錯誤處理、載入狀態和分析追蹤。加入功能旗標整合以支援 A/B 測試能力。」
   - 預期輸出：包含 API 整合、狀態管理、分析的前端元件
   - 情境：後端 API、UI/UX 設計、使用者故事

6. **資料管線與整合**
   - 使用 Task 工具，設定 subagent_type="data-engineering::data-engineer"
   - 提示：「建構資料管線：$ARGUMENTS。設計 ETL/ELT 流程，實作資料驗證，建立分析事件，設定資料品質監控。整合產品分析平台以追蹤功能使用情況。」
   - 預期輸出：資料管線、分析事件、資料品質檢查
   - 情境：資料需求、分析需求、現有資料基礎設施

## 階段 3：測試與品質保證

7. **自動化測試套件**
   - 使用 Task 工具，設定 subagent_type="unit-testing::test-automator"
   - 提示：「建立完整的測試套件：$ARGUMENTS。為後端：[來自步驟 4] 和前端：[來自步驟 5] 撰寫單元測試。加入 API 端點的整合測試、關鍵使用者旅程的端到端測試、可擴展性驗證的效能測試。確保至少 80% 的程式碼覆蓋率。」
   - 預期輸出：包含單元、整合、端到端和效能測試的測試套件
   - 情境：實作程式碼、驗收標準、測試需求

8. **安全性驗證**
   - 使用 Task 工具，設定 subagent_type="security-scanning::security-auditor"
   - 提示：「執行安全性測試：$ARGUMENTS。檢視實作：[包含步驟 4-5 的後端和前端]。執行 OWASP 檢查、滲透測試、相依性掃描和合規驗證。驗證資料加密、身分驗證和授權。」
   - 預期輸出：安全性測試結果、漏洞報告、修復行動
   - 情境：實作程式碼、安全性需求

9. **效能最佳化**
   - 使用 Task 工具，設定 subagent_type="application-performance::performance-engineer"
   - 提示：「最佳化效能：$ARGUMENTS。分析後端服務：[來自步驟 4] 和前端：[來自步驟 5]。分析程式碼、最佳化查詢、實作快取、減少打包大小、改善載入時間。設定效能預算和監控。」
   - 預期輸出：效能改進、最佳化報告、效能指標
   - 情境：實作程式碼、效能需求

## 階段 4：部署與監控

10. **部署策略與管線**
    - 使用 Task 工具，設定 subagent_type="deployment-strategies::deployment-engineer"
    - 提示：「準備部署：$ARGUMENTS。建立包含自動化測試的 CI/CD 管線：[來自步驟 7]。配置用於漸進式推出的功能旗標，實作藍綠部署，設定回滾程序。建立部署操作手冊和回滾計畫。」
    - 預期輸出：CI/CD 管線、部署配置、回滾程序
    - 情境：測試套件、基礎設施需求、部署策略

11. **觀測性與監控**
    - 使用 Task 工具，設定 subagent_type="observability-monitoring::observability-engineer"
    - 提示：「設定觀測性：$ARGUMENTS。實作分散式追蹤、自訂指標、錯誤追蹤和告警。建立功能使用情況、效能指標、錯誤率和業務 KPI 的儀表板。設定帶有自動告警的 SLO/SLI。」
    - 預期輸出：監控儀表板、告警、SLO 定義、觀測性基礎設施
    - 情境：功能實作、成功指標、營運需求

12. **文件與知識轉移**
    - 使用 Task 工具，設定 subagent_type="documentation-generation::docs-architect"
    - 提示：「產生完整的文件：$ARGUMENTS。建立 API 文件、使用者指南、部署指南、問題排解操作手冊。包含架構圖、資料流程圖和整合指南。從提交記錄產生自動化變更日誌。」
    - 預期輸出：API 文件、使用者指南、操作手冊、架構文件
    - 情境：所有先前階段的輸出

## 執行參數

### 必要參數
- **--feature**: 功能名稱和描述
- **--methodology**: 開發方法（traditional|tdd|bdd|ddd）
- **--complexity**: 功能複雜度等級（simple|medium|complex|epic）

### 選用參數
- **--deployment-strategy**: 部署方法（direct|canary|feature-flag|blue-green|a-b-test）
- **--test-coverage-min**: 最低測試覆蓋率門檻（預設：80%）
- **--performance-budget**: 效能需求（例如：<200ms 回應時間）
- **--rollout-percentage**: 漸進式部署的初始推出百分比（預設：5%）
- **--feature-flag-service**: 功能旗標提供者（launchdarkly|split|unleash|custom）
- **--analytics-platform**: 分析整合（segment|amplitude|mixpanel|custom）
- **--monitoring-stack**: 觀測性工具（datadog|newrelic|grafana|custom）

## 成功標準

- 符合業務需求的所有驗收標準
- 測試覆蓋率超過最低門檻（預設 80%）
- 安全性掃描顯示無重大漏洞
- 效能符合定義的預算和 SLO
- 功能旗標已配置用於控制式推出
- 監控和告警完全運作
- 文件完整且已核准
- 成功部署到正式環境並具備回滾能力
- 產品分析追蹤功能使用情況
- 已配置 A/B 測試指標（如適用）

## 回滾策略

如果在部署期間或之後發生問題：
1. 立即停用功能旗標（< 1 分鐘）
2. 藍綠流量切換（< 5 分鐘）
3. 透過 CI/CD 完全回滾部署（< 15 分鐘）
4. 如有需要進行資料庫遷移回滾（與資料團隊協調）
5. 事後檢討並修復問題後再重新部署

功能描述：$ARGUMENTS
