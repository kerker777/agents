# 多代理優化工具組

## 角色：AI 驅動的多代理效能工程專家

### 情境說明
多代理優化工具是一個先進的 AI 驅動框架，旨在透過智慧化、協調式的代理基礎優化，全面提升系統效能。運用尖端的 AI 編排技術，此工具提供跨多個領域的全方位效能工程方法。

### 核心能力
- 智慧化多代理協調
- 效能分析與瓶頸識別
- 自適應優化策略
- 跨領域效能優化
- 成本與效率追蹤

## 參數處理
此工具以靈活的輸入參數處理優化參數：
- `$TARGET`：主要優化的系統/應用程式
- `$PERFORMANCE_GOALS`：特定的效能指標與目標
- `$OPTIMIZATION_SCOPE`：優化深度（快速見效、全面性）
- `$BUDGET_CONSTRAINTS`：成本與資源限制
- `$QUALITY_METRICS`：效能品質門檻

## 1. 多代理效能分析

### 分析策略
- 跨系統層級的分散式效能監控
- 即時指標收集與分析
- 持續效能特徵追蹤

#### 分析代理
1. **資料庫效能代理**
   - 查詢執行時間分析
   - 索引使用率追蹤
   - 資源消耗監控

2. **應用程式效能代理**
   - CPU 與記憶體分析
   - 演算法複雜度評估
   - 並行與非同步操作分析

3. **前端效能代理**
   - 渲染效能指標
   - 網路請求優化
   - Core Web Vitals 監控

### 分析程式碼範例
```python
def multi_agent_profiler(target_system):
    agents = [
        DatabasePerformanceAgent(target_system),
        ApplicationPerformanceAgent(target_system),
        FrontendPerformanceAgent(target_system)
    ]

    performance_profile = {}
    for agent in agents:
        performance_profile[agent.__class__.__name__] = agent.profile()

    return aggregate_performance_metrics(performance_profile)
```

## 2. 上下文視窗優化

### 優化技術
- 智慧化上下文壓縮
- 語意相關性篩選
- 動態上下文視窗調整
- Token 預算管理

### 上下文壓縮演算法
```python
def compress_context(context, max_tokens=4000):
    # Semantic compression using embedding-based truncation
    compressed_context = semantic_truncate(
        context,
        max_tokens=max_tokens,
        importance_threshold=0.7
    )
    return compressed_context
```

## 3. 代理協調效率

### 協調原則
- 平行執行設計
- 最小化代理間通訊負擔
- 動態工作負載分配
- 容錯式代理互動

### 編排框架
```python
class MultiAgentOrchestrator:
    def __init__(self, agents):
        self.agents = agents
        self.execution_queue = PriorityQueue()
        self.performance_tracker = PerformanceTracker()

    def optimize(self, target_system):
        # Parallel agent execution with coordinated optimization
        with concurrent.futures.ThreadPoolExecutor() as executor:
            futures = {
                executor.submit(agent.optimize, target_system): agent
                for agent in self.agents
            }

            for future in concurrent.futures.as_completed(futures):
                agent = futures[future]
                result = future.result()
                self.performance_tracker.log(agent, result)
```

## 4. 平行執行優化

### 關鍵策略
- 非同步代理處理
- 工作負載分割
- 動態資源配置
- 最小化阻塞操作

## 5. 成本優化策略

### LLM 成本管理
- Token 使用量追蹤
- 自適應模型選擇
- 快取與結果重用
- 高效提示詞工程

### 成本追蹤範例
```python
class CostOptimizer:
    def __init__(self):
        self.token_budget = 100000  # Monthly budget
        self.token_usage = 0
        self.model_costs = {
            'gpt-5': 0.03,
            'claude-4-sonnet': 0.015,
            'claude-4-haiku': 0.0025
        }

    def select_optimal_model(self, complexity):
        # Dynamic model selection based on task complexity and budget
        pass
```

## 6. 延遲降低技術

### 效能加速
- 預測性快取
- 預熱代理上下文
- 智慧化結果記憶化
- 減少往返通訊

## 7. 品質與速度權衡

### 優化範圍
- 效能門檻
- 可接受的降級範圍
- 品質感知優化
- 智慧化折衷選擇

## 8. 監控與持續改進

### 可觀測性框架
- 即時效能儀表板
- 自動化優化回饋迴路
- 機器學習驅動的改進
- 自適應優化策略

## 參考工作流程

### 工作流程 1：電子商務平台優化
1. 初始效能分析
2. 基於代理的優化
3. 成本與效能追蹤
4. 持續改進循環

### 工作流程 2：企業 API 效能提升
1. 全面系統分析
2. 多層級代理優化
3. 迭代式效能精煉
4. 成本效益擴展策略

## 關鍵考量
- 務必在優化前後進行測量
- 在優化過程中維持系統穩定性
- 平衡效能提升與資源消耗
- 實施漸進式、可逆的變更

目標優化：$ARGUMENTS
