---
name: python-testing-patterns
description: 使用 pytest、fixtures、mocking 和測試驅動開發實作全面的測試策略。適用於撰寫 Python 測試、設定測試套件或實作測試最佳實務。
---

# Python 測試模式

全面指南：使用 pytest、fixtures、mocking、參數化和測試驅動開發實務在 Python 中實作強健的測試策略。

## 何時使用此技能

- 為 Python 程式碼撰寫單元測試
- 設定測試套件和測試基礎架構
- 實作測試驅動開發 (TDD)
- 為 API 和服務建立整合測試
- 模擬外部相依性和服務
- 測試非同步程式碼和並行操作
- 在 CI/CD 中設定持續測試
- 實作基於屬性的測試
- 測試資料庫操作
- 除錯失敗的測試

## 核心概念

### 1. 測試類型
- **單元測試 (Unit Tests)**：獨立測試個別函式/類別
- **整合測試 (Integration Tests)**：測試元件之間的互動
- **功能測試 (Functional Tests)**：端對端測試完整功能
- **效能測試 (Performance Tests)**：測量速度和資源使用情況

### 2. 測試結構 (AAA 模式)
- **安排 (Arrange)**：設定測試資料和前置條件
- **執行 (Act)**：執行被測試的程式碼
- **斷言 (Assert)**：驗證結果

### 3. 測試涵蓋率
- 測量測試執行了哪些程式碼
- 識別未測試的程式碼路徑
- 追求有意義的涵蓋率，而非只是高百分比

### 4. 測試隔離
- 測試應該是獨立的
- 測試之間不共享狀態
- 每個測試都應該自行清理

## 快速開始

```python
# test_example.py
def add(a, b):
    return a + b

def test_add():
    """基本測試範例。"""
    result = add(2, 3)
    assert result == 5

def test_add_negative():
    """使用負數進行測試。"""
    assert add(-1, 1) == 0

# 執行方式：pytest test_example.py
```

## 基礎模式

### 模式 1：基本 pytest 測試

```python
# test_calculator.py
import pytest

class Calculator:
    """簡單的計算機類別用於測試。"""

    def add(self, a: float, b: float) -> float:
        return a + b

    def subtract(self, a: float, b: float) -> float:
        return a - b

    def multiply(self, a: float, b: float) -> float:
        return a * b

    def divide(self, a: float, b: float) -> float:
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b


def test_addition():
    """測試加法。"""
    calc = Calculator()
    assert calc.add(2, 3) == 5
    assert calc.add(-1, 1) == 0
    assert calc.add(0, 0) == 0


def test_subtraction():
    """測試減法。"""
    calc = Calculator()
    assert calc.subtract(5, 3) == 2
    assert calc.subtract(0, 5) == -5


def test_multiplication():
    """測試乘法。"""
    calc = Calculator()
    assert calc.multiply(3, 4) == 12
    assert calc.multiply(0, 5) == 0


def test_division():
    """測試除法。"""
    calc = Calculator()
    assert calc.divide(6, 3) == 2
    assert calc.divide(5, 2) == 2.5


def test_division_by_zero():
    """測試除以零會引發錯誤。"""
    calc = Calculator()
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        calc.divide(5, 0)
```

### 模式 2：使用 Fixtures 進行設定和清理

