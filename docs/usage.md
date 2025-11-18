# 使用指南

關於使用代理程式、斜線指令和多代理工作流程的完整指南。

## 概述

外掛生態系統提供兩種主要介面：

1. **斜線指令** - 直接呼叫工具和工作流程
2. **自然語言** - Claude 會推理應使用哪些代理程式

## 斜線指令

斜線指令是與代理程式和工作流程互動的主要介面。每個外掛都提供可直接執行的命名空間指令。

### 指令格式

```bash
/plugin-name:command-name [arguments]
```

### 探索指令

列出已安裝外掛的所有可用斜線指令：

```bash
/plugin
```

### 斜線指令的優點

- **直接呼叫** - 無需用自然語言描述您的需求
- **結構化參數** - 明確傳遞參數以進行精確控制
- **可組合性** - 將指令串連在一起以實現複雜的工作流程
- **可探索性** - 使用 `/plugin` 查看所有可用指令

## 自然語言

當您需要 Claude 推理應使用哪位專家時，也可以透過自然語言呼叫代理程式：

```
"Use backend-architect to design the authentication API"
"Have security-auditor scan for OWASP vulnerabilities"
"Get performance-engineer to optimize this database query"
```

Claude Code 會根據您的請求自動選擇並協調適當的代理程式。

## 按類別分類的指令參考

### 開發與功能

| Command | Description |
|---------|-------------|
| `/backend-development:feature-development` | 端對端後端功能開發 |
| `/full-stack-orchestration:full-stack-feature` | 完整的全端功能實作 |
| `/multi-platform-apps:multi-platform` | 跨平台應用程式開發協調 |

### 測試與品質

| Command | Description |
|---------|-------------|
| `/unit-testing:test-generate` | 產生完整的單元測試 |
| `/tdd-workflows:tdd-cycle` | 完整的 TDD 紅燈-綠燈-重構循環 |
| `/tdd-workflows:tdd-red` | 先撰寫失敗的測試 |
| `/tdd-workflows:tdd-green` | 實作程式碼以通過測試 |
| `/tdd-workflows:tdd-refactor` | 在測試通過的情況下重構 |

### 程式碼品質與審查

| Command | Description |
|---------|-------------|
| `/code-review-ai:ai-review` | AI 驅動的程式碼審查 |
| `/comprehensive-review:full-review` | 多角度分析 |
| `/comprehensive-review:pr-enhance` | 增強拉取請求 |

### 除錯與疑難排解

| Command | Description |
|---------|-------------|
| `/debugging-toolkit:smart-debug` | 互動式智慧除錯 |
| `/incident-response:incident-response` | 正式環境事件管理 |
| `/incident-response:smart-fix` | 自動化事件解決 |
| `/error-debugging:error-analysis` | 深度錯誤分析 |
| `/error-debugging:error-trace` | 堆疊追蹤除錯 |
| `/error-diagnostics:smart-debug` | 智慧診斷除錯 |
| `/distributed-debugging:debug-trace` | 分散式系統追蹤 |

### 資訊安全

| Command | Description |
|---------|-------------|
| `/security-scanning:security-hardening` | 全面的安全加固 |
| `/security-scanning:security-sast` | 靜態應用程式安全測試 |
| `/security-scanning:security-dependencies` | 相依套件漏洞掃描 |
| `/security-compliance:compliance-check` | SOC2/HIPAA/GDPR 合規檢查 |
| `/frontend-mobile-security:xss-scan` | XSS 漏洞掃描 |

### 基礎架構與部署

| Command | Description |
|---------|-------------|
| `/observability-monitoring:monitor-setup` | 設定監控基礎架構 |
| `/observability-monitoring:slo-implement` | 實作 SLO/SLI 指標 |
| `/deployment-validation:config-validate` | 部署前驗證 |
| `/cicd-automation:workflow-automate` | CI/CD 流水線自動化 |

### 資料與機器學習

