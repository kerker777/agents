# 少樣本學習指南

## 概述

少樣本學習使 LLM 能夠透過在提示詞中提供少量範例（通常 1-10 個）來執行任務。這種技術對於需要特定格式、風格或領域知識的任務非常有效。

## 範例選擇策略

### 1. 語意相似度
使用基於嵌入的檢索選擇與輸入查詢最相似的範例。

```python
from sentence_transformers import SentenceTransformer
import numpy as np

class SemanticExampleSelector:
    def __init__(self, examples, model_name='all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
        self.examples = examples
        self.example_embeddings = self.model.encode([ex['input'] for ex in examples])

    def select(self, query, k=3):
        query_embedding = self.model.encode([query])
        similarities = np.dot(self.example_embeddings, query_embedding.T).flatten()
        top_indices = np.argsort(similarities)[-k:][::-1]
        return [self.examples[i] for i in top_indices]
```

**最適合**：問答、文字分類、擷取任務

### 2. 多樣性取樣
最大化不同模式和邊緣案例的覆蓋範圍。

```python
from sklearn.cluster import KMeans

class DiversityExampleSelector:
    def __init__(self, examples, model_name='all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
        self.examples = examples
        self.embeddings = self.model.encode([ex['input'] for ex in examples])

    def select(self, k=5):
        # Use k-means to find diverse cluster centers
        kmeans = KMeans(n_clusters=k, random_state=42)
        kmeans.fit(self.embeddings)

        # Select example closest to each cluster center
        diverse_examples = []
        for center in kmeans.cluster_centers_:
            distances = np.linalg.norm(self.embeddings - center, axis=1)
            closest_idx = np.argmin(distances)
            diverse_examples.append(self.examples[closest_idx])

        return diverse_examples
```

**最適合**：展示任務變化性、邊緣案例處理

### 3. 基於難度的選擇
逐步增加範例複雜度以搭建學習鷹架。

```python
class ProgressiveExampleSelector:
    def __init__(self, examples):
        # Examples should have 'difficulty' scores (0-1)
        self.examples = sorted(examples, key=lambda x: x['difficulty'])

    def select(self, k=3):
        # Select examples with linearly increasing difficulty
        step = len(self.examples) // k
        return [self.examples[i * step] for i in range(k)]
```

**最適合**：複雜推理任務、程式碼生成

### 4. 基於錯誤的選擇
包含處理常見失敗模式的範例。

```python
class ErrorGuidedSelector:
    def __init__(self, examples, error_patterns):
        self.examples = examples
        self.error_patterns = error_patterns  # Common mistakes to avoid

    def select(self, query, k=3):
        # Select examples demonstrating correct handling of error patterns
        selected = []
        for pattern in self.error_patterns[:k]:
            matching = [ex for ex in self.examples if pattern in ex['demonstrates']]
            if matching:
                selected.append(matching[0])
        return selected
```

**最適合**：具有已知失敗模式的任務、安全關鍵應用

## 範例建構最佳實務

### 格式一致性
所有範例應遵循相同的格式：

```python
# Good: Consistent format
examples = [
    {
        "input": "What is the capital of France?",
        "output": "Paris"
    },
    {
        "input": "What is the capital of Germany?",
        "output": "Berlin"
    }
]

# Bad: Inconsistent format
examples = [
    "Q: What is the capital of France? A: Paris",
    {"question": "What is the capital of Germany?", "answer": "Berlin"}
]
```

### 輸入-輸出對齊
確保範例展示您希望模型執行的確切任務：

```python
# Good: Clear input-output relationship
example = {
    "input": "情感：這部電影糟糕又無聊。",
    "output": "負面"
}

# Bad: Ambiguous relationship
example = {
    "input": "這部電影糟糕又無聊。",
    "output": "這條評論對電影表達了負面情感。"
}
```

### 複雜度平衡
包含涵蓋預期難度範圍的範例：

```python
examples = [
    # Simple case
    {"input": "2 + 2", "output": "4"},

    # Moderate case
    {"input": "15 * 3 + 8", "output": "53"},

    # Complex case
    {"input": "(12 + 8) * 3 - 15 / 5", "output": "57"}
]
```

## 上下文視窗管理

### 令牌預算分配
4K 上下文視窗的典型分配：

```
系統提示詞：       500 令牌  (12%)
少樣本範例：      1500 令牌  (38%)
使用者輸入：       500 令牌  (12%)
回應：           1500 令牌  (38%)
```

### 動態範例截斷
```python
class TokenAwareSelector:
    def __init__(self, examples, tokenizer, max_tokens=1500):
        self.examples = examples
        self.tokenizer = tokenizer
        self.max_tokens = max_tokens

    def select(self, query, k=5):
        selected = []
        total_tokens = 0

        # Start with most relevant examples
        candidates = self.rank_by_relevance(query)

        for example in candidates[:k]:
            example_tokens = len(self.tokenizer.encode(
                f"Input: {example['input']}\nOutput: {example['output']}\n\n"
            ))

            if total_tokens + example_tokens <= self.max_tokens:
                selected.append(example)
                total_tokens += example_tokens
            else:
                break

        return selected
```

