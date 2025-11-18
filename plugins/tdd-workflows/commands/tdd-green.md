實作最少程式碼以通過 TDD 綠燈階段的失敗測試：

[延伸思考：此工具使用 test-automator 代理來實作使測試通過所需的最少程式碼。它專注於簡單性，避免過度工程化，同時確保所有測試變為綠燈。]

## 實作流程

使用 Task 工具搭配 subagent_type="unit-testing::test-automator" 來實作最少的通過程式碼。

提示：「實作最少程式碼以通過這些失敗的測試：$ARGUMENTS。遵循 TDD 綠燈階段原則：

1. **實作前分析**
   - 檢視所有失敗的測試及其錯誤訊息
   - 識別使測試通過的最簡單路徑
   - 將測試需求對應到最少的實作需求
   - 避免過早優化或過度工程化
   - 只專注於讓測試變綠燈，而非完美的程式碼

2. **實作策略**
   - **假裝實作（Fake It）**：在適當時回傳寫死的值
   - **明顯實作（Obvious Implementation）**：當解決方案簡單明確時
   - **三角定位（Triangulation）**：僅在多個測試需要時才泛化
   - 從最簡單的測試開始並逐步推進
   - 一次一個測試 - 不要試圖一次全部通過

3. **程式碼結構指南**
   - 撰寫可能可行的最少程式碼
   - 避免加入測試未要求的功能
   - 初始使用簡單的資料結構
   - 將架構決策延後到重構階段
   - 保持方法/函式小巧且專注
   - 除非測試要求，否則不要加入錯誤處理

4. **語言特定模式**
   - **JavaScript/TypeScript**：簡單函式，初期避免類別
   - **Python**：函式優先於類別，簡單回傳
   - **Java**：最少的類別結構，還不需要模式
   - **C#**：基本實作，還不需要介面
   - **Go**：簡單函式，延後 goroutines/channels
   - **Ruby**：盡可能先用程序式而非物件導向

5. **漸進式實作**
   - 以最簡單的程式碼讓第一個測試通過
   - 每次變更後執行測試以驗證進度
   - 為下一個失敗測試僅加入足夠的程式碼
   - 抵抗超出測試需求實作的衝動
   - 追蹤技術債以供重構階段使用
   - 記錄所採取的假設和捷徑

6. **常見綠燈階段技巧**
   - 初始測試使用寫死的回傳值
   - 有限測試案例使用簡單的 if/else
   - 僅在迭代測試需要時使用基本迴圈
   - 最少的資料結構（陣列優先於複雜物件）
   - 記憶體儲存優先於資料庫整合
   - 同步優先於非同步實作

7. **成功標準**
   ✓ 所有測試通過（綠燈）
   ✓ 沒有超出測試需求的額外功能
   ✓ 程式碼可讀，即使不是最佳化
   ✓ 沒有破壞現有功能
   ✓ 實作時間最小化
   ✓ 明確識別重構路徑

8. **要避免的反模式**
   - 鍍金或加入未要求的功能
   - 過早實作設計模式
   - 沒有測試理由的複雜抽象
   - 沒有指標的效能優化
   - 在綠燈階段加入測試
   - 在實作期間重構
   - 忽略測試失敗以繼續前進

9. **實作指標**
   - 到達綠燈時間：追蹤實作持續時間
   - 程式碼行數：測量實作大小
   - 循環複雜度：初期保持低水平
   - 測試通過率：必須達到 100%
   - 程式碼覆蓋率：驗證所有路徑都已測試

10. **驗證步驟**
    - 執行所有測試並確認通過
    - 驗證現有測試沒有退化
    - 檢查實作確實是最少的
    - 記錄任何產生的技術債
    - 為重構階段準備筆記

輸出應包括：
- 完整的實作程式碼
- 顯示全部綠燈的測試執行結果
- 為後續重構所採取的捷徑列表
- 實作時間指標
- 技術債文件
- 重構階段的準備度評估」

## 實作後檢查

實作後：
1. 執行完整測試套件以確認所有測試通過
2. 驗證沒有破壞現有測試
3. 記錄需要重構的區域
4. 檢查實作確實是最少的
5. 記錄實作時間以供指標使用

## 復原流程

