使用專門的效能與最佳化代理程式，端到端優化應用程式效能：

[Extended thinking: 此工作流程協調整個應用程式堆疊的全面效能最佳化流程。從深度效能分析和建立基準開始，工作流程逐步進行各系統層級的針對性優化、透過負載測試驗證改善成果，並建立持續監控以維持效能。每個階段都建立在先前階段的洞察之上，創建一個資料驅動的最佳化策略，解決實際瓶頸而非理論上的改善。此工作流程強調現代可觀測性實踐、以使用者為中心的效能指標，以及具成本效益的最佳化策略。]

## 階段 1：效能分析與基準建立

### 1. 全面效能分析
- 使用 Task tool，設定 subagent_type="performance-engineer"
- 提示詞："針對 $ARGUMENTS 進行全面的應用程式效能分析。產生 CPU 使用的 flame graphs、記憶體分析的 heap dumps、追蹤 I/O 操作，並識別熱點路徑。若可用，使用 APM 工具如 DataDog 或 New Relic。包含資料庫查詢分析、API 回應時間和前端渲染指標。為所有關鍵使用者旅程建立效能基準。"
- 情境：初始效能調查
- 輸出：詳細的效能分析報告，包含 flame graphs、記憶體分析、瓶頸識別、基準指標

### 2. 可觀測性堆疊評估
- 使用 Task tool，設定 subagent_type="observability-engineer"
- 提示詞："評估 $ARGUMENTS 的當前可觀測性設定。檢視現有的監控、使用 OpenTelemetry 的分散式追蹤、日誌聚合和指標收集。識別可見性缺口、缺少的指標，以及需要更好儀表化的區域。建議 APM 工具整合和業務關鍵操作的自訂指標。"
- 情境：來自步驟 1 的效能分析
- 輸出：可觀測性評估報告、儀表化缺口、監控建議

### 3. 使用者體驗分析
- 使用 Task tool，設定 subagent_type="performance-engineer"
- 提示詞："分析 $ARGUMENTS 的使用者體驗指標。測量 Core Web Vitals（LCP、FID、CLS）、頁面載入時間、可互動時間和感知效能。若可用，使用 Real User Monitoring (RUM) 資料。識別效能不佳的使用者旅程及其業務影響。"
- 情境：來自步驟 1 的效能基準
- 輸出：使用者體驗效能報告、Core Web Vitals 分析、使用者影響評估

## 階段 2：資料庫與後端最佳化

### 4. 資料庫效能最佳化
- 使用 Task tool，設定 subagent_type="database-cloud-optimization::database-optimizer"
- 提示詞："根據效能分析資料 {context_from_phase_1}，優化 $ARGUMENTS 的資料庫效能。分析慢查詢日誌、建立缺少的索引、優化執行計畫、使用 Redis/Memcached 實作查詢結果快取。檢視連線池、預編譯語句和批次處理機會。若需要，考慮讀取副本和資料庫分片。"
- 情境：來自階段 1 的效能瓶頸
- 輸出：優化的查詢、新索引、快取策略、連線池配置

### 5. 後端程式碼與 API 最佳化
- 使用 Task tool，設定 subagent_type="backend-development::backend-architect"
- 提示詞："針對瓶頸 {context_from_phase_1}，優化 $ARGUMENTS 的後端服務。實作高效演算法、新增應用程式層級快取、優化 N+1 查詢、有效使用 async/await 模式。實作分頁、回應壓縮、GraphQL 查詢優化和批次 API 操作。新增斷路器和艙壁模式以提升韌性。"
- 情境：來自步驟 4 的資料庫優化、來自階段 1 的效能分析資料
- 輸出：優化的後端程式碼、快取實作、API 改善、韌性模式

### 6. 微服務與分散式系統最佳化
- 使用 Task tool，設定 subagent_type="performance-engineer"
- 提示詞："優化 $ARGUMENTS 的分散式系統效能。分析服務間通訊、實作 service mesh 優化、優化訊息佇列效能（Kafka/RabbitMQ）、減少網路跳轉。實作分散式快取策略並優化序列化/反序列化。"
- 情境：來自步驟 5 的後端優化
- 輸出：服務通訊改善、訊息佇列優化、分散式快取設定

## 階段 3：前端與 CDN 最佳化

### 7. 前端打包與載入最佳化
- 使用 Task tool，設定 subagent_type="frontend-developer"
- 提示詞："針對 Core Web Vitals {context_from_phase_1}，優化 $ARGUMENTS 的前端效能。實作程式碼分割、tree shaking、延遲載入和動態匯入。使用 webpack/rollup 分析優化打包大小。實作資源提示（prefetch、preconnect、preload）。優化關鍵渲染路徑並消除阻礙渲染的資源。"
- 情境：來自階段 1 的使用者體驗分析、來自階段 2 的後端優化
- 輸出：優化的打包檔案、延遲載入實作、改善的 Core Web Vitals

