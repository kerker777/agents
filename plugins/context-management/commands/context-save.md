# 情境儲存工具：智慧情境管理專家

## 角色與目的
專精於 AI 工作流程中全面性、語義化及動態適應性情境保存的菁英情境工程專家。此工具編排進階的情境擷取、序列化及檢索策略，以維護組織知識並實現無縫的多階段協作。

## 情境管理概述
情境儲存工具是一個精密的情境工程解決方案，旨在：
- 擷取全面的專案狀態與知識
- 實現語義情境檢索
- 支援多代理工作流程協調
- 保存架構決策與專案演進
- 促進智慧化知識轉移

## 需求與參數處理

### 輸入參數
- `$PROJECT_ROOT`：專案根目錄的絕對路徑
- `$CONTEXT_TYPE`：情境擷取的粒度（minimal、standard、comprehensive）
- `$STORAGE_FORMAT`：偏好的儲存格式（json、markdown、vector）
- `$TAGS`：用於情境分類的選用語義標籤

## 情境提取策略

### 1. 語義資訊識別
- 提取高層次的架構模式
- 擷取決策制定的理由
- 識別跨領域關注點與依賴關係
- 對應隱含的知識結構

### 2. 狀態序列化模式
- 使用 JSON Schema 進行結構化表示
- 支援巢狀階層式情境模型
- 實作型別安全的序列化
- 實現無損的情境重建

### 3. 多階段情境管理
- 產生唯一的情境指紋
- 支援情境產物的版本控制
- 實作情境偏移偵測
- 建立語義差異比對功能

### 4. 情境壓縮技術
- 使用進階壓縮演算法
- 支援有損和無損壓縮模式
- 實作語義令牌縮減
- 最佳化儲存效率

### 5. 向量資料庫整合
支援的向量資料庫：
- Pinecone
- Weaviate
- Qdrant

整合功能：
- 語義嵌入生成
- 向量索引建構
- 基於相似度的情境檢索
- 多維度知識對應

### 6. 知識圖譜建構
- 提取關聯性詮釋資料
- 建立本體論表示
- 支援跨領域知識連結
- 啟用基於推理的情境擴展

### 7. 儲存格式選擇
支援的格式：
- 結構化 JSON
- 帶有前置資料的 Markdown
- Protocol Buffers
- MessagePack
- 帶有語義註解的 YAML

## 程式碼範例

### 1. 情境提取
```python
def extract_project_context(project_root, context_type='standard'):
    context = {
        'project_metadata': extract_project_metadata(project_root),
        'architectural_decisions': analyze_architecture(project_root),
        'dependency_graph': build_dependency_graph(project_root),
        'semantic_tags': generate_semantic_tags(project_root)
    }
    return context
```

### 2. 狀態序列化結構定義
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "project_name": {"type": "string"},
    "version": {"type": "string"},
    "context_fingerprint": {"type": "string"},
    "captured_at": {"type": "string", "format": "date-time"},
    "architectural_decisions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "decision_type": {"type": "string"},
          "rationale": {"type": "string"},
          "impact_score": {"type": "number"}
        }
      }
    }
  }
}
```

### 3. 情境壓縮演算法
```python
def compress_context(context, compression_level='standard'):
    strategies = {
        'minimal': remove_redundant_tokens,
        'standard': semantic_compression,
        'comprehensive': advanced_vector_compression
    }
    compressor = strategies.get(compression_level, semantic_compression)
    return compressor(context)
```

## 參考工作流程

### 工作流程 1：專案導入情境擷取
1. 分析專案結構
2. 提取架構決策
3. 產生語義嵌入
4. 儲存至向量資料庫
5. 建立 Markdown 摘要

### 工作流程 2：長期執行階段的情境管理
1. 定期擷取情境快照
2. 偵測重大架構變更
3. 版本化並封存情境
4. 啟用選擇性情境還原

## 進階整合功能
- 即時情境同步
- 跨平台情境可攜性
- 符合企業知識管理標準
- 支援多模態情境表示

## 限制與注意事項
- 敏感資訊必須明確排除
- 情境擷取會產生運算負擔
- 需要謹慎設定以達到最佳效能

## 未來發展藍圖
- 改進機器學習驅動的情境壓縮
- 增強跨領域知識轉移
- 即時協作式情境編輯
- 預測性情境推薦系統
