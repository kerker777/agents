---
name: architecture-patterns
description: 實作經過驗證的後端架構模式，包括 Clean Architecture、Hexagonal Architecture 和 Domain-Driven Design。適用於設計複雜的後端系統或重構既有應用程式以提升可維護性。
---

# 架構模式

掌握經過驗證的後端架構模式，包括 Clean Architecture、Hexagonal Architecture 和 Domain-Driven Design，以建立可維護、可測試且可擴展的系統。

## 何時使用此技能

- 從零開始設計新的後端系統
- 重構單體應用程式以提升可維護性
- 為團隊建立架構標準
- 從緊耦合架構遷移至鬆耦合架構
- 實作領域驅動設計原則
- 建立可測試和可模擬的程式碼庫
- 規劃微服務分解策略

## 核心概念

### 1. Clean Architecture (Uncle Bob)

**分層架構（依賴關係由外向內流動）：**
- **Entities**：核心業務模型
- **Use Cases**：應用程式業務規則
- **Interface Adapters**：控制器、呈現器、閘道器
- **Frameworks & Drivers**：使用者介面、資料庫、外部服務

**核心原則：**
- 依賴關係指向內層
- 內層不知道外層的存在
- 業務邏輯獨立於框架
- 無需使用者介面、資料庫或外部服務即可測試

### 2. Hexagonal Architecture (Ports and Adapters)

**組成元件：**
- **Domain Core**：業務邏輯
- **Ports**：定義互動方式的介面
- **Adapters**：Port 的實作（資料庫、REST、訊息佇列）

**優勢：**
- 輕鬆替換實作（用於測試的模擬）
- 技術中立的核心
- 清楚的關注點分離

### 3. Domain-Driven Design (DDD)

**策略模式：**
- **Bounded Contexts**：為不同領域分離模型
- **Context Mapping**：上下文之間的關聯方式
- **Ubiquitous Language**：共通術語

**戰術模式：**
- **Entities**：具有識別性的物件
- **Value Objects**：由屬性定義的不可變物件
- **Aggregates**：一致性邊界
- **Repositories**：資料存取抽象層
- **Domain Events**：已發生的事件

## Clean Architecture 模式

### 目錄結構
```
app/
├── domain/           # Entities & business rules
│   ├── entities/
│   │   ├── user.py
│   │   └── order.py
│   ├── value_objects/
│   │   ├── email.py
│   │   └── money.py
│   └── interfaces/   # Abstract interfaces
│       ├── user_repository.py
│       └── payment_gateway.py
├── use_cases/        # Application business rules
│   ├── create_user.py
│   ├── process_order.py
│   └── send_notification.py
├── adapters/         # Interface implementations
│   ├── repositories/
│   │   ├── postgres_user_repository.py
│   │   └── redis_cache_repository.py
│   ├── controllers/
│   │   └── user_controller.py
│   └── gateways/
│       ├── stripe_payment_gateway.py
│       └── sendgrid_email_gateway.py
└── infrastructure/   # Framework & external concerns
    ├── database.py
    ├── config.py
    └── logging.py
```

### 實作範例

