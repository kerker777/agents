# 思維鏈提示

## 概述

思維鏈 (Chain-of-Thought, CoT) 提示可以引導 LLM 進行逐步推理，在複雜推理、數學和邏輯任務上顯著提升效能。

## 核心技術

### 零樣本 CoT
加入簡單的觸發片語來引導推理：

```python
def zero_shot_cot(query):
    return f"""{query}

讓我們一步步思考："""

# Example
query = "如果一列火車以每小時 60 英里的速度行駛 2.5 小時，它會走多遠？"
prompt = zero_shot_cot(query)

# Model output:
# "讓我們一步步思考：
# 1. 速度 = 每小時 60 英里
# 2. 時間 = 2.5 小時
# 3. 距離 = 速度 × 時間
# 4. 距離 = 60 × 2.5 = 150 英里
# 答案：150 英里"
```

### 少樣本 CoT
提供帶有明確推理鏈的範例：

```python
few_shot_examples = """
問：Roger 有 5 個網球。他又買了 2 罐網球。每罐有 3 個球。他現在有多少個網球？
答：讓我們一步步思考：
1. Roger 一開始有 5 個球
2. 他買了 2 罐，每罐有 3 個球
3. 罐子裡的球：2 × 3 = 6 個球
4. 總計：5 + 6 = 11 個球
答案：11

問：自助餐廳有 23 個蘋果。如果他們用了 20 個做午餐，又買了 6 個，他們現在有多少個？
答：讓我們一步步思考：
1. 一開始有 23 個蘋果
2. 用了 20 個做午餐：23 - 20 = 3 個蘋果剩下
3. 又買了 6 個：3 + 6 = 9 個蘋果
答案：9

問：{user_query}
答：讓我們一步步思考："""
```

### 自我一致性
生成多個推理路徑並採用多數投票：

```python
import openai
from collections import Counter

def self_consistency_cot(query, n=5, temperature=0.7):
    prompt = f"{query}\n\n讓我們一步步思考："

    responses = []
    for _ in range(n):
        response = openai.ChatCompletion.create(
            model="gpt-5",
            messages=[{"role": "user", "content": prompt}],
            temperature=temperature
        )
        responses.append(extract_final_answer(response))

    # Take majority vote
    answer_counts = Counter(responses)
    final_answer = answer_counts.most_common(1)[0][0]

    return {
        'answer': final_answer,
        'confidence': answer_counts[final_answer] / n,
        'all_responses': responses
    }
```

## 進階模式

### 由簡到繁提示
將複雜問題分解為更簡單的子問題：

```python
def least_to_most_prompt(complex_query):
    # Stage 1: Decomposition
    decomp_prompt = f"""將這個複雜問題分解為更簡單的子問題：

問題：{complex_query}

子問題："""

    subproblems = get_llm_response(decomp_prompt)

    # Stage 2: Sequential solving
    solutions = []
    context = ""

    for subproblem in subproblems:
        solve_prompt = f"""{context}

解決這個子問題：
{subproblem}

解答："""
        solution = get_llm_response(solve_prompt)
        solutions.append(solution)
        context += f"\n\n先前已解決：{subproblem}\n解答：{solution}"

    # Stage 3: Final integration
    final_prompt = f"""給定這些子問題的解答：
{context}

提供以下問題的最終答案：{complex_query}

最終答案："""

    return get_llm_response(final_prompt)
```

### 思維樹 (Tree-of-Thought, ToT)
探索多個推理分支：

```python
class TreeOfThought:
    def __init__(self, llm_client, max_depth=3, branches_per_step=3):
        self.client = llm_client
        self.max_depth = max_depth
        self.branches_per_step = branches_per_step

    def solve(self, problem):
        # Generate initial thought branches
        initial_thoughts = self.generate_thoughts(problem, depth=0)

        # Evaluate each branch
        best_path = None
        best_score = -1

        for thought in initial_thoughts:
            path, score = self.explore_branch(problem, thought, depth=1)
            if score > best_score:
                best_score = score
                best_path = path

        return best_path

    def generate_thoughts(self, problem, context="", depth=0):
        prompt = f"""問題：{problem}
{context}

生成 {self.branches_per_step} 個不同的下一步來解決這個問題：

1."""
        response = self.client.complete(prompt)
        return self.parse_thoughts(response)

    def evaluate_thought(self, problem, thought_path):
        prompt = f"""問題：{problem}

目前的推理路徑：
{thought_path}

從 0-10 評分這個推理路徑的：
- 正確性
- 達成解答的可能性
- 邏輯連貫性

分數："""
        return float(self.client.complete(prompt))
```

### 驗證步驟
加入明確的驗證以捕捉錯誤：

