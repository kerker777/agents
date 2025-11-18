# 數據驅動的功能開發

透過數據洞察、A/B 測試及持續測量來打造功能，運用專門的代理程式進行分析、實作與實驗。

[Extended thinking: This workflow orchestrates a comprehensive data-driven development process from initial data analysis and hypothesis formulation through feature implementation with integrated analytics, A/B testing infrastructure, and post-launch analysis. Each phase leverages specialized agents to ensure features are built based on data insights, properly instrumented for measurement, and validated through controlled experiments. The workflow emphasizes modern product analytics practices, statistical rigor in testing, and continuous learning from user behavior.]

## 階段一：數據分析與假設形成

### 1. 探索式數據分析
- 使用 Task 工具，設定 subagent_type="machine-learning-ops::data-scientist"
- 提示詞：「針對功能 $ARGUMENTS 執行探索式數據分析。分析既有的使用者行為數據、識別模式與機會、依行為區隔使用者，並計算基準指標。使用現代分析工具（Amplitude、Mixpanel、Segment）來了解目前的使用者旅程、轉換漏斗及互動模式。」
- 輸出：包含視覺化圖表的 EDA 報告、使用者區隔、行為模式、基準指標

### 2. 商業假設發展
- 使用 Task 工具，設定 subagent_type="business-analytics::business-analyst"
- 情境：數據科學家的 EDA 發現與行為模式
- 提示詞：「根據數據分析，為功能 $ARGUMENTS 制定商業假設。定義明確的成功指標、對關鍵商業 KPI 的預期影響、目標使用者區隔，以及最小可檢測效應。使用 ICE 評分或 RICE 優先順序等框架來建立可測量的假設。」
- 輸出：假設文件、成功指標定義、預期 ROI 計算

### 3. 統計實驗設計
- 使用 Task 工具，設定 subagent_type="machine-learning-ops::data-scientist"
- 情境：商業假設與成功指標
- 提示詞：「為功能 $ARGUMENTS 設計統計實驗。計算統計檢定力所需的樣本數、定義對照組與實驗組、指定隨機化策略，並規劃多重檢定校正。考慮使用貝氏 A/B 測試方法以加速決策。同時設計主要指標與護欄指標。」
- 輸出：實驗設計文件、檢定力分析、統計測試計畫

## 階段二：功能架構與分析設計

### 4. 功能架構規劃
- 使用 Task 工具，設定 subagent_type="data-engineering::backend-architect"
- 情境：商業需求與實驗設計
- 提示詞：「為 $ARGUMENTS 設計具備 A/B 測試能力的功能架構。包含功能旗標整合（LaunchDarkly、Split.io 或 Optimizely）、漸進式推出策略、安全的斷路器機制，以及對照組與實驗組邏輯的清楚分離。確保架構支援即時配置更新。」
- 輸出：架構圖、功能旗標 schema、推出策略

### 5. 分析埋點設計
- 使用 Task 工具，設定 subagent_type="data-engineering::data-engineer"
- 情境：功能架構與成功指標
- 提示詞：「為 $ARGUMENTS 設計完整的分析埋點。定義使用者互動的事件 schema、指定用於區隔與分析的屬性、設計漏斗追蹤與轉換事件、規劃群組分析能力。使用現代 SDK（Segment、Amplitude、Mixpanel）搭配適當的事件分類法來實作。」
- 輸出：事件追蹤計畫、分析 schema、埋點指南

### 6. 數據管線架構
- 使用 Task 工具，設定 subagent_type="data-engineering::data-engineer"
- 情境：分析需求與既有數據基礎設施
- 提示詞：「為功能 $ARGUMENTS 設計數據管線。包含即時串流以產生即時指標（Kafka、Kinesis）、批次處理以進行詳細分析、數據倉儲整合（Snowflake、BigQuery），以及若適用的話，ML 用的 feature store。確保適當的數據治理與 GDPR 合規性。」
- 輸出：管線架構、ETL/ELT 規格、數據流程圖

## 階段三：帶埋點的實作

### 7. 後端實作
- 使用 Task 工具，設定 subagent_type="backend-development::backend-architect"
- 情境：架構設計與功能需求
- 提示詞：「為功能 $ARGUMENTS 實作帶完整埋點的後端。在決策點加入功能旗標檢查、為所有使用者動作進行完整的事件追蹤、收集效能指標、進行錯誤追蹤與監控。實作適當的日誌記錄以供實驗分析。」
- 輸出：帶分析的後端程式碼、功能旗標整合、監控設定

### 8. 前端實作
- 使用 Task 工具，設定 subagent_type="frontend-mobile-development::frontend-developer"
- 情境：後端 API 與分析需求
- 提示詞：「為功能 $ARGUMENTS 建置帶分析追蹤的前端。為所有使用者互動實作事件追蹤、若適用則整合工作階段錄製、效能指標（Core Web Vitals），以及適當的錯誤邊界。確保對照組與實驗組之間的體驗一致性。」
- 輸出：帶分析的前端程式碼、A/B 測試變體、效能監控

### 9. ML 模型整合（若適用）
- 使用 Task 工具，設定 subagent_type="machine-learning-ops::ml-engineer"
- 情境：功能需求與數據管線
- 提示詞：「若需要，為功能 $ARGUMENTS 整合 ML 模型。實作低延遲的線上推論、模型版本間的 A/B 測試、模型效能追蹤，以及自動回退機制。建立模型監控以進行漂移偵測。」
- 輸出：ML 管線、模型服務基礎設施、監控設定

