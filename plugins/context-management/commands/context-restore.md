# 情境還原：進階語義記憶重建

## 角色說明

專業的情境還原專家，專注於在複雜的多代理 AI 工作流程中進行智慧化、語義感知的情境檢索與重建。專精於以高精確度和最小資訊損失保存及重建專案知識。

## 情境概述

情境還原工具是一個精密的記憶管理系統，旨在：
- 在分散式 AI 工作流程中復原和重建專案情境
- 在複雜的長期專案中實現無縫的延續性
- 提供智慧化、語義感知的情境重建
- 維護歷史知識完整性和決策可追溯性

## 核心需求與參數

### 輸入參數
- `context_source`：主要情境儲存位置（向量資料庫、檔案系統）
- `project_identifier`：唯一專案命名空間
- `restoration_mode`：
  - `full`：完整情境還原
  - `incremental`：部分情境更新
  - `diff`：比較並合併情境版本
- `token_budget`：要還原的最大情境令牌數（預設：8192）
- `relevance_threshold`：情境組件的語義相似度門檻（預設：0.75）

## 進階情境檢索策略

### 1. 語義向量搜尋
- 利用多維嵌入模型進行情境檢索
- 採用餘弦相似度和向量聚類技術
- 支援多模態嵌入（文字、程式碼、架構圖）

```python
def semantic_context_retrieve(project_id, query_vector, top_k=5):
    """Semantically retrieve most relevant context vectors"""
    vector_db = VectorDatabase(project_id)
    matching_contexts = vector_db.search(
        query_vector,
        similarity_threshold=0.75,
        max_results=top_k
    )
    return rank_and_filter_contexts(matching_contexts)
```

### 2. 相關性篩選與排序
- 實作多階段相關性評分
- 考量時間衰減、語義相似度和歷史影響
- 動態權重調整情境組件

```python
def rank_context_components(contexts, current_state):
    """Rank context components based on multiple relevance signals"""
    ranked_contexts = []
    for context in contexts:
        relevance_score = calculate_composite_score(
            semantic_similarity=context.semantic_score,
            temporal_relevance=context.age_factor,
            historical_impact=context.decision_weight
        )
        ranked_contexts.append((context, relevance_score))

    return sorted(ranked_contexts, key=lambda x: x[1], reverse=True)
```

### 3. 情境重建模式
- 實作漸進式情境載入
- 支援部分和完整情境重建
- 動態管理令牌預算

```python
def rehydrate_context(project_context, token_budget=8192):
    """Intelligent context rehydration with token budget management"""
    context_components = [
        'project_overview',
        'architectural_decisions',
        'technology_stack',
        'recent_agent_work',
        'known_issues'
    ]

    prioritized_components = prioritize_components(context_components)
    restored_context = {}

    current_tokens = 0
    for component in prioritized_components:
        component_tokens = estimate_tokens(component)
        if current_tokens + component_tokens <= token_budget:
            restored_context[component] = load_component(component)
            current_tokens += component_tokens

    return restored_context
```

### 4. 工作階段狀態重建
- 重建代理工作流程狀態
- 保存決策軌跡和推理情境
- 支援多代理協作歷程

### 5. 情境合併與衝突解決
- 實作三方合併策略
- 偵測並解決語義衝突
- 維護來源和決策可追溯性

### 6. 漸進式情境載入
- 支援情境組件的延遲載入
- 為大型專案實作情境串流
- 啟用動態情境擴展

### 7. 情境驗證與完整性檢查
- 情境密碼學簽章
- 語義一致性驗證
- 版本相容性檢查

### 8. 效能最佳化
- 實作高效的快取機制
- 使用機率型資料結構進行情境索引
- 最佳化向量搜尋演算法

## 參考工作流程

### 工作流程 1：專案恢復
1. 檢索最新的專案情境
2. 對照當前程式碼庫驗證情境
3. 選擇性還原相關組件
4. 產生恢復摘要

### 工作流程 2：跨專案知識轉移
1. 從來源專案提取語義向量
2. 對應並轉移相關知識
3. 將情境適配到目標專案領域
4. 驗證知識可轉移性

## 使用範例

```bash
# Full context restoration
context-restore project:ai-assistant --mode full

# Incremental context update
context-restore project:web-platform --mode incremental

# Semantic context query
context-restore project:ml-pipeline --query "model training strategy"
```

## 整合模式
- RAG（檢索增強生成）管線
- 多代理工作流程協調
- 持續學習系統
- 企業知識管理

## 未來發展藍圖
- 增強多模態嵌入支援
- 量子啟發式向量搜尋演算法
- 自我修復的情境重建
- 自適應學習情境策略
