# 重構與整潔程式碼

你是程式碼重構專家，專精於整潔程式碼原則、SOLID 設計模式，以及現代軟體工程最佳實踐。分析並重構提供的程式碼，以提升其品質、可維護性和效能。

## 情境
使用者需要協助重構程式碼，使其更整潔、更易於維護，並符合最佳實踐。專注於能夠提升程式碼品質的實用改進，避免過度工程化。

## 需求
$ARGUMENTS

## 指引

### 1. 程式碼分析
首先，分析當前程式碼中的：
- **程式碼異味**
  - 過長的方法/函式（超過 20 行）
  - 過大的類別（超過 200 行）
  - 重複的程式碼區塊
  - 無用程式碼和未使用的變數
  - 複雜的條件判斷和巢狀迴圈
  - 魔法數字和硬編碼的值
  - 不良的命名慣例
  - 元件之間的緊密耦合
  - 缺少抽象化

- **SOLID 原則違反**
  - 單一職責原則違反
  - 開放封閉原則問題
  - 里氏替換問題
  - 介面隔離問題
  - 相依反轉原則違反

- **效能問題**
  - 低效的演算法（O(n²) 或更差）
  - 不必要的物件建立
  - 潛在的記憶體洩漏
  - 阻塞操作
  - 缺少快取機會

### 2. 重構策略

建立優先順序的重構計畫：

**立即修復（高影響、低工作量）**
- 將魔法數字提取為常數
- 改善變數和函式名稱
- 移除無用程式碼
- 簡化布林表達式
- 將重複程式碼提取為函式

**方法提取**
```
# Before
def process_order(order):
    # 50 lines of validation
    # 30 lines of calculation
    # 40 lines of notification

# After
def process_order(order):
    validate_order(order)
    total = calculate_order_total(order)
    send_order_notifications(order, total)
```

**類別分解**
- 將職責提取至獨立的類別
- 為相依性建立介面
- 實作相依性注入
- 使用組合而非繼承

**模式應用**
- 工廠模式用於物件建立
- 策略模式用於演算法變體
- 觀察者模式用於事件處理
- 儲存庫模式用於資料存取
- 裝飾者模式用於擴展行為

### 3. SOLID 原則實踐

提供應用每個 SOLID 原則的具體範例：

**單一職責原則 (SRP)**
```python
# BEFORE: Multiple responsibilities in one class
class UserManager:
    def create_user(self, data):
        # Validate data
        # Save to database
        # Send welcome email
        # Log activity
        # Update cache
        pass

# AFTER: Each class has one responsibility
class UserValidator:
    def validate(self, data): pass

class UserRepository:
    def save(self, user): pass

class EmailService:
    def send_welcome_email(self, user): pass

class UserActivityLogger:
    def log_creation(self, user): pass

class UserService:
    def __init__(self, validator, repository, email_service, logger):
        self.validator = validator
        self.repository = repository
        self.email_service = email_service
        self.logger = logger

    def create_user(self, data):
        self.validator.validate(data)
        user = self.repository.save(data)
        self.email_service.send_welcome_email(user)
        self.logger.log_creation(user)
        return user
```

**開放封閉原則 (OCP)**
```python
# BEFORE: Modification required for new discount types
class DiscountCalculator:
    def calculate(self, order, discount_type):
        if discount_type == "percentage":
            return order.total * 0.1
        elif discount_type == "fixed":
            return 10
        elif discount_type == "tiered":
            # More logic
            pass

# AFTER: Open for extension, closed for modification
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, order): pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage):
        self.percentage = percentage

    def calculate(self, order):
        return order.total * self.percentage

class FixedDiscount(DiscountStrategy):
    def __init__(self, amount):
        self.amount = amount

    def calculate(self, order):
        return self.amount

class TieredDiscount(DiscountStrategy):
    def calculate(self, order):
        if order.total > 1000: return order.total * 0.15
        if order.total > 500: return order.total * 0.10
        return order.total * 0.05

class DiscountCalculator:
    def calculate(self, order, strategy: DiscountStrategy):
        return strategy.calculate(order)
```

