# REST API 最佳實踐

## URL 結構

### 資源命名
```
# 良好做法 - 使用複數名詞
GET /api/users
GET /api/orders
GET /api/products

# 不良做法 - 使用動詞或混合慣例
GET /api/getUser
GET /api/user  (不一致的單數形式)
POST /api/createOrder
```

### 巢狀資源
```
# 淺層巢狀（建議使用）
GET /api/users/{id}/orders
GET /api/orders/{id}

# 深層巢狀（應避免）
GET /api/users/{id}/orders/{orderId}/items/{itemId}/reviews
# 更好的做法：
GET /api/order-items/{id}/reviews
```

## HTTP 方法與狀態碼

### GET - 取得資源
```
GET /api/users              → 200 OK (包含清單)
GET /api/users/{id}         → 200 OK 或 404 Not Found
GET /api/users?page=2       → 200 OK (分頁)
```

### POST - 建立資源
```
POST /api/users
  Body: {"name": "John", "email": "john@example.com"}
  → 201 Created
  Location: /api/users/123
  Body: {"id": "123", "name": "John", ...}

POST /api/users (驗證錯誤)
  → 422 Unprocessable Entity
  Body: {"errors": [...]}
```

### PUT - 替換資源
```
PUT /api/users/{id}
  Body: {完整的使用者物件}
  → 200 OK (已更新)
  → 404 Not Found (不存在)

# 必須包含所有欄位
```

### PATCH - 部分更新
```
PATCH /api/users/{id}
  Body: {"name": "Jane"}  (僅變更的欄位)
  → 200 OK
  → 404 Not Found
```

### DELETE - 移除資源
```
DELETE /api/users/{id}
  → 204 No Content (已刪除)
  → 404 Not Found
  → 409 Conflict (因參考關係無法刪除)
```

## 篩選、排序與搜尋

### 查詢參數
```
# 篩選
GET /api/users?status=active
GET /api/users?role=admin&status=active

# 排序
GET /api/users?sort=created_at
GET /api/users?sort=-created_at  (降冪)
GET /api/users?sort=name,created_at

# 搜尋
GET /api/users?search=john
GET /api/users?q=john

# 欄位選擇（稀疏欄位集）
GET /api/users?fields=id,name,email
```

## 分頁模式

### 基於偏移量的分頁
```python
GET /api/users?page=2&page_size=20

Response:
{
  "items": [...],
  "page": 2,
  "page_size": 20,
  "total": 150,
  "pages": 8
}
```

### 基於遊標的分頁（適用於大型資料集）
```python
GET /api/users?limit=20&cursor=eyJpZCI6MTIzfQ

Response:
{
  "items": [...],
  "next_cursor": "eyJpZCI6MTQzfQ",
  "has_more": true
}
```

### Link Header 分頁（RESTful）
```
GET /api/users?page=2

Response Headers:
Link: <https://api.example.com/users?page=3>; rel="next",
      <https://api.example.com/users?page=1>; rel="prev",
      <https://api.example.com/users?page=1>; rel="first",
      <https://api.example.com/users?page=8>; rel="last"
```

## 版本控制策略

### URL 版本控制（建議使用）
```
/api/v1/users
/api/v2/users

優點：清楚明確，易於路由
缺點：相同資源有多個 URL
```

### Header 版本控制
```
GET /api/users
Accept: application/vnd.api+json; version=2

優點：URL 簡潔
缺點：較不明顯，測試較困難
```

### 查詢參數
```
GET /api/users?version=2

優點：易於測試
缺點：可選參數容易被遺忘
```

## 速率限制

### Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 742
X-RateLimit-Reset: 1640000000

被限制時的回應：
429 Too Many Requests
Retry-After: 3600
```

### 實作模式
```python
from fastapi import HTTPException, Request
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, calls: int, period: int):
        self.calls = calls
        self.period = period
        self.cache = {}

    def check(self, key: str) -> bool:
        now = datetime.now()
        if key not in self.cache:
            self.cache[key] = []

        # Remove old requests
        self.cache[key] = [
            ts for ts in self.cache[key]
            if now - ts < timedelta(seconds=self.period)
        ]

        if len(self.cache[key]) >= self.calls:
            return False

        self.cache[key].append(now)
        return True

limiter = RateLimiter(calls=100, period=60)

@app.get("/api/users")
async def get_users(request: Request):
    if not limiter.check(request.client.host):
        raise HTTPException(
            status_code=429,
            headers={"Retry-After": "60"}
        )
    return {"users": [...]}
```

## 認證與授權

### Bearer Token
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

401 Unauthorized - 缺少或無效的 token
403 Forbidden - Token 有效，但權限不足
```

### API Keys
```
X-API-Key: your-api-key-here
```

## 錯誤回應格式

### 一致的結構
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "value": "not-an-email"
      }
    ],
    "timestamp": "2025-10-16T12:00:00Z",
    "path": "/api/users"
  }
}
```

### 狀態碼指南
- `200 OK`：成功的 GET、PATCH、PUT
- `201 Created`：成功的 POST
- `204 No Content`：成功的 DELETE
- `400 Bad Request`：格式錯誤的請求
- `401 Unauthorized`：需要驗證
- `403 Forbidden`：已驗證但未授權
- `404 Not Found`：資源不存在
- `409 Conflict`：狀態衝突（重複的電子郵件等）
- `422 Unprocessable Entity`：驗證錯誤
- `429 Too Many Requests`：速率限制
- `500 Internal Server Error`：伺服器錯誤
- `503 Service Unavailable`：暫時停機

## 快取

### Cache Headers
```
# 用戶端快取
Cache-Control: public, max-age=3600

# 不快取
Cache-Control: no-cache, no-store, must-revalidate

# 條件式請求
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
→ 304 Not Modified
```

## 批量操作

### 批次端點
```python
POST /api/users/batch
{
  "items": [
    {"name": "User1", "email": "user1@example.com"},
    {"name": "User2", "email": "user2@example.com"}
  ]
}

Response:
{
  "results": [
    {"id": "1", "status": "created"},
    {"id": null, "status": "failed", "error": "Email already exists"}
  ]
}
```

## 冪等性

### 冪等性金鑰
```
POST /api/orders
Idempotency-Key: unique-key-123

如果是重複請求：
→ 200 OK (回傳快取的回應)
```

## CORS 設定

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## 使用 OpenAPI 撰寫文件

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="API for managing users",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

@app.get(
    "/api/users/{user_id}",
    summary="Get user by ID",
    response_description="User details",
    tags=["Users"]
)
async def get_user(
    user_id: str = Path(..., description="The user ID")
):
    """
    Retrieve user by ID.

    Returns full user profile including:
    - Basic information
    - Contact details
    - Account status
    """
    pass
```

## 健康檢查與監控端點

```python
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "version": "1.0.0",
        "timestamp": datetime.now().isoformat()
    }

@app.get("/health/detailed")
async def detailed_health():
    return {
        "status": "healthy",
        "checks": {
            "database": await check_database(),
            "redis": await check_redis(),
            "external_api": await check_external_api()
        }
    }
```
