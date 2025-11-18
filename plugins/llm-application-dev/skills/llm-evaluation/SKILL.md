---
name: llm-evaluation
description: Implement comprehensive evaluation strategies for LLM applications using automated metrics, human feedback, and benchmarking. Use when testing LLM performance, measuring AI application quality, or establishing evaluation frameworks.
---

# LLM 評估

掌握 LLM 應用的全面評估策略，從自動化指標到人工評估和 A/B 測試。

## 何時使用此技能

- 系統化地衡量 LLM 應用效能
- 比較不同模型或提示詞
- 在部署前偵測效能衰退
- 驗證提示詞變更帶來的改進
- 對生產系統建立信心
- 建立基準並追蹤隨時間的進展
- 除錯意外的模型行為

## 核心評估類型

### 1. 自動化指標
使用計算分數進行快速、可重複、可擴展的評估。

**文字生成：**
- **BLEU**：N-gram 重疊（翻譯）
- **ROUGE**：以召回率為導向（摘要）
- **METEOR**：語意相似度
- **BERTScore**：基於嵌入的相似度
- **Perplexity**：語言模型信心度

**分類：**
- **Accuracy**：正確百分比
- **Precision/Recall/F1**：特定類別的效能
- **Confusion Matrix**：錯誤模式
- **AUC-ROC**：排名品質

**檢索 (RAG)：**
- **MRR**：平均倒數排名
- **NDCG**：正規化折扣累積增益
- **Precision@K**：前 K 名中的相關項目
- **Recall@K**：前 K 名的覆蓋率

### 2. 人工評估
對難以自動化的品質面向進行人工評估。

**維度：**
- **Accuracy**：事實正確性
- **Coherence**：邏輯流暢性
- **Relevance**：回答問題
- **Fluency**：自然語言品質
- **Safety**：無有害內容
- **Helpfulness**：對使用者有用

### 3. LLM 作為評審
使用更強的 LLM 來評估較弱模型的輸出。

**方法：**
- **Pointwise**：評分單一回應
- **Pairwise**：比較兩個回應
- **Reference-based**：與黃金標準比較
- **Reference-free**：無需真實答案的判斷

## 快速入門

```python
from llm_eval import EvaluationSuite, Metric

# Define evaluation suite
suite = EvaluationSuite([
    Metric.accuracy(),
    Metric.bleu(),
    Metric.bertscore(),
    Metric.custom(name="groundedness", fn=check_groundedness)
])

# Prepare test cases
test_cases = [
    {
        "input": "What is the capital of France?",
        "expected": "Paris",
        "context": "France is a country in Europe. Paris is its capital."
    },
    # ... more test cases
]

# Run evaluation
results = suite.evaluate(
    model=your_model,
    test_cases=test_cases
)

print(f"Overall Accuracy: {results.metrics['accuracy']}")
print(f"BLEU Score: {results.metrics['bleu']}")
```

## 自動化指標實作

### BLEU 分數
```python
from nltk.translate.bleu_score import sentence_bleu, SmoothingFunction

def calculate_bleu(reference, hypothesis):
    """Calculate BLEU score between reference and hypothesis."""
    smoothie = SmoothingFunction().method4

    return sentence_bleu(
        [reference.split()],
        hypothesis.split(),
        smoothing_function=smoothie
    )

# Usage
bleu = calculate_bleu(
    reference="The cat sat on the mat",
    hypothesis="A cat is sitting on the mat"
)
```

### ROUGE 分數
```python
from rouge_score import rouge_scorer

def calculate_rouge(reference, hypothesis):
    """Calculate ROUGE scores."""
    scorer = rouge_scorer.RougeScorer(['rouge1', 'rouge2', 'rougeL'], use_stemmer=True)
    scores = scorer.score(reference, hypothesis)

    return {
        'rouge1': scores['rouge1'].fmeasure,
        'rouge2': scores['rouge2'].fmeasure,
        'rougeL': scores['rougeL'].fmeasure
    }
```

### BERTScore
```python
from bert_score import score

def calculate_bertscore(references, hypotheses):
    """Calculate BERTScore using pre-trained BERT."""
    P, R, F1 = score(
        hypotheses,
        references,
        lang='en',
        model_type='microsoft/deberta-xlarge-mnli'
    )

    return {
        'precision': P.mean().item(),
        'recall': R.mean().item(),
        'f1': F1.mean().item()
    }
```