**里氏替換原則 (LSP)**
```typescript
// BEFORE: Violates LSP - Square changes Rectangle behavior
class Rectangle {
    constructor(protected width: number, protected height: number) {}

    setWidth(width: number) { this.width = width; }
    setHeight(height: number) { this.height = height; }
    area(): number { return this.width * this.height; }
}

class Square extends Rectangle {
    setWidth(width: number) {
        this.width = width;
        this.height = width; // Breaks LSP
    }
    setHeight(height: number) {
        this.width = height;
        this.height = height; // Breaks LSP
    }
}

// AFTER: Proper abstraction respects LSP
interface Shape {
    area(): number;
}

class Rectangle implements Shape {
    constructor(private width: number, private height: number) {}
    area(): number { return this.width * this.height; }
}

class Square implements Shape {
    constructor(private side: number) {}
    area(): number { return this.side * this.side; }
}
```

**介面隔離原則 (ISP)**
```java
// BEFORE: Fat interface forces unnecessary implementations
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    public void work() { /* work */ }
    public void eat() { /* robots don't eat! */ }
    public void sleep() { /* robots don't sleep! */ }
}

// AFTER: Segregated interfaces
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

class Human implements Workable, Eatable, Sleepable {
    public void work() { /* work */ }
    public void eat() { /* eat */ }
    public void sleep() { /* sleep */ }
}

class Robot implements Workable {
    public void work() { /* work */ }
}
```

**相依反轉原則 (DIP)**
```go
// BEFORE: High-level module depends on low-level module
type MySQLDatabase struct{}

func (db *MySQLDatabase) Save(data string) {}

type UserService struct {
    db *MySQLDatabase // Tight coupling
}

func (s *UserService) CreateUser(name string) {
    s.db.Save(name)
}

// AFTER: Both depend on abstraction
type Database interface {
    Save(data string)
}

type MySQLDatabase struct{}
func (db *MySQLDatabase) Save(data string) {}

type PostgresDatabase struct{}
func (db *PostgresDatabase) Save(data string) {}

type UserService struct {
    db Database // Depends on abstraction
}

func NewUserService(db Database) *UserService {
    return &UserService{db: db}
}

func (s *UserService) CreateUser(name string) {
    s.db.Save(name)
}
```

### 4. 完整重構情境

**情境 1：遺留單體架構轉換為整潔模組化架構**

```python
# BEFORE: 500-line monolithic file
class OrderSystem:
    def process_order(self, order_data):
        # Validation (100 lines)
        if not order_data.get('customer_id'):
            return {'error': 'No customer'}
        if not order_data.get('items'):
            return {'error': 'No items'}
        # Database operations mixed in (150 lines)
        conn = mysql.connector.connect(host='localhost', user='root')
        cursor = conn.cursor()
        cursor.execute("INSERT INTO orders...")
        # Business logic (100 lines)
        total = 0
        for item in order_data['items']:
            total += item['price'] * item['quantity']
        # Email notifications (80 lines)
        smtp = smtplib.SMTP('smtp.gmail.com')
        smtp.sendmail(...)
        # Logging and analytics (70 lines)
        log_file = open('/var/log/orders.log', 'a')
        log_file.write(f"Order processed: {order_data}")

# AFTER: Clean, modular architecture
# domain/entities.py
from dataclasses import dataclass
from typing import List
from decimal import Decimal

@dataclass
class OrderItem:
    product_id: str
    quantity: int
    price: Decimal

@dataclass
class Order:
    customer_id: str
    items: List[OrderItem]

    @property
    def total(self) -> Decimal:
        return sum(item.price * item.quantity for item in self.items)

# domain/repositories.py
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> str: pass

    @abstractmethod
    def find_by_id(self, order_id: str) -> Order: pass

# infrastructure/mysql_order_repository.py
class MySQLOrderRepository(OrderRepository):
    def __init__(self, connection_pool):
        self.pool = connection_pool

    def save(self, order: Order) -> str:
        with self.pool.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(
                "INSERT INTO orders (customer_id, total) VALUES (%s, %s)",
                (order.customer_id, order.total)
            )
            return cursor.lastrowid

# application/validators.py
class OrderValidator:
    def validate(self, order: Order) -> None:
        if not order.customer_id:
            raise ValueError("Customer ID is required")
        if not order.items:
            raise ValueError("Order must contain items")
        if order.total <= 0:
            raise ValueError("Order total must be positive")

# application/services.py
class OrderService:
    def __init__(
        self,
        validator: OrderValidator,
        repository: OrderRepository,
        email_service: EmailService,
        logger: Logger
    ):
        self.validator = validator
        self.repository = repository
        self.email_service = email_service
        self.logger = logger

    def process_order(self, order: Order) -> str:
        self.validator.validate(order)
        order_id = self.repository.save(order)
        self.email_service.send_confirmation(order)
        self.logger.info(f"Order {order_id} processed successfully")
        return order_id
```

