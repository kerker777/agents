---
name: async-python-patterns
description: 精通 Python asyncio、並行程式設計（concurrent programming）和 async/await 模式，用於建構高效能應用程式。適用於建構非同步 API、並行系統，或需要非阻塞操作的 I/O 密集型應用程式。
---

# Async Python 模式

完整的指南，用於使用 asyncio、並行程式設計模式和 async/await 實作非同步 Python 應用程式，建構高效能、非阻塞的系統。

## 何時使用此技能

- 建構非同步 Web API（FastAPI、aiohttp、Sanic）
- 實作並行 I/O 操作（資料庫、檔案、網路）
- 建立具有並行請求的網路爬蟲
- 開發即時應用程式（WebSocket 伺服器、聊天系統）
- 同時處理多個獨立任務
- 建構具有非同步通訊的微服務
- 最佳化 I/O 密集型工作負載
- 實作非同步背景任務和佇列

## 核心概念

### 1. Event Loop（事件迴圈）
事件迴圈是 asyncio 的核心，負責管理和排程非同步任務。

**主要特性：**
- 單執行緒協作式多工（single-threaded cooperative multitasking）
- 排程協程（coroutines）執行
- 處理 I/O 操作而不阻塞
- 管理回呼（callbacks）和 futures

### 2. Coroutines（協程）
使用 `async def` 定義的函式，可以暫停和恢復執行。

**語法：**
```python
async def my_coroutine():
    result = await some_async_operation()
    return result
```

### 3. Tasks（任務）
在事件迴圈上並行執行的已排程協程。

### 4. Futures
代表非同步操作最終結果的低階物件。

### 5. Async Context Managers（非同步上下文管理器）
支援 `async with` 以進行適當清理的資源。

### 6. Async Iterators（非同步迭代器）
支援 `async for` 以迭代非同步資料來源的物件。

## 快速入門

```python
import asyncio

async def main():
    print("Hello")
    await asyncio.sleep(1)
    print("World")

# Python 3.7+
asyncio.run(main())
```

## 基礎模式

### 模式 1：基本 Async/Await

```python
import asyncio

async def fetch_data(url: str) -> dict:
    """非同步從 URL 取得資料。"""
    await asyncio.sleep(1)  # 模擬 I/O
    return {"url": url, "data": "result"}

async def main():
    result = await fetch_data("https://api.example.com")
    print(result)

asyncio.run(main())
```

### 模式 2：使用 gather() 並行執行

```python
import asyncio
from typing import List

async def fetch_user(user_id: int) -> dict:
    """取得使用者資料。"""
    await asyncio.sleep(0.5)
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_all_users(user_ids: List[int]) -> List[dict]:
    """並行取得多個使用者。"""
    tasks = [fetch_user(uid) for uid in user_ids]
    results = await asyncio.gather(*tasks)
    return results

async def main():
    user_ids = [1, 2, 3, 4, 5]
    users = await fetch_all_users(user_ids)
    print(f"Fetched {len(users)} users")

asyncio.run(main())
```

### 模式 3：任務建立與管理

```python
import asyncio

async def background_task(name: str, delay: int):
    """長時間執行的背景任務。"""
    print(f"{name} started")
    await asyncio.sleep(delay)
    print(f"{name} completed")
    return f"Result from {name}"

async def main():
    # 建立任務
    task1 = asyncio.create_task(background_task("Task 1", 2))
    task2 = asyncio.create_task(background_task("Task 2", 1))

    # 執行其他工作
    print("Main: doing other work")
    await asyncio.sleep(0.5)

    # 等待任務
    result1 = await task1
    result2 = await task2

    print(f"Results: {result1}, {result2}")

asyncio.run(main())
```

### 模式 4：非同步程式碼中的錯誤處理

```python
import asyncio
from typing import List, Optional

async def risky_operation(item_id: int) -> dict:
    """可能會失敗的操作。"""
    await asyncio.sleep(0.1)
    if item_id % 3 == 0:
        raise ValueError(f"Item {item_id} failed")
    return {"id": item_id, "status": "success"}

async def safe_operation(item_id: int) -> Optional[dict]:
    """具有錯誤處理的包裝器。"""
    try:
        return await risky_operation(item_id)
    except ValueError as e:
        print(f"Error: {e}")
        return None

async def process_items(item_ids: List[int]):
    """使用錯誤處理來處理多個項目。"""
    tasks = [safe_operation(iid) for iid in item_ids]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # 過濾掉失敗的項目
    successful = [r for r in results if r is not None and not isinstance(r, Exception)]
    failed = [r for r in results if isinstance(r, Exception)]

    print(f"Success: {len(successful)}, Failed: {len(failed)}")
    return successful

asyncio.run(process_items([1, 2, 3, 4, 5, 6]))
```

