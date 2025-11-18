# Agent 參考文件

完整參考所有 **86 個專門的 AI agents**，依類別組織並包含模型分配。

## Agent 類別

### 架構與系統設計

#### 核心架構

| Agent | Model | 描述 |
|-------|-------|-------------|
| [backend-architect](../plugins/backend-development/agents/backend-architect.md) | opus | RESTful API 設計、微服務邊界、資料庫架構 |
| [frontend-developer](../plugins/multi-platform-apps/agents/frontend-developer.md) | sonnet | React 元件、響應式版面配置、客戶端狀態管理 |
| [graphql-architect](../plugins/backend-development/agents/graphql-architect.md) | opus | GraphQL 架構、解析器、聯邦架構 |
| [architect-reviewer](../plugins/comprehensive-review/agents/architect-review.md) | opus | 架構一致性分析與模式驗證 |
| [cloud-architect](../plugins/cloud-infrastructure/agents/cloud-architect.md) | opus | AWS/Azure/GCP 基礎設施設計與成本最佳化 |
| [hybrid-cloud-architect](../plugins/cloud-infrastructure/agents/hybrid-cloud-architect.md) | opus | 跨雲端與地端環境的多雲策略 |
| [kubernetes-architect](../plugins/kubernetes-operations/agents/kubernetes-architect.md) | opus | 基於 Kubernetes 和 GitOps 的雲原生基礎設施 |

#### UI/UX 與行動開發

| Agent | Model | 描述 |
|-------|-------|-------------|
| [ui-ux-designer](../plugins/multi-platform-apps/agents/ui-ux-designer.md) | sonnet | 介面設計、線框圖、設計系統 |
| [ui-visual-validator](../plugins/accessibility-compliance/agents/ui-visual-validator.md) | sonnet | 視覺回歸測試與 UI 驗證 |
| [mobile-developer](../plugins/multi-platform-apps/agents/mobile-developer.md) | sonnet | React Native 與 Flutter 應用程式開發 |
| [ios-developer](../plugins/multi-platform-apps/agents/ios-developer.md) | sonnet | 使用 Swift/SwiftUI 的原生 iOS 開發 |
| [flutter-expert](../plugins/multi-platform-apps/agents/flutter-expert.md) | sonnet | 包含狀態管理的進階 Flutter 開發 |

### 程式語言

#### 系統與低階程式設計

| Agent | Model | 描述 |
|-------|-------|-------------|
| [c-pro](../plugins/systems-programming/agents/c-pro.md) | sonnet | 包含記憶體管理與作業系統介面的系統程式設計 |
| [cpp-pro](../plugins/systems-programming/agents/cpp-pro.md) | sonnet | 使用 RAII、智慧指標、STL 演算法的現代 C++ |
| [rust-pro](../plugins/systems-programming/agents/rust-pro.md) | sonnet | 使用所有權模式的記憶體安全系統程式設計 |
| [golang-pro](../plugins/systems-programming/agents/golang-pro.md) | sonnet | 使用 goroutines 與 channels 的並發程式設計 |

#### Web 與應用程式

| Agent | Model | 描述 |
|-------|-------|-------------|
| [javascript-pro](../plugins/javascript-typescript/agents/javascript-pro.md) | sonnet | 使用 ES6+、非同步模式、Node.js 的現代 JavaScript |
| [typescript-pro](../plugins/javascript-typescript/agents/typescript-pro.md) | sonnet | 包含型別系統與泛型的進階 TypeScript |
| [python-pro](../plugins/python-development/agents/python-pro.md) | sonnet | 包含進階功能與最佳化的 Python 開發 |
| [temporal-python-pro](../plugins/backend-development/agents/temporal-python-pro.md) | sonnet | 使用 Python SDK 的 Temporal 工作流程編排、持久性工作流程、saga 模式 |
| [ruby-pro](../plugins/web-scripting/agents/ruby-pro.md) | sonnet | 包含元程式設計、Rails 模式、gem 開發的 Ruby |
| [php-pro](../plugins/web-scripting/agents/php-pro.md) | sonnet | 使用框架與效能最佳化的現代 PHP |

#### 企業與 JVM

| Agent | Model | 描述 |
|-------|-------|-------------|
| [java-pro](../plugins/jvm-languages/agents/java-pro.md) | sonnet | 包含 streams、並發、JVM 最佳化的現代 Java |
| [scala-pro](../plugins/jvm-languages/agents/scala-pro.md) | sonnet | 包含函數式程式設計與分散式系統的企業級 Scala |
| [csharp-pro](../plugins/jvm-languages/agents/csharp-pro.md) | sonnet | 使用 .NET 框架與模式的 C# 開發 |

#### 專業平台

