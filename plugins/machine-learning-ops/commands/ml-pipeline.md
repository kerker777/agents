# Machine Learning Pipeline - 多 Agent MLOps 協調

為以下需求設計和實作完整的 ML 流程：$ARGUMENTS

## 思路

此工作流程協調多個專業 agent 以遵循現代 MLOps 最佳實務建構生產就緒的 ML 流程。此方法強調：

- **階段式協調**：每個階段建立在先前的輸出上，agent 之間有清晰的交接
- **現代工具整合**：實驗使用 MLflow/W&B，特徵使用 Feast/Tecton，服務使用 KServe/Seldon
- **生產優先思維**：每個元件都針對規模、監控和可靠性進行設計
- **可重現性**：資料、模型和基礎設施的版本控制
- **持續改進**：自動化重新訓練、A/B 測試和漂移偵測

多 agent 方法確保每個面向都由領域專家處理：
- 資料工程師處理資料擷取和品質
- 資料科學家設計特徵和實驗
- ML 工程師實作訓練流程
- MLOps 工程師處理生產環境部署
- 可觀測性工程師確保監控

## 階段 1：資料與需求分析

<Task>
subagent_type: data-engineer
prompt: |
  為 ML 系統分析和設計資料流程，需求如下：$ARGUMENTS

  交付項目：
  1. 資料來源稽核和擷取策略：
     - 來源系統和連線模式
     - 使用 Pydantic/Great Expectations 的模式驗證
     - 使用 DVC 或 lakeFS 的資料版本控制
     - 增量載入和 CDC 策略

  2. 資料品質框架：
     - 剖析和統計生成
     - 異常偵測規則
     - 資料譜系追蹤
     - 品質閘道和 SLA

  3. 儲存架構：
     - 原始/已處理/特徵層
     - 分割策略
     - 保留原則
     - 成本優化

  提供關鍵元件的實作程式碼和整合模式。
</Task>

<Task>
subagent_type: data-scientist
prompt: |
  為以下需求設計特徵工程和模型需求：$ARGUMENTS
  使用來自以下的資料架構：{phase1.data-engineer.output}

  交付項目：
  1. 特徵工程流程：
     - 轉換規格
     - 特徵儲存模式 (Feast/Tecton)
     - 統計驗證規則
     - 遺漏資料/離群值的處理策略

  2. 模型需求：
     - 演算法選擇理由
     - 效能指標和基準
     - 訓練資料需求
     - 評估標準和閾值

  3. 實驗設計：
     - 假設和成功指標
     - A/B 測試方法論
     - 樣本數計算
     - 偏差偵測方法

  包含特徵轉換程式碼和統計驗證邏輯。
</Task>

## 階段 2：模型開發與訓練

<Task>
subagent_type: ml-engineer
prompt: |
  根據需求實作訓練流程：{phase1.data-scientist.output}
  使用資料流程：{phase1.data-engineer.output}

  建構全面的訓練系統：
  1. 訓練流程實作：
     - 具備清晰介面的模組化訓練程式碼
     - 超參數優化 (Optuna/Ray Tune)
     - 分散式訓練支援 (Horovod/PyTorch DDP)
     - 交叉驗證和集成策略

  2. 實驗追蹤設定：
     - MLflow/Weights & Biases 整合
     - 指標記錄和視覺化
     - 產出物管理 (模型、圖表、資料樣本)
     - 實驗比較和分析工具

  3. 模型註冊表整合：
     - 版本控制和標記策略
     - 模型中繼資料和譜系
     - 升級工作流程 (dev -> staging -> prod)
     - 回滾程序

  提供完整的訓練程式碼和配置管理。
</Task>

<Task>
subagent_type: python-pro
prompt: |
  優化和生產化來自以下的 ML 程式碼：{phase2.ml-engineer.output}

  重點領域：
  1. 程式碼品質和結構：
     - 重構至生產標準
     - 新增全面的錯誤處理
     - 實作具備結構化格式的適當日誌記錄
     - 建立可重用的元件和工具

  2. 效能優化：
     - 剖析和優化瓶頸
     - 實作快取策略
     - 優化資料載入和預處理
     - 大規模訓練的記憶體管理

  3. 測試框架：
     - 資料轉換的單元測試
     - 流程元件的整合測試
     - 模型品質測試 (不變性、方向性)
     - 效能回歸測試

  交付具備完整測試覆蓋率的生產就緒、可維護程式碼。
</Task>

## 階段 3：生產環境部署與服務