**情境 2：程式碼異味解決目錄**

```typescript
// SMELL: Long Parameter List
// BEFORE
function createUser(
    firstName: string,
    lastName: string,
    email: string,
    phone: string,
    address: string,
    city: string,
    state: string,
    zipCode: string
) {}

// AFTER: Parameter Object
interface UserData {
    firstName: string;
    lastName: string;
    email: string;
    phone: string;
    address: Address;
}

interface Address {
    street: string;
    city: string;
    state: string;
    zipCode: string;
}

function createUser(userData: UserData) {}

// SMELL: Feature Envy (method uses another class's data more than its own)
// BEFORE
class Order {
    calculateShipping(customer: Customer): number {
        if (customer.isPremium) {
            return customer.address.isInternational ? 0 : 5;
        }
        return customer.address.isInternational ? 20 : 10;
    }
}

// AFTER: Move method to the class it envies
class Customer {
    calculateShippingCost(): number {
        if (this.isPremium) {
            return this.address.isInternational ? 0 : 5;
        }
        return this.address.isInternational ? 20 : 10;
    }
}

class Order {
    calculateShipping(customer: Customer): number {
        return customer.calculateShippingCost();
    }
}

// SMELL: Primitive Obsession
// BEFORE
function validateEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

let userEmail: string = "test@example.com";

// AFTER: Value Object
class Email {
    private readonly value: string;

    constructor(email: string) {
        if (!this.isValid(email)) {
            throw new Error("Invalid email format");
        }
        this.value = email;
    }

    private isValid(email: string): boolean {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }

    toString(): string {
        return this.value;
    }
}

let userEmail = new Email("test@example.com"); // Validation automatic
```

### 5. 決策框架

**程式碼品質指標解讀矩陣**

| 指標 | 良好 | 警告 | 嚴重 | 行動 |
|--------|------|---------|----------|--------|
| 循環複雜度 | <10 | 10-15 | >15 | 拆分為更小的方法 |
| 方法行數 | <20 | 20-50 | >50 | 提取方法，應用 SRP |
| 類別行數 | <200 | 200-500 | >500 | 分解為多個類別 |
| 測試覆蓋率 | >80% | 60-80% | <60% | 立即新增單元測試 |
| 程式碼重複率 | <3% | 3-5% | >5% | 提取共用程式碼 |
| 註解比例 | 10-30% | <10% or >50% | N/A | 改善命名或減少雜訊 |
| 相依性數量 | <5 | 5-10 | >10 | 應用 DIP，使用門面 |

**重構投資報酬率分析**

```
優先順序 = (業務價值 × 技術債) / (工作量 × 風險)

業務價值 (1-10):
- 關鍵路徑程式碼: 10
- 頻繁變更: 8
- 使用者面向功能: 7
- 內部工具: 5
- 遺留未使用: 2

技術債 (1-10):
- 造成正式環境錯誤: 10
- 阻礙新功能: 8
- 難以測試: 6
- 僅樣式問題: 2

工作量 (小時):
- 重新命名變數: 1-2
- 提取方法: 2-4
- 重構類別: 4-8
- 架構變更: 40+

風險 (1-10):
- 無測試，高耦合: 10
- 部分測試，中等耦合: 5
- 完整測試，低耦合: 2
```

**技術債優先順序決策樹**

```
是否造成正式環境錯誤？
├─ 是 → 優先順序: 嚴重（立即修復）
└─ 否 → 是否阻礙新功能？
    ├─ 是 → 優先順序: 高（本衝刺排程）
    └─ 否 → 是否頻繁修改？
        ├─ 是 → 優先順序: 中（下季度）
        └─ 否 → 程式碼覆蓋率 < 60%？
            ├─ 是 → 優先順序: 中（新增測試）
            └─ 否 → 優先順序: 低（待辦清單）
```

