---
name: debugging-strategies
description: 精通系統化除錯技術、效能分析工具和根本原因分析，以高效追蹤任何程式碼庫或技術堆疊中的錯誤。適用於調查錯誤、效能問題或非預期行為。
---

# 除錯策略

透過經過驗證的策略、強大的工具和有條理的方法，將除錯從令人沮喪的猜測轉變為系統化的問題解決。

## 何時使用此技能

- 追蹤難以捉摸的錯誤
- 調查效能問題
- 理解不熟悉的程式碼庫
- 除錯生產環境問題
- 分析崩潰轉儲和堆疊追蹤
- 分析應用程式效能
- 調查記憶體洩漏
- 除錯分散式系統

## 核心原則

### 1. 科學方法

**1. 觀察**：實際行為是什麼？
**2. 假設**：可能是什麼原因？
**3. 實驗**：測試你的假設
**4. 分析**：證實或推翻你的理論了嗎？
**5. 重複**：直到找到根本原因

### 2. 除錯心態

**不要假設：**
- "不可能是 X" - 可以
- "我沒改 Y" - 還是檢查一下
- "在我的機器上可以運作" - 找出為什麼

**要做：**
- 一致地重現
- 隔離問題
- 保持詳細記錄
- 質疑一切
- 卡住時休息一下

### 3. 橡皮鴨除錯

大聲解釋你的程式碼和問題（對橡皮鴨、同事或自己）。通常會揭示問題。

## 系統化除錯流程

### 階段 1：重現

```markdown
## 重現檢查清單

1. **你能重現嗎？**
   - 總是？有時候？隨機？
   - 需要特定條件嗎？
   - 其他人能重現嗎？

2. **建立最小重現**
   - 簡化成最小範例
   - 移除無關程式碼
   - 隔離問題

3. **記錄步驟**
   - 寫下確切步驟
   - 記錄環境細節
   - 捕捉錯誤訊息
```

### 階段 2：收集資訊

```markdown
## 資訊收集

1. **錯誤訊息**
   - 完整堆疊追蹤
   - 錯誤代碼
   - 主控台/日誌輸出

2. **環境**
   - 作業系統版本
   - 語言/執行時版本
   - 相依套件版本
   - 環境變數

3. **最近變更**
   - Git 歷史記錄
   - 部署時間軸
   - 設定變更

4. **範圍**
   - 影響所有使用者還是特定使用者？
   - 所有瀏覽器還是特定瀏覽器？
   - 僅生產環境還是開發環境也有？
```

### 階段 3：形成假設

```markdown
## 假設形成

根據收集的資訊，問：

1. **什麼改變了？**
   - 最近的程式碼變更
   - 相依套件更新
   - 基礎設施變更

2. **什麼不同？**
   - 正常 vs 損壞的環境
   - 正常 vs 損壞的使用者
   - 之前 vs 之後

3. **哪裡可能失敗？**
   - 輸入驗證
   - 業務邏輯
   - 資料層
   - 外部服務
```

### 階段 4：測試與驗證

```markdown
## 測試策略

1. **二分搜尋**
   - 註解掉一半程式碼
   - 縮小有問題的部分
   - 重複直到找到

2. **加入日誌**
   - 策略性的 console.log/print
   - 追蹤變數值
   - 追蹤執行流程

3. **隔離元件**
   - 分別測試每個部分
   - Mock 相依項
   - 移除複雜性

4. **比較正常與損壞**
   - 比對設定
   - 比對環境
   - 比對資料
```

## 除錯工具

### JavaScript/TypeScript 除錯

```typescript
// Chrome DevTools Debugger
function processOrder(order: Order) {
    debugger;  // Execution pauses here

    const total = calculateTotal(order);
    console.log('Total:', total);

    // Conditional breakpoint
    if (order.items.length > 10) {
        debugger;  // Only breaks if condition true
    }

    return total;
}

// Console debugging techniques
console.log('Value:', value);                    // Basic
console.table(arrayOfObjects);                   // Table format
console.time('operation'); /* code */ console.timeEnd('operation');  // Timing
console.trace();                                 // Stack trace
console.assert(value > 0, 'Value must be positive');  // Assertion

// Performance profiling
performance.mark('start-operation');
// ... operation code
performance.mark('end-operation');
performance.measure('operation', 'start-operation', 'end-operation');
console.log(performance.getEntriesByType('measure'));
```

