---
name: workflow-orchestration-patterns
description: Design durable workflows with Temporal for distributed systems. Covers workflow vs activity separation, saga patterns, state management, and determinism constraints. Use when building long-running processes, distributed transactions, or microservice orchestration.
---

# Workflow Orchestration Patterns

掌握使用 Temporal 的工作流程編排架構，涵蓋基本設計決策、韌性模式，以及建構可靠分散式系統的最佳實務。

## 何時使用工作流程編排

### 理想的使用情境（來源：docs.temporal.io）

- **多步驟流程**：跨越多台機器、服務或資料庫
- **分散式交易**：需要全有或全無語意
- **長時間執行的工作流程**：從數小時到數年，具備自動狀態持久化
- **故障恢復**：必須從最後成功的步驟繼續執行
- **業務流程**：預訂、訂單、活動、審批
- **實體生命週期管理**：庫存追蹤、帳戶管理、購物車工作流程
- **基礎設施自動化**：CI/CD 流水線、資源配置、部署
- **人工介入**系統，需要逾時和升級機制

### 不適用的情境

- 簡單的 CRUD 操作（使用直接的 API 呼叫）
- 純資料處理流水線（使用 Airflow、批次處理）
- 無狀態的請求/回應（使用標準 API）
- 即時串流（使用 Kafka、事件處理器）

## 關鍵設計決策：Workflows vs Activities

**基本原則**（來源：temporal.io/blog/workflow-engine-principles）：
- **Workflows** = 編排邏輯和決策制定
- **Activities** = 外部互動（API、資料庫、網路呼叫）

### Workflows（編排）

**特性：**
- 包含業務邏輯和協調
- **必須具備確定性**（相同輸入 → 相同輸出）
- **不能**執行直接的外部呼叫
- 狀態會在故障期間自動保存
- 即使基礎設施發生故障，也能執行數年

**工作流程任務範例：**
- 決定要執行哪些步驟
- 處理補償邏輯
- 管理逾時和重試
- 協調子工作流程

### Activities（外部互動）

**特性：**
- 處理所有外部系統互動
- 可以是非確定性的（API 呼叫、資料庫寫入）
- 包含內建的逾時和重試邏輯
- **必須具備冪等性**（呼叫 N 次 = 呼叫 1 次）
- 短期執行（通常為數秒到數分鐘）

**活動任務範例：**
- 呼叫支付閘道 API
- 寫入資料庫
- 發送電子郵件或通知
- 查詢外部服務

### 設計決策框架

```
是否涉及外部系統？ → Activity
是否為編排/決策邏輯？ → Workflow
```

## 核心工作流程模式

### 1. Saga Pattern 與補償機制

**目的**：實現具備回滾能力的分散式交易

**模式**（來源：temporal.io/blog/compensating-actions-part-of-a-complete-breakfast-with-sagas）：

```
對於每個步驟：
  1. 在執行之前註冊補償動作
  2. 執行該步驟（透過 activity）
  3. 失敗時，以相反順序執行所有補償動作（LIFO）
```

**範例：支付工作流程**
1. 預留庫存（補償：釋放庫存）
2. 扣款（補償：退款）
3. 履行訂單（補償：取消履行）

**關鍵要求：**
- 補償動作必須具備冪等性
- 在執行步驟之前註冊補償動作
- 以相反順序執行補償動作
- 優雅地處理部分失敗

### 2. Entity Workflows（Actor Model）

**目的**：代表單一實體實例的長時間執行工作流程

**模式**（來源：docs.temporal.io/evaluate/use-cases-design-patterns）：
- 一個工作流程執行 = 一個實體（購物車、帳戶、庫存項目）
- 工作流程在實體生命週期內持續存在
- 接收信號以進行狀態變更
- 支援查詢當前狀態

**使用情境範例：**
- 購物車（新增商品、結帳、過期）
- 銀行帳戶（存款、提款、餘額查詢）
- 產品庫存（庫存更新、預留）

**優勢：**
- 封裝實體行為
- 保證每個實體的一致性
- 自然的事件溯源

### 3. Fan-Out/Fan-In（平行執行）

**目的**：平行執行多個任務，彙總結果

**模式：**
- 產生子工作流程或平行活動
- 等待全部完成
- 彙總結果
- 處理部分失敗

**擴展規則**（來源：temporal.io/blog/workflow-engine-principles）：
- 不要擴展單個工作流程
- 對於 1M 個任務：產生 1K 個子工作流程 × 每個 1K 個任務
- 保持每個工作流程有界

### 4. Async Callback Pattern

**目的**：等待外部事件或人工審批

