---
name: uv-package-manager
description: 掌握 uv 套件管理器，實現快速的 Python 依賴管理、虛擬環境和現代化 Python 專案工作流程。適用於設置 Python 專案、管理依賴套件，或使用 uv 優化 Python 開發工作流程。
---

# UV 套件管理器

全面指南：使用 uv（一個以 Rust 編寫的超快速 Python 套件安裝器與解析器）進行現代化 Python 專案管理和依賴工作流程。

## 何時使用此技能

- 快速設置新的 Python 專案
- 比 pip 更快速地管理 Python 依賴套件
- 建立和管理虛擬環境
- 安裝 Python 直譯器
- 有效解決依賴衝突
- 從 pip/pip-tools/poetry 遷移
- 加速 CI/CD 流水線
- 管理 monorepo Python 專案
- 使用鎖定檔案進行可重現的建置
- 優化包含 Python 依賴的 Docker 建置

## 核心概念

### 1. 什麼是 uv？
- **超高速套件安裝器**：比 pip 快 10-100 倍
- **以 Rust 編寫**：利用 Rust 的效能優勢
- **可直接替代 pip**：與 pip 工作流程相容
- **虛擬環境管理器**：建立和管理虛擬環境
- **Python 安裝器**：下載和管理 Python 版本
- **解析器**：進階依賴解析功能
- **鎖定檔案支援**：可重現的安裝

### 2. 主要特性
- 極速安裝速度
- 透過全域快取節省磁碟空間
- 與 pip、pip-tools、poetry 相容
- 全面的依賴解析
- 跨平台支援（Linux、macOS、Windows）
- 安裝時不需要 Python
- 內建虛擬環境支援

### 3. UV 與傳統工具的比較
- **vs pip**：快 10-100 倍，更好的解析器
- **vs pip-tools**：更快、更簡單、更好的使用體驗
- **vs poetry**：更快、較不固執己見、更輕量
- **vs conda**：更快、專注於 Python

## 安裝

### 快速安裝

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# 使用 pip（如果您已安裝 Python）
pip install uv

# 使用 Homebrew（macOS）
brew install uv

# 使用 cargo（如果您已安裝 Rust）
cargo install --git https://github.com/astral-sh/uv uv
```

### 驗證安裝

```bash
uv --version
# uv 0.x.x
```

## 快速開始

### 建立新專案

```bash
# 建立新專案並包含虛擬環境
uv init my-project
cd my-project

# 或在當前目錄中建立
uv init .

# 初始化會建立：
# - .python-version（Python 版本）
# - pyproject.toml（專案設定）
# - README.md
# - .gitignore
```

### 安裝依賴套件

```bash
# 安裝套件（如需要會建立虛擬環境）
uv add requests pandas

# 安裝開發依賴套件
uv add --dev pytest black ruff

# 從 requirements.txt 安裝
uv pip install -r requirements.txt

# 從 pyproject.toml 安裝
uv sync
```

## 虛擬環境管理

### 模式 1：建立虛擬環境

```bash
# 使用 uv 建立虛擬環境
uv venv

# 使用特定 Python 版本建立
uv venv --python 3.12

# 使用自訂名稱建立
uv venv my-env

# 建立並包含系統套件
uv venv --system-site-packages

# 指定位置
uv venv /path/to/venv
```

### 模式 2：啟用虛擬環境

```bash
# Linux/macOS
source .venv/bin/activate

# Windows (Command Prompt)
.venv\Scripts\activate.bat

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# 或使用 uv run（不需要啟用）
uv run python script.py
uv run pytest
```

### 模式 3：使用 uv run

```bash
# 執行 Python 腳本（自動啟用虛擬環境）
uv run python app.py

# 執行已安裝的 CLI 工具
uv run black .
uv run pytest

# 使用特定 Python 版本執行
uv run --python 3.11 python script.py

# 傳遞參數
uv run python script.py --arg value
```

## 套件管理

### 模式 4：新增依賴套件

```bash
# 新增套件（加入至 pyproject.toml）
uv add requests

# 新增並指定版本限制
uv add "django>=4.0,<5.0"

