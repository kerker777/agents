# Machine Learning Pipeline - 多代理 MLOps 協調編排

為以下需求設計並實作完整的 ML 流程管線：$ARGUMENTS

## 設計思維

此工作流程協調多個專業代理，遵循現代 MLOps 最佳實踐建構可投入生產環境的 ML 流程管線。此方法強調：

- **基於階段的協調**：每個階段建立在前一階段的輸出之上，代理之間有明確的交接
- **現代工具整合**：MLflow/W&B 用於實驗追蹤、Feast/Tecton 用於特徵管理、KServe/Seldon 用於模型服務
- **生產環境優先思維**：每個元件都為擴展性、監控和可靠性而設計
- **可重現性**：對資料、模型和基礎設施進行版本控制
- **持續改進**：自動化重新訓練、A/B 測試和漂移偵測

多代理方法確保每個面向都由領域專家處理：
- 資料工程師處理資料擷取和品質管控
- 資料科學家設計特徵和實驗
- ML 工程師實作訓練流程管線
- MLOps 工程師處理生產環境部署
- 可觀測性工程師確保監控機制

## 階段 1：資料與需求分析

<Task>
subagent_type: data-engineer
prompt: |
  分析並設計 ML 系統的資料流程管線，需求如下：$ARGUMENTS

  交付成果：
  1. 資料來源稽核與擷取策略：
     - 來源系統與連線模式
     - 使用 Pydantic/Great Expectations 進行架構驗證
     - 使用 DVC 或 lakeFS 進行資料版本控制
     - 增量載入與 CDC（變更資料擷取）策略

  2. 資料品質框架：
     - 資料剖析與統計資料生成
     - 異常偵測規則
     - 資料血緣追蹤
     - 品質閘門與 SLA

  3. 儲存架構：
     - 原始/處理/特徵資料層
     - 分區策略
     - 保留政策
     - 成本最佳化

  提供關鍵元件的實作程式碼與整合模式。
</Task>

<Task>
subagent_type: data-scientist
prompt: |
  為以下需求設計特徵工程與模型要求：$ARGUMENTS
  使用資料架構來自：{phase1.data-engineer.output}

  交付成果：
  1. 特徵工程流程管線：
     - 轉換規格
     - 特徵儲存架構（Feast/Tecton）
     - 統計驗證規則
     - 遺漏資料/離群值的處理策略

  2. 模型需求：
     - 演算法選擇理由
     - 效能指標與基準線
     - 訓練資料需求
     - 評估標準與閾值

  3. 實驗設計：
     - 假設與成功指標
     - A/B 測試方法論
     - 樣本數計算
     - 偏差偵測方法

  包含特徵轉換程式碼與統計驗證邏輯。
</Task>

## 階段 2：模型開發與訓練

<Task>
subagent_type: ml-engineer
prompt: |
  根據需求實作訓練流程管線：{phase1.data-scientist.output}
  使用資料流程管線：{phase1.data-engineer.output}

  建構完整的訓練系統：
  1. 訓練流程管線實作：
     - 具備清晰介面的模組化訓練程式碼
     - 超參數最佳化（Optuna/Ray Tune）
     - 分散式訓練支援（Horovod/PyTorch DDP）
     - 交叉驗證與集成策略

  2. 實驗追蹤設定：
     - MLflow/Weights & Biases 整合
     - 指標記錄與視覺化
     - 產出物管理（模型、圖表、資料樣本）
     - 實驗比較與分析工具

  3. 模型註冊中心整合：
     - 版本控制與標籤策略
     - 模型後設資料與血緣
     - 晉升工作流程（dev -> staging -> prod）
     - 回退程序

  提供完整的訓練程式碼與配置管理。
</Task>

<Task>
subagent_type: python-pro
prompt: |
  最佳化並生產化來自以下的 ML 程式碼：{phase2.ml-engineer.output}

  重點領域：
  1. 程式碼品質與結構：
     - 重構至生產環境標準
     - 新增完整的錯誤處理
     - 實作結構化格式的適當日誌記錄
     - 建立可重用的元件與工具程式

  2. 效能最佳化：
     - 分析並最佳化瓶頸
     - 實作快取策略
     - 最佳化資料載入與預處理
     - 大規模訓練的記憶體管理

  3. 測試框架：
     - 資料轉換的單元測試
     - 流程管線元件的整合測試
     - 模型品質測試（不變性、方向性）
     - 效能回歸測試

  交付可投入生產環境、易於維護且具備完整測試覆蓋率的程式碼。
</Task>

## 階段 3：生產環境部署與服務