### 模式 5：逾時處理

```python
import asyncio

async def slow_operation(delay: int) -> str:
    """需要時間的操作。"""
    await asyncio.sleep(delay)
    return f"Completed after {delay}s"

async def with_timeout():
    """執行帶有逾時的操作。"""
    try:
        result = await asyncio.wait_for(slow_operation(5), timeout=2.0)
        print(result)
    except asyncio.TimeoutError:
        print("Operation timed out")

asyncio.run(with_timeout())
```

## 進階模式

### 模式 6：非同步上下文管理器

```python
import asyncio
from typing import Optional

class AsyncDatabaseConnection:
    """非同步資料庫連線上下文管理器。"""

    def __init__(self, dsn: str):
        self.dsn = dsn
        self.connection: Optional[object] = None

    async def __aenter__(self):
        print("Opening connection")
        await asyncio.sleep(0.1)  # 模擬連線
        self.connection = {"dsn": self.dsn, "connected": True}
        return self.connection

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Closing connection")
        await asyncio.sleep(0.1)  # 模擬清理
        self.connection = None

async def query_database():
    """使用非同步上下文管理器。"""
    async with AsyncDatabaseConnection("postgresql://localhost") as conn:
        print(f"Using connection: {conn}")
        await asyncio.sleep(0.2)  # 模擬查詢
        return {"rows": 10}

asyncio.run(query_database())
```

### 模式 7：非同步迭代器和生成器

```python
import asyncio
from typing import AsyncIterator

async def async_range(start: int, end: int, delay: float = 0.1) -> AsyncIterator[int]:
    """非同步生成器，以延遲方式產生數字。"""
    for i in range(start, end):
        await asyncio.sleep(delay)
        yield i

async def fetch_pages(url: str, max_pages: int) -> AsyncIterator[dict]:
    """非同步取得分頁資料。"""
    for page in range(1, max_pages + 1):
        await asyncio.sleep(0.2)  # 模擬 API 呼叫
        yield {
            "page": page,
            "url": f"{url}?page={page}",
            "data": [f"item_{page}_{i}" for i in range(5)]
        }

async def consume_async_iterator():
    """消費非同步迭代器。"""
    async for number in async_range(1, 5):
        print(f"Number: {number}")

    print("\nFetching pages:")
    async for page_data in fetch_pages("https://api.example.com/items", 3):
        print(f"Page {page_data['page']}: {len(page_data['data'])} items")

asyncio.run(consume_async_iterator())
```

### 模式 8：生產者-消費者模式

```python
import asyncio
from asyncio import Queue
from typing import Optional

async def producer(queue: Queue, producer_id: int, num_items: int):
    """生產項目並放入佇列。"""
    for i in range(num_items):
        item = f"Item-{producer_id}-{i}"
        await queue.put(item)
        print(f"Producer {producer_id} produced: {item}")
        await asyncio.sleep(0.1)
    await queue.put(None)  # 發出完成訊號

async def consumer(queue: Queue, consumer_id: int):
    """從佇列消費項目。"""
    while True:
        item = await queue.get()
        if item is None:
            queue.task_done()
            break

        print(f"Consumer {consumer_id} processing: {item}")
        await asyncio.sleep(0.2)  # 模擬工作
        queue.task_done()

async def producer_consumer_example():
    """執行生產者-消費者模式。"""
    queue = Queue(maxsize=10)

    # 建立任務
    producers = [
        asyncio.create_task(producer(queue, i, 5))
        for i in range(2)
    ]

    consumers = [
        asyncio.create_task(consumer(queue, i))
        for i in range(3)
    ]

    # 等待生產者
    await asyncio.gather(*producers)

    # 等待佇列清空
    await queue.join()

    # 取消消費者
    for c in consumers:
        c.cancel()

asyncio.run(producer_consumer_example())
```

### 模式 9：使用 Semaphore 進行速率限制

