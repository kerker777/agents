---
name: api-design-principles
description: 精通 REST 和 GraphQL API 設計原則，打造直覺、可擴展且易於維護的 API，讓開發者愛不釋手。適用於設計新 API、審查 API 規格或建立 API 設計標準時使用。
---

# API 設計原則

精通 REST 和 GraphQL API 設計原則，打造直覺、可擴展且易於維護的 API，讓開發者愛不釋手，並經得起時間的考驗。

## 何時使用此技能

- 設計新的 REST 或 GraphQL API
- 重構現有 API 以提升可用性
- 為團隊建立 API 設計標準
- 在實作前審查 API 規格
- 在不同 API 範式間遷移（REST 到 GraphQL 等）
- 建立對開發者友善的 API 文件
- 針對特定使用場景優化 API（行動裝置、第三方整合）

## 核心概念

### 1. RESTful 設計原則

**資源導向架構**
- 資源應為名詞（users、orders、products），而非動詞
- 使用 HTTP 方法來表示動作（GET、POST、PUT、PATCH、DELETE）
- URL 代表資源階層
- 一致的命名慣例

**HTTP 方法語意：**
- `GET`：取得資源（冪等、安全）
- `POST`：建立新資源
- `PUT`：取代整個資源（冪等）
- `PATCH`：部分更新資源
- `DELETE`：移除資源（冪等）

### 2. GraphQL 設計原則

**Schema 優先開發**
- 型別定義你的領域模型
- 使用 Query 讀取資料
- 使用 Mutation 修改資料
- 使用 Subscription 進行即時更新

**Query 結構：**
- 客戶端只請求所需的資料
- 單一端點，多重操作
- 強型別 schema
- 內建自省功能

### 3. API 版本控制策略

**URL 版本控制：**
```
/api/v1/users
/api/v2/users
```

**Header 版本控制：**
```
Accept: application/vnd.api+json; version=1
```

**Query 參數版本控制：**
```
/api/users?version=1
```

## REST API 設計模式

### 模式 1：資源集合設計

```python
# 良好：資源導向端點
GET    /api/users              # 列出使用者（含分頁）
POST   /api/users              # 建立使用者
GET    /api/users/{id}         # 取得特定使用者
PUT    /api/users/{id}         # 取代使用者
PATCH  /api/users/{id}         # 更新使用者欄位
DELETE /api/users/{id}         # 刪除使用者

# 巢狀資源
GET    /api/users/{id}/orders  # 取得使用者的訂單
POST   /api/users/{id}/orders  # 為使用者建立訂單

# 不良：動作導向端點（應避免）
POST   /api/createUser
POST   /api/getUserById
POST   /api/deleteUser
```

### 模式 2：分頁與篩選

```python
from typing import List, Optional
from pydantic import BaseModel, Field

class PaginationParams(BaseModel):
    page: int = Field(1, ge=1, description="Page number")
    page_size: int = Field(20, ge=1, le=100, description="Items per page")

class FilterParams(BaseModel):
    status: Optional[str] = None
    created_after: Optional[str] = None
    search: Optional[str] = None

class PaginatedResponse(BaseModel):
    items: List[dict]
    total: int
    page: int
    page_size: int
    pages: int

    @property
    def has_next(self) -> bool:
        return self.page < self.pages

    @property
    def has_prev(self) -> bool:
        return self.page > 1

# FastAPI endpoint example
from fastapi import FastAPI, Query, Depends

app = FastAPI()

@app.get("/api/users", response_model=PaginatedResponse)
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    status: Optional[str] = Query(None),
    search: Optional[str] = Query(None)
):
    # Apply filters
    query = build_query(status=status, search=search)

    # Count total
    total = await count_users(query)

    # Fetch page
    offset = (page - 1) * page_size
    users = await fetch_users(query, limit=page_size, offset=offset)

    return PaginatedResponse(
        items=users,
        total=total,
        page=page,
        page_size=page_size,
        pages=(total + page_size - 1) // page_size
    )
```

### 模式 3：錯誤處理與狀態碼