# 新增多個套件
uv add numpy pandas matplotlib

# 新增開發依賴套件
uv add --dev pytest pytest-cov

# 新增可選依賴群組
uv add --optional docs sphinx

# 從 git 新增
uv add git+https://github.com/user/repo.git

# 從 git 新增並指定特定參照
uv add git+https://github.com/user/repo.git@v1.0.0

# 從本地路徑新增
uv add ./local-package

# 新增可編輯的本地套件
uv add -e ./local-package
```

### 模式 5：移除依賴套件

```bash
# 移除套件
uv remove requests

# 移除開發依賴套件
uv remove --dev pytest

# 移除多個套件
uv remove numpy pandas matplotlib
```

### 模式 6：升級依賴套件

```bash
# 升級特定套件
uv add --upgrade requests

# 升級所有套件
uv sync --upgrade

# 升級套件至最新版本
uv add --upgrade requests

# 顯示可升級的套件
uv tree --outdated
```

### 模式 7：鎖定依賴套件

```bash
# 產生 uv.lock 檔案
uv lock

# 更新鎖定檔案
uv lock --upgrade

# 鎖定但不安裝
uv lock --no-install

# 鎖定特定套件
uv lock --upgrade-package requests
```

## Python 版本管理

### 模式 8：安裝 Python 版本

```bash
# 安裝 Python 版本
uv python install 3.12

# 安裝多個版本
uv python install 3.11 3.12 3.13

# 安裝最新版本
uv python install

# 列出已安裝的版本
uv python list

# 尋找可用版本
uv python list --all-versions
```

### 模式 9：設定 Python 版本

```bash
# 為專案設定 Python 版本
uv python pin 3.12

# 這會建立/更新 .python-version 檔案

# 使用特定 Python 版本執行指令
uv --python 3.11 run python script.py

# 使用特定版本建立虛擬環境
uv venv --python 3.12
```

## 專案設定

### 模式 10：使用 uv 的 pyproject.toml

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "My awesome project"
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "requests>=2.31.0",
    "pydantic>=2.0.0",
    "click>=8.1.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "black>=23.0.0",
    "ruff>=0.1.0",
    "mypy>=1.5.0",
]
docs = [
    "sphinx>=7.0.0",
    "sphinx-rtd-theme>=1.3.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.uv]
dev-dependencies = [
    # 由 uv 管理的額外開發依賴套件
]

[tool.uv.sources]
# 自訂套件來源
my-package = { git = "https://github.com/user/repo.git" }
```

### 模式 11：在現有專案中使用 uv

```bash
# 從 requirements.txt 遷移
uv add -r requirements.txt

# 從 poetry 遷移
# 已有 pyproject.toml，只需使用：
uv sync

# 匯出至 requirements.txt
uv pip freeze > requirements.txt

# 匯出並包含雜湊值
uv pip freeze --require-hashes > requirements.txt
```

## 進階工作流程

### 模式 12：Monorepo 支援

```bash
# 專案結構
# monorepo/
#   packages/
#     package-a/
#       pyproject.toml
#     package-b/
#       pyproject.toml
#   pyproject.toml (root)

# 根目錄的 pyproject.toml
[tool.uv.workspace]
members = ["packages/*"]

# 安裝所有工作區套件
uv sync

# 新增工作區依賴
uv add --path ./packages/package-a
```

### 模式 13：CI/CD 整合

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v2
        with:
          enable-cache: true

      - name: Set up Python
        run: uv python install 3.12

      - name: Install dependencies
        run: uv sync --all-extras --dev

      - name: Run tests
        run: uv run pytest

      - name: Run linting
        run: |
          uv run ruff check .
          uv run black --check .
```

### 模式 14：Docker 整合

```dockerfile
# Dockerfile
FROM python:3.12-slim

# 安裝 uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# 設定工作目錄
WORKDIR /app

# 複製依賴檔案
COPY pyproject.toml uv.lock ./

# 安裝依賴套件
RUN uv sync --frozen --no-dev

# 複製應用程式程式碼
COPY . .