如果測試仍然失敗：
- 仔細檢視測試需求
- 檢查是否誤解斷言
- 加入最少程式碼以解決特定失敗
- 避免從頭重寫的誘惑
- 考慮測試本身是否需要調整

## 整合點

- 從 tdd-red.md 測試建立接續
- 為 tdd-refactor.md 改進做準備
- 更新測試覆蓋率指標
- 觸發 CI/CD 管道驗證
- 記錄技術債以供追蹤

## 最佳實踐

- 在此階段接受「夠好」的標準
- 速度優先於完美（完美在重構階段達成）
- 先讓它運作，再讓它正確，然後讓它快速
- 相信重構階段會改善程式碼
- 保持變更小而漸進
- 慶祝達到綠燈狀態！

## 完整實作範例

### 範例 1：最少實作 → 生產就緒（使用者服務）

**測試需求：**
```typescript
describe('UserService', () => {
  it('should create a new user', async () => {
    const user = await userService.create({ email: 'test@example.com', name: 'Test' });
    expect(user.id).toBeDefined();
    expect(user.email).toBe('test@example.com');
  });

  it('should find user by email', async () => {
    await userService.create({ email: 'test@example.com', name: 'Test' });
    const user = await userService.findByEmail('test@example.com');
    expect(user).toBeDefined();
  });
});
```

**階段 1：假裝實作（最少）**
```typescript
class UserService {
  create(data: { email: string; name: string }) {
    return { id: '123', email: data.email, name: data.name };
  }

  findByEmail(email: string) {
    return { id: '123', email: email, name: 'Test' };
  }
}
```
*測試通過。實作顯然是假的，但驗證了測試結構。*

**階段 2：簡單的真實實作**
```typescript
class UserService {
  private users: Map<string, User> = new Map();
  private nextId = 1;

  create(data: { email: string; name: string }) {
    const user = { id: String(this.nextId++), ...data };
    this.users.set(user.email, user);
    return user;
  }

  findByEmail(email: string) {
    return this.users.get(email) || null;
  }
}
```
*記憶體儲存。測試通過。對綠燈階段來說夠好了。*

**階段 3：生產就緒（重構階段）**
```typescript
class UserService {
  constructor(private db: Database) {}

  async create(data: { email: string; name: string }) {
    const existing = await this.db.query('SELECT * FROM users WHERE email = ?', [data.email]);
    if (existing) throw new Error('User exists');

    const id = await this.db.insert('users', data);
    return { id, ...data };
  }

  async findByEmail(email: string) {
    return this.db.queryOne('SELECT * FROM users WHERE email = ?', [email]);
  }
}
```
*資料庫整合、錯誤處理、驗證 - 留待重構階段。*

### 範例 2：API 優先實作（Express）

**測試需求：**
```javascript
describe('POST /api/tasks', () => {
  it('should create task and return 201', async () => {
    const res = await request(app)
      .post('/api/tasks')
      .send({ title: 'Test Task' });

    expect(res.status).toBe(201);
    expect(res.body.id).toBeDefined();
    expect(res.body.title).toBe('Test Task');
  });
});
```

**階段 1：寫死的回應**
```javascript
app.post('/api/tasks', (req, res) => {
  res.status(201).json({ id: '1', title: req.body.title });
});
```
*測試立即通過。還不需要邏輯。*

**階段 2：簡單邏輯**
```javascript
let tasks = [];
let nextId = 1;

app.post('/api/tasks', (req, res) => {
  const task = { id: String(nextId++), title: req.body.title };
  tasks.push(task);
  res.status(201).json(task);
});
```
*最少的狀態管理。準備好接受更多測試。*

**階段 3：分層架構（重構）**
```javascript
// Controller
app.post('/api/tasks', async (req, res) => {
  try {
    const task = await taskService.create(req.body);
    res.status(201).json(task);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Service layer
class TaskService {
  constructor(private repository: TaskRepository) {}

  async create(data: CreateTaskDto): Promise<Task> {
    this.validate(data);
    return this.repository.save(data);
  }
}
```
*在重構階段加入適當的關注點分離。*

### 範例 3：資料庫整合（Django）

**測試需求：**
```python
def test_product_creation():
    product = Product.objects.create(name="Widget", price=9.99)
    assert product.id is not None
    assert product.name == "Widget"

def test_product_price_validation():
    with pytest.raises(ValidationError):
        Product.objects.create(name="Widget", price=-1)
```

