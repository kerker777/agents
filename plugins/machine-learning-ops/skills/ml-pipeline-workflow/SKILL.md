---
name: ml-pipeline-workflow
description: 建構從資料準備到模型訓練、驗證和生產環境部署的端到端 MLOps 流程。適用於建立 ML 流程、實作 MLOps 實務或自動化模型訓練和部署工作流程。
---

# ML Pipeline Workflow

從資料準備到模型部署的完整端到端 MLOps 流程協調。

## 概述

此技能提供建構生產環境 ML 流程的全面指引，處理完整生命週期：資料擷取 → 準備 → 訓練 → 驗證 → 部署 → 監控。

## 何時使用此技能

- 從零開始建構新的 ML 流程
- 為 ML 系統設計工作流程協調
- 實作資料 → 模型 → 部署自動化
- 設定可重現的訓練工作流程
- 建立基於 DAG 的 ML 協調
- 將 ML 元件整合到生產系統中

## 此技能提供的內容

### 核心能力

1. **流程架構**
   - 端到端工作流程設計
   - DAG 協調模式 (Airflow、Dagster、Kubeflow)
   - 元件相依性和資料流
   - 錯誤處理和重試策略

2. **資料準備**
   - 資料驗證和品質檢查
   - 特徵工程流程
   - 資料版本控制和譜系
   - 訓練/驗證/測試分割策略

3. **模型訓練**
   - 訓練作業協調
   - 超參數管理
   - 實驗追蹤整合
   - 分散式訓練模式

4. **模型驗證**
   - 驗證框架和指標
   - A/B 測試基礎設施
   - 效能回歸偵測
   - 模型比較工作流程

5. **部署自動化**
   - 模型服務模式
   - 金絲雀部署
   - 藍綠部署策略
   - 回滾機制

### 參考文件

詳細指南請參閱 `references/` 目錄：
- **data-preparation.md** - 資料清理、驗證和特徵工程
- **model-training.md** - 訓練工作流程和最佳實務
- **model-validation.md** - 驗證策略和指標
- **model-deployment.md** - 部署模式和服務架構

### 資產和範本

`assets/` 目錄包含：
- **pipeline-dag.yaml.template** - 工作流程協調的 DAG 範本
- **training-config.yaml** - 訓練配置範本
- **validation-checklist.md** - 部署前驗證檢查清單

## 使用模式

### 基本流程設定

```python
# 1. Define pipeline stages
stages = [
    "data_ingestion",
    "data_validation",
    "feature_engineering",
    "model_training",
    "model_validation",
    "model_deployment"
]

# 2. Configure dependencies
# See assets/pipeline-dag.yaml.template for full example
```

### 生產環境工作流程

1. **資料準備階段**
   - 從來源擷取原始資料
   - 執行資料品質檢查
   - 應用特徵轉換
   - 版本化已處理的資料集

2. **訓練階段**
   - 載入版本化的訓練資料
   - 執行訓練作業
   - 追蹤實驗和指標
   - 儲存訓練好的模型

3. **驗證階段**
   - 執行驗證測試套件
   - 與基準比較
   - 生成效能報告
   - 核准部署

4. **部署階段**
   - 打包模型產出物
   - 部署到服務基礎設施
   - 配置監控
   - 驗證生產環境流量

## 最佳實務

### 流程設計

- **模組化**：每個階段應該可以獨立測試
- **冪等性**：重新執行階段應該是安全的
- **可觀測性**：在每個階段記錄指標
- **版本控制**：追蹤資料、程式碼和模型版本
- **故障處理**：實作重試邏輯和警報

### 資料管理

- 使用資料驗證函式庫 (Great Expectations、TFX)
- 使用 DVC 或類似工具版本化資料集
- 記錄特徵工程轉換
- 維護資料譜系追蹤

### 模型操作

- 分離訓練和服務基礎設施
- 使用模型註冊表 (MLflow、Weights & Biases)
- 為新模型實作漸進式推出
- 監控模型效能漂移
- 維護回滾能力

### 部署策略

- 從影子部署開始
- 使用金絲雀發布進行驗證
- 實作 A/B 測試基礎設施
- 設定自動化回滾觸發器
- 監控延遲和吞吐量

## 整合點

### 協調工具

- **Apache Airflow**：基於 DAG 的工作流程協調
- **Dagster**：基於資產的流程協調
- **Kubeflow Pipelines**：Kubernetes 原生 ML 工作流程
- **Prefect**：現代資料流自動化

### 實驗追蹤

- MLflow 用於實驗追蹤和模型註冊表
- Weights & Biases 用於視覺化和協作
- TensorBoard 用於訓練指標

### 部署平台

- AWS SageMaker 用於託管 ML 基礎設施
- Google Vertex AI 用於 GCP 部署
- Azure ML 用於 Azure 雲端
- Kubernetes + KServe 用於雲端無關的服務

## 漸進式揭露

從基礎開始，逐步增加複雜度：

1. **第 1 級**：簡單線性流程 (資料 → 訓練 → 部署)
2. **第 2 級**：新增驗證和監控階段
3. **第 3 級**：實作超參數調整
4. **第 4 級**：新增 A/B 測試和漸進式推出
5. **第 5 級**：具備集成策略的多模型流程

## 常見模式

### 批次訓練流程

```yaml
# See assets/pipeline-dag.yaml.template
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

### 即時特徵流程

```python
# Stream processing for real-time features
# Combined with batch training
# See references/data-preparation.md
```

### 持續訓練

```python
# Automated retraining on schedule
# Triggered by data drift detection
# See references/model-training.md
```

## 疑難排解

### 常見問題

- **流程失敗**：檢查相依性和資料可用性
- **訓練不穩定**：檢視超參數和資料品質
- **部署問題**：驗證模型產出物和服務配置
- **效能下降**：監控資料漂移和模型指標

### 除錯步驟

1. 檢查每個階段的流程日誌
2. 在邊界處驗證輸入/輸出資料
3. 隔離測試元件
4. 檢視實驗追蹤指標
5. 檢查模型產出物和中繼資料

## 後續步驟

設定流程後：

1. 探索 **hyperparameter-tuning** 技能以進行優化
2. 學習 **experiment-tracking-setup** 以使用 MLflow/W&B
3. 檢視 **model-deployment-patterns** 以了解服務策略
4. 使用可觀測性工具實作監控

## 相關技能

- **experiment-tracking-setup**：MLflow 和 Weights & Biases 整合
- **hyperparameter-tuning**：自動化超參數優化
- **model-deployment-patterns**：進階部署策略