```python
# domain/entities/user.py
from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class User:
    """Core user entity - no framework dependencies."""
    id: str
    email: str
    name: str
    created_at: datetime
    is_active: bool = True

    def deactivate(self):
        """Business rule: deactivating user."""
        self.is_active = False

    def can_place_order(self) -> bool:
        """Business rule: active users can order."""
        return self.is_active

# domain/interfaces/user_repository.py
from abc import ABC, abstractmethod
from typing import Optional, List
from domain.entities.user import User

class IUserRepository(ABC):
    """Port: defines contract, no implementation."""

    @abstractmethod
    async def find_by_id(self, user_id: str) -> Optional[User]:
        pass

    @abstractmethod
    async def find_by_email(self, email: str) -> Optional[User]:
        pass

    @abstractmethod
    async def save(self, user: User) -> User:
        pass

    @abstractmethod
    async def delete(self, user_id: str) -> bool:
        pass

# use_cases/create_user.py
from domain.entities.user import User
from domain.interfaces.user_repository import IUserRepository
from dataclasses import dataclass
from datetime import datetime
import uuid

@dataclass
class CreateUserRequest:
    email: str
    name: str

@dataclass
class CreateUserResponse:
    user: User
    success: bool
    error: Optional[str] = None

class CreateUserUseCase:
    """Use case: orchestrates business logic."""

    def __init__(self, user_repository: IUserRepository):
        self.user_repository = user_repository

    async def execute(self, request: CreateUserRequest) -> CreateUserResponse:
        # Business validation
        existing = await self.user_repository.find_by_email(request.email)
        if existing:
            return CreateUserResponse(
                user=None,
                success=False,
                error="Email already exists"
            )

        # Create entity
        user = User(
            id=str(uuid.uuid4()),
            email=request.email,
            name=request.name,
            created_at=datetime.now(),
            is_active=True
        )

        # Persist
        saved_user = await self.user_repository.save(user)

        return CreateUserResponse(
            user=saved_user,
            success=True
        )

# adapters/repositories/postgres_user_repository.py
from domain.interfaces.user_repository import IUserRepository
from domain.entities.user import User
from typing import Optional
import asyncpg

class PostgresUserRepository(IUserRepository):
    """Adapter: PostgreSQL implementation."""

    def __init__(self, pool: asyncpg.Pool):
        self.pool = pool

    async def find_by_id(self, user_id: str) -> Optional[User]:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM users WHERE id = $1", user_id
            )
            return self._to_entity(row) if row else None

    async def find_by_email(self, email: str) -> Optional[User]:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM users WHERE email = $1", email
            )
            return self._to_entity(row) if row else None

    async def save(self, user: User) -> User:
        async with self.pool.acquire() as conn:
            await conn.execute(
                """
                INSERT INTO users (id, email, name, created_at, is_active)
                VALUES ($1, $2, $3, $4, $5)
                ON CONFLICT (id) DO UPDATE
                SET email = $2, name = $3, is_active = $5
                """,
                user.id, user.email, user.name, user.created_at, user.is_active
            )
            return user

    async def delete(self, user_id: str) -> bool:
        async with self.pool.acquire() as conn:
            result = await conn.execute(
                "DELETE FROM users WHERE id = $1", user_id
            )
            return result == "DELETE 1"

    def _to_entity(self, row) -> User:
        """Map database row to entity."""
        return User(
            id=row["id"],
            email=row["email"],
            name=row["name"],
            created_at=row["created_at"],
            is_active=row["is_active"]
        )

# adapters/controllers/user_controller.py
from fastapi import APIRouter, Depends, HTTPException
from use_cases.create_user import CreateUserUseCase, CreateUserRequest
from pydantic import BaseModel

router = APIRouter()

class CreateUserDTO(BaseModel):
    email: str
    name: str

@router.post("/users")
async def create_user(
    dto: CreateUserDTO,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case)
):
    """Controller: handles HTTP concerns only."""
    request = CreateUserRequest(email=dto.email, name=dto.name)
    response = await use_case.execute(request)

    if not response.success:
        raise HTTPException(status_code=400, detail=response.error)

    return {"user": response.user}
```

## Hexagonal Architecture 模式

```python
# Core domain (hexagon center)
class OrderService:
    """Domain service - no infrastructure dependencies."""

    def __init__(
        self,
        order_repository: OrderRepositoryPort,
        payment_gateway: PaymentGatewayPort,
        notification_service: NotificationPort
    ):
        self.orders = order_repository
        self.payments = payment_gateway
        self.notifications = notification_service

    async def place_order(self, order: Order) -> OrderResult:
        # Business logic
        if not order.is_valid():
            return OrderResult(success=False, error="Invalid order")

        # Use ports (interfaces)
        payment = await self.payments.charge(
            amount=order.total,
            customer=order.customer_id
        )

        if not payment.success:
            return OrderResult(success=False, error="Payment failed")

        order.mark_as_paid()
        saved_order = await self.orders.save(order)

        await self.notifications.send(
            to=order.customer_email,
            subject="Order confirmed",
            body=f"Order {order.id} confirmed"
        )

        return OrderResult(success=True, order=saved_order)

# Ports (interfaces)
class OrderRepositoryPort(ABC):
    @abstractmethod
    async def save(self, order: Order) -> Order:
        pass

class PaymentGatewayPort(ABC):
    @abstractmethod
    async def charge(self, amount: Money, customer: str) -> PaymentResult:
        pass

class NotificationPort(ABC):
    @abstractmethod
    async def send(self, to: str, subject: str, body: str):
        pass

# Adapters (implementations)
class StripePaymentAdapter(PaymentGatewayPort):
    """Primary adapter: connects to Stripe API."""

    def __init__(self, api_key: str):
        self.stripe = stripe
        self.stripe.api_key = api_key

    async def charge(self, amount: Money, customer: str) -> PaymentResult:
        try:
            charge = self.stripe.Charge.create(
                amount=amount.cents,
                currency=amount.currency,
                customer=customer
            )
            return PaymentResult(success=True, transaction_id=charge.id)
        except stripe.error.CardError as e:
            return PaymentResult(success=False, error=str(e))

class MockPaymentAdapter(PaymentGatewayPort):
    """Test adapter: no external dependencies."""

    async def charge(self, amount: Money, customer: str) -> PaymentResult:
        return PaymentResult(success=True, transaction_id="mock-123")
```