```python
from fastapi import HTTPException, status
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    error: str
    message: str
    details: Optional[dict] = None
    timestamp: str
    path: str

class ValidationErrorDetail(BaseModel):
    field: str
    message: str
    value: Any

# Consistent error responses
STATUS_CODES = {
    "success": 200,
    "created": 201,
    "no_content": 204,
    "bad_request": 400,
    "unauthorized": 401,
    "forbidden": 403,
    "not_found": 404,
    "conflict": 409,
    "unprocessable": 422,
    "internal_error": 500
}

def raise_not_found(resource: str, id: str):
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail={
            "error": "NotFound",
            "message": f"{resource} not found",
            "details": {"id": id}
        }
    )

def raise_validation_error(errors: List[ValidationErrorDetail]):
    raise HTTPException(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        detail={
            "error": "ValidationError",
            "message": "Request validation failed",
            "details": {"errors": [e.dict() for e in errors]}
        }
    )

# Example usage
@app.get("/api/users/{user_id}")
async def get_user(user_id: str):
    user = await fetch_user(user_id)
    if not user:
        raise_not_found("User", user_id)
    return user
```

### 模式 4：HATEOAS（Hypermedia as the Engine of Application State）

```python
class UserResponse(BaseModel):
    id: str
    name: str
    email: str
    _links: dict

    @classmethod
    def from_user(cls, user: User, base_url: str):
        return cls(
            id=user.id,
            name=user.name,
            email=user.email,
            _links={
                "self": {"href": f"{base_url}/api/users/{user.id}"},
                "orders": {"href": f"{base_url}/api/users/{user.id}/orders"},
                "update": {
                    "href": f"{base_url}/api/users/{user.id}",
                    "method": "PATCH"
                },
                "delete": {
                    "href": f"{base_url}/api/users/{user.id}",
                    "method": "DELETE"
                }
            }
        )
```

## GraphQL 設計模式

### 模式 1：Schema 設計

```graphql
# schema.graphql

# Clear type definitions
type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!

  # Relationships
  orders(
    first: Int = 20
    after: String
    status: OrderStatus
  ): OrderConnection!

  profile: UserProfile
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Money!
  items: [OrderItem!]!
  createdAt: DateTime!

  # Back-reference
  user: User!
}

# Pagination pattern (Relay-style)
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type OrderEdge {
  node: Order!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Enums for type safety
enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}

# Custom scalars
scalar DateTime
scalar Money

# Query root
type Query {
  user(id: ID!): User
  users(
    first: Int = 20
    after: String
    search: String
  ): UserConnection!

  order(id: ID!): Order
}

# Mutation root
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!

  createOrder(input: CreateOrderInput!): CreateOrderPayload!
}

# Input types for mutations
input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

# Payload types for mutations
type CreateUserPayload {
  user: User
  errors: [Error!]
}

type Error {
  field: String
  message: String!
}
```

### 模式 2：Resolver 設計

```python
from typing import Optional, List
from ariadne import QueryType, MutationType, ObjectType
from dataclasses import dataclass

query = QueryType()
mutation = MutationType()
user_type = ObjectType("User")

@query.field("user")
async def resolve_user(obj, info, id: str) -> Optional[dict]:
    """Resolve single user by ID."""
    return await fetch_user_by_id(id)

@query.field("users")
async def resolve_users(
    obj,
    info,
    first: int = 20,
    after: Optional[str] = None,
    search: Optional[str] = None
) -> dict:
    """Resolve paginated user list."""
    # Decode cursor
    offset = decode_cursor(after) if after else 0

    # Fetch users
    users = await fetch_users(
        limit=first + 1,  # Fetch one extra to check hasNextPage
        offset=offset,
        search=search
    )

    # Pagination
    has_next = len(users) > first
    if has_next:
        users = users[:first]

    edges = [
        {
            "node": user,
            "cursor": encode_cursor(offset + i)
        }
        for i, user in enumerate(users)
    ]

    return {
        "edges": edges,
        "pageInfo": {
            "hasNextPage": has_next,
            "hasPreviousPage": offset > 0,
            "startCursor": edges[0]["cursor"] if edges else None,
            "endCursor": edges[-1]["cursor"] if edges else None
        },
        "totalCount": await count_users(search=search)
    }

@user_type.field("orders")
async def resolve_user_orders(user: dict, info, first: int = 20) -> dict:
    """Resolve user's orders (N+1 prevention with DataLoader)."""
    # Use DataLoader to batch requests
    loader = info.context["loaders"]["orders_by_user"]
    orders = await loader.load(user["id"])

    return paginate_orders(orders, first)

@mutation.field("createUser")
async def resolve_create_user(obj, info, input: dict) -> dict:
    """Create new user."""
    try:
        # Validate input
        validate_user_input(input)

        # Create user
        user = await create_user(
            email=input["email"],
            name=input["name"],
            password=hash_password(input["password"])
        )

        return {
            "user": user,
            "errors": []
        }
    except ValidationError as e:
        return {
            "user": None,
            "errors": [{"field": e.field, "message": e.message}]
        }
```

