---
name: ml-pipeline-workflow
description: 建立從資料準備到模型訓練、驗證及正式部署的端對端 MLOps 流水線。適用於建立 ML 流水線、實施 MLOps 實務，或自動化模型訓練和部署工作流程。
---

# ML Pipeline Workflow（ML 流水線工作流程）

從資料準備到模型部署的完整端對端 MLOps 流水線編排。

## 概述

本技能提供建立正式環境 ML 流水線的全面性指引，涵蓋完整的生命週期：資料擷取 → 準備 → 訓練 → 驗證 → 部署 → 監控。

## 何時使用本技能

- 從頭開始建立新的 ML 流水線
- 為 ML 系統設計工作流程編排
- 實作資料 → 模型 → 部署的自動化
- 建立可重現的訓練工作流程
- 建立基於 DAG 的 ML 編排
- 將 ML 元件整合至正式環境系統

## 本技能提供的功能

### 核心能力

1. **流水線架構（Pipeline Architecture）**
   - 端對端工作流程設計
   - DAG 編排模式（Airflow、Dagster、Kubeflow）
   - 元件相依性和資料流
   - 錯誤處理和重試策略

2. **資料準備（Data Preparation）**
   - 資料驗證和品質檢查
   - 特徵工程流水線
   - 資料版本控制和資料譜系
   - 訓練/驗證/測試資料集分割策略

3. **模型訓練（Model Training）**
   - 訓練作業編排
   - 超參數管理
   - 實驗追蹤整合
   - 分散式訓練模式

4. **模型驗證（Model Validation）**
   - 驗證框架和指標
   - A/B 測試基礎設施
   - 效能衰退偵測
   - 模型比較工作流程

5. **部署自動化（Deployment Automation）**
   - 模型服務模式
   - 金絲雀部署（Canary deployments）
   - 藍綠部署策略（Blue-green deployment）
   - 回滾機制

### 參考文件

詳細指南請參閱 `references/` 目錄：
- **data-preparation.md** - 資料清理、驗證和特徵工程
- **model-training.md** - 訓練工作流程和最佳實務
- **model-validation.md** - 驗證策略和指標
- **model-deployment.md** - 部署模式和服務架構

### 資源和範本

`assets/` 目錄包含：
- **pipeline-dag.yaml.template** - 工作流程編排的 DAG 範本
- **training-config.yaml** - 訓練設定範本
- **validation-checklist.md** - 部署前驗證檢查清單

## 使用模式

### 基礎流水線設定

```python
# 1. 定義流水線階段
stages = [
    "data_ingestion",
    "data_validation",
    "feature_engineering",
    "model_training",
    "model_validation",
    "model_deployment"
]

# 2. 設定相依性
# 完整範例請參閱 assets/pipeline-dag.yaml.template
```

### 正式環境工作流程

1. **資料準備階段**
   - 從來源擷取原始資料
   - 執行資料品質檢查
   - 套用特徵轉換
   - 為處理後的資料集建立版本

2. **訓練階段**
   - 載入已版本控制的訓練資料
   - 執行訓練作業
   - 追蹤實驗和指標
   - 儲存訓練完成的模型

3. **驗證階段**
   - 執行驗證測試套件
   - 與基準進行比較
   - 產生效能報告
   - 核准部署

4. **部署階段**
   - 打包模型產出物
   - 部署至服務基礎設施
   - 設定監控
   - 驗證正式環境流量

## 最佳實務

### 流水線設計

- **模組化（Modularity）**：每個階段都應該能獨立測試
- **冪等性（Idempotency）**：重新執行階段應該是安全的
- **可觀測性（Observability）**：在每個階段記錄指標
- **版本控制（Versioning）**：追蹤資料、程式碼和模型版本
- **故障處理（Failure Handling）**：實作重試邏輯和告警

### 資料管理

- 使用資料驗證函式庫（Great Expectations、TFX）
- 使用 DVC 或類似工具為資料集建立版本
- 記錄特徵工程轉換
- 維護資料譜系追蹤

### 模型維運

- 分離訓練和服務基礎設施
- 使用模型註冊表（MLflow、Weights & Biases）
- 實作新模型的逐步推出
- 監控模型效能漂移
- 維護回滾能力

### 部署策略

- 從影子部署（shadow deployments）開始
- 使用金絲雀發布（canary releases）進行驗證
- 實作 A/B 測試基礎設施
- 設定自動回滾觸發器
- 監控延遲和吞吐量

## 整合點

### 編排工具

- **Apache Airflow**：基於 DAG 的工作流程編排
- **Dagster**：基於資產的流水線編排
- **Kubeflow Pipelines**：原生 Kubernetes 的 ML 工作流程
- **Prefect**：現代化資料流自動化

### 實驗追蹤

- MLflow 用於實驗追蹤和模型註冊表
- Weights & Biases 用於視覺化和協作
- TensorBoard 用於訓練指標

### 部署平台

- AWS SageMaker 用於託管式 ML 基礎設施
- Google Vertex AI 用於 GCP 部署
- Azure ML 用於 Azure 雲端
- Kubernetes + KServe 用於雲端中立的服務

## 漸進式揭露

從基礎開始逐步增加複雜度：

1. **Level 1**：簡單線性流水線（資料 → 訓練 → 部署）
2. **Level 2**：新增驗證和監控階段
3. **Level 3**：實作超參數調整
4. **Level 4**：新增 A/B 測試和逐步推出
5. **Level 5**：具有集成策略的多模型流水線

## 常見模式

### 批次訓練流水線

```yaml
# 參閱 assets/pipeline-dag.yaml.template
stages:
  - name: data_preparation
    dependencies: []
  - name: model_training
    dependencies: [data_preparation]
  - name: model_evaluation
    dependencies: [model_training]
  - name: model_deployment
    dependencies: [model_evaluation]
```

### 即時特徵流水線

```python
# 用於即時特徵的串流處理
# 與批次訓練結合
# 參閱 references/data-preparation.md
```

### 持續訓練

```python
# 按排程自動重新訓練
# 由資料漂移偵測觸發
# 參閱 references/model-training.md
```

## 疑難排解

### 常見問題

- **流水線失敗**：檢查相依性和資料可用性
- **訓練不穩定**：檢視超參數和資料品質
- **部署問題**：驗證模型產出物和服務設定
- **效能降低**：監控資料漂移和模型指標

### 除錯步驟

1. 檢查每個階段的流水線日誌
2. 驗證邊界處的輸入/輸出資料
3. 獨立測試元件
4. 檢視實驗追蹤指標
5. 檢查模型產出物和中繼資料

## 下一步

設定流水線後：

1. 探索 **hyperparameter-tuning** 技能以進行最佳化
2. 學習 **experiment-tracking-setup** 以使用 MLflow/W&B
3. 檢視 **model-deployment-patterns** 以了解服務策略
4. 使用可觀測性工具實作監控

## 相關技能

- **experiment-tracking-setup**：MLflow 和 Weights & Biases 整合
- **hyperparameter-tuning**：自動化超參數最佳化
- **model-deployment-patterns**：進階部署策略