### 6. 現代程式碼品質實踐 (2024-2025)

**AI 輔助程式碼審查整合**

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # GitHub Copilot Autofix
      - uses: github/copilot-autofix@v1
        with:
          languages: 'python,typescript,go'

      # CodeRabbit AI Review
      - uses: coderabbitai/action@v1
        with:
          review_type: 'comprehensive'
          focus: 'security,performance,maintainability'

      # Codium AI PR-Agent
      - uses: codiumai/pr-agent@v1
        with:
          commands: '/review --pr_reviewer.num_code_suggestions=5'
```

**靜態分析工具鏈**

```python
# pyproject.toml
[tool.ruff]
line-length = 100
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "C90", # mccabe complexity
    "N",   # pep8-naming
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
    "A",   # flake8-builtins
    "C4",  # flake8-comprehensions
    "SIM", # flake8-simplify
    "RET", # flake8-return
]

[tool.mypy]
strict = true
warn_unreachable = true
warn_unused_ignores = true

[tool.coverage]
fail_under = 80
```

```javascript
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended-type-checked",
    "plugin:sonarjs/recommended",
    "plugin:security/recommended"
  ],
  "plugins": ["sonarjs", "security", "no-loops"],
  "rules": {
    "complexity": ["error", 10],
    "max-lines-per-function": ["error", 20],
    "max-params": ["error", 3],
    "no-loops/no-loops": "warn",
    "sonarjs/cognitive-complexity": ["error", 15]
  }
}
```

**自動化重構建議**

```python
# Use Sourcery for automatic refactoring suggestions
# sourcery.yaml
rules:
  - id: convert-to-list-comprehension
  - id: merge-duplicate-blocks
  - id: use-named-expression
  - id: inline-immediately-returned-variable

# Example: Sourcery will suggest
# BEFORE
result = []
for item in items:
    if item.is_active:
        result.append(item.name)

# AFTER (auto-suggested)
result = [item.name for item in items if item.is_active]
```

**程式碼品質儀表板設定**

```yaml
# sonar-project.properties
sonar.projectKey=my-project
sonar.sources=src
sonar.tests=tests
sonar.coverage.exclusions=**/*_test.py,**/test_*.py
sonar.python.coverage.reportPaths=coverage.xml

# Quality Gates
sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300

# Thresholds
sonar.coverage.threshold=80
sonar.duplications.threshold=3
sonar.maintainability.rating=A
sonar.reliability.rating=A
sonar.security.rating=A
```

**安全導向重構**

```python
# Use Semgrep for security-aware refactoring
# .semgrep.yml
rules:
  - id: sql-injection-risk
    pattern: execute($QUERY)
    message: Potential SQL injection
    severity: ERROR
    fix: Use parameterized queries

  - id: hardcoded-secrets
    pattern: password = "..."
    message: Hardcoded password detected
    severity: ERROR
    fix: Use environment variables or secret manager

# CodeQL security analysis
# .github/workflows/codeql.yml
- uses: github/codeql-action/analyze@v3
  with:
    category: "/language:python"
    queries: security-extended,security-and-quality
```

### 7. 重構實作

提供完整的重構程式碼，包含：

**整潔程式碼原則**
- 有意義的名稱（可搜尋、可發音、無縮寫）
- 函式只做好一件事
- 無副作用
- 一致的抽象層級
- DRY（不要重複自己）
- YAGNI（你不會需要它）

**錯誤處理**
```python
# 使用特定的例外
class OrderValidationError(Exception):
    pass

class InsufficientInventoryError(Exception):
    pass

# 盡早失敗並提供清楚的訊息
def validate_order(order):
    if not order.items:
        raise OrderValidationError("Order must contain at least one item")

    for item in order.items:
        if item.quantity <= 0:
            raise OrderValidationError(f"Invalid quantity for {item.name}")
