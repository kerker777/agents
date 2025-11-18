跨後端、前端和基礎設施層協調全端功能開發，採用現代 API 優先方法：

[延伸思考：此工作流程協調多個專業 agents，透過架構到部署交付完整的全端功能。它遵循 API 優先開發原則，確保契約驅動開發，其中 API 規格驅動後端實作和前端消費。每個階段建立在先前輸出之上，建立具有適當關注點分離、全面測試和準備部署到正式環境的一致系統。工作流程強調現代實務，如元件驅動的 UI 開發、功能旗標、可觀測性和漸進式推出策略。]

## 階段 1：架構與設計基礎

### 1. 資料庫架構設計
- 使用 Task 工具，subagent_type="database-design::database-architect"
- 提示："為以下設計資料庫 schema 和資料模型：$ARGUMENTS。考慮可擴展性、查詢模式、索引策略和資料一致性要求。如果修改現有 schema，請包含遷移策略。提供邏輯和實體資料模型。"
- 預期輸出：實體關係圖、資料表 schema、索引策略、遷移腳本、資料存取模式
- 情境：初始需求和業務領域模型

### 2. 後端服務架構
- 使用 Task 工具，subagent_type="backend-development::backend-architect"
- 提示："為以下設計後端服務架構：$ARGUMENTS。使用前一步驟的資料庫設計，建立服務邊界、定義 API 契約（OpenAPI/GraphQL）、設計認證/授權策略，並指定服務間通訊模式。包含韌性模式（斷路器、重試）和快取策略。"
- 預期輸出：服務架構圖、OpenAPI 規格、認證流程、快取架構、訊息佇列設計（如適用）
- 情境：步驟 1 的資料庫 schema、非功能需求

### 3. 前端元件架構
- 使用 Task 工具，subagent_type="frontend-mobile-development::frontend-developer"
- 提示："為以下設計前端架構和元件結構：$ARGUMENTS。基於前一步驟的 API 契約，設計元件階層、狀態管理方法（Redux/Zustand/Context）、路由結構和資料擷取模式。包含無障礙要求和響應式設計策略。規劃 Storybook 元件文件。"
- 預期輸出：元件樹狀圖、狀態管理設計、路由設定、設計系統整合計畫、無障礙檢查清單
- 情境：步驟 2 的 API 規格、UI/UX 需求

## 階段 2：平行實作

### 4. 後端服務實作
- 使用 Task 工具，subagent_type="python-development::python-pro"（或 "golang-pro"/"nodejs-expert" 基於技術堆疊）
- 提示："實作以下後端服務：$ARGUMENTS。使用階段 1 的架構和 API 規格，建構 RESTful/GraphQL 端點，包含適當的驗證、錯誤處理和日誌記錄。實作業務邏輯、資料存取層、認證中介軟體和與外部服務的整合。包含可觀測性（結構化日誌記錄、指標、追蹤）。"
- 預期輸出：後端服務程式碼、API 端點、中介軟體、背景工作、單元測試、整合測試
- 情境：階段 1 的架構設計、資料庫 schema

### 5. 前端實作
- 使用 Task 工具，subagent_type="frontend-mobile-development::frontend-developer"
- 提示："實作以下前端應用程式：$ARGUMENTS。使用階段 1 的元件架構建構 React/Next.js 元件。實作狀態管理、包含適當錯誤處理和載入狀態的 API 整合、表單驗證和響應式版面。為元件建立 Storybook stories。確保無障礙（WCAG 2.1 AA 合規）。"
- 預期輸出：React 元件、狀態管理實作、API 客戶端程式碼、Storybook stories、響應式樣式、無障礙實作
- 情境：步驟 3 的元件架構、API 契約

### 6. 資料庫實作與最佳化
- 使用 Task 工具，subagent_type="database-design::sql-pro"
- 提示："實作並最佳化以下資料庫層：$ARGUMENTS。建立遷移腳本、預存程序（如需要）、最佳化後端實作識別的查詢、設定適當的索引，並實作資料驗證約束。包含資料庫層級的安全措施和備份策略。"
- 預期輸出：遷移腳本、最佳化的查詢、預存程序、索引定義、資料庫安全設定
- 情境：步驟 1 的資料庫設計、後端實作的查詢模式

## 階段 3：整合與測試