| Agent | Model | 描述 |
|-------|-------|-------------|
| [elixir-pro](../plugins/functional-programming/agents/elixir-pro.md) | sonnet | 包含 OTP 模式與 Phoenix 框架的 Elixir |
| [django-pro](../plugins/api-scaffolding/agents/django-pro.md) | sonnet | 包含 ORM 與非同步視圖的 Django 開發 |
| [fastapi-pro](../plugins/api-scaffolding/agents/fastapi-pro.md) | sonnet | 使用非同步模式與 Pydantic 的 FastAPI |
| [unity-developer](../plugins/game-development/agents/unity-developer.md) | sonnet | Unity 遊戲開發與最佳化 |
| [minecraft-bukkit-pro](../plugins/game-development/agents/minecraft-bukkit-pro.md) | sonnet | Minecraft 伺服器插件開發 |
| [sql-pro](../plugins/database-design/agents/sql-pro.md) | sonnet | 複雜 SQL 查詢與資料庫最佳化 |

### 基礎設施與維運

#### DevOps 與部署

| Agent | Model | 描述 |
|-------|-------|-------------|
| [devops-troubleshooter](../plugins/incident-response/agents/devops-troubleshooter.md) | sonnet | 正式環境除錯、日誌分析、部署疑難排解 |
| [deployment-engineer](../plugins/cloud-infrastructure/agents/deployment-engineer.md) | sonnet | CI/CD 管線、容器化、雲端部署 |
| [terraform-specialist](../plugins/cloud-infrastructure/agents/terraform-specialist.md) | sonnet | 使用 Terraform 模組與狀態管理的基礎設施即程式碼 |
| [dx-optimizer](../plugins/team-collaboration/agents/dx-optimizer.md) | sonnet | 開發者體驗最佳化與工具改善 |

#### 資料庫管理

| Agent | Model | 描述 |
|-------|-------|-------------|
| [database-optimizer](../plugins/observability-monitoring/agents/database-optimizer.md) | sonnet | 查詢最佳化、索引設計、遷移策略 |
| [database-admin](../plugins/database-migrations/agents/database-admin.md) | sonnet | 資料庫操作、備份、複寫、監控 |
| [database-architect](../plugins/database-design/agents/database-architect.md) | opus | 從零開始的資料庫設計、技術選型、架構建模 |

#### 事件響應與網路

| Agent | Model | 描述 |
|-------|-------|-------------|
| [incident-responder](../plugins/incident-response/agents/incident-responder.md) | opus | 正式環境事件管理與解決 |
| [network-engineer](../plugins/observability-monitoring/agents/network-engineer.md) | sonnet | 網路除錯、負載平衡、流量分析 |

### 品質保證與安全性

#### 程式碼品質與審查

| Agent | Model | 描述 |
|-------|-------|-------------|
| [code-reviewer](../plugins/comprehensive-review/agents/code-reviewer.md) | opus | 著重於安全性與正式環境可靠性的程式碼審查 |
| [security-auditor](../plugins/comprehensive-review/agents/security-auditor.md) | opus | 弱點評估與 OWASP 合規性 |
| [backend-security-coder](../plugins/data-validation-suite/agents/backend-security-coder.md) | opus | 安全的後端編碼實務、API 安全實作 |
| [frontend-security-coder](../plugins/frontend-mobile-security/agents/frontend-security-coder.md) | opus | XSS 防護、CSP 實作、客戶端安全性 |
| [mobile-security-coder](../plugins/frontend-mobile-security/agents/mobile-security-coder.md) | opus | 行動安全模式、WebView 安全性、生物辨識驗證 |

#### 測試與除錯

| Agent | Model | 描述 |
|-------|-------|-------------|
| [test-automator](../plugins/codebase-cleanup/agents/test-automator.md) | sonnet | 完整測試套件建立（單元、整合、端對端） |
| [tdd-orchestrator](../plugins/backend-development/agents/tdd-orchestrator.md) | sonnet | 測試驅動開發方法論指導 |
| [debugger](../plugins/error-debugging/agents/debugger.md) | sonnet | 錯誤解決與測試失敗分析 |
| [error-detective](../plugins/error-debugging/agents/error-detective.md) | sonnet | 日誌分析與錯誤模式識別 |

#### 效能與可觀測性

| Agent | Model | 描述 |
|-------|-------|-------------|
| [performance-engineer](../plugins/observability-monitoring/agents/performance-engineer.md) | opus | 應用程式效能分析與最佳化 |
| [observability-engineer](../plugins/observability-monitoring/agents/observability-engineer.md) | opus | 正式環境監控、分散式追蹤、SLI/SLO 管理 |
| [search-specialist](../plugins/content-marketing/agents/search-specialist.md) | haiku | 進階網路研究與資訊整合 |