```python
import asyncio
from typing import List

async def api_call(url: str, semaphore: asyncio.Semaphore) -> dict:
    """使用速率限制進行 API 呼叫。"""
    async with semaphore:
        print(f"Calling {url}")
        await asyncio.sleep(0.5)  # 模擬 API 呼叫
        return {"url": url, "status": 200}

async def rate_limited_requests(urls: List[str], max_concurrent: int = 5):
    """使用速率限制進行多個請求。"""
    semaphore = asyncio.Semaphore(max_concurrent)
    tasks = [api_call(url, semaphore) for url in urls]
    results = await asyncio.gather(*tasks)
    return results

async def main():
    urls = [f"https://api.example.com/item/{i}" for i in range(20)]
    results = await rate_limited_requests(urls, max_concurrent=3)
    print(f"Completed {len(results)} requests")

asyncio.run(main())
```

### 模式 10：非同步鎖和同步化

```python
import asyncio

class AsyncCounter:
    """執行緒安全的非同步計數器。"""

    def __init__(self):
        self.value = 0
        self.lock = asyncio.Lock()

    async def increment(self):
        """安全地遞增計數器。"""
        async with self.lock:
            current = self.value
            await asyncio.sleep(0.01)  # 模擬工作
            self.value = current + 1

    async def get_value(self) -> int:
        """取得目前值。"""
        async with self.lock:
            return self.value

async def worker(counter: AsyncCounter, worker_id: int):
    """遞增計數器的 worker。"""
    for _ in range(10):
        await counter.increment()
        print(f"Worker {worker_id} incremented")

async def test_counter():
    """測試並行計數器。"""
    counter = AsyncCounter()

    workers = [asyncio.create_task(worker(counter, i)) for i in range(5)]
    await asyncio.gather(*workers)

    final_value = await counter.get_value()
    print(f"Final counter value: {final_value}")

asyncio.run(test_counter())
```

## 實際應用

### 使用 aiohttp 進行網路爬蟲

```python
import asyncio
import aiohttp
from typing import List, Dict

async def fetch_url(session: aiohttp.ClientSession, url: str) -> Dict:
    """取得單一 URL。"""
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as response:
            text = await response.text()
            return {
                "url": url,
                "status": response.status,
                "length": len(text)
            }
    except Exception as e:
        return {"url": url, "error": str(e)}

async def scrape_urls(urls: List[str]) -> List[Dict]:
    """並行爬取多個 URL。"""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

async def main():
    urls = [
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/status/404",
    ]

    results = await scrape_urls(urls)
    for result in results:
        print(result)

asyncio.run(main())
```

### 非同步資料庫操作

```python
import asyncio
from typing import List, Optional

# 模擬的非同步資料庫客戶端
class AsyncDB:
    """模擬的非同步資料庫。"""

    async def execute(self, query: str) -> List[dict]:
        """執行查詢。"""
        await asyncio.sleep(0.1)
        return [{"id": 1, "name": "Example"}]

    async def fetch_one(self, query: str) -> Optional[dict]:
        """取得單一列。"""
        await asyncio.sleep(0.1)
        return {"id": 1, "name": "Example"}

async def get_user_data(db: AsyncDB, user_id: int) -> dict:
    """並行取得使用者和相關資料。"""
    user_task = db.fetch_one(f"SELECT * FROM users WHERE id = {user_id}")
    orders_task = db.execute(f"SELECT * FROM orders WHERE user_id = {user_id}")
    profile_task = db.fetch_one(f"SELECT * FROM profiles WHERE user_id = {user_id}")

    user, orders, profile = await asyncio.gather(user_task, orders_task, profile_task)

    return {
        "user": user,
        "orders": orders,
        "profile": profile
    }

async def main():
    db = AsyncDB()
    user_data = await get_user_data(db, 1)
    print(user_data)

asyncio.run(main())
```

### WebSocket 伺服器

```python
import asyncio
from typing import Set

# 模擬的 WebSocket 連線
class WebSocket:
    """模擬的 WebSocket。"""

    def __init__(self, client_id: str):
        self.client_id = client_id

    async def send(self, message: str):
        """發送訊息。"""
        print(f"Sending to {self.client_id}: {message}")
        await asyncio.sleep(0.01)

    async def recv(self) -> str:
        """接收訊息。"""
        await asyncio.sleep(1)
        return f"Message from {self.client_id}"

class WebSocketServer:
    """簡單的 WebSocket 伺服器。"""

    def __init__(self):
        self.clients: Set[WebSocket] = set()

    async def register(self, websocket: WebSocket):
        """註冊新客戶端。"""
        self.clients.add(websocket)
        print(f"Client {websocket.client_id} connected")

    async def unregister(self, websocket: WebSocket):
        """取消註冊客戶端。"""
        self.clients.remove(websocket)
        print(f"Client {websocket.client_id} disconnected")

    async def broadcast(self, message: str):
        """廣播訊息給所有客戶端。"""
        if self.clients:
            tasks = [client.send(message) for client in self.clients]
            await asyncio.gather(*tasks)

    async def handle_client(self, websocket: WebSocket):
        """處理個別客戶端連線。"""
        await self.register(websocket)
        try:
            async for message in self.message_iterator(websocket):
                await self.broadcast(f"{websocket.client_id}: {message}")
        finally:
            await self.unregister(websocket)

    async def message_iterator(self, websocket: WebSocket):
        """迭代來自客戶端的訊息。"""
        for _ in range(3):  # 模擬 3 則訊息
            yield await websocket.recv()
```