### 7. API 契約測試
- 使用 Task 工具，subagent_type="test-automator"
- 提示："為以下建立契約測試：$ARGUMENTS。實作 Pact/Dredd 測試，以驗證後端和前端之間的 API 契約。為所有 API 端點建立整合測試、測試認證流程、驗證錯誤回應，並確保適當的 CORS 設定。包含負載測試情境。"
- 預期輸出：契約測試套件、整合測試、負載測試情境、API 文件驗證
- 情境：階段 2 的 API 實作

### 8. 端對端測試
- 使用 Task 工具，subagent_type="test-automator"
- 提示："實作以下端對端測試：$ARGUMENTS。建立 Playwright/Cypress 測試，涵蓋關鍵使用者旅程、跨瀏覽器相容性、行動響應性和錯誤情境。測試功能旗標整合、分析追蹤和效能指標。包含視覺回歸測試。"
- 預期輸出：E2E 測試套件、視覺回歸基準、效能基準、測試報告
- 情境：階段 2 的前端和後端實作

### 9. 安全稽核與強化
- 使用 Task 工具，subagent_type="security-auditor"
- 提示："執行以下安全稽核：$ARGUMENTS。檢查 API 安全（認證、授權、速率限制）、檢查 OWASP Top 10 漏洞、稽核前端的 XSS/CSRF 風險、驗證輸入淨化，並檢查密鑰管理。提供滲透測試結果和修復步驟。"
- 預期輸出：安全稽核報告、漏洞評估、修復建議、安全 headers 設定
- 情境：階段 2 的所有實作

## 階段 4：部署與維運

### 10. 基礎設施與 CI/CD 設定
- 使用 Task 工具，subagent_type="deployment-engineer"
- 提示："設定以下部署基礎設施：$ARGUMENTS。建立 Docker 容器、Kubernetes manifests（或雲端特定設定）、實作包含自動化測試閘道的 CI/CD 流程、設定功能旗標（LaunchDarkly/Unleash），並設定監控/警報。包含 blue-green 部署策略和回滾程序。"
- 預期輸出：Dockerfiles、K8s manifests、CI/CD 流程設定、功能旗標設定、IaC 範本（Terraform/CloudFormation）
- 情境：前面階段的所有實作和測試

### 11. 可觀測性與監控
- 使用 Task 工具，subagent_type="deployment-engineer"
- 提示："實作以下可觀測性堆疊：$ARGUMENTS。設定分散式追蹤（OpenTelemetry）、設定應用程式指標（Prometheus/DataDog）、實作集中式日誌記錄（ELK/Splunk）、為關鍵指標建立儀表板，並定義 SLI/SLO。包含警報規則和待命程序。"
- 預期輸出：可觀測性設定、儀表板定義、警報規則、runbooks、SLI/SLO 定義
- 情境：步驟 10 的基礎設施設定

### 12. 效能最佳化
- 使用 Task 工具，subagent_type="performance-engineer"
- 提示："最佳化以下堆疊的效能：$ARGUMENTS。分析並最佳化資料庫查詢、實作快取策略（Redis/CDN）、最佳化前端 bundle 大小和載入效能、設定延遲載入和程式碼分割，並調校後端服務效能。包含前後指標。"
- 預期輸出：效能改善、快取設定、CDN 設定、最佳化的 bundles、效能指標報告
- 情境：步驟 11 的監控資料、負載測試結果

## 設定選項
- `stack`：指定技術堆疊（例如 "React/FastAPI/PostgreSQL"、"Next.js/Django/MongoDB"）
- `deployment_target`：雲端平台（AWS/GCP/Azure）或地端
- `feature_flags`：啟用/停用功能旗標整合
- `api_style`：REST 或 GraphQL
- `testing_depth`：全面或基本
- `compliance`：特定合規要求（GDPR、HIPAA、SOC2）

## 成功標準
- 所有 API 契約透過契約測試驗證
- 前端和後端整合測試通過
- E2E 測試涵蓋關鍵使用者旅程
- 安全稽核通過，無嚴重漏洞
- 效能指標符合定義的 SLO
- 可觀測性堆疊擷取所有關鍵指標
- 功能旗標設定用於漸進式推出
- 所有元件的文件完整
- CI/CD 流程包含自動化品質閘道
- 零停機時間部署能力已驗證

## 協調備註
- 每個階段建立在前面階段的輸出之上
- 階段 2 的平行任務可以同時執行，但必須為階段 3 收斂
- 維護需求和實作之間的可追蹤性
- 在所有服務中使用關聯 ID 進行分散式追蹤
- 在 ADR 中記錄所有架構決策
- 確保跨服務的一致錯誤處理和 API 回應

要實作的功能：$ARGUMENTS