**VS Code 除錯器設定：**
```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Debug Program",
            "program": "${workspaceFolder}/src/index.ts",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outFiles": ["${workspaceFolder}/dist/**/*.js"],
            "skipFiles": ["<node_internals>/**"]
        },
        {
            "type": "node",
            "request": "launch",
            "name": "Debug Tests",
            "program": "${workspaceFolder}/node_modules/jest/bin/jest",
            "args": ["--runInBand", "--no-cache"],
            "console": "integratedTerminal"
        }
    ]
}
```

### Python 除錯

```python
# Built-in debugger (pdb)
import pdb

def calculate_total(items):
    total = 0
    pdb.set_trace()  # Debugger starts here

    for item in items:
        total += item.price * item.quantity

    return total

# Breakpoint (Python 3.7+)
def process_order(order):
    breakpoint()  # More convenient than pdb.set_trace()
    # ... code

# Post-mortem debugging
try:
    risky_operation()
except Exception:
    import pdb
    pdb.post_mortem()  # Debug at exception point

# IPython debugging (ipdb)
from ipdb import set_trace
set_trace()  # Better interface than pdb

# Logging for debugging
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

def fetch_user(user_id):
    logger.debug(f'Fetching user: {user_id}')
    user = db.query(User).get(user_id)
    logger.debug(f'Found user: {user}')
    return user

# Profile performance
import cProfile
import pstats

cProfile.run('slow_function()', 'profile_stats')
stats = pstats.Stats('profile_stats')
stats.sort_stats('cumulative')
stats.print_stats(10)  # Top 10 slowest
```

### Go 除錯

```go
// Delve debugger
// Install: go install github.com/go-delve/delve/cmd/dlv@latest
// Run: dlv debug main.go

import (
    "fmt"
    "runtime"
    "runtime/debug"
)

// Print stack trace
func debugStack() {
    debug.PrintStack()
}

// Panic recovery with debugging
func processRequest() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Panic:", r)
            debug.PrintStack()
        }
    }()

    // ... code that might panic
}

// Memory profiling
import _ "net/http/pprof"
// Visit http://localhost:6060/debug/pprof/

// CPU profiling
import (
    "os"
    "runtime/pprof"
)

f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()
// ... code to profile
```

## 進階除錯技巧

### 技巧 1：二分搜尋除錯

```bash
# Git bisect for finding regression
git bisect start
git bisect bad                    # Current commit is bad
git bisect good v1.0.0            # v1.0.0 was good

# Git checks out middle commit
# Test it, then:
git bisect good   # if it works
git bisect bad    # if it's broken

# Continue until bug found
git bisect reset  # when done
```

### 技巧 2：差異除錯

比較正常 vs 損壞：

```markdown
## 什麼不同？

| 面向         | 正常            | 損壞            |
|--------------|-----------------|-----------------|
| 環境         | 開發環境        | 生產環境        |
| Node 版本    | 18.16.0         | 18.15.0         |
| 資料         | 空資料庫        | 100 萬筆記錄    |
| 使用者       | 管理員          | 一般使用者      |
| 瀏覽器       | Chrome          | Safari          |
| 時間         | 白天            | 午夜後          |

假設：時間相關問題？檢查時區處理。
```

### 技巧 3：追蹤除錯

```typescript
// Function call tracing
function trace(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with args:`, args);
        const result = originalMethod.apply(this, args);
        console.log(`${propertyKey} returned:`, result);
        return result;
    };

    return descriptor;
}

class OrderService {
    @trace
    calculateTotal(items: Item[]): number {
        return items.reduce((sum, item) => sum + item.price, 0);
    }
}
```

### 技巧 4：記憶體洩漏偵測

```typescript
// Chrome DevTools Memory Profiler
// 1. Take heap snapshot
// 2. Perform action
// 3. Take another snapshot
// 4. Compare snapshots

