# Agent 技能

Agent 技能是模組化的套件，能夠透過專業領域知識擴展 Claude 的能力，遵循 Anthropic 的 [Agent Skills Specification](https://github.com/anthropics/skills/blob/main/agent_skills_spec.md)。這個外掛程式生態系統包含 **57 個專業技能**，分佈於 15 個外掛程式中，實現漸進式揭露和高效的 token 使用。

## 概述

技能為 Claude 提供特定領域的深度專業知識，而無需預先將所有內容載入上下文。每個技能包括：

- **YAML Frontmatter**：名稱和啟動條件
- **漸進式揭露**：中繼資料 → 指示 → 資源
- **啟動觸發條件**：清楚的「使用時機」條款，用於自動調用

## 依外掛程式分類的技能

### Kubernetes Operations (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **k8s-manifest-generator** | 建立符合最佳實務的生產就緒 Kubernetes 清單檔案，包括 Deployments、Services、ConfigMaps 和 Secrets |
| **helm-chart-scaffolding** | 設計、組織和管理 Helm charts，用於模板化和打包 Kubernetes 應用程式 |
| **gitops-workflow** | 使用 ArgoCD 和 Flux 實作 GitOps 工作流程，實現自動化的宣告式部署 |
| **k8s-security-policies** | 實作 Kubernetes 安全政策，包括 NetworkPolicy、PodSecurityPolicy 和 RBAC |

### LLM Application Development (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **langchain-architecture** | 使用 LangChain 框架設計 LLM 應用程式，包含代理、記憶體和工具整合 |
| **prompt-engineering-patterns** | 精通進階提示工程技術，提升 LLM 效能和可靠性 |
| **rag-implementation** | 建構檢索增強生成系統，結合向量資料庫和語意搜尋 |
| **llm-evaluation** | 實作全面的評估策略，包含自動化指標和基準測試 |

### Backend Development (5 個技能)

| 技能 | 描述 |
|-------|-------------|
| **api-design-principles** | 精通 REST 和 GraphQL API 設計，打造直觀、可擴展且易於維護的 API |
| **architecture-patterns** | 實作整潔架構、六角架構和領域驅動設計 |
| **microservices-patterns** | 設計微服務架構，包含服務邊界、事件驅動通訊和韌性設計 |
| **workflow-orchestration-patterns** | 使用 Temporal 設計持久化工作流程，適用於分散式系統、saga 模式和狀態管理 |
| **temporal-python-testing** | 使用 pytest 測試 Temporal 工作流程，包含時間跳躍和模擬策略，達成全面測試覆蓋率 |

### Developer Essentials (8 個技能)

| 技能 | 描述 |
|-------|-------------|
| **git-advanced-workflows** | 精通進階 Git 工作流程，包括 rebasing、cherry-picking、bisect、worktrees 和 reflog |
| **sql-optimization-patterns** | 最佳化 SQL 查詢、索引策略和 EXPLAIN 分析，提升資料庫效能 |
| **error-handling-patterns** | 實作健全的錯誤處理機制，包含例外處理、Result 型別和優雅降級 |
| **code-review-excellence** | 提供有效的程式碼審查，包含建設性回饋和系統性分析 |
| **e2e-testing-patterns** | 使用 Playwright 和 Cypress 建構可靠的端對端測試套件，涵蓋關鍵使用者工作流程 |
| **auth-implementation-patterns** | 實作身份驗證和授權機制，包含 JWT、OAuth2、sessions 和 RBAC |
| **debugging-strategies** | 精通系統化除錯技術、效能分析工具和根因分析 |
| **monorepo-management** | 使用 Turborepo、Nx 和 pnpm workspaces 管理 monorepo，適用於可擴展的多套件專案 |

### Blockchain & Web3 (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **defi-protocol-templates** | 實作 DeFi 協定，包含質押、AMMs、治理和借貸的範本 |
| **nft-standards** | 實作 NFT 標準（ERC-721、ERC-1155），包含中繼資料和市場整合 |
| **solidity-security** | 精通智能合約安全性，防止漏洞並實作安全模式 |
| **web3-testing** | 使用 Hardhat 和 Foundry 測試智能合約，包含單元測試和主網分叉 |

### CI/CD Automation (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **deployment-pipeline-design** | 設計多階段 CI/CD 管線，包含審核關卡和安全性檢查 |
| **github-actions-templates** | 建立生產就緒的 GitHub Actions 工作流程，用於測試、建置和部署 |
| **gitlab-ci-patterns** | 建構 GitLab CI/CD 管線，包含多階段工作流程和分散式 runners |
| **secrets-management** | 使用 Vault、AWS Secrets Manager 或原生解決方案實作安全的機密管理 |

### Cloud Infrastructure (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **terraform-module-library** | 建構可重複使用的 Terraform 模組，適用於 AWS、Azure 和 GCP 基礎設施 |
| **multi-cloud-architecture** | 設計多雲架構，避免廠商鎖定 |
| **hybrid-cloud-networking** | 配置地端和雲端平台之間的安全連線 |
| **cost-optimization** | 透過適當調整規模、標記和預留執行個體來最佳化雲端成本 |

### Framework Migration (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **react-modernization** | 升級 React 應用程式、遷移至 hooks 並採用並行功能 |
| **angular-migration** | 使用混合模式和漸進式重寫，從 AngularJS 遷移至 Angular |
| **database-migration** | 執行零停機時間的資料庫遷移策略和資料轉換 |
| **dependency-upgrade** | 管理主要相依性升級，包含相容性分析和測試 |

### Observability & Monitoring (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **prometheus-configuration** | 設定 Prometheus 以進行全面的指標收集和監控 |
| **grafana-dashboards** | 建立生產環境的 Grafana 儀表板，用於即時系統視覺化 |
| **distributed-tracing** | 使用 Jaeger 和 Tempo 實作分散式追蹤，追蹤請求流程 |
| **slo-implementation** | 定義 SLIs 和 SLOs，包含錯誤預算和告警機制 |

### Payment Processing (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **stripe-integration** | 實作 Stripe 金流處理，包含結帳、訂閱和 webhooks |
| **paypal-integration** | 整合 PayPal 金流處理，包含快速結帳和訂閱功能 |
| **pci-compliance** | 實作 PCI DSS 合規性，確保支付卡資料的安全處理 |
| **billing-automation** | 建構自動化帳務系統，處理定期付款和開立發票 |

### Python Development (5 個技能)

| 技能 | 描述 |
|-------|-------------|
| **async-python-patterns** | 精通 Python asyncio、並行程式設計和 async/await 模式 |
| **python-testing-patterns** | 使用 pytest、fixtures 和 mocking 實作全面的測試 |
| **python-packaging** | 建立可分發的 Python 套件，包含適當的結構和 PyPI 發布 |
| **python-performance-optimization** | 使用 cProfile 和效能最佳實務來分析和最佳化 Python 程式碼 |
| **uv-package-manager** | 精通 uv 套件管理器，實現快速的相依性管理和虛擬環境 |

### JavaScript/TypeScript (4 個技能)

| 技能 | 描述 |
|-------|-------------|
| **typescript-advanced-types** | 精通 TypeScript 的進階型別系統，包括泛型和條件型別 |
| **nodejs-backend-patterns** | 使用 Express/Fastify 和最佳實務建構生產就緒的 Node.js 服務 |
| **javascript-testing-patterns** | 使用 Jest、Vitest 和 Testing Library 實作全面的測試 |
| **modern-javascript-patterns** | 精通 ES6+ 功能，包括 async/await、解構和函數式程式設計 |

### API Scaffolding (1 個技能)

| 技能 | 描述 |
|-------|-------------|
| **fastapi-templates** | 建立生產就緒的 FastAPI 專案，包含非同步模式和錯誤處理 |

### Machine Learning Operations (1 個技能)

| 技能 | 描述 |
|-------|-------------|
| **ml-pipeline-workflow** | 建構端對端的 MLOps 管線，從資料準備到部署 |

### Security Scanning (1 個技能)

| 技能 | 描述 |
|-------|-------------|
| **sast-configuration** | 配置靜態應用程式安全測試工具，用於漏洞偵測 |

## 技能運作方式

### 啟動

當 Claude 偵測到您的請求中符合的模式時，技能會自動啟動：

```
User: "Set up Kubernetes deployment with Helm chart"
→ Activates: helm-chart-scaffolding, k8s-manifest-generator

User: "Build a RAG system for document Q&A"
→ Activates: rag-implementation, prompt-engineering-patterns

User: "Optimize Python async performance"
→ Activates: async-python-patterns, python-performance-optimization
```

### 漸進式揭露

技能使用三層架構來提高 token 效率：

1. **中繼資料**（Frontmatter）：名稱和啟動條件（始終載入）
2. **指示**：核心指導和模式（啟動時載入）
3. **資源**：範例和範本（依需求載入）

### 與 Agents 的整合

技能與代理協同運作，提供深度領域專業知識：

- **Agents**：高階推理和協調
- **Skills**：專業知識和實作模式

範例工作流程：
```
backend-architect agent → Plans API architecture
  ↓
api-design-principles skill → Provides REST/GraphQL best practices
  ↓
fastapi-templates skill → Supplies production-ready templates
```

## 規格合規性

所有 55 個技能都遵循 [Agent Skills Specification](https://github.com/anthropics/skills/blob/main/agent_skills_spec.md)：

- ✓ 必要的 `name` 欄位（連字號命名）
- ✓ 必要的 `description` 欄位，包含「使用時機」條款
- ✓ 描述少於 1024 個字元
- ✓ 完整、未截斷的描述
- ✓ 正確的 YAML frontmatter 格式

## 建立新技能

要將技能新增至外掛程式：

1. 建立 `plugins/{plugin-name}/skills/{skill-name}/SKILL.md`
2. 新增 YAML frontmatter：
   ```yaml
   ---
   name: skill-name
   description: What the skill does. Use when [activation trigger].
   ---
   ```
3. 使用漸進式揭露撰寫完整的技能內容
4. 將技能路徑新增至 `marketplace.json`：
   ```json
   {
     "name": "plugin-name",
     "skills": ["./skills/skill-name"]
   }
   ```

### 技能結構

```
plugins/{plugin-name}/
└── skills/
    └── {skill-name}/
        └── SKILL.md        # Frontmatter + content
```

## 優勢

- **Token 效率**：僅在需要時載入相關知識
- **專業知識**：深度領域知識且不造成負擔
- **清楚的啟動條件**：明確的觸發條件可防止意外調用
- **可組合性**：在工作流程中混合和搭配技能
- **可維護性**：獨立的更新不會影響其他技能

## 資源

- [Anthropic Skills Repository](https://github.com/anthropics/skills)
- [Agent Skills Documentation](https://docs.claude.com/en/docs/claude-code/skills)
