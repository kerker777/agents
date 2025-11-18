---
name: code-review-excellence
description: 精通有效的程式碼審查實務，提供建設性回饋、及早發現錯誤，並在維持團隊士氣的同時促進知識分享。適用於審查拉取請求、建立審查標準或指導開發人員。
---

# 程式碼審查卓越

透過建設性回饋、系統化分析和協作改進，將程式碼審查從把關轉變為知識分享。

## 何時使用此技能

- 審查拉取請求和程式碼變更
- 為團隊建立程式碼審查標準
- 透過審查指導初級開發人員
- 進行架構審查
- 建立審查檢查清單和指南
- 改善團隊協作
- 減少程式碼審查週期時間
- 維護程式碼品質標準

## 核心原則

### 1. 審查心態

**程式碼審查的目標：**
- 捕捉錯誤和邊緣案例
- 確保程式碼可維護性
- 在團隊間分享知識
- 執行編碼標準
- 改進設計和架構
- 建立團隊文化

**不是目標：**
- 展示知識
- 挑剔格式（使用 linter）
- 不必要地阻擋進度
- 改寫成你偏好的方式

### 2. 有效回饋

**好的回饋是：**
- 具體且可行
- 教育性的，而非批判性的
- 針對程式碼，而非個人
- 平衡的（也讚美好的工作）
- 有優先順序的（關鍵 vs 最好有）

```markdown
❌ 不好："這是錯的。"
✅ 好："當多個使用者同時存取時，這可能導致競態條件。
         考慮在這裡使用互斥鎖。"

❌ 不好："為什麼你不用 X 模式？"
✅ 好："你有考慮過倉儲模式嗎？這會讓測試更容易。
         這裡有個範例：[連結]"

❌ 不好："重新命名這個變數。"
✅ 好："[小建議] 考慮用 `userCount` 而非 `uc` 以提高
         清晰度。如果你偏好保留也沒問題。"
```

### 3. 審查範圍

**應該審查的：**
- 邏輯正確性和邊緣案例
- 安全漏洞
- 效能影響
- 測試覆蓋率和品質
- 錯誤處理
- 文件和註解
- API 設計和命名
- 架構適配性

**不應手動審查的：**
- 程式碼格式（使用 Prettier、Black 等）
- import 組織
- Linting 違規
- 簡單的拼寫錯誤

## 審查流程

### 階段 1：收集背景資訊（2-3 分鐘）

```markdown
在深入程式碼之前，先理解：

1. 閱讀 PR 描述和相關議題
2. 檢查 PR 大小（超過 400 行？要求分割）
3. 檢視 CI/CD 狀態（測試通過了嗎？）
4. 理解業務需求
5. 注意任何相關的架構決策
```

### 階段 2：高層次審查（5-10 分鐘）

```markdown
1. **架構與設計**
   - 解決方案符合問題嗎？
   - 有更簡單的方法嗎？
   - 與現有模式一致嗎？
   - 能擴展嗎？

2. **檔案組織**
   - 新檔案放在正確位置嗎？
   - 程式碼邏輯分組了嗎？
   - 有重複的檔案嗎？

3. **測試策略**
   - 有測試嗎？
   - 測試涵蓋邊緣案例嗎？
   - 測試可讀嗎？
```

### 階段 3：逐行審查（10-20 分鐘）

```markdown
對每個檔案：

1. **邏輯與正確性**
   - 處理邊緣案例了嗎？
   - 有差一錯誤嗎？
   - Null/undefined 檢查？
   - 競態條件？

2. **安全性**
   - 輸入驗證？
   - SQL 注入風險？
   - XSS 漏洞？
   - 敏感資料暴露？

3. **效能**
   - N+1 查詢？
   - 不必要的迴圈？
   - 記憶體洩漏？
   - 阻塞操作？

4. **可維護性**
   - 清楚的變數名稱？
   - 函式只做一件事？
   - 複雜程式碼有註解？
   - 魔術數字已提取？
```

### 階段 4：總結與決策（2-3 分鐘）

```markdown
1. 總結主要關注點
2. 強調你喜歡的部分
3. 做出明確決定：
   - ✅ 核准
   - 💬 評論（小建議）
   - 🔄 請求變更（必須處理）
4. 如果複雜則提議配對
```

## 審查技巧

### 技巧 1：檢查清單方法