<Task>
subagent_type: mlops-engineer
prompt: |
  為來自以下的模型設計生產環境部署：{phase2.ml-engineer.output}
  使用來自以下的優化程式碼：{phase2.python-pro.output}

  實作需求：
  1. 模型服務基礎設施：
     - 使用 FastAPI/TorchServe 的 REST/gRPC API
     - 批次預測流程 (Airflow/Kubeflow)
     - 串流處理 (Kafka/Kinesis 整合)
     - 模型服務平台 (KServe/Seldon Core)

  2. 部署策略：
     - 零停機時間的藍綠部署
     - 帶流量分割的金絲雀發布
     - 用於驗證的影子部署
     - A/B 測試基礎設施

  3. CI/CD 流程：
     - GitHub Actions/GitLab CI 工作流程
     - 自動化測試閘道
     - 部署前的模型驗證
     - 用於 GitOps 部署的 ArgoCD

  4. 基礎設施即程式碼：
     - 雲端資源的 Terraform 模組
     - Kubernetes 部署的 Helm chart
     - 優化的 Docker 多階段建構
     - 使用 Vault/Secrets Manager 的機密管理

  提供完整的部署配置和自動化指令碼。
</Task>

<Task>
subagent_type: kubernetes-architect
prompt: |
  為來自以下的 ML 工作負載設計 Kubernetes 基礎設施：{phase3.mlops-engineer.output}

  Kubernetes 專屬需求：
  1. 工作負載協調：
     - 使用 Kubeflow 的訓練作業排程
     - GPU 資源配置和共用
     - 現貨/可佔用執行個體整合
     - 優先級類別和資源配額

  2. 服務基礎設施：
     - 用於自動擴展的 HPA/VPA
     - 用於事件驅動擴展的 KEDA
     - 用於流量管理的 Istio 服務網格
     - 模型快取和暖機策略

  3. 儲存和資料存取：
     - 訓練資料的 PVC 策略
     - 使用 CSI 驅動程式的模型產出物儲存
     - 特徵儲存的分散式儲存
     - 用於推論優化的快取層

  提供整個 ML 平台的 Kubernetes manifest 和 Helm chart。
</Task>

## 階段 4：監控與持續改進

<Task>
subagent_type: observability-engineer
prompt: |
  為部署在以下的 ML 系統實作全面監控：{phase3.mlops-engineer.output}
  使用 Kubernetes 基礎設施：{phase3.kubernetes-architect.output}

  監控框架：
  1. 模型效能監控：
     - 預測準確性追蹤
     - 延遲和吞吐量指標
     - 特徵重要性變化
     - 商業 KPI 關聯

  2. 資料和模型漂移偵測：
     - 統計漂移偵測 (KS 測試、PSI)
     - 概念漂移監控
     - 特徵分布追蹤
     - 自動化漂移警報和報告

  3. 系統可觀測性：
     - 所有元件的 Prometheus 指標
     - 視覺化的 Grafana 儀表板
     - 使用 Jaeger/Zipkin 的分散式追蹤
     - 使用 ELK/Loki 的日誌聚合

  4. 警報和自動化：
     - PagerDuty/Opsgenie 整合
     - 自動化重新訓練觸發器
     - 效能下降工作流程
     - 事件回應手冊

  5. 成本追蹤：
     - 資源使用指標
     - 依模型/實驗的成本分配
     - 優化建議
     - 預算警報和控制

  交付監控配置、儀表板和警報規則。
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

1. **資料流程成功**：
   - 生產環境中 < 0.1% 資料品質問題
   - 自動化資料驗證通過率 99.9%
   - 完整資料譜系追蹤
   - 次秒級特徵服務延遲

2. **模型效能**：
   - 達到或超越基準指標
   - 重新訓練前 < 5% 效能下降
   - 具備統計顯著性的成功 A/B 測試
   - > 24 小時內無未偵測到的模型漂移

3. **營運卓越**：
   - 模型服務 99.9% 正常運行時間
   - < 200ms p99 推論延遲
   - 5 分鐘內自動化回滾
   - 完整可觀測性，< 1 分鐘警報時間

4. **開發速度**：
   - 從提交到生產 < 1 小時
   - 平行實驗執行
   - 可重現的訓練執行
   - 自助式模型部署

5. **成本效益**：
   - < 20% 基礎設施浪費
   - 優化的資源配置
   - 根據負載自動擴展
   - 現貨執行個體使用率 > 60%

## 最終交付項目

完成後，協調的流程將提供：
- 具備完全自動化的端到端 ML 流程
- 全面的文件和手冊
- 生產就緒的基礎設施即程式碼
- 完整的監控和警報系統
- 持續改進的 CI/CD 流程
- 成本優化和擴展策略
- 災難復原和回滾程序