```python
# test_database.py
import pytest
from typing import Generator

class Database:
    """簡單的資料庫類別。"""

    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.connected = False

    def connect(self):
        """連接到資料庫。"""
        self.connected = True

    def disconnect(self):
        """中斷資料庫連線。"""
        self.connected = False

    def query(self, sql: str) -> list:
        """執行查詢。"""
        if not self.connected:
            raise RuntimeError("Not connected")
        return [{"id": 1, "name": "Test"}]


@pytest.fixture
def db() -> Generator[Database, None, None]:
    """提供已連線資料庫的 fixture。"""
    # 設定
    database = Database("sqlite:///:memory:")
    database.connect()

    # 提供給測試使用
    yield database

    # 清理
    database.disconnect()


def test_database_query(db):
    """使用 fixture 測試資料庫查詢。"""
    results = db.query("SELECT * FROM users")
    assert len(results) == 1
    assert results[0]["name"] == "Test"


@pytest.fixture(scope="session")
def app_config():
    """session 範圍的 fixture - 每個測試會話建立一次。"""
    return {
        "database_url": "postgresql://localhost/test",
        "api_key": "test-key",
        "debug": True
    }


@pytest.fixture(scope="module")
def api_client(app_config):
    """module 範圍的 fixture - 每個測試模組建立一次。"""
    # 設定耗費資源的資源
    client = {"config": app_config, "session": "active"}
    yield client
    # 清理
    client["session"] = "closed"


def test_api_client(api_client):
    """測試使用 api client fixture。"""
    assert api_client["session"] == "active"
    assert api_client["config"]["debug"] is True
```

### 模式 3：參數化測試

```python
# test_validation.py
import pytest

def is_valid_email(email: str) -> bool:
    """檢查電子郵件是否有效。"""
    return "@" in email and "." in email.split("@")[1]


@pytest.mark.parametrize("email,expected", [
    ("user@example.com", True),
    ("test.user@domain.co.uk", True),
    ("invalid.email", False),
    ("@example.com", False),
    ("user@domain", False),
    ("", False),
])
def test_email_validation(email, expected):
    """使用各種輸入測試電子郵件驗證。"""
    assert is_valid_email(email) == expected


@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
    (-5, -5, -10),
])
def test_addition_parameterized(a, b, expected):
    """使用多組參數測試加法。"""
    from test_calculator import Calculator
    calc = Calculator()
    assert calc.add(a, b) == expected


# 使用 pytest.param 處理特殊情況
@pytest.mark.parametrize("value,expected", [
    pytest.param(1, True, id="positive"),
    pytest.param(0, False, id="zero"),
    pytest.param(-1, False, id="negative"),
])
def test_is_positive(value, expected):
    """使用自訂測試 ID 進行測試。"""
    assert (value > 0) == expected
```

### 模式 4：使用 unittest.mock 進行 Mocking

```python
# test_api_client.py
import pytest
from unittest.mock import Mock, patch, MagicMock
import requests

class APIClient:
    """簡單的 API 客戶端。"""

    def __init__(self, base_url: str):
        self.base_url = base_url

    def get_user(self, user_id: int) -> dict:
        """從 API 取得使用者。"""
        response = requests.get(f"{self.base_url}/users/{user_id}")
        response.raise_for_status()
        return response.json()

    def create_user(self, data: dict) -> dict:
        """建立新使用者。"""
        response = requests.post(f"{self.base_url}/users", json=data)
        response.raise_for_status()
        return response.json()


def test_get_user_success():
    """使用 mock 測試成功的 API 呼叫。"""
    client = APIClient("https://api.example.com")

    mock_response = Mock()
    mock_response.json.return_value = {"id": 1, "name": "John Doe"}
    mock_response.raise_for_status.return_value = None

    with patch("requests.get", return_value=mock_response) as mock_get:
        user = client.get_user(1)

        assert user["id"] == 1
        assert user["name"] == "John Doe"
        mock_get.assert_called_once_with("https://api.example.com/users/1")


def test_get_user_not_found():
    """測試 404 錯誤的 API 呼叫。"""
    client = APIClient("https://api.example.com")

    mock_response = Mock()
    mock_response.raise_for_status.side_effect = requests.HTTPError("404 Not Found")

    with patch("requests.get", return_value=mock_response):
        with pytest.raises(requests.HTTPError):
            client.get_user(999)


@patch("requests.post")
def test_create_user(mock_post):
    """使用裝飾器語法測試使用者建立。"""
    client = APIClient("https://api.example.com")

    mock_post.return_value.json.return_value = {"id": 2, "name": "Jane Doe"}
    mock_post.return_value.raise_for_status.return_value = None

    user_data = {"name": "Jane Doe", "email": "jane@example.com"}
    result = client.create_user(user_data)

    assert result["id"] == 2
    mock_post.assert_called_once()
    call_args = mock_post.call_args
    assert call_args.kwargs["json"] == user_data
```