## 效能最佳實踐

### 1. 使用連線池

```python
import asyncio
import aiohttp

async def with_connection_pool():
    """使用連線池以提高效率。"""
    connector = aiohttp.TCPConnector(limit=100, limit_per_host=10)

    async with aiohttp.ClientSession(connector=connector) as session:
        tasks = [session.get(f"https://api.example.com/item/{i}") for i in range(50)]
        responses = await asyncio.gather(*tasks)
        return responses
```

### 2. 批次操作

```python
async def batch_process(items: List[str], batch_size: int = 10):
    """以批次方式處理項目。"""
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        tasks = [process_item(item) for item in batch]
        await asyncio.gather(*tasks)
        print(f"Processed batch {i // batch_size + 1}")

async def process_item(item: str):
    """處理單一項目。"""
    await asyncio.sleep(0.1)
    return f"Processed: {item}"
```

### 3. 避免阻塞操作

```python
import asyncio
import concurrent.futures
from typing import Any

def blocking_operation(data: Any) -> Any:
    """CPU 密集型的阻塞操作。"""
    import time
    time.sleep(1)
    return data * 2

async def run_in_executor(data: Any) -> Any:
    """在執行緒池中執行阻塞操作。"""
    loop = asyncio.get_event_loop()
    with concurrent.futures.ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, blocking_operation, data)
        return result

async def main():
    results = await asyncio.gather(*[run_in_executor(i) for i in range(5)])
    print(results)

asyncio.run(main())
```

## 常見陷阱

### 1. 忘記使用 await

```python
# 錯誤 - 回傳協程物件，不會執行
result = async_function()

# 正確
result = await async_function()
```

### 2. 阻塞事件迴圈

```python
# 錯誤 - 阻塞事件迴圈
import time
async def bad():
    time.sleep(1)  # 阻塞了！

# 正確
async def good():
    await asyncio.sleep(1)  # 非阻塞
```

### 3. 未處理取消操作

```python
async def cancelable_task():
    """處理取消操作的任務。"""
    try:
        while True:
            await asyncio.sleep(1)
            print("Working...")
    except asyncio.CancelledError:
        print("Task cancelled, cleaning up...")
        # 執行清理
        raise  # 重新拋出以傳播取消操作
```

### 4. 混用同步和非同步程式碼

```python
# 錯誤 - 無法直接從同步函式呼叫非同步函式
def sync_function():
    result = await async_function()  # SyntaxError!

# 正確
def sync_function():
    result = asyncio.run(async_function())
```

## 測試非同步程式碼

```python
import asyncio
import pytest

# 使用 pytest-asyncio
@pytest.mark.asyncio
async def test_async_function():
    """測試非同步函式。"""
    result = await fetch_data("https://api.example.com")
    assert result is not None

@pytest.mark.asyncio
async def test_with_timeout():
    """測試逾時功能。"""
    with pytest.raises(asyncio.TimeoutError):
        await asyncio.wait_for(slow_operation(5), timeout=1.0)
```

## 資源

- **Python asyncio 文件**：https://docs.python.org/3/library/asyncio.html
- **aiohttp**：非同步 HTTP 客戶端/伺服器
- **FastAPI**：現代化的非同步 Web 框架
- **asyncpg**：非同步 PostgreSQL 驅動程式
- **motor**：非同步 MongoDB 驅動程式

## 最佳實踐摘要

1. **使用 asyncio.run()** 作為進入點（Python 3.7+）
2. **總是 await 協程**以執行它們
3. **使用 gather() 並行執行**多個任務
4. **實作適當的錯誤處理**，使用 try/except
5. **使用逾時**以防止操作掛起
6. **池化連線**以獲得更好的效能
7. **避免在非同步程式碼中使用阻塞操作**
8. **使用 semaphores** 進行速率限制
9. **適當處理任務取消**
10. **使用 pytest-asyncio 測試非同步程式碼**
