遵循 TDD 紅燈階段原則撰寫全面性的失敗測試。

[延伸思考：使用 test-automator 代理程式生成能適當定義預期行為的失敗測試。]

## 角色

使用 Task 工具並搭配 subagent_type="unit-testing::test-automator" 來生成失敗測試。

## 提示詞範本

"為以下項目生成全面性的失敗測試：$ARGUMENTS

## 核心要求

1. **測試結構**
   - 符合測試框架的設定（Jest/pytest/JUnit/Go/RSpec）
   - Arrange-Act-Assert 模式
   - should_X_when_Y 命名慣例
   - 獨立的 fixtures，不相互依賴

2. **行為覆蓋範圍**
   - 正常路徑情境（Happy path）
   - 邊界情況（空值、null、邊界值）
   - 錯誤處理和例外狀況
   - 併發存取（如適用）

3. **失敗驗證**
   - 測試執行時必須失敗
   - 失敗原因必須正確（而非語法或匯入錯誤）
   - 提供有意義的診斷錯誤訊息
   - 避免連鎖失敗

4. **測試類別**
   - Unit：獨立元件行為
   - Integration：元件互動
   - Contract：API/介面契約
   - Property：數學不變性

## 測試框架模式

**JavaScript/TypeScript (Jest/Vitest)**
- 使用 `vi.fn()` 或 `jest.fn()` 模擬（mock）依賴項
- 針對 React 元件使用 `@testing-library`
- 使用 `fast-check` 進行屬性測試

**Python (pytest)**
- 使用適當範圍的 Fixtures
- 使用 Parametrize 處理多個測試案例
- 使用 Hypothesis 進行基於屬性的測試

**Go**
- 使用 subtests 進行表格驅動測試
- 使用 `t.Parallel()` 進行平行執行
- 使用 `testify/assert` 以獲得更清晰的斷言

**Ruby (RSpec)**
- 使用 `let` 進行延遲載入，`let!` 進行立即載入
- 使用 Contexts 處理不同情境
- 使用 Shared examples 處理共同行為

## 品質檢查清單

- 可讀的測試名稱，記錄測試意圖
- 每個測試只測試一個行為
- 不洩漏實作細節
- 使用有意義的測試資料（避免 'foo'/'bar'）
- 測試作為活文件（living documentation）

## 應避免的反模式

- 測試立即通過
- 測試實作而非行為
- 複雜的設定程式碼
- 每個測試包含多個職責
- 與特定細節過度耦合的脆弱測試

## 邊界情況類別

- **Null/Empty**：undefined、null、空字串/陣列/物件
- **Boundaries**：最小/最大值、單一元素、容量限制
- **Special Cases**：Unicode、空白字元、特殊字元
- **State**：無效的狀態轉換、併發修改
- **Errors**：網路失敗、逾時、權限問題

## 輸出要求

- 包含匯入語句的完整測試檔案
- 測試目的說明文件
- 執行和驗證失敗的命令
- 指標：測試數量、覆蓋範圍
- 綠燈階段的後續步驟"

## 驗證

生成後：
1. 執行測試 - 確認測試失敗
2. 驗證失敗訊息是否有幫助
3. 檢查測試獨立性
4. 確保全面的覆蓋範圍

## 範例（最小化）

```typescript
// auth.service.test.ts
describe('AuthService', () => {
  let authService: AuthService;
  let mockUserRepo: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockUserRepo = { findByEmail: jest.fn() } as any;
    authService = new AuthService(mockUserRepo);
  });

  it('should_return_token_when_valid_credentials', async () => {
    const user = { id: '1', email: 'test@example.com', passwordHash: 'hashed' };
    mockUserRepo.findByEmail.mockResolvedValue(user);

    const result = await authService.authenticate('test@example.com', 'pass');

    expect(result.success).toBe(true);
    expect(result.token).toBeDefined();
  });

  it('should_fail_when_user_not_found', async () => {
    mockUserRepo.findByEmail.mockResolvedValue(null);

    const result = await authService.authenticate('none@example.com', 'pass');

    expect(result.success).toBe(false);
    expect(result.error).toBe('INVALID_CREDENTIALS');
  });
});
```

測試需求：$ARGUMENTS