**階段 1：僅模型**
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
```
*第一個測試通過。第二個測試失敗 - 驗證未實作。*

**階段 2：加入驗證**
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    def clean(self):
        if self.price < 0:
            raise ValidationError("Price cannot be negative")

    def save(self, *args, **kwargs):
        self.clean()
        super().save(*args, **kwargs)
```
*所有測試通過。加入最少的驗證邏輯。*

**階段 3：豐富的領域模型（重構）**
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        indexes = [models.Index(fields=['category', '-created_at'])]

    def clean(self):
        if self.price < 0:
            raise ValidationError("Price cannot be negative")
        if self.price > 10000:
            raise ValidationError("Price exceeds maximum")

    def apply_discount(self, percentage: float) -> Decimal:
        return self.price * (1 - percentage / 100)
```
*需要時加入額外功能、索引、業務邏輯。*

### 範例 4：React 元件實作

**測試需求：**
```typescript
describe('UserProfile', () => {
  it('should display user name', () => {
    render(<UserProfile user={{ name: 'John', email: 'john@test.com' }} />);
    expect(screen.getByText('John')).toBeInTheDocument();
  });

  it('should display email', () => {
    render(<UserProfile user={{ name: 'John', email: 'john@test.com' }} />);
    expect(screen.getByText('john@test.com')).toBeInTheDocument();
  });
});
```

**階段 1：最少的 JSX**
```typescript
interface UserProfileProps {
  user: { name: string; email: string };
}

const UserProfile: React.FC<UserProfileProps> = ({ user }) => (
  <div>
    <div>{user.name}</div>
    <div>{user.email}</div>
  </div>
);
```
*測試通過。沒有樣式，沒有結構。*

**階段 2：基本結構**
```typescript
const UserProfile: React.FC<UserProfileProps> = ({ user }) => (
  <div className="user-profile">
    <h2>{user.name}</h2>
    <p>{user.email}</p>
  </div>
);
```
*加入語意化 HTML，className 作為樣式掛鉤。*

**階段 3：生產元件（重構）**
```typescript
const UserProfile: React.FC<UserProfileProps> = ({ user }) => {
  const [isEditing, setIsEditing] = useState(false);

  return (
    <div className="user-profile" role="article" aria-label="User profile">
      <header>
        <h2>{user.name}</h2>
        <button onClick={() => setIsEditing(true)} aria-label="Edit profile">
          Edit
        </button>
      </header>
      <section>
        <p>{user.email}</p>
        {user.bio && <p>{user.bio}</p>}
      </section>
    </div>
  );
};
```
*漸進式加入無障礙性、互動、額外功能。*

## 決策框架

### 框架 1：假裝實作 vs. 真實實作

**何時該假裝實作：**
- 新功能的第一個測試
- 複雜的外部相依性（支付閘道、API）
- 實作方法仍不確定
- 需要先驗證測試結構
- 時間壓力需要看到所有測試綠燈

**何時該真實實作：**
- 第二或第三個測試揭示模式
- 實作顯而易見且簡單
- 假裝實作會比真實程式碼更複雜
- 需要測試整合點
- 測試明確要求真實行為

**決策矩陣：**
```
複雜度     低        | 高
         ↓         | ↓
簡單    → 真實實作  | 先假裝，稍後真實
複雜    → 真實實作  | 假裝，評估替代方案
```

### 框架 2：複雜度權衡分析

**簡單度分數計算：**
```
分數 = (程式碼行數) + (循環複雜度 × 2) + (相依性 × 3)

< 20  → 夠簡單，直接實作
20-50 → 考慮更簡單的替代方案
> 50  → 將複雜度延後到重構階段
```

**範例評估：**
```typescript
// 選項 A：直接實作（分數：45）
function calculateShipping(weight: number, distance: number, express: boolean): number {
  let base = weight * 0.5 + distance * 0.1;
  if (express) base *= 2;
  if (weight > 50) base += 10;
  if (distance > 1000) base += 20;
  return base;
}

// 選項 B：綠燈階段最簡單（分數：15）
function calculateShipping(weight: number, distance: number, express: boolean): number {
  return express ? 50 : 25; // 假裝實作，直到更多測試驅動真實邏輯
}
```
*綠燈階段選擇選項 B，隨著測試需求演進到選項 A。*

### 框架 3：效能考量時機

**綠燈階段：專注於正確性**
```
❌ 避免：
- 快取策略
- 資料庫查詢優化
- 演算法複雜度改進
- 過早的記憶體優化