| Command | Description |
|---------|-------------|
| `/machine-learning-ops:ml-pipeline` | 機器學習訓練流水線編排 |
| `/data-engineering:data-pipeline` | ETL/ELT 流水線建構 |
| `/data-engineering:data-driven-feature` | 資料驅動功能開發 |

### 文件

| Command | Description |
|---------|-------------|
| `/code-documentation:doc-generate` | 產生完整文件 |
| `/code-documentation:code-explain` | 解釋程式碼功能 |
| `/documentation-generation:doc-generate` | OpenAPI 規格、圖表、教學文件 |

### 重構與維護

| Command | Description |
|---------|-------------|
| `/code-refactoring:refactor-clean` | 程式碼清理和重構 |
| `/code-refactoring:tech-debt` | 技術債管理 |
| `/codebase-cleanup:deps-audit` | 相依套件稽核 |
| `/codebase-cleanup:tech-debt` | 技術債削減 |
| `/framework-migration:legacy-modernize` | 舊版程式碼現代化 |
| `/framework-migration:code-migrate` | 框架遷移 |
| `/framework-migration:deps-upgrade` | 相依套件升級 |

### 資料庫

| Command | Description |
|---------|-------------|
| `/database-migrations:sql-migrations` | SQL 遷移自動化 |
| `/database-migrations:migration-observability` | 遷移監控 |
| `/database-cloud-optimization:cost-optimize` | 資料庫和雲端最佳化 |

### Git 與拉取請求工作流程

| Command | Description |
|---------|-------------|
| `/git-pr-workflows:pr-enhance` | 提升拉取請求品質 |
| `/git-pr-workflows:onboard` | 團隊入職自動化 |
| `/git-pr-workflows:git-workflow` | Git 工作流程自動化 |

### 專案腳手架

| Command | Description |
|---------|-------------|
| `/python-development:python-scaffold` | FastAPI/Django 專案設定 |
| `/javascript-typescript:typescript-scaffold` | Next.js/React + Vite 設定 |
| `/systems-programming:rust-project` | Rust 專案腳手架 |

### AI 與 LLM 開發

| Command | Description |
|---------|-------------|
| `/llm-application-dev:langchain-agent` | LangChain 代理程式開發 |
| `/llm-application-dev:ai-assistant` | AI 助理實作 |
| `/llm-application-dev:prompt-optimize` | 提示工程最佳化 |
| `/agent-orchestration:multi-agent-optimize` | 多代理程式最佳化 |
| `/agent-orchestration:improve-agent` | 代理程式改進工作流程 |

### 測試與效能

| Command | Description |
|---------|-------------|
| `/performance-testing-review:ai-review` | 效能分析 |
| `/application-performance:performance-optimization` | 應用程式最佳化 |

### 團隊協作

| Command | Description |
|---------|-------------|
| `/team-collaboration:issue` | 議題管理自動化 |
| `/team-collaboration:standup-notes` | 站立會議記錄產生 |

### 無障礙網頁

| Command | Description |
|---------|-------------|
| `/accessibility-compliance:accessibility-audit` | WCAG 合規稽核 |

### API 開發

| Command | Description |
|---------|-------------|
| `/api-testing-observability:api-mock` | API 模擬和測試 |

### 上下文管理

| Command | Description |
|---------|-------------|
| `/context-management:context-save` | 儲存對話上下文 |
| `/context-management:context-restore` | 還原先前上下文 |

## 多代理工作流程範例

外掛提供可透過斜線指令存取的預先設定多代理工作流程。

### 全端開發

```bash
# 基於指令的工作流程呼叫
/full-stack-orchestration:full-stack-feature "user dashboard with real-time analytics"

# 自然語言替代方案
"Implement user dashboard with real-time analytics"
```

**編排順序：** backend-architect → database-architect → frontend-developer → test-automator → security-auditor → deployment-engineer → observability-engineer

**執行內容：**

1. 資料庫綱要設計與遷移
2. 後端 API 實作（REST/GraphQL）
3. 前端元件與狀態管理
4. 完整測試套件（單元/整合/E2E）
5. 資訊安全稽核和加固
6. CI/CD 流水線設定與功能旗標
7. 可觀測性和監控設定