### 模式 3：DataLoader（預防 N+1 問題）

```python
from aiodataloader import DataLoader
from typing import List, Optional

class UserLoader(DataLoader):
    """Batch load users by ID."""

    async def batch_load_fn(self, user_ids: List[str]) -> List[Optional[dict]]:
        """Load multiple users in single query."""
        users = await fetch_users_by_ids(user_ids)

        # Map results back to input order
        user_map = {user["id"]: user for user in users}
        return [user_map.get(user_id) for user_id in user_ids]

class OrdersByUserLoader(DataLoader):
    """Batch load orders by user ID."""

    async def batch_load_fn(self, user_ids: List[str]) -> List[List[dict]]:
        """Load orders for multiple users in single query."""
        orders = await fetch_orders_by_user_ids(user_ids)

        # Group orders by user_id
        orders_by_user = {}
        for order in orders:
            user_id = order["user_id"]
            if user_id not in orders_by_user:
                orders_by_user[user_id] = []
            orders_by_user[user_id].append(order)

        # Return in input order
        return [orders_by_user.get(user_id, []) for user_id in user_ids]

# Context setup
def create_context():
    return {
        "loaders": {
            "user": UserLoader(),
            "orders_by_user": OrdersByUserLoader()
        }
    }
```

## 最佳實務

### REST APIs
1. **一致的命名**：集合使用複數名詞（`/users`，而非 `/user`）
2. **無狀態**：每個請求都包含所有必要資訊
3. **正確使用 HTTP 狀態碼**：2xx 成功、4xx 客戶端錯誤、5xx 伺服器錯誤
4. **為 API 加上版本**：從第一天就規劃好重大變更
5. **分頁**：大型集合務必使用分頁
6. **速率限制**：使用速率限制保護你的 API
7. **文件**：使用 OpenAPI/Swagger 建立互動式文件

### GraphQL APIs
1. **Schema 優先**：在撰寫 resolver 前先設計 schema
2. **避免 N+1**：使用 DataLoader 進行高效的資料擷取
3. **輸入驗證**：在 schema 和 resolver 層級進行驗證
4. **錯誤處理**：在 mutation payload 中回傳結構化錯誤
5. **分頁**：使用游標式分頁（Relay 規範）
6. **棄用**：使用 `@deprecated` 指令進行漸進式遷移
7. **監控**：追蹤查詢複雜度和執行時間

## 常見陷阱

- **過度擷取/擷取不足（REST）**：GraphQL 可解決此問題，但需要 DataLoader
- **重大變更**：為 API 加上版本或使用棄用策略
- **不一致的錯誤格式**：標準化錯誤回應
- **缺少速率限制**：沒有限制的 API 容易遭到濫用
- **文件不足**：缺乏文件的 API 會讓開發者感到挫折
- **忽略 HTTP 語意**：對冪等操作使用 POST 會違反預期
- **緊密耦合**：API 結構不應直接對應資料庫 schema

## 資源

- **references/rest-best-practices.md**：完整的 REST API 設計指南
- **references/graphql-schema-design.md**：GraphQL schema 模式與反模式
- **references/api-versioning-strategies.md**：版本控制方法與遷移路徑
- **assets/rest-api-template.py**：FastAPI REST API 範本
- **assets/graphql-schema-template.graphql**：完整的 GraphQL schema 範例
- **assets/api-design-checklist.md**：實作前審查檢查清單
- **scripts/openapi-generator.py**：從程式碼產生 OpenAPI 規格