✓ 接受：
- O(n²) 如果它讓程式碼更簡單
- 多次資料庫查詢
- 同步操作
- 效率低但清楚的演算法
```

**綠燈階段何時效能很重要：**
1. 效能是明確的測試需求
2. 實作會導致測試套件逾時
3. 記憶體洩漏會使測試當機
4. 資源耗盡阻止測試

**效能測試整合：**
```typescript
// 功能測試通過後才加入效能測試
describe('Performance', () => {
  it('should handle 1000 users within 100ms', () => {
    const start = Date.now();
    for (let i = 0; i < 1000; i++) {
      userService.create({ email: `user${i}@test.com`, name: `User ${i}` });
    }
    expect(Date.now() - start).toBeLessThan(100);
  });
});
```

## 框架特定模式

### React 模式

**簡單元件 → Hooks → Context：**
```typescript
// 綠燈階段：僅使用 Props
const Counter = ({ count, onIncrement }) => (
  <button onClick={onIncrement}>{count}</button>
);

// 重構：加入 hooks
const Counter = () => {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
};

// 重構：提取到 context
const Counter = () => {
  const { count, increment } = useCounter();
  return <button onClick={increment}>{count}</button>;
};
```

### Django 模式

**函式視圖 → 類別視圖 → 泛型視圖：**
```python
# 綠燈階段：簡單函式
def product_list(request):
    products = Product.objects.all()
    return JsonResponse({'products': list(products.values())})

# 重構：類別式視圖
class ProductListView(View):
    def get(self, request):
        products = Product.objects.all()
        return JsonResponse({'products': list(products.values())})

# 重構：泛型視圖
class ProductListView(ListView):
    model = Product
    context_object_name = 'products'
```

### Express 模式

**行內 → 中介軟體 → 服務層：**
```javascript
// 綠燈階段：行內邏輯
app.post('/api/users', (req, res) => {
  const user = { id: Date.now(), ...req.body };
  users.push(user);
  res.json(user);
});

// 重構：提取中介軟體
app.post('/api/users', validateUser, (req, res) => {
  const user = userService.create(req.body);
  res.json(user);
});

// 重構：完整分層
app.post('/api/users',
  validateUser,
  asyncHandler(userController.create)
);
```

## 重構抵抗模式

### 模式 1：測試錨點

透過維護介面契約，在重構期間保持測試綠燈：

```typescript
// 原始實作（測試綠燈）
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// 重構：加入稅金計算（保持介面）
function calculateTotal(items: Item[]): number {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  const tax = subtotal * 0.1;
  return subtotal + tax;
}

// 測試仍然綠燈，因為回傳型別/行為未改變
```

### 模式 2：平行實作

同時執行舊實作和新實作：

```python
def process_order(order):
    # 舊實作（測試依賴於此）
    result_old = legacy_process(order)

    # 新實作（平行測試）
    result_new = new_process(order)

    # 驗證它們相符
    assert result_old == result_new, "Implementation mismatch"

    return result_old  # 保持測試綠燈
```

### 模式 3：重構的功能旗標

```javascript
class PaymentService {
  processPayment(amount) {
    if (config.USE_NEW_PAYMENT_PROCESSOR) {
      return this.newPaymentProcessor(amount);
    }
    return this.legacyPaymentProcessor(amount);
  }
}
```

## 效能優先的綠燈階段策略

### 策略 1：型別驅動開發

使用型別來指導最少實作：

```typescript
// 型別定義契約
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// 綠燈階段：記憶體實作
class InMemoryUserRepository implements UserRepository {
  private users = new Map<string, User>();

  async findById(id: string) {
    return this.users.get(id) || null;
  }

  async save(user: User) {
    this.users.set(user.id, user);
  }
}

// 重構：資料庫實作（相同介面）
class DatabaseUserRepository implements UserRepository {
  constructor(private db: Database) {}

  async findById(id: string) {
    return this.db.query('SELECT * FROM users WHERE id = ?', [id]);
  }