## 邊緣案例處理

### 包含邊界範例
```python
edge_case_examples = [
    # Empty input
    {"input": "", "output": "請提供輸入文字。"},

    # Very long input (truncated in example)
    {"input": "..." + "word " * 1000, "output": "輸入超過最大長度。"},

    # Ambiguous input
    {"input": "bank", "output": "模稜兩可：可能指金融機構或河岸。"},

    # Invalid input
    {"input": "!@#$%", "output": "無效的輸入格式。請提供有效文字。"}
]
```

## 少樣本提示詞模板

### 分類模板
```python
def build_classification_prompt(examples, query, labels):
    prompt = f"將文字分類到以下類別之一：{', '.join(labels)}\n\n"

    for ex in examples:
        prompt += f"文字：{ex['input']}\n類別：{ex['output']}\n\n"

    prompt += f"文字：{query}\n類別："
    return prompt
```

### 擷取模板
```python
def build_extraction_prompt(examples, query):
    prompt = "從文字中擷取結構化資訊。\n\n"

    for ex in examples:
        prompt += f"文字：{ex['input']}\n擷取：{json.dumps(ex['output'])}\n\n"

    prompt += f"文字：{query}\n擷取："
    return prompt
```

### 轉換模板
```python
def build_transformation_prompt(examples, query):
    prompt = "根據範例中顯示的模式轉換輸入。\n\n"

    for ex in examples:
        prompt += f"輸入：{ex['input']}\n輸出：{ex['output']}\n\n"

    prompt += f"輸入：{query}\n輸出："
    return prompt
```

## 評估與最佳化

### 範例品質指標
```python
def evaluate_example_quality(example, validation_set):
    metrics = {
        'clarity': rate_clarity(example),  # 0-1 score
        'representativeness': calculate_similarity_to_validation(example, validation_set),
        'difficulty': estimate_difficulty(example),
        'uniqueness': calculate_uniqueness(example, other_examples)
    }
    return metrics
```

### A/B 測試範例集
```python
class ExampleSetTester:
    def __init__(self, llm_client):
        self.client = llm_client

    def compare_example_sets(self, set_a, set_b, test_queries):
        results_a = self.evaluate_set(set_a, test_queries)
        results_b = self.evaluate_set(set_b, test_queries)

        return {
            'set_a_accuracy': results_a['accuracy'],
            'set_b_accuracy': results_b['accuracy'],
            'winner': 'A' if results_a['accuracy'] > results_b['accuracy'] else 'B',
            'improvement': abs(results_a['accuracy'] - results_b['accuracy'])
        }

    def evaluate_set(self, examples, test_queries):
        correct = 0
        for query in test_queries:
            prompt = build_prompt(examples, query['input'])
            response = self.client.complete(prompt)
            if response == query['expected_output']:
                correct += 1
        return {'accuracy': correct / len(test_queries)}
```

## 進階技術

### 元學習（學習選擇）
訓練一個小型模型來預測哪些範例最有效：

```python
from sklearn.ensemble import RandomForestClassifier

class LearnedExampleSelector:
    def __init__(self):
        self.selector_model = RandomForestClassifier()

    def train(self, training_data):
        # training_data: list of (query, example, success) tuples
        features = []
        labels = []

        for query, example, success in training_data:
            features.append(self.extract_features(query, example))
            labels.append(1 if success else 0)

        self.selector_model.fit(features, labels)

    def extract_features(self, query, example):
        return [
            semantic_similarity(query, example['input']),
            len(example['input']),
            len(example['output']),
            keyword_overlap(query, example['input'])
        ]

    def select(self, query, candidates, k=3):
        scores = []
        for example in candidates:
            features = self.extract_features(query, example)
            score = self.selector_model.predict_proba([features])[0][1]
            scores.append((score, example))

        return [ex for _, ex in sorted(scores, reverse=True)[:k]]
```

### 自適應範例數量
根據任務難度動態調整範例數量：

```python
class AdaptiveExampleSelector:
    def __init__(self, examples):
        self.examples = examples

    def select(self, query, max_examples=5):
        # Start with 1 example
        for k in range(1, max_examples + 1):
            selected = self.get_top_k(query, k)

            # Quick confidence check (could use a lightweight model)
            if self.estimated_confidence(query, selected) > 0.9:
                return selected

        return selected  # Return max_examples if never confident enough
```

## 常見錯誤

1. **過多範例**：更多不一定更好；可能會稀釋焦點
2. **不相關範例**：範例應與目標任務緊密匹配
3. **格式不一致**：混淆模型對輸出格式的理解
4. **過度擬合範例**：模型過於照字面複製範例模式
5. **忽略令牌限制**：用盡實際輸入/輸出的空間

## 資源

- 範例資料集儲存庫
- 常見任務的預建範例選擇器
- 少樣本效能評估框架
- 不同模型的令牌計數工具