```markdown
## 安全檢查清單
- [ ] 使用者輸入已驗證和清理
- [ ] SQL 查詢使用參數化
- [ ] 已檢查身份驗證/授權
- [ ] 沒有硬編碼的密鑰
- [ ] 錯誤訊息不洩漏資訊

## 效能檢查清單
- [ ] 沒有 N+1 查詢
- [ ] 資料庫查詢已建立索引
- [ ] 大列表已分頁
- [ ] 昂貴操作已快取
- [ ] 熱路徑中沒有阻塞 I/O

## 測試檢查清單
- [ ] 已測試正常路徑
- [ ] 涵蓋邊緣案例
- [ ] 已測試錯誤情況
- [ ] 測試名稱具描述性
- [ ] 測試是確定性的
```

### 技巧 2：提問方法

與其陳述問題，不如提出問題以鼓勵思考：

```markdown
❌ "如果列表為空，這會失敗。"
✅ "如果 `items` 是空陣列會發生什麼？"

❌ "你需要在這裡處理錯誤。"
✅ "如果 API 呼叫失敗，這應該如何運作？"

❌ "這效率不高。"
✅ "我看到這會遍歷所有使用者。我們有考慮過在有 10 萬
    使用者時的效能影響嗎？"
```

### 技巧 3：建議，不要命令

```markdown
## 使用協作語言

❌ "你必須將這改為使用 async/await"
✅ "建議：async/await 可能會讓這更易讀：
    ```typescript
    async function fetchUser(id: string) {
        const user = await db.query('SELECT * FROM users WHERE id = ?', id);
        return user;
    }
    ```
    你覺得如何？"

❌ "將這提取成函式"
✅ "這個邏輯出現在 3 個地方。是否考慮將它提取成
    共用的工具函式？"
```

### 技巧 4：區分嚴重程度

```markdown
使用標籤指示優先順序：

🔴 [blocking] - 合併前必須修正
🟡 [important] - 應該修正，如不同意請討論
🟢 [nit] - 最好有，不阻擋
💡 [suggestion] - 可考慮的替代方法
📚 [learning] - 教育性評論，無需行動
🎉 [praise] - 做得好，繼續保持！

範例：
"🔴 [blocking] 這個 SQL 查詢容易受注入攻擊。
 請使用參數化查詢。"

"🟢 [nit] 考慮將 `data` 重新命名為 `userData` 以提高清晰度。"

"🎉 [praise] 優秀的測試覆蓋率！這會捕捉邊緣案例。"
```

## 特定語言模式

### Python 程式碼審查

```python
# 檢查 Python 特定問題

# ❌ 可變預設參數
def add_item(item, items=[]):  # 錯誤！跨呼叫共用
    items.append(item)
    return items

# ✅ 使用 None 作為預設值
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# ❌ 捕捉過於廣泛
try:
    result = risky_operation()
except:  # 捕捉所有，甚至 KeyboardInterrupt！
    pass

# ✅ 捕捉特定例外
try:
    result = risky_operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise

# ❌ 使用可變類別屬性
class User:
    permissions = []  # 在所有實例間共用！

# ✅ 在 __init__ 中初始化
class User:
    def __init__(self):
        self.permissions = []
```

### TypeScript/JavaScript 程式碼審查

```typescript
// 檢查 TypeScript 特定問題

// ❌ 使用 any 會破壞型別安全
function processData(data: any) {  // 避免 any
    return data.value;
}

// ✅ 使用適當的型別
interface DataPayload {
    value: string;
}
function processData(data: DataPayload) {
    return data.value;
}

// ❌ 未處理非同步錯誤
async function fetchUser(id: string) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();  // 如果網路失敗怎麼辦？
}

// ✅ 正確處理錯誤
async function fetchUser(id: string): Promise<User> {
    try {
        const response = await fetch(`/api/users/${id}`);
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw error;
    }
}

// ❌ 修改 props
function UserProfile({ user }: Props) {
    user.lastViewed = new Date();  // 修改 prop！
    return <div>{user.name}</div>;
}

// ✅ 不要修改 props
function UserProfile({ user, onView }: Props) {
    useEffect(() => {
        onView(user.id);  // 通知父元件更新
    }, [user.id]);
    return <div>{user.name}</div>;
}
```

## 進階審查模式

### 模式 1：架構審查

```markdown
審查重大變更時：

1. **先設計文件**
   - 對於大型功能，在程式碼之前要求設計文件
   - 在實作前與團隊審查設計
   - 就方法達成共識以避免重做

2. **分階段審查**
   - 第一個 PR：核心抽象和介面
   - 第二個 PR：實作
   - 第三個 PR：整合和測試
   - 更容易審查，更快迭代

3. **考慮替代方案**
   - "我們有考慮過使用 [模式/函式庫] 嗎？"
   - "與更簡單的方法相比，取捨是什麼？"
   - "隨著需求變化，這將如何演進？"
```

### 模式 2：測試品質審查