### 資料與 AI

#### 資料工程與分析

| Agent | Model | 描述 |
|-------|-------|-------------|
| [data-scientist](../plugins/machine-learning-ops/agents/data-scientist.md) | opus | 資料分析、SQL 查詢、BigQuery 操作 |
| [data-engineer](../plugins/data-engineering/agents/data-engineer.md) | sonnet | ETL 管線、資料倉儲、串流架構 |

#### 機器學習與 AI

| Agent | Model | 描述 |
|-------|-------|-------------|
| [ai-engineer](../plugins/llm-application-dev/agents/ai-engineer.md) | opus | LLM 應用程式、RAG 系統、提示詞管線 |
| [ml-engineer](../plugins/machine-learning-ops/agents/ml-engineer.md) | opus | ML 管線、模型服務、特徵工程 |
| [mlops-engineer](../plugins/machine-learning-ops/agents/mlops-engineer.md) | opus | ML 基礎設施、實驗追蹤、模型註冊表 |
| [prompt-engineer](../plugins/llm-application-dev/agents/prompt-engineer.md) | opus | LLM 提示詞最佳化與工程 |

### 文件與技術寫作

| Agent | Model | 描述 |
|-------|-------|-------------|
| [docs-architect](../plugins/code-documentation/agents/docs-architect.md) | opus | 完整技術文件生成 |
| [api-documenter](../plugins/api-testing-observability/agents/api-documenter.md) | sonnet | OpenAPI/Swagger 規範與開發者文件 |
| [reference-builder](../plugins/documentation-generation/agents/reference-builder.md) | haiku | 技術參考文件與 API 文件 |
| [tutorial-engineer](../plugins/code-documentation/agents/tutorial-engineer.md) | sonnet | 逐步教學與教育內容 |
| [mermaid-expert](../plugins/documentation-generation/agents/mermaid-expert.md) | sonnet | 圖表建立（流程圖、序列圖、ERD） |

### 商業與營運

#### 商業分析與財務

| Agent | Model | 描述 |
|-------|-------|-------------|
| [business-analyst](../plugins/business-analytics/agents/business-analyst.md) | sonnet | 指標分析、報表製作、KPI 追蹤 |
| [quant-analyst](../plugins/quantitative-trading/agents/quant-analyst.md) | opus | 財務建模、交易策略、市場分析 |
| [risk-manager](../plugins/quantitative-trading/agents/risk-manager.md) | sonnet | 投資組合風險監控與管理 |

#### 行銷與業務

| Agent | Model | 描述 |
|-------|-------|-------------|
| [content-marketer](../plugins/content-marketing/agents/content-marketer.md) | sonnet | 部落格文章、社群媒體、電子郵件行銷活動 |
| [sales-automator](../plugins/customer-sales-automation/agents/sales-automator.md) | haiku | 開發信、後續追蹤、提案生成 |

#### 客服與法務

| Agent | Model | 描述 |
|-------|-------|-------------|
| [customer-support](../plugins/customer-sales-automation/agents/customer-support.md) | sonnet | 客服工單、FAQ 回覆、客戶溝通 |
| [hr-pro](../plugins/hr-legal-compliance/agents/hr-pro.md) | opus | 人資作業、政策、員工關係 |
| [legal-advisor](../plugins/hr-legal-compliance/agents/legal-advisor.md) | opus | 隱私政策、服務條款、法律文件 |

### SEO 與內容最佳化

| Agent | Model | 描述 |
|-------|-------|-------------|
| [seo-content-auditor](../plugins/seo-content-creation/agents/seo-content-auditor.md) | sonnet | 內容品質分析、E-E-A-T 信號評估 |
| [seo-meta-optimizer](../plugins/seo-technical-optimization/agents/seo-meta-optimizer.md) | haiku | Meta 標題與描述最佳化 |
| [seo-keyword-strategist](../plugins/seo-technical-optimization/agents/seo-keyword-strategist.md) | haiku | 關鍵字分析與語意變化 |
| [seo-structure-architect](../plugins/seo-technical-optimization/agents/seo-structure-architect.md) | haiku | 內容結構與 schema 標記 |
| [seo-snippet-hunter](../plugins/seo-technical-optimization/agents/seo-snippet-hunter.md) | haiku | 精選摘要格式化 |
| [seo-content-refresher](../plugins/seo-analysis-monitoring/agents/seo-content-refresher.md) | haiku | 內容新鮮度分析 |
| [seo-cannibalization-detector](../plugins/seo-analysis-monitoring/agents/seo-cannibalization-detector.md) | haiku | 關鍵字重疊偵測 |
| [seo-authority-builder](../plugins/seo-analysis-monitoring/agents/seo-authority-builder.md) | sonnet | E-E-A-T 信號分析 |
| [seo-content-writer](../plugins/seo-content-creation/agents/seo-content-writer.md) | sonnet | SEO 最佳化內容創作 |
| [seo-content-planner](../plugins/seo-content-creation/agents/seo-content-planner.md) | haiku | 內容規劃與主題叢集 |