  async save(user: User) {
    await this.db.insert('users', user);
  }
}
```

### 策略 2：契約測試整合

```typescript
// 定義契約
const userServiceContract = {
  create: {
    input: { email: 'string', name: 'string' },
    output: { id: 'string', email: 'string', name: 'string' }
  }
};

// 綠燈階段：實作符合契約
class UserService {
  create(data: { email: string; name: string }) {
    return { id: '123', ...data }; // 最少但符合契約
  }
}

// 契約測試確保符合性
describe('UserService Contract', () => {
  it('should match create contract', () => {
    const result = userService.create({ email: 'test@test.com', name: 'Test' });
    expect(typeof result.id).toBe('string');
    expect(typeof result.email).toBe('string');
    expect(typeof result.name).toBe('string');
  });
});
```

### 策略 3：持續重構工作流程

**綠燈階段的微重構：**

```python
# 測試通過
def calculate_discount(price, customer_type):
    if customer_type == 'premium':
        return price * 0.8
    return price

# 立即微重構（測試仍然綠燈）
DISCOUNT_RATES = {
    'premium': 0.8,
    'standard': 1.0
}

def calculate_discount(price, customer_type):
    rate = DISCOUNT_RATES.get(customer_type, 1.0)
    return price * rate
```

**安全重構檢查清單：**
- ✓ 重構前測試綠燈
- ✓ 一次改變一件事
- ✓ 每次變更後執行測試
- ✓ 每次成功重構後提交
- ✓ 不改變行為，只改變結構

## 現代開發實踐（2024/2025）

### 型別驅動開發

**Python 型別提示：**
```python
from typing import Optional, List
from dataclasses import dataclass

@dataclass
class User:
    id: str
    email: str
    name: str

class UserService:
    def create(self, email: str, name: str) -> User:
        return User(id="123", email=email, name=name)

    def find_by_email(self, email: str) -> Optional[User]:
        return None  # 最少實作
```

**TypeScript 嚴格模式：**
```typescript
// 在 tsconfig.json 中啟用嚴格模式
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}

// 由型別指導的實作
interface CreateUserDto {
  email: string;
  name: string;
}

class UserService {
  create(data: CreateUserDto): User {
    // 型別系統強制執行契約
    return { id: '123', email: data.email, name: data.name };
  }
}
```

### AI 輔助綠燈階段

**使用 Copilot/AI 工具：**
1. 先寫測試（人類驅動）
2. 讓 AI 建議最少實作
3. 驗證建議通過測試
4. 如果確實是最少的就接受，如果過度工程化就拒絕
5. 與 AI 迭代進行重構階段

**AI 提示模式：**
```
給定這些失敗的測試：
[貼上測試]

提供使測試通過的最少實作。
不要加入錯誤處理、驗證或超出測試需求的功能。
專注於簡單性而非完整性。
```

### 雲原生模式

**本地 → 容器 → 雲端：**
```javascript
// 綠燈階段：本地實作
class CacheService {
  private cache = new Map();

  get(key) { return this.cache.get(key); }
  set(key, value) { this.cache.set(key, value); }
}

// 重構：Redis 相容介面
class CacheService {
  constructor(private redis) {}

  async get(key) { return this.redis.get(key); }
  async set(key, value) { return this.redis.set(key, value); }
}

// 生產環境：具備備援的分散式快取
class CacheService {
  constructor(private redis, private fallback) {}

  async get(key) {
    try {
      return await this.redis.get(key);
    } catch {
      return this.fallback.get(key);
    }
  }
}
```

### 可觀測性驅動開發

**在綠燈階段加入可觀測性掛鉤：**
```typescript
class OrderService {
  async createOrder(data: CreateOrderDto): Promise<Order> {
    console.log('[OrderService] Creating order', { data }); // 簡單的日誌記錄

    const order = { id: '123', ...data };

    console.log('[OrderService] Order created', { orderId: order.id }); // 成功日誌

    return order;
  }
}

// 重構：結構化日誌記錄
class OrderService {
  constructor(private logger: Logger) {}

  async createOrder(data: CreateOrderDto): Promise<Order> {
    this.logger.info('order.create.start', { data });

    const order = await this.repository.save(data);

    this.logger.info('order.create.success', {
      orderId: order.id,
      duration: Date.now() - start
    });

    return order;
  }
}
```

要使其通過的測試：$ARGUMENTS