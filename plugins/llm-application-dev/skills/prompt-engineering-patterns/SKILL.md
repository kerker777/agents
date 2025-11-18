---
name: prompt-engineering-patterns
description: Master advanced prompt engineering techniques to maximize LLM performance, reliability, and controllability in production. Use when optimizing prompts, improving LLM outputs, or designing production prompt templates.
---

# 提示詞工程模式

掌握進階提示詞工程技術，最大化 LLM 的效能、可靠性和可控性。

## 何時使用此技能

- 為生產環境 LLM 應用設計複雜的提示詞
- 最佳化提示詞效能和一致性
- 實作結構化推理模式（思維鏈、思維樹）
- 建構具有動態範例選擇的少樣本學習系統
- 建立具有變數插值的可重複使用提示詞模板
- 除錯和改進產生不一致輸出的提示詞
- 為專業 AI 助理實作系統提示詞

## 核心能力

### 1. 少樣本學習
- 範例選擇策略（語意相似度、多樣性取樣）
- 在上下文視窗限制下平衡範例數量
- 使用輸入-輸出配對建構有效的示範
- 從知識庫動態檢索範例
- 透過策略性範例選擇處理邊緣案例

### 2. 思維鏈提示
- 逐步推理引導
- 零樣本 CoT 使用「讓我們一步步思考」
- 具有推理軌跡的少樣本 CoT
- 自我一致性技術（取樣多個推理路徑）
- 驗證與確認步驟

### 3. 提示詞最佳化
- 迭代改進工作流程
- A/B 測試提示詞變體
- 衡量提示詞效能指標（準確性、一致性、延遲）
- 在維持品質的同時減少令牌使用
- 處理邊緣案例和失敗模式

### 4. 模板系統
- 變數插值和格式化
- 條件式提示詞區段
- 多輪對話模板
- 基於角色的提示詞組合
- 模組化提示詞元件

### 5. 系統提示詞設計
- 設定模型行為和限制
- 定義輸出格式和結構
- 建立角色和專業知識
- 安全指南和內容政策
- 上下文設定和背景資訊

## 快速入門

```python
from prompt_optimizer import PromptTemplate, FewShotSelector

# Define a structured prompt template
template = PromptTemplate(
    system="You are an expert SQL developer. Generate efficient, secure SQL queries.",
    instruction="Convert the following natural language query to SQL:\n{query}",
    few_shot_examples=True,
    output_format="SQL code block with explanatory comments"
)

# Configure few-shot learning
selector = FewShotSelector(
    examples_db="sql_examples.jsonl",
    selection_strategy="semantic_similarity",
    max_examples=3
)

# Generate optimized prompt
prompt = template.render(
    query="Find all users who registered in the last 30 days",
    examples=selector.select(query="user registration date filter")
)
```

## 關鍵模式

### 漸進式揭露
從簡單提示詞開始，僅在需要時增加複雜性：

1. **第 1 級**：直接指令
   - "摘要這篇文章"

2. **第 2 級**：加入限制
   - "以 3 個重點摘要這篇文章，著重於關鍵發現"

3. **第 3 級**：加入推理
   - "閱讀這篇文章，識別主要發現，然後以 3 個重點摘要"

4. **第 4 級**：加入範例
   - 包含 2-3 個帶有輸入-輸出配對的範例摘要

### 指令層級
```
[系統上下文] → [任務指令] → [範例] → [輸入資料] → [輸出格式]
```

### 錯誤恢復
建構能優雅處理失敗的提示詞：
- 包含降級指令
- 要求信心分數
- 不確定時要求替代解釋
- 指定如何表示缺少的資訊

## 最佳實務

1. **具體明確**：模糊的提示詞產生不一致的結果
2. **展示，而非描述**：範例比描述更有效
3. **廣泛測試**：在多樣化、具代表性的輸入上評估
4. **快速迭代**：小改變可能產生大影響
5. **監控效能**：在生產環境中追蹤指標
6. **版本控制**：像程式碼一樣進行適當的版本管理
7. **記錄意圖**：解釋為什麼提示詞如此結構化

## 常見陷阱

- **過度工程**：在嘗試簡單提示詞之前就開始使用複雜提示詞
- **範例污染**：使用與目標任務不符的範例
- **上下文溢位**：過多範例超出令牌限制
- **模糊指令**：留下多種解釋的空間
- **忽略邊緣案例**：未在不尋常或邊界輸入上測試

## 整合模式

### 與 RAG 系統整合
```python
# Combine retrieved context with prompt engineering
prompt = f"""根據以下上下文：
{retrieved_context}

{few_shot_examples}

問題：{user_question}

僅根據上述上下文提供詳細答案。如果上下文不包含足夠資訊，明確說明缺少什麼。"""
```

### 與驗證整合
```python
# Add self-verification step
prompt = f"""{main_task_prompt}

生成回應後，驗證它是否符合這些標準：
1. 直接回答問題
2. 僅使用提供的上下文資訊
3. 引用具體來源
4. 承認任何不確定性

如果驗證失敗，請修改您的回應。"""
```

## 效能最佳化

### 令牌效率
- 移除冗餘的詞彙和片語
- 在首次定義後一致使用縮寫
- 合併類似的指令
- 將穩定內容移至系統提示詞

### 延遲降低
- 在不犧牲品質的情況下最小化提示詞長度
- 對長篇輸出使用串流
- 快取常見的提示詞前綴
- 盡可能批次處理類似請求

## 資源

- **references/few-shot-learning.md**：深入探討範例選擇和建構
- **references/chain-of-thought.md**：進階推理引導技術
- **references/prompt-optimization.md**：系統化改進工作流程
- **references/prompt-templates.md**：可重複使用的模板模式
- **references/system-prompts.md**：系統級提示詞設計
- **assets/prompt-template-library.md**：經過實戰考驗的提示詞模板
- **assets/few-shot-examples.json**：精選的範例資料集
- **scripts/optimize-prompt.py**：自動化提示詞最佳化工具

## 成功指標

追蹤提示詞的這些關鍵績效指標：
- **準確性**：輸出的正確性
- **一致性**：類似輸入間的可重現性
- **延遲**：回應時間（P50、P95、P99）
- **令牌使用**：每個請求的平均令牌數
- **成功率**：有效輸出的百分比
- **使用者滿意度**：評分和回饋

## 下一步

1. 檢視提示詞模板庫以了解常見模式
2. 針對您的特定使用案例試驗少樣本學習
3. 實作提示詞版本管理和 A/B 測試
4. 建立自動化評估流程
5. 記錄您的提示詞工程決策和學習