```python
def cot_with_verification(query):
    # Step 1: Generate reasoning and answer
    reasoning_prompt = f"""{query}

讓我們一步步解決："""

    reasoning_response = get_llm_response(reasoning_prompt)

    # Step 2: Verify the reasoning
    verification_prompt = f"""原始問題：{query}

提議的解答：
{reasoning_response}

透過以下方式驗證此解答：
1. 檢查每個步驟是否有邏輯錯誤
2. 驗證算術計算
3. 確保最終答案合理

此解答是否正確？如果不正確，有什麼問題？

驗證："""

    verification = get_llm_response(verification_prompt)

    # Step 3: Revise if needed
    if "不正確" in verification or "錯誤" in verification:
        revision_prompt = f"""先前的解答有錯誤：
{verification}

請提供正確的解答：{query}

修正的解答："""
        return get_llm_response(revision_prompt)

    return reasoning_response
```

## 領域特定 CoT

### 數學問題
```python
math_cot_template = """
問題：{problem}

解答：
步驟 1：識別我們知道的
- {list_known_values}

步驟 2：識別我們需要找到的
- {target_variable}

步驟 3：選擇相關公式
- {formulas}

步驟 4：代入數值
- {substitution}

步驟 5：計算
- {calculation}

步驟 6：驗證並陳述答案
- {verification}

答案：{final_answer}
"""
```

### 程式碼除錯
```python
debug_cot_template = """
有錯誤的程式碼：
{code}

錯誤訊息：
{error}

除錯過程：
步驟 1：理解錯誤訊息
- {interpret_error}

步驟 2：定位有問題的行
- {identify_line}

步驟 3：分析為什麼這行失敗
- {root_cause}

步驟 4：決定修正方法
- {proposed_fix}

步驟 5：驗證修正是否解決錯誤
- {verification}

修正後的程式碼：
{corrected_code}
"""
```

### 邏輯推理
```python
logic_cot_template = """
前提：
{premises}

問題：{question}

推理：
步驟 1：列出所有給定的事實
{facts}

步驟 2：識別邏輯關係
{relationships}

步驟 3：應用演繹推理
{deductions}

步驟 4：得出結論
{conclusion}

答案：{final_answer}
"""
```

## 效能最佳化

### 快取推理模式
```python
class ReasoningCache:
    def __init__(self):
        self.cache = {}

    def get_similar_reasoning(self, problem, threshold=0.85):
        problem_embedding = embed(problem)

        for cached_problem, reasoning in self.cache.items():
            similarity = cosine_similarity(
                problem_embedding,
                embed(cached_problem)
            )
            if similarity > threshold:
                return reasoning

        return None

    def add_reasoning(self, problem, reasoning):
        self.cache[problem] = reasoning
```

### 自適應推理深度
```python
def adaptive_cot(problem, initial_depth=3):
    depth = initial_depth

    while depth <= 10:  # Max depth
        response = generate_cot(problem, num_steps=depth)

        # Check if solution seems complete
        if is_solution_complete(response):
            return response

        depth += 2  # Increase reasoning depth

    return response  # Return best attempt
```

## 評估指標

```python
def evaluate_cot_quality(reasoning_chain):
    metrics = {
        'coherence': measure_logical_coherence(reasoning_chain),
        'completeness': check_all_steps_present(reasoning_chain),
        'correctness': verify_final_answer(reasoning_chain),
        'efficiency': count_unnecessary_steps(reasoning_chain),
        'clarity': rate_explanation_clarity(reasoning_chain)
    }
    return metrics
```

## 最佳實務

1. **清晰的步驟標記**：使用編號步驟或清晰的分隔符
2. **展示所有工作**：不要跳過步驟，即使是明顯的步驟
3. **驗證計算**：加入明確的驗證步驟
4. **陳述假設**：將隱含假設明確化
5. **檢查邊緣案例**：考慮邊界條件
6. **使用範例**：先用範例展示推理模式

## 常見陷阱

- **過早結論**：未完整推理就跳到答案
- **循環邏輯**：使用結論來證明推理
- **遺漏步驟**：跳過中間計算
- **過度複雜化**：加入不必要的步驟造成混淆
- **格式不一致**：推理過程中改變步驟結構

## 何時使用 CoT

**使用 CoT 於：**
- 數學和算術問題
- 邏輯推理任務
- 多步驟規劃
- 程式碼生成和除錯
- 複雜決策制定

**略過 CoT 於：**
- 簡單的事實查詢
- 直接查找
- 創意寫作
- 需要簡潔的任務
- 即時、延遲敏感的應用

## 資源

- 用於 CoT 評估的基準資料集
- 預建的 CoT 提示詞模板
- 推理驗證工具
- 步驟擷取和解析工具