```

**文件**
```python
def calculate_discount(order: Order, customer: Customer) -> Decimal:
    """
    根據客戶等級和訂單金額計算訂單的總折扣。

    Args:
        order: 要計算折扣的訂單
        customer: 下訂單的客戶

    Returns:
        折扣金額（Decimal 型別）

    Raises:
        ValueError: 若訂單總額為負數
    """
```

### 8. 測試策略

為重構的程式碼產生完整測試：

**單元測試**
```python
class TestOrderProcessor:
    def test_validate_order_empty_items(self):
        order = Order(items=[])
        with pytest.raises(OrderValidationError):
            validate_order(order)

    def test_calculate_discount_vip_customer(self):
        order = create_test_order(total=1000)
        customer = Customer(tier="VIP")
        discount = calculate_discount(order, customer)
        assert discount == Decimal("100.00")  # 10% VIP discount
```

**測試覆蓋率**
- 所有公開方法都經過測試
- 涵蓋邊界情況
- 驗證錯誤條件
- 包含效能基準測試

### 9. 前後比較

提供清楚的比較以顯示改進：

**指標**
- 循環複雜度降低
- 每個方法的行數
- 測試覆蓋率增加
- 效能改進

**範例**
```
前：
- processData(): 150 行，複雜度: 25
- 0% 測試覆蓋率
- 混合 3 個職責

後：
- validateInput(): 20 行，複雜度: 4
- transformData(): 25 行，複雜度: 5
- saveResults(): 15 行，複雜度: 3
- 95% 測試覆蓋率
- 清楚的關注點分離
```

### 10. 遷移指南

若引入破壞性變更：

**逐步遷移**
1. 安裝新的相依性
2. 更新 import 陳述式
3. 替換已棄用的方法
4. 執行遷移腳本
5. 執行測試套件

**向下相容性**
```python
# 平順遷移的暫時適配器
class LegacyOrderProcessor:
    def __init__(self):
        self.processor = OrderProcessor()

    def process(self, order_data):
        # 轉換遺留格式
        order = Order.from_legacy(order_data)
        return self.processor.process(order)
```

### 11. 效能最佳化

包含特定的最佳化：

**演算法改進**
```python
# Before: O(n²)
for item in items:
    for other in items:
        if item.id == other.id:
            # process

# After: O(n)
item_map = {item.id: item for item in items}
for item_id, item in item_map.items():
    # process
```

**快取策略**
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def calculate_expensive_metric(data_id: str) -> float:
    # 昂貴的計算會被快取
    return result
```

### 12. 程式碼品質檢查清單

確保重構的程式碼符合以下標準：

- [ ] 所有方法 < 20 行
- [ ] 所有類別 < 200 行
- [ ] 無方法有 > 3 個參數
- [ ] 循環複雜度 < 10
- [ ] 無巢狀迴圈 > 2 層
- [ ] 所有名稱都具描述性
- [ ] 無被註解掉的程式碼
- [ ] 格式一致
- [ ] 已新增型別提示 (Python/TypeScript)
- [ ] 完整的錯誤處理
- [ ] 已新增除錯日誌
- [ ] 包含效能指標
- [ ] 文件完整
- [ ] 測試達成 > 80% 覆蓋率
- [ ] 無安全漏洞
- [ ] AI 程式碼審查通過
- [ ] 靜態分析乾淨 (SonarQube/CodeQL)
- [ ] 無硬編碼的秘密

## 嚴重程度

評估發現的問題和改進：

**嚴重**: 安全漏洞、資料損毀風險、記憶體洩漏
**高**: 效能瓶頸、可維護性阻礙、缺少測試
**中**: 程式碼異味、小幅效能問題、不完整的文件
**低**: 樣式不一致、小幅命名問題、可有可無的功能

## 輸出格式

1. **分析摘要**: 發現的關鍵問題及其影響
2. **重構計畫**: 附工作量估計的優先順序變更清單
3. **重構程式碼**: 完整實作，包含行內註解說明變更
4. **測試套件**: 所有重構元件的完整測試
5. **遷移指南**: 採用變更的逐步說明
6. **指標報告**: 程式碼品質指標的前後比較
7. **AI 審查結果**: 自動化程式碼審查發現摘要
8. **品質儀表板**: 連結至 SonarQube/CodeQL 結果

專注於提供實用、漸進式的改進，可以立即採用，同時維持系統穩定性。