<Task>
subagent_type: mlops-engineer
prompt: |
  為來自以下的模型設計生產環境部署：{phase2.ml-engineer.output}
  使用最佳化程式碼來自：{phase2.python-pro.output}

  實作需求：
  1. 模型服務基礎設施：
     - 使用 FastAPI/TorchServe 的 REST/gRPC API
     - 批次預測流程管線（Airflow/Kubeflow）
     - 串流處理（Kafka/Kinesis 整合）
     - 模型服務平台（KServe/Seldon Core）

  2. 部署策略：
     - 藍綠部署以實現零停機時間
     - 金絲雀發布與流量分配
     - 影子部署用於驗證
     - A/B 測試基礎設施

  3. CI/CD 流程管線：
     - GitHub Actions/GitLab CI 工作流程
     - 自動化測試閘門
     - 部署前的模型驗證
     - 使用 ArgoCD 進行 GitOps 部署

  4. 基礎設施即程式碼（Infrastructure as Code）：
     - 雲端資源的 Terraform 模組
     - Kubernetes 部署的 Helm charts
     - Docker 多階段建置以進行最佳化
     - 使用 Vault/Secrets Manager 進行機密管理

  提供完整的部署配置與自動化腳本。
</Task>

<Task>
subagent_type: kubernetes-architect
prompt: |
  為來自以下的 ML 工作負載設計 Kubernetes 基礎設施：{phase3.mlops-engineer.output}

  Kubernetes 特定需求：
  1. 工作負載編排：
     - 使用 Kubeflow 進行訓練任務排程
     - GPU 資源分配與共享
     - Spot/可搶佔執行個體整合
     - 優先級類別與資源配額

  2. 服務基礎設施：
     - HPA/VPA 用於自動擴展
     - KEDA 用於事件驅動擴展
     - Istio 服務網格用於流量管理
     - 模型快取與預熱策略

  3. 儲存與資料存取：
     - 訓練資料的 PVC 策略
     - 使用 CSI 驅動程式的模型產出物儲存
     - 特徵儲存的分散式儲存
     - 推論最佳化的快取層

  提供整個 ML 平台的 Kubernetes manifests 與 Helm charts。
</Task>

## 階段 4：監控與持續改進

<Task>
subagent_type: observability-engineer
prompt: |
  為部署於以下位置的 ML 系統實作全面監控：{phase3.mlops-engineer.output}
  使用 Kubernetes 基礎設施：{phase3.kubernetes-architect.output}

  監控框架：
  1. 模型效能監控：
     - 預測準確度追蹤
     - 延遲與吞吐量指標
     - 特徵重要性變化
     - 業務 KPI 關聯性

  2. 資料與模型漂移偵測：
     - 統計漂移偵測（KS test、PSI）
     - 概念漂移監控
     - 特徵分布追蹤
     - 自動化漂移告警與報告

  3. 系統可觀測性：
     - 所有元件的 Prometheus 指標
     - 視覺化的 Grafana 儀表板
     - 使用 Jaeger/Zipkin 進行分散式追蹤
     - 使用 ELK/Loki 進行日誌聚合

  4. 告警與自動化：
     - PagerDuty/Opsgenie 整合
     - 自動化重新訓練觸發器
     - 效能降級工作流程
     - 事件回應操作手冊

  5. 成本追蹤：
     - 資源使用率指標
     - 按模型/實驗的成本分配
     - 最佳化建議
     - 預算告警與控制

  交付監控配置、儀表板與告警規則。
</Task>

## 配置選項

- **experiment_tracking**：mlflow | wandb | neptune | clearml
- **feature_store**：feast | tecton | databricks | custom
- **serving_platform**：kserve | seldon | torchserve | triton
- **orchestration**：kubeflow | airflow | prefect | dagster
- **cloud_provider**：aws | azure | gcp | multi-cloud
- **deployment_mode**：realtime | batch | streaming | hybrid
- **monitoring_stack**：prometheus | datadog | newrelic | custom

## 成功標準

1. **資料流程管線成功**：
   - 生產環境中資料品質問題 < 0.1%
   - 自動化資料驗證通過率達 99.9%
   - 完整的資料血緣追蹤
   - 特徵服務延遲低於 1 秒

2. **模型效能**：
   - 達到或超越基準線指標
   - 重新訓練前的效能降低 < 5%
   - A/B 測試成功且具統計顯著性
   - 模型漂移未偵測時間 < 24 小時

3. **營運卓越**：
   - 模型服務正常運作時間達 99.9%
   - p99 推論延遲 < 200ms
   - 5 分鐘內自動回退
   - 完整的可觀測性，告警時間 < 1 分鐘

4. **開發速度**：
   - 從提交到生產環境 < 1 小時
   - 平行實驗執行
   - 可重現的訓練執行
   - 自助式模型部署

5. **成本效率**：
   - 基礎設施浪費 < 20%
   - 最佳化的資源分配
   - 基於負載的自動擴展
   - Spot 執行個體使用率 > 60%

## 最終交付成果

完成後，協調編排的流程管線將提供：
- 具備完整自動化的端到端 ML 流程管線
- 全面的文件與操作手冊
- 可投入生產環境的基礎設施即程式碼
- 完整的監控與告警系統
- 用於持續改進的 CI/CD 流程管線
- 成本最佳化與擴展策略
- 災難復原與回退程序