### 模式 5：測試例外情況

```python
# test_exceptions.py
import pytest

def divide(a: float, b: float) -> float:
    """將 a 除以 b。"""
    if b == 0:
        raise ZeroDivisionError("Division by zero")
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError("Arguments must be numbers")
    return a / b


def test_zero_division():
    """測試除以零會引發例外。"""
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)


def test_zero_division_with_message():
    """測試例外訊息。"""
    with pytest.raises(ZeroDivisionError, match="Division by zero"):
        divide(5, 0)


def test_type_error():
    """測試型別錯誤例外。"""
    with pytest.raises(TypeError, match="must be numbers"):
        divide("10", 5)


def test_exception_info():
    """測試存取例外資訊。"""
    with pytest.raises(ValueError) as exc_info:
        int("not a number")

    assert "invalid literal" in str(exc_info.value)
```

## 進階模式

### 模式 6：測試非同步程式碼

```python
# test_async.py
import pytest
import asyncio

async def fetch_data(url: str) -> dict:
    """非同步取得資料。"""
    await asyncio.sleep(0.1)
    return {"url": url, "data": "result"}


@pytest.mark.asyncio
async def test_fetch_data():
    """測試非同步函式。"""
    result = await fetch_data("https://api.example.com")
    assert result["url"] == "https://api.example.com"
    assert "data" in result


@pytest.mark.asyncio
async def test_concurrent_fetches():
    """測試並行的非同步操作。"""
    urls = ["url1", "url2", "url3"]
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)

    assert len(results) == 3
    assert all("data" in r for r in results)


@pytest.fixture
async def async_client():
    """非同步 fixture。"""
    client = {"connected": True}
    yield client
    client["connected"] = False


@pytest.mark.asyncio
async def test_with_async_fixture(async_client):
    """測試使用非同步 fixture。"""
    assert async_client["connected"] is True
```

### 模式 7：使用 Monkeypatch 進行測試

```python
# test_environment.py
import os
import pytest

def get_database_url() -> str:
    """從環境變數取得資料庫 URL。"""
    return os.environ.get("DATABASE_URL", "sqlite:///:memory:")


def test_database_url_default():
    """測試預設的資料庫 URL。"""
    # 如果有設定環境變數，將使用實際的環境變數
    url = get_database_url()
    assert url


def test_database_url_custom(monkeypatch):
    """使用 monkeypatch 測試自訂資料庫 URL。"""
    monkeypatch.setenv("DATABASE_URL", "postgresql://localhost/test")
    assert get_database_url() == "postgresql://localhost/test"


def test_database_url_not_set(monkeypatch):
    """測試未設定環境變數時的情況。"""
    monkeypatch.delenv("DATABASE_URL", raising=False)
    assert get_database_url() == "sqlite:///:memory:"


class Config:
    """設定類別。"""

    def __init__(self):
        self.api_key = "production-key"

    def get_api_key(self):
        return self.api_key


def test_monkeypatch_attribute(monkeypatch):
    """測試 monkeypatch 物件屬性。"""
    config = Config()
    monkeypatch.setattr(config, "api_key", "test-key")
    assert config.get_api_key() == "test-key"
```

### 模式 8：臨時檔案和目錄

```python
# test_file_operations.py
import pytest
from pathlib import Path

def save_data(filepath: Path, data: str):
    """將資料儲存到檔案。"""
    filepath.write_text(data)


def load_data(filepath: Path) -> str:
    """從檔案載入資料。"""
    return filepath.read_text()


def test_file_operations(tmp_path):
    """使用臨時目錄測試檔案操作。"""
    # tmp_path 是一個 pathlib.Path 物件
    test_file = tmp_path / "test_data.txt"

    # 儲存資料
    save_data(test_file, "Hello, World!")

    # 驗證檔案存在
    assert test_file.exists()

    # 載入並驗證資料
    data = load_data(test_file)
    assert data == "Hello, World!"


def test_multiple_files(tmp_path):
    """使用多個臨時檔案進行測試。"""
    files = {
        "file1.txt": "Content 1",
        "file2.txt": "Content 2",
        "file3.txt": "Content 3"
    }

    for filename, content in files.items():
        filepath = tmp_path / filename
        save_data(filepath, content)

    # 驗證所有檔案都已建立
    assert len(list(tmp_path.iterdir())) == 3

    # 驗證內容
    for filename, expected_content in files.items():
        filepath = tmp_path / filename
        assert load_data(filepath) == expected_content
```