# 執行應用程式
CMD ["uv", "run", "python", "app.py"]
```

**優化的多階段建置：**

```dockerfile
# 多階段 Dockerfile
FROM python:3.12-slim AS builder

# 安裝 uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

WORKDIR /app

# 將依賴套件安裝至虛擬環境
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-editable

# 執行階段
FROM python:3.12-slim

WORKDIR /app

# 從建置器複製虛擬環境
COPY --from=builder /app/.venv .venv
COPY . .

# 使用虛擬環境
ENV PATH="/app/.venv/bin:$PATH"

CMD ["python", "app.py"]
```

### 模式 15：鎖定檔案工作流程

```bash
# 建立鎖定檔案（uv.lock）
uv lock

# 從鎖定檔案安裝（精確版本）
uv sync --frozen

# 更新鎖定檔案但不安裝
uv lock --no-install

# 升級鎖定檔案中的特定套件
uv lock --upgrade-package requests

# 檢查鎖定檔案是否為最新
uv lock --check

# 匯出鎖定檔案至 requirements.txt
uv export --format requirements-txt > requirements.txt

# 匯出並包含雜湊值以提升安全性
uv export --format requirements-txt --hash > requirements.txt
```

## 效能優化

### 模式 16：使用全域快取

```bash
# UV 自動使用全域快取，位於：
# Linux: ~/.cache/uv
# macOS: ~/Library/Caches/uv
# Windows: %LOCALAPPDATA%\uv\cache

# 清除快取
uv cache clean

# 檢查快取大小
uv cache dir
```

### 模式 17：平行安裝

```bash
# UV 預設以平行方式安裝套件

# 控制平行度
uv pip install --jobs 4 package1 package2

# 不使用平行（循序執行）
uv pip install --jobs 1 package
```

### 模式 18：離線模式

```bash
# 僅從快取安裝（無網路）
uv pip install --offline package

# 離線從鎖定檔案同步
uv sync --frozen --offline
```

## 與其他工具的比較

### uv vs pip

```bash
# pip
python -m venv .venv
source .venv/bin/activate
pip install requests pandas numpy
# 約 30 秒

# uv
uv venv
uv add requests pandas numpy
# 約 2 秒（快 10-15 倍）
```

### uv vs poetry

```bash
# poetry
poetry init
poetry add requests pandas
poetry install
# 約 20 秒

# uv
uv init
uv add requests pandas
uv sync
# 約 3 秒（快 6-7 倍）
```

### uv vs pip-tools

```bash
# pip-tools
pip-compile requirements.in
pip-sync requirements.txt
# 約 15 秒

# uv
uv lock
uv sync --frozen
# 約 2 秒（快 7-8 倍）
```

## 常見工作流程

### 模式 19：開始新專案

```bash
# 完整工作流程
uv init my-project
cd my-project

# 設定 Python 版本
uv python pin 3.12

# 新增依賴套件
uv add fastapi uvicorn pydantic

# 新增開發依賴套件
uv add --dev pytest black ruff mypy

# 建立結構
mkdir -p src/my_project tests

# 執行測試
uv run pytest

# 格式化程式碼
uv run black .
uv run ruff check .
```

### 模式 20：維護現有專案

```bash
# 複製儲存庫
git clone https://github.com/user/project.git
cd project

# 安裝依賴套件（自動建立虛擬環境）
uv sync

# 安裝並包含開發依賴套件
uv sync --all-extras

# 更新依賴套件
uv lock --upgrade

# 執行應用程式
uv run python app.py

# 執行測試
uv run pytest

# 新增新的依賴套件
uv add new-package

# 提交更新的檔案
git add pyproject.toml uv.lock
git commit -m "Add new-package dependency"
```

## 工具整合

### 模式 21：Pre-commit 掛鉤

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: uv-lock
        name: uv lock
        entry: uv lock
        language: system
        pass_filenames: false

      - id: ruff
        name: ruff
        entry: uv run ruff check --fix
        language: system
        types: [python]

      - id: black
        name: black
        entry: uv run black
        language: system
        types: [python]
```

### 模式 22：VS Code 整合

```json
// .vscode/settings.json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true,
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["-v"],
  "python.linting.enabled": true,
  "python.formatting.provider": "black",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true
  }
}
```