### 專業領域

| Agent | Model | 描述 |
|-------|-------|-------------|
| [arm-cortex-expert](../plugins/arm-cortex-microcontrollers/agents/arm-cortex-expert.md) | sonnet | ARM Cortex-M 韌體與周邊驅動程式開發 |
| [blockchain-developer](../plugins/blockchain-web3/agents/blockchain-developer.md) | sonnet | Web3 應用程式、智慧合約、DeFi 協定 |
| [payment-integration](../plugins/payment-processing/agents/payment-integration.md) | sonnet | 金流處理器整合（Stripe、PayPal） |
| [legacy-modernizer](../plugins/framework-migration/agents/legacy-modernizer.md) | sonnet | 舊版程式碼重構與現代化 |
| [context-manager](../plugins/agent-orchestration/agents/context-manager.md) | haiku | 多 agent 上下文管理 |

## 模型設定

Agents 根據任務複雜度與運算需求被分配到特定的 Claude 模型。

### 模型分配摘要

| Model | Agent 數量 | 使用案例 |
|-------|-------------|----------|
| Haiku | 47 | 快速執行任務：測試、文件、維運、資料庫最佳化、商業 |
| Sonnet | 97 | 複雜推理、架構、語言專業、編排、安全性 |

### 模型選擇標準

#### Haiku - 快速執行與確定性任務

**使用時機：**
- 根據明確定義的規格生成程式碼
- 遵循既有模式建立測試
- 使用清晰範本撰寫文件
- 執行基礎設施操作
- 執行資料庫查詢最佳化
- 處理客戶支援回覆
- 處理 SEO 最佳化任務
- 管理部署管線

#### Sonnet - 複雜推理與架構

**使用時機：**
- 設計系統架構
- 進行技術選型決策
- 執行安全性稽核
- 審查程式碼的架構模式
- 建立複雜的 AI/ML 管線
- 提供特定語言的專業知識
- 編排多 agent 工作流程
- 處理業務關鍵的法務/人資事務

### 混合編排模式

Plugin 生態系統運用 Sonnet + Haiku 編排以達到最佳效能與成本效益：

#### 模式 1：規劃 → 執行
```
Sonnet: backend-architect（設計 API 架構）
  ↓
Haiku: 根據規格生成 API 端點
  ↓
Haiku: test-automator（生成完整測試）
  ↓
Sonnet: code-reviewer（架構審查）
```

#### 模式 2：推理 → 行動（事件響應）
```
Sonnet: incident-responder（診斷問題、建立策略）
  ↓
Haiku: devops-troubleshooter（執行修復）
  ↓
Haiku: deployment-engineer（部署熱修復）
  ↓
Haiku: 實作監控告警
```

#### 模式 3：複雜 → 簡單（資料庫設計）
```
Sonnet: database-architect（架構設計、技術選型）
  ↓
Haiku: sql-pro（生成遷移腳本）
  ↓
Haiku: database-admin（執行遷移）
  ↓
Haiku: database-optimizer（調校查詢效能）
```

#### 模式 4：多 Agent 工作流程
```
全端功能開發：
Sonnet: backend-architect + frontend-developer（設計元件）
  ↓
Haiku: 根據設計生成程式碼
  ↓
Haiku: test-automator（單元 + 整合測試）
  ↓
Sonnet: security-auditor（安全性審查）
  ↓
Haiku: deployment-engineer（CI/CD 設定）
  ↓
Haiku: 設定可觀測性堆疊
```

## Agent 呼叫

### 自然語言

當您需要 Claude 推理應該使用哪個專家時，可以透過自然語言呼叫 agents：

```
"使用 backend-architect 來設計身份驗證 API"
"讓 security-auditor 掃描 OWASP 弱點"
"請 performance-engineer 最佳化這個資料庫查詢"
```

### Slash 指令

許多 agents 可以透過 plugin slash 指令直接呼叫：

```bash
/backend-development:feature-development user authentication
/security-scanning:security-sast
/incident-response:smart-fix "memory leak in payment service"
```

## 貢獻

要新增一個 agent：

1. 建立 `plugins/{plugin-name}/agents/{agent-name}.md`
2. 在 frontmatter 中加入名稱、描述與模型分配
3. 撰寫完整的系統提示詞
4. 在 `.claude-plugin/marketplace.json` 中更新 plugin 定義

詳見[貢獻指南](../CONTRIBUTING.md)。
