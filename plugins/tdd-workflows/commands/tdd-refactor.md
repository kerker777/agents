運用全面的測試安全網，自信地重構程式碼：

[Extended thinking: 此工具使用 tdd-orchestrator agent（opus model）進行精密的重構，同時保持所有測試通過。它應用設計模式、改善程式碼品質，並在全面測試覆蓋的安全保障下優化效能。]

## 使用方式

使用 Task 工具並設定 subagent_type="tdd-orchestrator" 來執行安全的重構。

提示詞：「重構此程式碼並保持所有測試通過：$ARGUMENTS。應用 TDD 重構階段：

## 核心流程

**1. 預先評估**
- 執行測試以建立綠燈基準線
- 分析程式碼異味（code smells）和測試覆蓋率
- 記錄當前效能指標
- 建立漸進式重構計畫

**2. 程式碼異味偵測**
- 重複程式碼 → 提取方法/類別
- 過長方法 → 拆解為專注的函式
- 大型類別 → 分割職責
- 過長參數列表 → 參數物件
- Feature Envy（依戀情結）→ 將方法移至適當的類別
- Primitive Obsession（基本型別偏執）→ 值物件
- Switch 陳述式 → 多型
- 死程式碼 → 移除

**3. 設計模式**
- 應用創建型模式（Factory、Builder、Singleton）
- 應用結構型模式（Adapter、Facade、Decorator）
- 應用行為型模式（Strategy、Observer、Command）
- 應用領域模式（Repository、Service、Value Objects）
- 僅在模式能帶來明確價值時才使用

**4. SOLID 原則**
- Single Responsibility（單一職責）：一個改變的理由
- Open/Closed（開放封閉）：對擴充開放，對修改封閉
- Liskov Substitution（里氏替換）：子型別可替換
- Interface Segregation（介面隔離）：小型、專注的介面
- Dependency Inversion（依賴反轉）：依賴於抽象

**5. 重構技術**
- Extract Method/Variable/Interface（提取方法/變數/介面）
- Inline（內聯）不必要的間接層
- Rename（重新命名）以提升清晰度
- Move Method/Field（移動方法/欄位）至適當的類別
- Replace Magic Numbers（替換魔術數字）為常數
- Encapsulate（封裝）欄位
- Replace Conditional with Polymorphism（以多型替換條件式）
- Introduce Null Object（引入空物件）

**6. 效能優化**
- 進行效能分析以識別瓶頸
- 優化演算法和資料結構
- 在有益的情況下實作快取
- 減少資料庫查詢（消除 N+1 問題）
- 延遲載入和分頁
- 始終在優化前後進行測量

**7. 漸進式步驟**
- 進行小型、原子性的變更
- 每次修改後執行測試
- 每次成功重構後提交
- 將重構與行為變更分開
- 需要時使用鷹架（scaffolding）

**8. 架構演進**
- 分層隔離和依賴管理
- 模組邊界和介面定義
- 事件驅動模式以解耦
- 資料庫存取模式優化

**9. 安全驗證**
- 每次變更後執行完整測試套件
- 效能回歸測試
- 變異測試（mutation testing）以評估測試有效性
- 重大變更的回滾計畫

**10. 進階模式**
- Strangler Fig：漸進式替換遺留系統
- Branch by Abstraction：大規模變更
- Parallel Change：擴展-收縮模式
- Mikado Method：依賴圖導航

## 輸出需求

- 應用改善後的重構程式碼
- 測試結果（全部通過）
- 前後指標比較
- 應用的重構技術清單
- 效能改善測量結果
- 剩餘技術債評估

## 安全檢查清單

提交前：
- ✓ 所有測試通過（100% 綠燈）
- ✓ 無功能性退化
- ✓ 效能指標可接受
- ✓ 程式碼覆蓋率維持/改善
- ✓ 文件已更新

## 復原協議

如果測試失敗：
- 立即還原上次變更
- 識別造成中斷的重構
- 應用更小的漸進式變更
- 使用版本控制進行安全實驗

## 範例：Extract Method Pattern

**重構前：**
```typescript
class OrderProcessor {
  processOrder(order: Order): ProcessResult {
    // Validation
    if (!order.customerId || order.items.length === 0) {
      return { success: false, error: "Invalid order" };
    }

    // Calculate totals
    let subtotal = 0;
    for (const item of order.items) {
      subtotal += item.price * item.quantity;
    }
    let total = subtotal + (subtotal * 0.08) + (subtotal > 100 ? 0 : 15);

    // Process payment...
    // Update inventory...
    // Send confirmation...
  }
}
```

**重構後：**
```typescript
class OrderProcessor {
  async processOrder(order: Order): Promise<ProcessResult> {
    const validation = this.validateOrder(order);
    if (!validation.isValid) return ProcessResult.failure(validation.error);

    const orderTotal = OrderTotal.calculate(order);
    const inventoryCheck = await this.inventoryService.checkAvailability(order.items);
    if (!inventoryCheck.available) return ProcessResult.failure(inventoryCheck.reason);

    await this.paymentService.processPayment(order.paymentMethod, orderTotal.total);
    await this.inventoryService.reserveItems(order.items);
    await this.notificationService.sendOrderConfirmation(order, orderTotal);

    return ProcessResult.success(order.id, orderTotal.total);
  }

  private validateOrder(order: Order): ValidationResult {
    if (!order.customerId) return ValidationResult.invalid("Customer ID required");
    if (order.items.length === 0) return ValidationResult.invalid("Order must contain items");
    return ValidationResult.valid();
  }
}
```

**應用技術：** Extract Method、Value Objects、Dependency Injection、Async patterns

要重構的程式碼：$ARGUMENTS"