## 疑難排解

### 常見問題

```bash
# 問題：找不到 uv
# 解決方案：加入 PATH 或重新安裝
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc

# 問題：錯誤的 Python 版本
# 解決方案：明確固定版本
uv python pin 3.12
uv venv --python 3.12

# 問題：依賴衝突
# 解決方案：檢查解析結果
uv lock --verbose

# 問題：快取問題
# 解決方案：清除快取
uv cache clean

# 問題：鎖定檔案不同步
# 解決方案：重新產生
uv lock --upgrade
```

## 最佳實踐

### 專案設置

1. **始終使用鎖定檔案**以確保可重現性
2. **使用 .python-version 固定 Python 版本**
3. **將開發依賴套件與生產環境分開**
4. **使用 uv run** 而非啟用虛擬環境
5. **將 uv.lock 提交至版本控制**
6. **在 CI 中使用 --frozen** 以確保一致的建置
7. **利用全域快取**提升速度
8. **針對 monorepo 使用工作區**
9. **匯出 requirements.txt** 以確保相容性
10. **保持 uv 更新**以獲得最新功能

### 效能提示

```bash
# 在 CI 中使用凍結安裝
uv sync --frozen

# 盡可能使用離線模式
uv sync --offline

# 平行操作（自動）
# uv 預設會這樣做

# 在不同環境間重用快取
# uv 在全域共用快取

# 使用鎖定檔案跳過解析
uv sync --frozen  # 跳過解析
```

## 遷移指南

### 從 pip + requirements.txt

```bash
# 之前
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 之後
uv venv
uv pip install -r requirements.txt
# 或更好的方式：
uv init
uv add -r requirements.txt
```

### 從 Poetry

```bash
# 之前
poetry install
poetry add requests

# 之後
uv sync
uv add requests

# 保留現有的 pyproject.toml
# uv 會讀取 [project] 和 [tool.poetry] 區段
```

### 從 pip-tools

```bash
# 之前
pip-compile requirements.in
pip-sync requirements.txt

# 之後
uv lock
uv sync --frozen
```

## 指令參考

### 基本指令

```bash
# 專案管理
uv init [PATH]              # 初始化專案
uv add PACKAGE              # 新增依賴套件
uv remove PACKAGE           # 移除依賴套件
uv sync                     # 安裝依賴套件
uv lock                     # 建立/更新鎖定檔案

# 虛擬環境
uv venv [PATH]              # 建立虛擬環境
uv run COMMAND              # 在虛擬環境中執行

# Python 管理
uv python install VERSION   # 安裝 Python
uv python list              # 列出已安裝的 Python
uv python pin VERSION       # 固定 Python 版本

# 套件安裝（與 pip 相容）
uv pip install PACKAGE      # 安裝套件
uv pip uninstall PACKAGE    # 解除安裝套件
uv pip freeze               # 列出已安裝的套件
uv pip list                 # 列出套件

# 工具
uv cache clean              # 清除快取
uv cache dir                # 顯示快取位置
uv --version                # 顯示版本
```

## 資源

- **官方文件**：https://docs.astral.sh/uv/
- **GitHub 儲存庫**：https://github.com/astral-sh/uv
- **Astral 部落格**：https://astral.sh/blog
- **遷移指南**：https://docs.astral.sh/uv/guides/
- **與其他工具的比較**：https://docs.astral.sh/uv/pip/compatibility/

## 最佳實踐摘要

1. **對所有新專案使用 uv** - 從 `uv init` 開始
2. **提交鎖定檔案** - 確保可重現的建置
3. **固定 Python 版本** - 使用 .python-version
4. **使用 uv run** - 避免手動啟用虛擬環境
5. **利用快取機制** - 讓 uv 管理全域快取
6. **在 CI 中使用 --frozen** - 精確重現
7. **保持 uv 更新** - 快速發展的專案
8. **使用工作區** - 用於 monorepo 專案
9. **匯出以確保相容性** - 在需要時產生 requirements.txt
10. **閱讀文件** - uv 功能豐富且持續演進
