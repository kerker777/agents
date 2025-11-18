# Claude Code 外掛：編排與自動化

> **⚡ 已針對 Sonnet 4.5 與 Haiku 4.5 更新** — 所有代理皆已針對最新模型進行混合編排最佳化
>
> **🎯 已啟用代理技能** — 47 項專業技能透過漸進式揭露擴展 Claude 在各外掛中的能力

一個完整的生產級系統，結合了 **85 個專業 AI 代理**、**15 個多代理工作流程編排器**、**47 項代理技能**，以及 **44 項開發工具**，組織成 **63 個專注且單一用途的外掛**，適用於 [Claude Code](https://docs.claude.com/en/docs/claude-code/overview)。

## 概述

這個統一的程式庫提供了現代軟體開發中智慧自動化和多代理編排所需的一切：

- **63 個專注外掛** - 針對最小化 token 使用量和可組合性最佳化的精細、單一用途外掛
- **85 個專業代理** - 具備架構、程式語言、基礎設施、品質、資料/AI、文件、業務營運和 SEO 等領域深度知識的領域專家
- **47 項代理技能** - 具備漸進式揭露的模組化知識包，提供專業能力
- **15 個工作流程編排器** - 用於複雜操作的多代理協調系統，如全端開發、安全加固、ML 流程和事件回應
- **44 項開發工具** - 最佳化的實用工具，包括專案腳手架、安全掃描、測試自動化和基礎設施設定

### 主要特色

- **精細的外掛架構**：63 個針對最小化 token 使用量最佳化的專注外掛
- **完整的工具集**：44 項開發工具，包括測試產生、腳手架和安全掃描
- **100% 代理覆蓋率**：所有外掛皆包含專業代理
- **代理技能**：47 項專業技能，遵循漸進式揭露和 token 效率原則
- **清晰的組織**：23 個類別，每個類別包含 1-6 個外掛，方便探索
- **高效設計**：每個外掛平均 3.4 個元件（遵循 Anthropic 的 2-8 模式）

### 運作方式

每個外掛都完全獨立，擁有自己的代理、命令和技能：

- **只安裝所需的外掛** - 每個外掛只載入其特定的代理、命令和技能
- **最小化 token 使用量** - 不會將不必要的資源載入上下文
- **混搭組合** - 組合多個外掛以建立複雜的工作流程
- **清晰的邊界** - 每個外掛都有單一且專注的用途
- **漸進式揭露** - 技能只在啟動時才載入知識

**範例**：安裝 `python-development` 會載入 3 個 Python 代理、1 個腳手架工具，並提供 5 項技能（約 300 個 tokens），而不是整個市場。

## 快速開始

### 步驟 1：新增市場

將此市場新增至 Claude Code：

```bash
/plugin marketplace add wshobson/agents
```

這會使所有 63 個外掛可供安裝，但**不會將任何代理或工具**載入您的上下文。

### 步驟 2：安裝外掛

瀏覽可用的外掛：

```bash
/plugin
```

安裝您需要的外掛：

```bash
# 基本開發外掛
/plugin install python-development          # Python 與 5 項專業技能
/plugin install javascript-typescript       # JS/TS 與 4 項專業技能
/plugin install backend-development         # 後端 API 與 3 項架構技能

# 基礎設施與營運
/plugin install kubernetes-operations       # K8s 與 4 項部署技能
/plugin install cloud-infrastructure        # AWS/Azure/GCP 與 4 項雲端技能

# 安全與品質
/plugin install security-scanning           # SAST 與安全技能
/plugin install code-review-ai             # AI 驅動的程式碼審查

# 全端編排
/plugin install full-stack-orchestration   # 多代理工作流程
```

每個已安裝的外掛只會將**其特定的代理、命令和技能**載入 Claude 的上下文。

## 文件

### 核心指南

- **[外掛參考](docs/plugins.md)** - 所有 63 個外掛的完整目錄
- **[代理參考](docs/agents.md)** - 所有 85 個代理按類別組織
- **[代理技能](docs/agent-skills.md)** - 47 項具備漸進式揭露的專業技能
- **[使用指南](docs/usage.md)** - 命令、工作流程和最佳實務
- **[架構](docs/architecture.md)** - 設計原則和模式

### 快速連結

- [安裝](#快速開始) - 2 個步驟即可開始使用
- [必備外掛](docs/plugins.md#quick-start---essential-plugins) - 立即提升生產力的頂級外掛
- [命令參考](docs/usage.md#command-reference-by-category) - 按類別組織的所有斜線命令
- [多代理工作流程](docs/usage.md#multi-agent-workflow-examples) - 預先設定的編排範例
- [模型設定](docs/agents.md#model-configuration) - Haiku/Sonnet 混合編排

## 最新功能

### 代理技能（14 個外掛中的 47 項技能）

遵循 Anthropic 漸進式揭露架構的專業知識包：

**程式語言開發：**
- **Python**（5 項技能）：非同步模式、測試、打包、效能、UV 套件管理器
- **JavaScript/TypeScript**（4 項技能）：進階型別、Node.js 模式、測試、現代 ES6+

**基礎設施與 DevOps：**
- **Kubernetes**（4 項技能）：manifests、Helm charts、GitOps、安全政策
- **雲端基礎設施**（4 項技能）：Terraform、多雲、混合網路、成本最佳化
- **CI/CD**（4 項技能）：流程設計、GitHub Actions、GitLab CI、機密管理

**開發與架構：**
- **後端**（3 項技能）：API 設計、架構模式、微服務
- **LLM 應用程式**（4 項技能）：LangChain、提示工程、RAG、評估

**區塊鏈與 Web3**（4 項技能）：DeFi 協定、NFT 標準、Solidity 安全、Web3 測試

**以及更多：** 框架遷移、可觀測性、支付處理、ML 營運、安全掃描

[→ 檢視完整技能文件](docs/agent-skills.md)

### 混合模型編排

針對最佳效能和成本的策略性模型分配：
- **47 個 Haiku 代理** - 針對確定性任務的快速執行
- **97 個 Sonnet 代理** - 複雜推理和架構

編排模式結合模型以提升效率：
```
Sonnet（規劃）→ Haiku（執行）→ Sonnet（審查）
```

[→ 檢視模型設定詳情](docs/agents.md#model-configuration)

## 熱門使用案例

### 全端功能開發

```bash
/full-stack-orchestration:full-stack-feature "user authentication with OAuth2"
```

協調 7+ 個代理：backend-architect → database-architect → frontend-developer → test-automator → security-auditor → deployment-engineer → observability-engineer

[→ 檢視所有工作流程範例](docs/usage.md#multi-agent-workflow-examples)

### 安全加固

```bash
/security-scanning:security-hardening --level comprehensive
```

具備 SAST、相依性掃描和程式碼審查的多代理安全評估。

### 使用現代工具進行 Python 開發

```bash
/python-development:python-scaffold fastapi-microservice
```

建立具備非同步模式的生產級 FastAPI 專案，啟動技能：
- `async-python-patterns` - AsyncIO 和並行處理
- `python-testing-patterns` - pytest 和 fixtures
- `uv-package-manager` - 快速相依性管理

### Kubernetes 部署

```bash
# 自動啟動 k8s 技能
"Create production Kubernetes deployment with Helm chart and GitOps"
```

使用具備 4 項專業技能的 kubernetes-architect 代理來建立生產級設定。

[→ 檢視完整使用指南](docs/usage.md)

## 外掛類別

**23 個類別，63 個外掛：**

- 🎨 **開發**（4）- 除錯、後端、前端、多平台
- 📚 **文件**（2）- 程式碼文件、API 規格、圖表
- 🔄 **工作流程**（3）- git、全端、TDD
- ✅ **測試**（2）- 單元測試、TDD 工作流程
- 🔍 **品質**（3）- 程式碼審查、完整審查、效能
- 🤖 **AI 與 ML**（4）- LLM 應用程式、代理編排、上下文、MLOps
- 📊 **資料**（2）- 資料工程、資料驗證
- 🗄️ **資料庫**（2）- 資料庫設計、遷移
- 🚨 **營運**（4）- 事件回應、診斷、分散式除錯、可觀測性
- ⚡ **效能**（2）- 應用程式效能、資料庫/雲端最佳化
- ☁️ **基礎設施**（5）- 部署、驗證、Kubernetes、雲端、CI/CD
- 🔒 **安全**（4）- 掃描、合規、後端/API、前端/行動裝置
- 💻 **程式語言**（7）- Python、JS/TS、系統、JVM、腳本、函數式、嵌入式
- 🔗 **區塊鏈**（1）- 智能合約、DeFi、Web3
- 💰 **金融**（1）- 量化交易、風險管理
- 💳 **支付**（1）- Stripe、PayPal、計費
- 🎮 **遊戲**（1）- Unity、Minecraft 外掛
- 📢 **行銷**（4）- SEO 內容、技術 SEO、SEO 分析、內容行銷
- 💼 **商業**（3）- 分析、人資/法律、客戶/銷售
- 以及更多...

[→ 檢視完整外掛目錄](docs/plugins.md)

## 架構重點

### 精細設計

- **單一職責** - 每個外掛專精一件事
- **最小化 token 使用量** - 每個外掛平均 3.4 個元件
- **可組合** - 混搭以建立複雜工作流程
- **100% 覆蓋率** - 所有 85 個代理皆可透過外掛存取

### 漸進式揭露（技能）

針對 token 效率的三層架構：
1. **中繼資料** - 名稱和啟動標準（始終載入）
2. **指示** - 核心指引（啟動時載入）
3. **資源** - 範例和範本（按需載入）

### 程式庫結構

```
claude-agents/
├── .claude-plugin/
│   └── marketplace.json          # 63 個外掛
├── plugins/
│   ├── python-development/
│   │   ├── agents/               # 3 個 Python 專家
│   │   ├── commands/             # 腳手架工具
│   │   └── skills/               # 5 項專業技能
│   ├── kubernetes-operations/
│   │   ├── agents/               # K8s 架構師
│   │   ├── commands/             # 部署工具
│   │   └── skills/               # 4 項 K8s 技能
│   └── ... (還有 61 個外掛)
├── docs/                          # 完整文件
└── README.md                      # 本檔案
```

[→ 檢視架構詳情](docs/architecture.md)

## 貢獻

新增代理、技能或命令：

1. 在 `plugins/` 中識別或建立適當的外掛目錄
2. 在適當的子目錄中建立 `.md` 檔案：
   - `agents/` - 用於專業代理
   - `commands/` - 用於工具和工作流程
   - `skills/` - 用於模組化知識包
3. 遵循命名慣例（小寫、連字號分隔）
4. 撰寫清晰的啟動標準和完整內容
5. 在 `.claude-plugin/marketplace.json` 中更新外掛定義

詳細指南請參閱[架構文件](docs/architecture.md)。

## 資源

### 文件
- [Claude Code 文件](https://docs.claude.com/en/docs/claude-code/overview)
- [外掛指南](https://docs.claude.com/en/docs/claude-code/plugins)
- [子代理指南](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [代理技能指南](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [斜線命令參考](https://docs.claude.com/en/docs/claude-code/slash-commands)

### 本程式庫
- [外掛參考](docs/plugins.md)
- [代理參考](docs/agents.md)
- [代理技能指南](docs/agent-skills.md)
- [使用指南](docs/usage.md)
- [架構](docs/architecture.md)

## 授權

MIT License - 詳情請參閱 [LICENSE](LICENSE) 檔案。

## Star 歷史

[![Star History Chart](https://api.star-history.com/svg?repos=wshobson/agents&type=date&legend=top-left)](https://www.star-history.com/#wshobson/agents&type=date&legend=top-left)