### 安全加固

```bash
# 全面的安全評估和修復
/security-scanning:security-hardening --level comprehensive

# 自然語言替代方案
"Perform security audit and implement OWASP best practices"
```

**編排順序：** security-auditor → backend-security-coder → frontend-security-coder → mobile-security-coder → test-automator

### 資料/機器學習流水線

```bash
# 機器學習功能開發與正式環境部署
/machine-learning-ops:ml-pipeline "customer churn prediction model"

# 自然語言替代方案
"Build customer churn prediction model with deployment"
```

**編排順序：** data-scientist → data-engineer → ml-engineer → mlops-engineer → performance-engineer

### 事件回應

```bash
# 智慧除錯與根本原因分析
/incident-response:smart-fix "production memory leak in payment service"

# 自然語言替代方案
"Debug production memory leak and create runbook"
```

**編排順序：** incident-responder → devops-troubleshooter → debugger → error-detective → observability-engineer

## 指令參數與選項

許多斜線指令支援參數以進行精確控制：

```bash
# 為特定檔案產生測試
/unit-testing:test-generate src/api/users.py

# 功能開發與方法論規格
/backend-development:feature-development OAuth2 integration with social login

# 安全相依套件掃描
/security-scanning:security-dependencies

# 元件腳手架
/frontend-mobile-development:component-scaffold UserProfile component with hooks

# TDD 工作流程循環
/tdd-workflows:tdd-red User can reset password
/tdd-workflows:tdd-green
/tdd-workflows:tdd-refactor

# 智慧除錯
/debugging-toolkit:smart-debug memory leak in checkout flow

# Python 專案腳手架
/python-development:python-scaffold fastapi-microservice
```

## 結合自然語言和指令

您可以混合使用兩種方法以獲得最佳彈性：

```
# 從指令開始進行結構化工作流程
/full-stack-orchestration:full-stack-feature "payment processing"

# 然後提供自然語言指導
"Ensure PCI-DSS compliance and integrate with Stripe"
"Add retry logic for failed transactions"
"Set up fraud detection rules"
```

## 最佳實務

### 何時使用斜線指令

- **結構化工作流程** - 具有明確階段的多步驟流程
- **重複性任務** - 您經常執行的操作
- **精確控制** - 當您需要特定參數時
- **探索** - 探索可用功能

### 何時使用自然語言

- **探索性工作** - 當您不確定要使用哪個工具時
- **複雜推理** - 當 Claude 需要協調多個代理程式時
- **情境決策** - 當正確的方法取決於情況時
- **臨時任務** - 不適合指令的一次性操作

### 工作流程組合

為複雜場景組合多個外掛：

```bash
# 1. 從功能開發開始
/backend-development:feature-development payment processing API

# 2. 新增安全加固
/security-scanning:security-hardening

# 3. 產生完整測試
/unit-testing:test-generate

# 4. 審查實作
/code-review-ai:ai-review

# 5. 設定 CI/CD
/cicd-automation:workflow-automate

# 6. 新增監控
/observability-monitoring:monitor-setup
```

## 代理技能整合

代理技能與指令配合使用以提供深度專業知識：

```
User: "Set up FastAPI project with async patterns"
→ Activates: fastapi-templates skill
→ Invokes: /python-development:python-scaffold
→ Result: Production-ready FastAPI project with best practices

User: "Implement Kubernetes deployment with Helm"
→ Activates: helm-chart-scaffolding, k8s-manifest-generator skills
→ Guides: kubernetes-architect agent
→ Result: Production-grade K8s manifests with Helm charts
```

有關 47 種專業技能的詳細資訊，請參閱[代理技能](./agent-skills.md)。

## 另請參閱

- [代理技能](./agent-skills.md) - 專業知識套件
- [代理參考](./agents.md) - 完整代理目錄
- [外掛參考](./plugins.md) - 所有 63 個外掛
- [架構](./architecture.md) - 設計原則