### 模式 9：自訂 Fixtures 和 Conftest

```python
# conftest.py
"""所有測試共享的 fixtures。"""
import pytest

@pytest.fixture(scope="session")
def database_url():
    """為所有測試提供資料庫 URL。"""
    return "postgresql://localhost/test_db"


@pytest.fixture(autouse=True)
def reset_database(database_url):
    """自動使用的 fixture，在每個測試前執行。"""
    # 設定：清空資料庫
    print(f"Clearing database: {database_url}")
    yield
    # 清理：清理
    print("Test completed")


@pytest.fixture
def sample_user():
    """提供範例使用者資料。"""
    return {
        "id": 1,
        "name": "Test User",
        "email": "test@example.com"
    }


@pytest.fixture
def sample_users():
    """提供範例使用者清單。"""
    return [
        {"id": 1, "name": "User 1"},
        {"id": 2, "name": "User 2"},
        {"id": 3, "name": "User 3"},
    ]


# 參數化的 fixture
@pytest.fixture(params=["sqlite", "postgresql", "mysql"])
def db_backend(request):
    """使用不同資料庫後端執行測試的 fixture。"""
    return request.param


def test_with_db_backend(db_backend):
    """此測試將使用不同的後端執行 3 次。"""
    print(f"Testing with {db_backend}")
    assert db_backend in ["sqlite", "postgresql", "mysql"]
```

### 模式 10：基於屬性的測試

```python
# test_properties.py
from hypothesis import given, strategies as st
import pytest

def reverse_string(s: str) -> str:
    """反轉字串。"""
    return s[::-1]


@given(st.text())
def test_reverse_twice_is_original(s):
    """屬性：反轉兩次會回到原始字串。"""
    assert reverse_string(reverse_string(s)) == s


@given(st.text())
def test_reverse_length(s):
    """屬性：反轉後的字串長度相同。"""
    assert len(reverse_string(s)) == len(s)


@given(st.integers(), st.integers())
def test_addition_commutative(a, b):
    """屬性：加法具有交換律。"""
    assert a + b == b + a


@given(st.lists(st.integers()))
def test_sorted_list_properties(lst):
    """屬性：排序後的串列是有序的。"""
    sorted_lst = sorted(lst)

    # 長度相同
    assert len(sorted_lst) == len(lst)

    # 所有元素都存在
    assert set(sorted_lst) == set(lst)

    # 是有序的
    for i in range(len(sorted_lst) - 1):
        assert sorted_lst[i] <= sorted_lst[i + 1]
```

## 測試最佳實務

### 測試組織

```python
# tests/
#   __init__.py
#   conftest.py           # 共享的 fixtures
#   test_unit/            # 單元測試
#     test_models.py
#     test_utils.py
#   test_integration/     # 整合測試
#     test_api.py
#     test_database.py
#   test_e2e/            # 端對端測試
#     test_workflows.py
```

### 測試命名

```python
# 良好的測試名稱
def test_user_creation_with_valid_data():
    """清晰的名稱描述正在測試什麼。"""
    pass


def test_login_fails_with_invalid_password():
    """名稱描述預期的行為。"""
    pass


def test_api_returns_404_for_missing_resource():
    """明確說明輸入和預期結果。"""
    pass


# 不良的測試名稱
def test_1():  # 沒有描述性
    pass


def test_user():  # 太模糊
    pass


def test_function():  # 沒有解釋測試什麼
    pass
```

### 測試標記