```typescript
// ❌ 不好的測試：測試實作細節
test('increments counter variable', () => {
    const component = render(<Counter />);
    const button = component.getByRole('button');
    fireEvent.click(button);
    expect(component.state.counter).toBe(1);  // 測試內部狀態
});

// ✅ 好的測試：測試行為
test('displays incremented count when clicked', () => {
    render(<Counter />);
    const button = screen.getByRole('button', { name: /increment/i });
    fireEvent.click(button);
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

// 測試的審查問題：
// - 測試描述的是行為，而非實作嗎？
// - 測試名稱清楚且具描述性嗎？
// - 測試涵蓋邊緣案例嗎？
// - 測試是獨立的（沒有共用狀態）嗎？
// - 測試可以以任何順序執行嗎？
```

### 模式 3：安全審查

```markdown
## 安全審查檢查清單

### 身份驗證與授權
- [ ] 需要的地方有身份驗證嗎？
- [ ] 每個動作前都有授權檢查嗎？
- [ ] JWT 驗證正確（簽章、過期）嗎？
- [ ] API 金鑰/密鑰妥善保護了嗎？

### 輸入驗證
- [ ] 所有使用者輸入都已驗證？
- [ ] 檔案上傳有限制（大小、類型）嗎？
- [ ] SQL 查詢參數化了嗎？
- [ ] XSS 保護（轉義輸出）？

### 資料保護
- [ ] 密碼已雜湊（bcrypt/argon2）？
- [ ] 敏感資料在靜態時加密？
- [ ] 敏感資料強制使用 HTTPS？
- [ ] 根據法規處理 PII？

### 常見漏洞
- [ ] 沒有 eval() 或類似的動態執行？
- [ ] 沒有硬編碼的密鑰？
- [ ] 狀態變更操作有 CSRF 保護？
- [ ] 公開端點有速率限制？
```

## 給予困難回饋

### 模式：三明治方法（改良版）

```markdown
傳統：讚美 + 批評 + 讚美（感覺虛假）

更好：背景 + 具體問題 + 有用的解決方案

範例：
"我注意到支付處理邏輯內嵌在控制器中。這使得測試
和重用變得更困難。

[具體問題]
calculateTotal() 函式混合了稅金計算、折扣邏輯和
資料庫查詢，使其難以單元測試和理解。

[有用的解決方案]
我們可以將這提取成 PaymentService 類別嗎？這會讓
它可測試和可重用。如果需要，我可以和你配對處理。"
```

### 處理分歧

```markdown
當作者不同意你的回饋時：

1. **尋求理解**
   "幫我理解你的方法。是什麼讓你選擇這個模式？"

2. **承認有效觀點**
   "關於 X 的觀點很好。我沒考慮到那個。"

3. **提供數據**
   "我擔心效能。我們可以加個基準測試來驗證方法嗎？"

4. **必要時升級**
   "讓 [架構師/資深開發] 對此發表意見吧。"

5. **知道何時放手**
   如果它能運作且不是關鍵問題，就核准它。
   完美是進步的敵人。
```

## 最佳實務

1. **及時審查**：24 小時內，最好當天
2. **限制 PR 大小**：最多 200-400 行以進行有效審查
3. **分時段審查**：最多 60 分鐘，休息一下
4. **使用審查工具**：GitHub、GitLab 或專用工具
5. **盡可能自動化**：Linter、格式化工具、安全掃描
6. **建立關係**：表情符號、讚美和同理心很重要
7. **保持可用**：在複雜問題上提議配對
8. **向他人學習**：審查其他人的審查評論

## 常見陷阱

- **完美主義**：因小的風格偏好而阻擋 PR
- **範圍蔓延**："既然你在做，可以也...嗎"
- **不一致**：對不同的人有不同標準
- **延遲審查**：讓 PR 擱置數天
- **消失**：請求變更後就消失
- **橡皮圖章**：未實際審查就核准
- **自行車棚**：廣泛討論瑣碎細節

## 範本

### PR 審查評論範本

```markdown
## 摘要
[簡要概述審查內容]

## 優點
- [做得好的地方]
- [好的模式或方法]

## 必要變更
🔴 [阻擋問題 1]
🔴 [阻擋問題 2]

## 建議
💡 [改進 1]
💡 [改進 2]

## 問題
❓ [需要對 X 的澄清]
❓ [替代方法考量]

## 結論
✅ 處理必要變更後核准
```

## 資源

- **references/code-review-best-practices.md**：全面的審查指南
- **references/common-bugs-checklist.md**：特定語言的錯誤檢查
- **references/security-review-guide.md**：專注於安全的審查檢查清單
- **assets/pr-review-template.md**：標準審查評論範本
- **assets/review-checklist.md**：快速參考檢查清單
- **scripts/pr-analyzer.py**：分析 PR 複雜度並建議審查者
