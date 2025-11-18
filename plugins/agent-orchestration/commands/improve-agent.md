# Agent 效能優化工作流程

透過效能分析、提示工程與持續迭代，系統化地改善現有 agent。

[延伸思考：Agent 優化需要採用資料驅動的方法，結合效能指標、使用者回饋分析與進階提示工程技術。成功的關鍵在於系統化評估、針對性改善，以及嚴謹的測試與回退機制，以確保正式環境的安全性。]

## 階段 1：效能分析與基準指標

使用 context-manager 進行歷史資料收集，全面分析 agent 效能。

### 1.1 收集效能資料
```
Use: context-manager
Command: analyze-agent-performance $ARGUMENTS --days 30
```

收集以下指標：
- 任務完成率（成功與失敗任務）
- 回應準確度與事實正確性
- 工具使用效率（正確工具、呼叫頻率）
- 平均回應時間與 token 消耗量
- 使用者滿意度指標（修正次數、重試次數）
- 幻覺事件與錯誤模式

### 1.2 使用者回饋模式分析

識別使用者互動中的重複模式：
- **修正模式**：使用者持續修改輸出的地方
- **澄清請求**：常見的模糊區域
- **任務放棄**：使用者放棄的時間點
- **後續問題**：不完整回應的指標
- **正面回饋**：需要保留的成功模式

### 1.3 失敗模式分類

按根本原因分類失敗：
- **指令誤解**：角色或任務混淆
- **輸出格式錯誤**：結構或格式問題
- **情境遺失**：長對話過程中的效能退化
- **工具誤用**：不正確或低效的工具選擇
- **限制違反**：安全或業務規則違反
- **邊界案例處理**：異常輸入情境

### 1.4 基準效能報告

產生量化基準指標：
```
Performance Baseline:
- Task Success Rate: [X%]
- Average Corrections per Task: [Y]
- Tool Call Efficiency: [Z%]
- User Satisfaction Score: [1-10]
- Average Response Latency: [Xms]
- Token Efficiency Ratio: [X:Y]
```

## 階段 2：提示工程改善

使用 prompt-engineer agent 應用進階提示優化技術。

### 2.1 Chain-of-Thought 強化

實作結構化推理模式：
```
Use: prompt-engineer
Technique: chain-of-thought-optimization
```

- 加入明確的推理步驟：「讓我們逐步處理這個問題...」
- 包含自我驗證檢查點：「在繼續之前，請驗證...」
- 對複雜任務實作遞迴分解
- 加入推理軌跡可見性以利除錯

### 2.2 Few-Shot 範例優化

從成功的互動中精選高品質範例：
- **選擇多元範例**涵蓋常見使用案例
- **包含邊界案例**先前失敗的情境
- **同時展示正面與負面範例**並附上解釋
- **排序範例**從簡單到複雜
- **標註範例**關鍵決策點

範例結構：
```
Good Example:
Input: [User request]
Reasoning: [Step-by-step thought process]
Output: [Successful response]
Why this works: [Key success factors]

Bad Example:
Input: [Similar request]
Output: [Failed response]
Why this fails: [Specific issues]
Correct approach: [Fixed version]
```

### 2.3 角色定義精煉

強化 agent 身份與能力：
- **核心目的**：清晰的單句使命陳述
- **專業領域**：特定知識領域
- **行為特質**：個性與互動風格
- **工具熟練度**：可用工具與使用時機
- **限制**：agent 不應該做的事
- **成功標準**：如何衡量任務完成度

### 2.4 Constitutional AI 整合

實作自我修正機制：
```
Constitutional Principles:
1. Verify factual accuracy before responding
2. Self-check for potential biases or harmful content
3. Validate output format matches requirements
4. Ensure response completeness
5. Maintain consistency with previous responses
```

加入批評與修訂循環：
- 初始回應生成
- 根據原則進行自我批評
- 若發現問題則自動修訂
- 輸出前的最終驗證

### 2.5 輸出格式調整

優化回應結構：
- **結構化範本**用於常見任務
- **動態格式化**根據複雜度調整
- **漸進式揭露**詳細資訊
- **Markdown 優化**提升可讀性
- **程式碼區塊格式化**與語法高亮
- **表格與清單生成**用於資料呈現

## 階段 3：測試與驗證