**模式：**
- 工作流程發送請求並等待信號
- 外部系統非同步處理
- 發送信號以恢復工作流程
- 工作流程繼續處理回應

**使用情境：**
- 人工審批工作流程
- Webhook 回調
- 長時間執行的外部流程

## 狀態管理和確定性

### 自動狀態保存

**Temporal 的運作方式**（來源：docs.temporal.io/workflows）：
- 完整的程式狀態自動保存
- Event History 記錄每個命令和事件
- 從崩潰中無縫恢復
- 應用程式恢復到故障前的狀態

### 確定性限制

**工作流程以狀態機方式執行**：
- 重播行為必須一致
- 相同輸入 → 每次都產生相同輸出

**工作流程中禁止的操作**（來源：docs.temporal.io/workflows）：
- ❌ 執行緒、鎖、同步原語
- ❌ 隨機數生成（`random()`）
- ❌ 全域狀態或靜態變數
- ❌ 系統時間（`datetime.now()`）
- ❌ 直接的檔案 I/O 或網路呼叫
- ❌ 非確定性的函式庫

**工作流程中允許的操作**：
- ✅ `workflow.now()`（確定性時間）
- ✅ `workflow.random()`（確定性隨機）
- ✅ 純函數和計算
- ✅ 呼叫 activities（非確定性操作）

### 版本控制策略

**挑戰**：在舊的執行仍在執行時變更工作流程程式碼

**解決方案**：
1. **版本控制 API**：使用 `workflow.get_version()` 進行安全變更
2. **新工作流程類型**：建立新工作流程，將新執行路由到該工作流程
3. **向後相容**：確保舊事件能正確重播

## 韌性和錯誤處理

### 重試策略

**預設行為**：Temporal 會永久重試 activities

**設定重試**：
- 初始重試間隔
- 退避係數（指數退避）
- 最大間隔（上限重試延遲）
- 最大嘗試次數（最終失敗）

**不可重試的錯誤**：
- 無效輸入（驗證失敗）
- 業務規則違規
- 永久性失敗（找不到資源）

### 冪等性要求

**為何至關重要**（來源：docs.temporal.io/activities）：
- Activities 可能執行多次
- 網路故障會觸發重試
- 重複執行必須是安全的

**實作策略**：
- 冪等性金鑰（去重）
- 使用唯一約束的檢查後執行
- 使用 upsert 操作而非 insert
- 追蹤已處理的請求 ID

### Activity Heartbeats

**目的**：偵測停滯的長時間執行活動

**模式**：
- Activity 定期發送心跳
- 包含進度資訊
- 如果未收到心跳則逾時
- 啟用基於進度的重試

## 最佳實務

### 工作流程設計

1. **保持工作流程專注** - 每個工作流程單一職責
2. **小型工作流程** - 使用子工作流程以提升可擴展性
3. **清晰的邊界** - 工作流程負責編排，activities 負責執行
4. **本地測試** - 使用時間跳躍測試環境

### 活動設計

1. **冪等操作** - 安全可重試
2. **短期執行** - 數秒到數分鐘，而非數小時
3. **逾時設定** - 始終設定逾時
4. **長任務的心跳** - 報告進度
5. **錯誤處理** - 區分可重試與不可重試

### 常見陷阱

**工作流程違規**：
- 使用 `datetime.now()` 而非 `workflow.now()`
- 在工作流程程式碼中使用執行緒或非同步操作
- 從工作流程直接呼叫外部 API
- 工作流程中的非確定性邏輯

**活動錯誤**：
- 非冪等操作（無法處理重試）
- 缺少逾時（活動永遠執行）
- 沒有錯誤分類（重試驗證錯誤）
- 忽略承載限制（每個參數 2MB）

### 營運考量

**監控**：
- 工作流程執行時間
- 活動失敗率
- 重試嘗試和退避
- 待處理工作流程數量

**可擴展性**：
- 使用 workers 進行水平擴展
- 任務佇列分區
- 子工作流程分解
- 適當時進行活動批次處理

## 額外資源

**官方文件**：
- Temporal Core Concepts: docs.temporal.io/workflows
- Workflow Patterns: docs.temporal.io/evaluate/use-cases-design-patterns
- Best Practices: docs.temporal.io/develop/best-practices
- Saga Pattern: temporal.io/blog/saga-pattern-made-easy

**關鍵原則**：
1. Workflows = 編排，Activities = 外部呼叫
2. 工作流程的確定性是不可妥協的
3. 活動的冪等性至關重要
4. 狀態保存是自動的
5. 為故障和恢復而設計