// Node.js memory debugging
if (process.memoryUsage().heapUsed > 500 * 1024 * 1024) {
    console.warn('High memory usage:', process.memoryUsage());

    // Generate heap dump
    require('v8').writeHeapSnapshot();
}

// Find memory leaks in tests
let beforeMemory: number;

beforeEach(() => {
    beforeMemory = process.memoryUsage().heapUsed;
});

afterEach(() => {
    const afterMemory = process.memoryUsage().heapUsed;
    const diff = afterMemory - beforeMemory;

    if (diff > 10 * 1024 * 1024) {  // 10MB threshold
        console.warn(`Possible memory leak: ${diff / 1024 / 1024}MB`);
    }
});
```

## 依問題類型的除錯模式

### 模式 1：間歇性錯誤

```markdown
## 不穩定錯誤的策略

1. **加入廣泛日誌**
   - 記錄時間資訊
   - 記錄所有狀態轉換
   - 記錄外部互動

2. **尋找競態條件**
   - 共用狀態的並行存取
   - 非同步操作以不同順序完成
   - 缺少同步

3. **檢查時間相依性**
   - setTimeout/setInterval
   - Promise 解析順序
   - 動畫幀時間

4. **壓力測試**
   - 多次執行
   - 變化時間
   - 模擬負載
```

### 模式 2：效能問題

```markdown
## 效能除錯

1. **先分析**
   - 不要盲目優化
   - 前後測量
   - 找出瓶頸

2. **常見問題**
   - N+1 查詢
   - 不必要的重新渲染
   - 大量資料處理
   - 同步 I/O

3. **工具**
   - 瀏覽器 DevTools Performance 分頁
   - Lighthouse
   - Python: cProfile, line_profiler
   - Node: clinic.js, 0x
```

### 模式 3：生產環境錯誤

```markdown
## 生產環境除錯

1. **收集證據**
   - 錯誤追蹤（Sentry、Bugsnag）
   - 應用程式日誌
   - 使用者回報
   - 指標/監控

2. **在本地重現**
   - 使用生產資料（匿名化）
   - 匹配環境
   - 遵循確切步驟

3. **安全調查**
   - 不要改變生產環境
   - 使用功能旗標
   - 加入監控/日誌
   - 在預備環境測試修正
```

## 最佳實務

1. **先重現**：無法重現就無法修正
2. **隔離問題**：移除複雜性直到最小案例
3. **閱讀錯誤訊息**：它們通常很有幫助
4. **檢查最近變更**：大多數錯誤是最近的
5. **使用版本控制**：Git bisect、blame、歷史記錄
6. **休息一下**：新鮮的眼光看得更清楚
7. **記錄發現**：幫助未來的你
8. **修正根本原因**：不只是症狀

## 常見除錯錯誤

- **進行多項變更**：一次改變一件事
- **不讀錯誤訊息**：閱讀完整的堆疊追蹤
- **假設它很複雜**：通常很簡單
- **在生產環境除錯日誌**：發佈前移除
- **不使用除錯器**：console.log 不總是最好的
- **太早放棄**：堅持會有回報
- **不測試修正**：驗證它確實有效

## 快速除錯檢查清單

```markdown
## 卡住時，檢查：

- [ ] 拼寫錯誤（變數名稱中的拼寫錯誤）
- [ ] 大小寫敏感性（fileName vs filename）
- [ ] Null/undefined 值
- [ ] 陣列索引差一
- [ ] 非同步時間（競態條件）
- [ ] 作用域問題（閉包、提升）
- [ ] 型別不匹配
- [ ] 缺少相依套件
- [ ] 環境變數
- [ ] 檔案路徑（絕對 vs 相對）
- [ ] 快取問題（清除快取）
- [ ] 過時資料（刷新資料庫）
```

## 資源

- **references/debugging-tools-guide.md**：全面的工具文件
- **references/performance-profiling.md**：效能除錯指南
- **references/production-debugging.md**：除錯即時系統
- **assets/debugging-checklist.md**：快速參考檢查清單
- **assets/common-bugs.md**：常見錯誤模式
- **scripts/debug-helper.ts**：除錯工具函式
