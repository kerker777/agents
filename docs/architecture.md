# 架構與設計原則

此市集遵循業界最佳實踐，專注於粒度化、可組合性和最小化 token 使用量。

## 核心理念

### 單一職責原則

- 每個插件**專注做好一件事**（Unix 哲學）
- 明確且專注的目標（可用 5-10 個字描述）
- 插件平均大小：**3.4 個元件**（遵循 Anthropic 的 2-8 模式）
- **零臃腫插件** - 所有插件都專注且目標明確

### 可組合性優於綁定

- 根據需求混合搭配插件
- 工作流程編排器組合專注的插件
- 無強制功能綁定
- 插件之間界線清晰

### 上下文效率

- 更小的工具 = 更快的處理速度
- 更適合 LLM 上下文視窗
- 更精確、更專注的回應
- 只安裝所需的內容

### 可維護性

- 單一目標 = 更容易更新
- 清晰界線 = 隔離變更
- 更少重複 = 更簡單的維護
- 獨立的依賴關係

## 粒度化插件架構

### 插件分布

- **63 個專注插件**針對特定使用案例最佳化
- **23 個清晰類別**，每個類別包含 1-6 個插件，便於探索
- 按領域組織：
  - **Development**：4 個插件（除錯、後端、前端、多平台）
  - **Security**：4 個插件（掃描、合規、後端 API、前端行動）
  - **Operations**：4 個插件（事件、診斷、分散式、可觀測性）
  - **Languages**：7 個插件（Python、JS/TS、系統、JVM、腳本、函數式、嵌入式）
  - **Infrastructure**：5 個插件（部署、驗證、K8s、雲端、CI/CD）
  - 以及 18 個專門類別

### 元件細分

**85 個專業 Agent**
- 具備深度知識的領域專家
- 組織涵蓋架構、語言、基礎設施、品質、資料/AI、文件、商業和 SEO
- 模型最佳化（47 個 Haiku、97 個 Sonnet）以提升效能和成本效益

**15 個工作流程編排器**
- 多 agent 協調系統
- 複雜操作，如全端開發、安全加固、ML 管線、事件回應
- 預先配置的 agent 工作流程

**44 個開發工具**
- 最佳化工具包括：
  - 專案腳手架（Python、TypeScript、Rust）
  - 安全掃描（SAST、依賴審計、XSS）
  - 測試生成（pytest、Jest）
  - 元件腳手架（React、React Native）
  - 基礎設施設定（Terraform、Kubernetes）

**47 個 Agent 技能**
- 模組化知識套件
- 漸進式揭露架構
- 跨 14 個插件的領域專業知識
- 規格相容（Anthropic Agent Skills Specification）

## 儲存庫結構

```
claude-agents/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog (63 plugins)
├── plugins/                       # Isolated plugin directories
│   ├── python-development/
│   │   ├── agents/               # Python language agents
│   │   │   ├── python-pro.md
│   │   │   ├── django-pro.md
│   │   │   └── fastapi-pro.md
│   │   ├── commands/             # Python tooling
│   │   │   └── python-scaffold.md
│   │   └── skills/               # Python skills (5 total)
│   │       ├── async-python-patterns/
│   │       ├── python-testing-patterns/
│   │       ├── python-packaging/
│   │       ├── python-performance-optimization/
│   │       └── uv-package-manager/
│   ├── backend-development/
│   │   ├── agents/
│   │   │   ├── backend-architect.md
│   │   │   ├── graphql-architect.md
│   │   │   └── tdd-orchestrator.md
│   │   ├── commands/
│   │   │   └── feature-development.md
│   │   └── skills/               # Backend skills (3 total)
│   │       ├── api-design-principles/
│   │       ├── architecture-patterns/
│   │       └── microservices-patterns/
│   ├── security-scanning/
│   │   ├── agents/
│   │   │   └── security-auditor.md
│   │   ├── commands/
│   │   │   ├── security-hardening.md
│   │   │   ├── security-sast.md
│   │   │   └── security-dependencies.md
│   │   └── skills/               # Security skills (1 total)
│   │       └── sast-configuration/
│   └── ... (60 more isolated plugins)
├── docs/                          # Documentation
│   ├── agent-skills.md           # Agent Skills guide
│   ├── agents.md                 # Agent reference
│   ├── plugins.md                # Plugin catalog
│   ├── usage.md                  # Usage guide
│   └── architecture.md           # This file
└── README.md                      # Quick start
```

## 插件結構

每個插件包含：