## Domain-Driven Design 模式

```python
# Value Objects (immutable)
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True)
class Email:
    """Value object: validated email."""
    value: str

    def __post_init__(self):
        if "@" not in self.value:
            raise ValueError("Invalid email")

@dataclass(frozen=True)
class Money:
    """Value object: amount with currency."""
    amount: int  # cents
    currency: str

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)

# Entities (with identity)
class Order:
    """Entity: has identity, mutable state."""

    def __init__(self, id: str, customer: Customer):
        self.id = id
        self.customer = customer
        self.items: List[OrderItem] = []
        self.status = OrderStatus.PENDING
        self._events: List[DomainEvent] = []

    def add_item(self, product: Product, quantity: int):
        """Business logic in entity."""
        item = OrderItem(product, quantity)
        self.items.append(item)
        self._events.append(ItemAddedEvent(self.id, item))

    def total(self) -> Money:
        """Calculated property."""
        return sum(item.subtotal() for item in self.items)

    def submit(self):
        """State transition with business rules."""
        if not self.items:
            raise ValueError("Cannot submit empty order")
        if self.status != OrderStatus.PENDING:
            raise ValueError("Order already submitted")

        self.status = OrderStatus.SUBMITTED
        self._events.append(OrderSubmittedEvent(self.id))

# Aggregates (consistency boundary)
class Customer:
    """Aggregate root: controls access to entities."""

    def __init__(self, id: str, email: Email):
        self.id = id
        self.email = email
        self._addresses: List[Address] = []
        self._orders: List[str] = []  # Order IDs, not full objects

    def add_address(self, address: Address):
        """Aggregate enforces invariants."""
        if len(self._addresses) >= 5:
            raise ValueError("Maximum 5 addresses allowed")
        self._addresses.append(address)

    @property
    def primary_address(self) -> Optional[Address]:
        return next((a for a in self._addresses if a.is_primary), None)

# Domain Events
@dataclass
class OrderSubmittedEvent:
    order_id: str
    occurred_at: datetime = field(default_factory=datetime.now)

# Repository (aggregate persistence)
class OrderRepository:
    """Repository: persist/retrieve aggregates."""

    async def find_by_id(self, order_id: str) -> Optional[Order]:
        """Reconstitute aggregate from storage."""
        pass

    async def save(self, order: Order):
        """Persist aggregate and publish events."""
        await self._persist(order)
        await self._publish_events(order._events)
        order._events.clear()
```

## 參考資源

- **references/clean-architecture-guide.md**：詳細的分層說明
- **references/hexagonal-architecture-guide.md**：Ports and Adapters 模式
- **references/ddd-tactical-patterns.md**：Entities、Value Objects、Aggregates
- **assets/clean-architecture-template/**：完整專案結構
- **assets/ddd-examples/**：領域建模範例

## 最佳實務

1. **依賴規則**：依賴關係永遠指向內層
2. **介面隔離**：小而專注的介面
3. **業務邏輯於領域層**：核心層保持框架獨立
4. **測試獨立性**：核心層無需基礎設施即可測試
5. **限界上下文**：明確的領域邊界
6. **共通語言**：一致的術語
7. **精簡控制器**：委派給使用案例
8. **豐富的領域模型**：行為與資料並存

## 常見陷阱

- **貧血領域模型**：實體只有資料，沒有行為
- **框架耦合**：業務邏輯依賴於框架
- **肥大控制器**：業務邏輯放在控制器中
- **儲存庫洩漏**：暴露 ORM 物件
- **缺少抽象層**：核心層使用具體依賴
- **過度工程**：簡單的 CRUD 使用 Clean Architecture