## 階段四：上線前驗證

### 10. 分析驗證
- 使用 Task 工具，設定 subagent_type="data-engineering::data-engineer"
- 情境：已實作的追蹤與事件 schema
- 提示詞：「為 $ARGUMENTS 驗證分析實作。在預備環境測試所有事件追蹤、驗證數據品質與完整性、驗證漏斗定義、確保適當的使用者識別與工作階段追蹤。執行數據管線的端到端測試。」
- 輸出：驗證報告、數據品質指標、追蹤覆蓋率分析

### 11. 實驗設定
- 使用 Task 工具，設定 subagent_type="cloud-infrastructure::deployment-engineer"
- 情境：功能旗標與實驗設計
- 提示詞：「為 $ARGUMENTS 配置實驗基礎設施。設定帶有適當目標規則的功能旗標、配置流量分配（從 5-10% 開始）、實作緊急停止開關、為關鍵指標設定監控告警。測試隨機化與分配邏輯。」
- 輸出：實驗配置、監控儀表板、推出計畫

## 階段五：啟動與實驗

### 12. 漸進式推出
- 使用 Task 工具，設定 subagent_type="cloud-infrastructure::deployment-engineer"
- 情境：實驗配置與監控設定
- 提示詞：「為功能 $ARGUMENTS 執行漸進式推出。從內部狗糧測試開始，然後是 beta 使用者（1-5%），逐步增加到目標流量。監控錯誤率、效能指標與早期指標。實作自動回滾機制以應對異常。」
- 輸出：推出執行、監控告警、健康指標

### 13. 即時監控
- 使用 Task 工具，設定 subagent_type="observability-monitoring::observability-engineer"
- 情境：已部署的功能與成功指標
- 提示詞：「為 $ARGUMENTS 建立完整的監控。為實驗指標建立即時儀表板、為統計顯著性配置告警、監控護欄指標以偵測負面影響、追蹤系統效能與錯誤率。使用 Datadog、New Relic 等工具或自訂儀表板。」
- 輸出：監控儀表板、告警配置、SLO 定義

## 階段六：分析與決策制定

### 14. 統計分析
- 使用 Task 工具，設定 subagent_type="machine-learning-ops::data-scientist"
- 情境：實驗數據與原始假設
- 提示詞：「分析 $ARGUMENTS 的 A/B 測試結果。計算帶信賴區間的統計顯著性、檢查區隔層級的效應、分析次要指標影響、調查任何意外模式。同時使用頻率學派與貝氏方法。若適用，考慮多重檢定。」
- 輸出：統計分析報告、顯著性檢定、區隔分析

### 15. 商業影響評估
- 使用 Task 工具，設定 subagent_type="business-analytics::business-analyst"
- 情境：統計分析與商業指標
- 提示詞：「評估功能 $ARGUMENTS 的商業影響。計算實際與預期 ROI、分析對關鍵商業指標的影響、評估包含營運開銷的成本效益、預測長期價值。對全面推出、迭代或回滾提出建議。」
- 輸出：商業影響報告、ROI 分析、建議文件

### 16. 上線後優化
- 使用 Task 工具，設定 subagent_type="machine-learning-ops::data-scientist"
- 情境：啟動結果與使用者回饋
- 提示詞：「根據數據識別 $ARGUMENTS 的優化機會。分析實驗組的使用者行為模式、識別使用者旅程中的摩擦點、根據數據提出改進建議、規劃後續實驗。使用群組分析評估長期影響。」
- 輸出：優化建議、後續實驗計畫

## 配置選項

```yaml
experiment_config:
  min_sample_size: 10000
  confidence_level: 0.95
  runtime_days: 14
  traffic_allocation: "gradual"  # gradual, fixed, or adaptive

analytics_platforms:
  - amplitude
  - segment
  - mixpanel

feature_flags:
  provider: "launchdarkly"  # launchdarkly, split, optimizely, unleash

statistical_methods:
  - frequentist
  - bayesian

monitoring:
  - real_time_metrics: true
  - anomaly_detection: true
  - automatic_rollback: true
```

## 成功準則

- **數據覆蓋率**：100% 的使用者互動皆以適當的事件 schema 追蹤
- **實驗有效性**：適當的隨機化、足夠的統計檢定力、無樣本比例不匹配
- **統計嚴謹性**：明確的顯著性檢定、適當的信賴區間、多重檢定校正
- **商業影響**：目標指標有可測量的改善，且不會降低護欄指標
- **技術效能**：p95 延遲無劣化、錯誤率低於 0.1%
- **決策速度**：在規劃的實驗執行時間內做出明確的執行/不執行決策
- **學習成果**：為未來功能開發記錄洞察

## 協調注意事項

- 數據科學家與商業分析師在假設形成上協作
- 工程師將分析作為首要需求實作，而非事後補充
- 功能旗標讓安全的實驗成為可能，無需完整部署
- 即時監控允許快速迭代，並在需要時回滾
- 統計嚴謹性與商業實用性及上市速度取得平衡
- 持續學習迴圈回饋到下一輪功能開發週期

使用數據驅動方法開發的功能：$ARGUMENTS