```python
# test_markers.py
import pytest

@pytest.mark.slow
def test_slow_operation():
    """標記慢速測試。"""
    import time
    time.sleep(2)


@pytest.mark.integration
def test_database_integration():
    """標記整合測試。"""
    pass


@pytest.mark.skip(reason="Feature not implemented yet")
def test_future_feature():
    """暫時跳過測試。"""
    pass


@pytest.mark.skipif(os.name == "nt", reason="Unix only test")
def test_unix_specific():
    """條件性跳過。"""
    pass


@pytest.mark.xfail(reason="Known bug #123")
def test_known_bug():
    """標記預期會失敗的測試。"""
    assert False


# 執行方式：
# pytest -m slow          # 只執行慢速測試
# pytest -m "not slow"    # 跳過慢速測試
# pytest -m integration   # 執行整合測試
```

### 涵蓋率報告

```bash
# 安裝 coverage
pip install pytest-cov

# 執行測試並產生涵蓋率報告
pytest --cov=myapp tests/

# 產生 HTML 報告
pytest --cov=myapp --cov-report=html tests/

# 如果涵蓋率低於閾值則失敗
pytest --cov=myapp --cov-fail-under=80 tests/

# 顯示未涵蓋的行數
pytest --cov=myapp --cov-report=term-missing tests/
```

## 測試資料庫程式碼

```python
# test_database_models.py
import pytest
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session

Base = declarative_base()


class User(Base):
    """使用者模型。"""
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    email = Column(String(100), unique=True)


@pytest.fixture(scope="function")
def db_session() -> Session:
    """建立記憶體內資料庫進行測試。"""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)

    SessionLocal = sessionmaker(bind=engine)
    session = SessionLocal()

    yield session

    session.close()


def test_create_user(db_session):
    """測試建立使用者。"""
    user = User(name="Test User", email="test@example.com")
    db_session.add(user)
    db_session.commit()

    assert user.id is not None
    assert user.name == "Test User"


def test_query_user(db_session):
    """測試查詢使用者。"""
    user1 = User(name="User 1", email="user1@example.com")
    user2 = User(name="User 2", email="user2@example.com")

    db_session.add_all([user1, user2])
    db_session.commit()

    users = db_session.query(User).all()
    assert len(users) == 2


def test_unique_email_constraint(db_session):
    """測試唯一電子郵件限制。"""
    from sqlalchemy.exc import IntegrityError

    user1 = User(name="User 1", email="same@example.com")
    user2 = User(name="User 2", email="same@example.com")

    db_session.add(user1)
    db_session.commit()

    db_session.add(user2)

    with pytest.raises(IntegrityError):
        db_session.commit()
```

## CI/CD 整合

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          pip install -e ".[dev]"
          pip install pytest pytest-cov

      - name: Run tests
        run: |
          pytest --cov=myapp --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

## 設定檔案

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    -v
    --strict-markers
    --tb=short
    --cov=myapp
    --cov-report=term-missing
markers =
    slow: marks tests as slow
    integration: marks integration tests
    unit: marks unit tests
    e2e: marks end-to-end tests
```

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = [
    "-v",
    "--cov=myapp",
    "--cov-report=term-missing",
]

[tool.coverage.run]
source = ["myapp"]
omit = ["*/tests/*", "*/migrations/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
]
```

## 資源

- **pytest 文件**：https://docs.pytest.org/
- **unittest.mock**：https://docs.python.org/3/library/unittest.mock.html
- **hypothesis**：基於屬性的測試
- **pytest-asyncio**：測試非同步程式碼
- **pytest-cov**：涵蓋率報告
- **pytest-mock**：pytest 的 mock 包裝器

## 最佳實務總結

1. **先撰寫測試** (TDD) 或與程式碼一起撰寫
2. **每個測試一個斷言**（盡可能）
3. **使用描述性的測試名稱**，說明行為
4. **保持測試獨立**且隔離
5. **使用 fixtures** 進行設定和清理
6. **適當地模擬外部相依性**
7. **參數化測試**以減少重複
8. **測試邊界情況**和錯誤條件
9. **測量涵蓋率**但專注於品質
10. **在 CI/CD 中執行測試**，每次提交都執行