### 自訂指標
```python
def calculate_groundedness(response, context):
    """Check if response is grounded in provided context."""
    # Use NLI model to check entailment
    from transformers import pipeline

    nli = pipeline("text-classification", model="microsoft/deberta-large-mnli")

    result = nli(f"{context} [SEP] {response}")[0]

    # Return confidence that response is entailed by context
    return result['score'] if result['label'] == 'ENTAILMENT' else 0.0

def calculate_toxicity(text):
    """Measure toxicity in generated text."""
    from detoxify import Detoxify

    results = Detoxify('original').predict(text)
    return max(results.values())  # Return highest toxicity score

def calculate_factuality(claim, knowledge_base):
    """Verify factual claims against knowledge base."""
    # Implementation depends on your knowledge base
    # Could use retrieval + NLI, or fact-checking API
    pass
```

## LLM 作為評審模式

### 單一輸出評估
```python
def llm_judge_quality(response, question):
    """Use GPT-5 to judge response quality."""
    prompt = f"""請針對以下回應評分 1-10 分：
1. 準確性（事實正確）
2. 實用性（回答問題）
3. 清晰度（寫得好且易於理解）

問題：{question}
回應：{response}

請以 JSON 格式提供評分：
{{
  "accuracy": <1-10>,
  "helpfulness": <1-10>,
  "clarity": <1-10>,
  "reasoning": "<簡短說明>"
}}
"""

    result = openai.ChatCompletion.create(
        model="gpt-5",
        messages=[{"role": "user", "content": prompt}],
        temperature=0
    )

    return json.loads(result.choices[0].message.content)
```

### 成對比較
```python
def compare_responses(question, response_a, response_b):
    """Compare two responses using LLM judge."""
    prompt = f"""請比較這兩個對問題的回應，並判斷哪個更好。

問題：{question}

回應 A：{response_a}

回應 B：{response_b}

哪個回應更好，為什麼？請考慮準確性、實用性和清晰度。

請以 JSON 格式回答：
{{
  "winner": "A" 或 "B" 或 "tie",
  "reasoning": "<說明>",
  "confidence": <1-10>
}}
"""

    result = openai.ChatCompletion.create(
        model="gpt-5",
        messages=[{"role": "user", "content": prompt}],
        temperature=0
    )

    return json.loads(result.choices[0].message.content)
```

## 人工評估框架

### 標註指南
```python
class AnnotationTask:
    """Structure for human annotation task."""

    def __init__(self, response, question, context=None):
        self.response = response
        self.question = question
        self.context = context

    def get_annotation_form(self):
        return {
            "question": self.question,
            "context": self.context,
            "response": self.response,
            "ratings": {
                "accuracy": {
                    "scale": "1-5",
                    "description": "回應是否在事實上正確？"
                },
                "relevance": {
                    "scale": "1-5",
                    "description": "是否回答問題？"
                },
                "coherence": {
                    "scale": "1-5",
                    "description": "是否邏輯一致？"
                }
            },
            "issues": {
                "factual_error": False,
                "hallucination": False,
                "off_topic": False,
                "unsafe_content": False
            },
            "feedback": ""
        }
```

### 評審者間一致性
```python
from sklearn.metrics import cohen_kappa_score

def calculate_agreement(rater1_scores, rater2_scores):
    """Calculate inter-rater agreement."""
    kappa = cohen_kappa_score(rater1_scores, rater2_scores)

    interpretation = {
        kappa < 0: "差",
        kappa < 0.2: "輕微",
        kappa < 0.4: "尚可",
        kappa < 0.6: "中等",
        kappa < 0.8: "相當",
        kappa <= 1.0: "幾乎完美"
    }

    return {
        "kappa": kappa,
        "interpretation": interpretation[True]
    }
```

## A/B 測試