### 8. CDN 與邊緣運算最佳化
- 使用 Task tool，設定 subagent_type="cloud-infrastructure::cloud-architect"
- 提示詞："優化 $ARGUMENTS 的 CDN 和邊緣運算效能。為最佳快取配置 CloudFlare/CloudFront、實作邊緣函式處理動態內容、設定響應式圖片和 WebP/AVIF 格式的圖片優化。配置 HTTP/2 和 HTTP/3、實作 Brotli 壓縮。為全球使用者設定地理分布。"
- 情境：來自步驟 7 的前端優化
- 輸出：CDN 配置、邊緣快取規則、壓縮設定、地理位置優化

### 9. 行動裝置與漸進式網頁應用程式最佳化
- 使用 Task tool，設定 subagent_type="frontend-mobile-development::mobile-developer"
- 提示詞："優化 $ARGUMENTS 的行動裝置體驗。實作 service workers 以支援離線功能、使用自適應載入優化慢速網路。減少行動裝置 CPU 的 JavaScript 執行時間。為長列表實作虛擬捲動。優化觸控回應和流暢動畫。若適用，考慮 React Native/Flutter 特定的優化。"
- 情境：來自步驟 7-8 的前端優化
- 輸出：針對行動裝置優化的程式碼、PWA 實作、離線功能

## 階段 4：負載測試與驗證

### 10. 全面負載測試
- 使用 Task tool，設定 subagent_type="performance-engineer"
- 提示詞："使用 k6/Gatling/Artillery 對 $ARGUMENTS 進行全面負載測試。根據生產環境流量模式設計真實負載情境。測試正常負載、尖峰負載和壓力測試情境。若適用，包含 API 測試、瀏覽器測試和 WebSocket 測試。測量各種負載等級下的回應時間、吞吐量、錯誤率和資源使用率。"
- 情境：來自階段 1-3 的所有優化
- 輸出：負載測試結果、負載下的效能表現、臨界點、可擴展性分析

### 11. 效能衰退測試
- 使用 Task tool，設定 subagent_type="performance-testing-review::test-automator"
- 提示詞："為 $ARGUMENTS 建立自動化效能衰退測試。為關鍵指標設定效能預算、使用 GitHub Actions 或類似工具整合至 CI/CD 流程。建立前端的 Lighthouse CI 測試、使用 Artillery 的 API 效能測試，以及資料庫效能基準測試。實作效能衰退的自動回滾觸發機制。"
- 情境：來自步驟 10 的負載測試結果、來自階段 1 的基準指標
- 輸出：效能測試套件、CI/CD 整合、衰退預防系統

## 階段 5：監控與持續最佳化

### 12. 生產環境監控設定
- 使用 Task tool，設定 subagent_type="observability-engineer"
- 提示詞："為 $ARGUMENTS 實作生產環境效能監控。使用 DataDog/New Relic/Dynatrace 設定 APM、使用 OpenTelemetry 配置分散式追蹤、實作自訂業務指標。為關鍵指標建立 Grafana 儀表板、設定 PagerDuty 警報以偵測效能下降。為關鍵服務定義 SLI/SLO 和錯誤預算。"
- 情境：來自所有先前階段的效能改善
- 輸出：監控儀表板、警報規則、SLI/SLO 定義、運維手冊

### 13. 持續效能最佳化
- 使用 Task tool，設定 subagent_type="performance-engineer"
- 提示詞："為 $ARGUMENTS 建立持續最佳化流程。建立效能預算追蹤、實作效能變更的 A/B 測試、設定生產環境的持續效能分析。記錄最佳化機會待辦清單、建立容量規劃模型，並建立定期效能審查週期。"
- 情境：來自步驟 12 的監控設定、所有先前的優化工作
- 輸出：效能預算追蹤、最佳化待辦清單、容量規劃、審查流程

## 配置選項

- **performance_focus**: "latency" | "throughput" | "cost" | "balanced" (預設："balanced")
- **optimization_depth**: "quick-wins" | "comprehensive" | "enterprise" (預設："comprehensive")
- **tools_available**: ["datadog", "newrelic", "prometheus", "grafana", "k6", "gatling"]
- **budget_constraints**: 設定基礎架構變更的最大可接受成本
- **user_impact_tolerance**: "zero-downtime" | "maintenance-window" | "gradual-rollout"

## 成功標準

- **回應時間**：關鍵端點的 P50 < 200ms、P95 < 1s、P99 < 2s
- **Core Web Vitals**：LCP < 2.5s、FID < 100ms、CLS < 0.1
- **吞吐量**：支援當前尖峰負載的 2 倍，錯誤率 <1%
- **資料庫效能**：查詢 P95 < 100ms，無查詢 > 1s
- **資源使用率**：正常負載下 CPU < 70%、記憶體 < 80%
- **成本效益**：每美元效能至少提升 30%
- **監控覆蓋率**：100% 關鍵路徑具備儀表化和警報機制

效能最佳化目標：$ARGUMENTS