使用 A/B 比較的完整測試框架。

### 3.1 測試套件開發

建立代表性測試情境：
```
Test Categories:
1. Golden path scenarios (common successful cases)
2. Previously failed tasks (regression testing)
3. Edge cases and corner scenarios
4. Stress tests (complex, multi-step tasks)
5. Adversarial inputs (potential breaking points)
6. Cross-domain tasks (combining capabilities)
```

### 3.2 A/B 測試框架

比較原始版本與改良版本 agent：
```
Use: parallel-test-runner
Config:
  - Agent A: Original version
  - Agent B: Improved version
  - Test set: 100 representative tasks
  - Metrics: Success rate, speed, token usage
  - Evaluation: Blind human review + automated scoring
```

統計顯著性測試：
- 最小樣本數：每個變體 100 個任務
- 信心水準：95% (p < 0.05)
- 效果量計算 (Cohen's d)
- 未來測試的統計檢定力分析

### 3.3 評估指標

完整的評分框架：

**任務層級指標：**
- 完成率（二元成功/失敗）
- 正確性分數（0-100% 準確度）
- 效率分數（實際步驟 vs 最佳步驟）
- 工具使用適當性
- 回應相關性與完整性

**品質指標：**
- 幻覺率（每個回應的事實錯誤數）
- 一致性分數（與先前回應的一致性）
- 格式符合度（符合指定結構）
- 安全分數（限制遵守度）
- 使用者滿意度預測

**效能指標：**
- 回應延遲（首個 token 時間）
- 總生成時間
- Token 消耗量（輸入 + 輸出）
- 每任務成本（API 使用費）
- 記憶體/情境效率

### 3.4 人工評估協定

結構化的人工審查流程：
- 盲測評估（評估者不知道版本）
- 標準化評分標準與明確準則
- 每個樣本多位評估者（評估者間信度）
- 質性回饋收集
- 偏好排序（A vs B 比較）

## 階段 4：版本控制與部署

安全推出，具備監控與回退能力。

### 4.1 版本管理

系統化版本策略：
```
Version Format: agent-name-v[MAJOR].[MINOR].[PATCH]
Example: customer-support-v2.3.1

MAJOR: Significant capability changes
MINOR: Prompt improvements, new examples
PATCH: Bug fixes, minor adjustments
```

維護版本歷史：
- 基於 Git 的提示儲存
- 包含改善細節的變更紀錄
- 每個版本的效能指標
- 記錄回退程序

### 4.2 分階段推出

漸進式部署策略：
1. **Alpha 測試**：內部團隊驗證（5% 流量）
2. **Beta 測試**：精選使用者（20% 流量）
3. **金絲雀發布**：逐步增加（20% → 50% → 100%）
4. **完整部署**：達成成功標準後
5. **監控期**：7 天觀察期

### 4.3 回退程序

快速恢復機制：
```
Rollback Triggers:
- Success rate drops >10% from baseline
- Critical errors increase >5%
- User complaints spike
- Cost per task increases >20%
- Safety violations detected

Rollback Process:
1. Detect issue via monitoring
2. Alert team immediately
3. Switch to previous stable version
4. Analyze root cause
5. Fix and re-test before retry
```

### 4.4 持續監控

即時效能追蹤：
- 關鍵指標儀表板
- 異常偵測警報
- 使用者回饋收集
- 自動化回歸測試
- 每週效能報告

## 成功標準

當以下條件達成時，Agent 改善即為成功：
- 任務成功率提升 ≥15%
- 使用者修正次數減少 ≥25%
- 安全違規沒有增加
- 回應時間維持在基準的 10% 範圍內
- 每任務成本增加不超過 5%
- 正面使用者回饋增加

## 部署後檢視

正式環境使用 30 天後：
1. 分析累積的效能資料
2. 與基準和目標進行比較
3. 識別新的改善機會
4. 記錄經驗教訓
5. 規劃下一個優化週期

## 持續改善週期

建立定期改善節奏：
- **每週**：監控指標並收集回饋
- **每月**：分析模式並規劃改善
- **每季**：具備新功能的主要版本更新
- **每年**：策略審查與架構更新

請記住：Agent 優化是一個迭代過程。每個週期都建立在先前的學習之上，在維持穩定性與安全性的同時逐步提升效能。