### 統計測試框架
```python
from scipy import stats
import numpy as np

class ABTest:
    def __init__(self, variant_a_name="A", variant_b_name="B"):
        self.variant_a = {"name": variant_a_name, "scores": []}
        self.variant_b = {"name": variant_b_name, "scores": []}

    def add_result(self, variant, score):
        """Add evaluation result for a variant."""
        if variant == "A":
            self.variant_a["scores"].append(score)
        else:
            self.variant_b["scores"].append(score)

    def analyze(self, alpha=0.05):
        """Perform statistical analysis."""
        a_scores = self.variant_a["scores"]
        b_scores = self.variant_b["scores"]

        # T-test
        t_stat, p_value = stats.ttest_ind(a_scores, b_scores)

        # Effect size (Cohen's d)
        pooled_std = np.sqrt((np.std(a_scores)**2 + np.std(b_scores)**2) / 2)
        cohens_d = (np.mean(b_scores) - np.mean(a_scores)) / pooled_std

        return {
            "variant_a_mean": np.mean(a_scores),
            "variant_b_mean": np.mean(b_scores),
            "difference": np.mean(b_scores) - np.mean(a_scores),
            "relative_improvement": (np.mean(b_scores) - np.mean(a_scores)) / np.mean(a_scores),
            "p_value": p_value,
            "statistically_significant": p_value < alpha,
            "cohens_d": cohens_d,
            "effect_size": self.interpret_cohens_d(cohens_d),
            "winner": "B" if np.mean(b_scores) > np.mean(a_scores) else "A"
        }

    @staticmethod
    def interpret_cohens_d(d):
        """Interpret Cohen's d effect size."""
        abs_d = abs(d)
        if abs_d < 0.2:
            return "negligible"
        elif abs_d < 0.5:
            return "small"
        elif abs_d < 0.8:
            return "medium"
        else:
            return "large"
```

## 回歸測試

### 回歸偵測
```python
class RegressionDetector:
    def __init__(self, baseline_results, threshold=0.05):
        self.baseline = baseline_results
        self.threshold = threshold

    def check_for_regression(self, new_results):
        """Detect if new results show regression."""
        regressions = []

        for metric in self.baseline.keys():
            baseline_score = self.baseline[metric]
            new_score = new_results.get(metric)

            if new_score is None:
                continue

            # Calculate relative change
            relative_change = (new_score - baseline_score) / baseline_score

            # Flag if significant decrease
            if relative_change < -self.threshold:
                regressions.append({
                    "metric": metric,
                    "baseline": baseline_score,
                    "current": new_score,
                    "change": relative_change
                })

        return {
            "has_regression": len(regressions) > 0,
            "regressions": regressions
        }
```

## 基準測試

### 執行基準測試
```python
class BenchmarkRunner:
    def __init__(self, benchmark_dataset):
        self.dataset = benchmark_dataset

    def run_benchmark(self, model, metrics):
        """Run model on benchmark and calculate metrics."""
        results = {metric.name: [] for metric in metrics}

        for example in self.dataset:
            # Generate prediction
            prediction = model.predict(example["input"])

            # Calculate each metric
            for metric in metrics:
                score = metric.calculate(
                    prediction=prediction,
                    reference=example["reference"],
                    context=example.get("context")
                )
                results[metric.name].append(score)

        # Aggregate results
        return {
            metric: {
                "mean": np.mean(scores),
                "std": np.std(scores),
                "min": min(scores),
                "max": max(scores)
            }
            for metric, scores in results.items()
        }
```

## 資源

- **references/metrics.md**：全面的指標指南
- **references/human-evaluation.md**：標註最佳實務
- **references/benchmarking.md**：標準基準測試
- **references/a-b-testing.md**：統計測試指南
- **references/regression-testing.md**：CI/CD 整合
- **assets/evaluation-framework.py**：完整的評估框架
- **assets/benchmark-dataset.jsonl**：範例資料集
- **scripts/evaluate-model.py**：自動化評估執行器

## 最佳實務

1. **多重指標**：使用多樣化的指標以獲得全面的視角
2. **代表性資料**：在真實世界、多樣化的範例上測試
3. **基準**：始終與基準效能進行比較
4. **統計嚴謹性**：使用適當的統計測試進行比較
5. **持續評估**：整合到 CI/CD 流程中
6. **人工驗證**：結合自動化指標與人工判斷
7. **錯誤分析**：調查失敗以了解弱點
8. **版本控制**：隨時間追蹤評估結果

## 常見陷阱

- **單一指標執著**：以犧牲其他指標為代價來最佳化一個指標
- **樣本數過小**：從太少範例中得出結論
- **資料污染**：在訓練資料上測試
- **忽略變異**：未考慮統計不確定性
- **指標不匹配**：使用與業務目標不一致的指標