- **agents/** - 該領域的專業 agent（選用）
- **commands/** - 該插件特定的工具和工作流程（選用）
- **skills/** - 具有漸進式揭露的模組化知識套件（選用）

### 最低需求

- 至少一個 agent 或一個 command
- 明確且專注的目標
- 所有檔案都有正確的 frontmatter
- 在 marketplace.json 中有項目

### 插件範例

```
plugins/kubernetes-operations/
├── agents/
│   └── kubernetes-architect.md   # K8s architecture and design
├── commands/
│   └── k8s-deploy.md            # Deployment automation
└── skills/
    ├── k8s-manifest-generator/   # Manifest creation skill
    ├── helm-chart-scaffolding/   # Helm chart skill
    ├── gitops-workflow/          # GitOps automation skill
    └── k8s-security-policies/    # Security policy skill
```

## Agent 技能架構

### 漸進式揭露

技能使用三層架構以提升 token 效率：

1. **中繼資料**（Frontmatter）：名稱和啟動條件（總是載入）
2. **指令**：核心指導和模式（啟動時載入）
3. **資源**：範例和範本（按需載入）

### 規格相容性

所有技能遵循 [Agent Skills Specification](https://github.com/anthropics/skills/blob/main/agent_skills_spec.md)：

```yaml
---
name: skill-name                  # Required: hyphen-case
description: What the skill does. Use when [trigger]. # Required: < 1024 chars
---

# Skill content with progressive disclosure
```

### 優勢

- **Token 效率**：僅在需要時載入相關知識
- **專業知識**：深度領域知識而無臃腫
- **清晰啟動**：明確的觸發器防止不必要的調用
- **可組合性**：跨工作流程混合搭配技能
- **可維護性**：獨立更新不影響其他技能

詳見 [Agent Skills](./agent-skills.md) 了解 47 個技能的完整細節。

## 模型配置策略

### 雙層架構

系統策略性地使用 Claude Opus 和 Sonnet 模型：

| 模型 | 數量 | 使用案例 |
|-------|-------|----------|
| Haiku | 47 個 agent | 快速執行、確定性任務 |
| Sonnet | 97 個 agent | 複雜推理、架構決策 |

### 選擇標準

**Haiku - 快速執行與確定性任務**
- 從明確定義的規格生成程式碼
- 遵循既定模式創建測試
- 使用清晰範本編寫文件
- 執行基礎設施操作
- 執行資料庫查詢最佳化
- 處理客戶支援回應
- 處理 SEO 最佳化任務
- 管理部署管線

**Sonnet - 複雜推理與架構**
- 設計系統架構
- 做出技術選型決策
- 執行安全稽核
- 檢視架構模式的程式碼
- 創建複雜的 AI/ML 管線
- 提供特定語言的專業知識
- 編排多 agent 工作流程
- 處理業務關鍵的法務/人資事務

### 混合編排

組合模型以獲得最佳效能和成本：

```
規劃階段 (Sonnet) → 執行階段 (Haiku) → 審查階段 (Sonnet)

範例：
backend-architect (Sonnet) 設計 API
  ↓
Generate endpoints (Haiku) 實作規格
  ↓
test-automator (Haiku) 創建測試
  ↓
code-reviewer (Sonnet) 驗證架構
```

## 效能與品質

### 最佳化 Token 使用量

- **隔離插件**僅載入所需內容
- **粒度化架構**減少不必要的上下文
- **漸進式揭露**（技能）按需載入知識
- **清晰界線**防止上下文污染

### 元件涵蓋範圍

- **100% agent 涵蓋率** - 所有插件至少包含一個 agent
- **100% 元件可用性** - 所有 85 個 agent 可跨插件存取
- **高效分布** - 平均每個插件 3.4 個元件

### 可探索性

- **清晰的插件名稱**立即傳達目標
- **邏輯分類**具有 23 個明確定義的類別
- **可搜尋文件**帶有交叉參照
- **易於找到**適合工作的工具

## 設計模式

### 模式 1：單一目標插件

每個插件專注於一個領域：

```
python-development/
├── agents/           # Python language experts
├── commands/         # Python project scaffolding
└── skills/           # Python-specific knowledge
```

**優勢：**
- 明確的職責
- 易於維護
- 最小化 token 使用量
- 可與其他插件組合

### 模式 2：工作流程編排

編排器插件協調多個 agent：

```
full-stack-orchestration/
└── commands/
    └── full-stack-feature.md    # Coordinates 7+ agents
```

**編排：**
1. backend-architect（設計 API）
2. database-architect（設計 schema）
3. frontend-developer（建構 UI）
4. test-automator（創建測試）
5. security-auditor（安全審查）
6. deployment-engineer（CI/CD）
7. observability-engineer（監控）

### 模式 3：Agent + 技能整合

Agent 提供推理，技能提供知識：

```
使用者：「使用非同步模式建構 FastAPI 專案」
  ↓
fastapi-pro agent（編排）
  ↓
fastapi-templates skill（提供模式）
  ↓
python-scaffold command（生成專案）
```

### 模式 4：多插件組合

複雜工作流程使用多個插件：

```
功能開發工作流程：
1. backend-development:feature-development
2. security-scanning:security-hardening
3. unit-testing:test-generate
4. code-review-ai:ai-review
5. cicd-automation:workflow-automate
6. observability-monitoring:monitor-setup
```

## 版本控制與更新

### Marketplace 更新

- Marketplace 目錄位於 `.claude-plugin/marketplace.json`
- 插件採用語意化版本控制
- 維持向後相容性
- 重大變更提供清晰的遷移指南

### 插件更新

- 個別插件更新不影響其他插件
- 技能可獨立更新
- Agent 可新增/移除而不破壞工作流程
- Command 維持穩定的介面

## 貢獻指南

### 新增插件

1. 創建插件目錄：`plugins/{plugin-name}/`
2. 新增 agent 和/或 command
3. 選擇性地新增技能
4. 更新 marketplace.json
5. 在適當的類別中記錄

### 新增 Agent

1. 創建 `plugins/{plugin-name}/agents/{agent-name}.md`
2. 新增 frontmatter（name、description、model）
3. 撰寫完整的系統提示
4. 更新插件定義

### 新增技能

1. 創建 `plugins/{plugin-name}/skills/{skill-name}/SKILL.md`
2. 新增 YAML frontmatter（name、description 帶有「Use when」）
3. 撰寫具有漸進式揭露的技能內容
4. 新增至 marketplace.json 中插件的 skills 陣列

### 品質標準

- **清晰命名** - 使用連字號格式，具描述性
- **專注範圍** - 單一職責
- **完整文件** - 說明是什麼、何時使用、如何使用
- **經過測試的功能** - 提交前驗證
- **規格相容** - 遵循 Anthropic 指南

## 另見

- [Agent Skills](./agent-skills.md) - 模組化知識套件
- [Agent Reference](./agents.md) - 完整 agent 目錄
- [Plugin Reference](./plugins.md) - 所有 63 個插件
- [Usage Guide](./usage.md) - 指令與工作流程
