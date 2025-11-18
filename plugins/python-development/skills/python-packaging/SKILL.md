---
name: python-packaging
description: 建立可散佈的 Python 套件，包含適當的專案結構、setup.py/pyproject.toml，以及發布到 PyPI。適用於打包 Python 函式庫、建立 CLI 工具，或散佈 Python 程式碼。
---

# Python Packaging（Python 套件打包）

全面介紹如何使用現代化打包工具、pyproject.toml 來建立、組織和散佈 Python 套件，以及發布到 PyPI 的完整指南。

## 何時使用此技能

- 建立用於散佈的 Python 函式庫
- 建置具有進入點的命令列工具
- 發布套件到 PyPI 或私有儲存庫
- 設定 Python 專案結構
- 建立具有相依性的可安裝套件
- 建置 wheel 和原始碼散佈檔
- Python 套件的版本控制與發布
- 建立命名空間套件
- 實作套件詮釋資料和分類器

## 核心概念

### 1. 套件結構
- **Source layout**（原始碼配置）：`src/package_name/`（建議使用）
- **Flat layout**（平面配置）：`package_name/`（較簡單但靈活性較低）
- **Package metadata**（套件詮釋資料）：pyproject.toml、setup.py 或 setup.cfg
- **Distribution formats**（散佈格式）：wheel (.whl) 和 source distribution 原始碼散佈檔 (.tar.gz)

### 2. 現代化打包標準
- **PEP 517/518**：建置系統需求
- **PEP 621**：pyproject.toml 中的詮釋資料
- **PEP 660**：可編輯安裝
- **pyproject.toml**：單一配置來源

### 3. 建置後端
- **setuptools**：傳統、廣泛使用
- **hatchling**：現代化、具有主見
- **flit**：輕量級，適用於純 Python
- **poetry**：相依性管理 + 打包

### 4. 散佈方式
- **PyPI**：Python Package Index（公開）
- **TestPyPI**：正式發布前的測試環境
- **Private repositories**（私有儲存庫）：JFrog、AWS CodeArtifact 等

## 快速入門

### 最小套件結構

```
my-package/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── module.py
└── tests/
    └── test_module.py
```

### 最小 pyproject.toml

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "0.1.0"
description = "A short description"
authors = [{name = "Your Name", email = "you@example.com"}]
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "requests>=2.28.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "black>=22.0",
]
```

## 套件結構模式

### 模式 1：Source Layout 原始碼配置（建議使用）

```
my-package/
├── pyproject.toml
├── README.md
├── LICENSE
├── .gitignore
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── core.py
│       ├── utils.py
│       └── py.typed          # 用於型別提示
├── tests/
│   ├── __init__.py
│   ├── test_core.py
│   └── test_utils.py
└── docs/
    └── index.md
```

**優點：**
- 防止意外從原始碼匯入
- 更乾淨的測試匯入
- 更好的隔離性

**Source layout 的 pyproject.toml：**
```toml
[tool.setuptools.packages.find]
where = ["src"]
```

### 模式 2：Flat Layout 平面配置

```
my-package/
├── pyproject.toml
├── README.md
├── my_package/
│   ├── __init__.py
│   └── module.py
└── tests/
    └── test_module.py
```

**較簡單但：**
- 可以在不安裝的情況下匯入套件
- 對於函式庫來說較不專業

### 模式 3：多套件專案

```
project/
├── pyproject.toml
├── packages/
│   ├── package-a/
│   │   └── src/
│   │       └── package_a/
│   └── package-b/
│       └── src/
│           └── package_b/
└── tests/
```

## 完整的 pyproject.toml 範例

### 模式 4：功能完整的 pyproject.toml

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-awesome-package"
version = "1.0.0"
description = "An awesome Python package"
readme = "README.md"
requires-python = ">=3.8"
license = {text = "MIT"}
authors = [
    {name = "Your Name", email = "you@example.com"},
]
maintainers = [
    {name = "Maintainer Name", email = "maintainer@example.com"},
]
keywords = ["example", "package", "awesome"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.8",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]

dependencies = [
    "requests>=2.28.0,<3.0.0",
    "click>=8.0.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-cov>=4.0.0",
    "black>=23.0.0",
    "ruff>=0.1.0",
    "mypy>=1.0.0",
]
docs = [
    "sphinx>=5.0.0",
    "sphinx-rtd-theme>=1.0.0",
]
all = [
    "my-awesome-package[dev,docs]",
]

[project.urls]
Homepage = "https://github.com/username/my-awesome-package"
Documentation = "https://my-awesome-package.readthedocs.io"
Repository = "https://github.com/username/my-awesome-package"
"Bug Tracker" = "https://github.com/username/my-awesome-package/issues"
Changelog = "https://github.com/username/my-awesome-package/blob/main/CHANGELOG.md"

[project.scripts]
my-cli = "my_package.cli:main"
awesome-tool = "my_package.tools:run"

[project.entry-points."my_package.plugins"]
plugin1 = "my_package.plugins:plugin1"

[tool.setuptools]
package-dir = {"" = "src"}
zip-safe = false

[tool.setuptools.packages.find]
where = ["src"]
include = ["my_package*"]
exclude = ["tests*"]

[tool.setuptools.package-data]
my_package = ["py.typed", "*.pyi", "data/*.json"]

# Black configuration
[tool.black]
line-length = 100
target-version = ["py38", "py39", "py310", "py311"]
include = '\.pyi?$'

# Ruff configuration
[tool.ruff]
line-length = 100
target-version = "py38"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

# MyPy configuration
[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

# Pytest configuration
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = "-v --cov=my_package --cov-report=term-missing"

# Coverage configuration
[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
]
```

### 模式 5：動態版本控制

```toml
[build-system]
requires = ["setuptools>=61.0", "setuptools-scm>=8.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
dynamic = ["version"]
description = "Package with dynamic version"

[tool.setuptools.dynamic]
version = {attr = "my_package.__version__"}

# 或使用 setuptools-scm 進行基於 git 的版本控制
[tool.setuptools_scm]
write_to = "src/my_package/_version.py"
```

**在 __init__.py 中：**
```python
# src/my_package/__init__.py
__version__ = "1.0.0"

# 或使用 setuptools-scm
from importlib.metadata import version
__version__ = version("my-package")
```

## 命令列介面 (CLI) 模式

### 模式 6：使用 Click 的 CLI

```python
# src/my_package/cli.py
import click

@click.group()
@click.version_option()
def cli():
    """My awesome CLI tool."""
    pass

@cli.command()
@click.argument("name")
@click.option("--greeting", default="Hello", help="Greeting to use")
def greet(name: str, greeting: str):
    """Greet someone."""
    click.echo(f"{greeting}, {name}!")

@cli.command()
@click.option("--count", default=1, help="Number of times to repeat")
def repeat(count: int):
    """Repeat a message."""
    for i in range(count):
        click.echo(f"Message {i + 1}")

def main():
    """Entry point for CLI."""
    cli()

if __name__ == "__main__":
    main()
```

**在 pyproject.toml 中註冊：**
```toml
[project.scripts]
my-tool = "my_package.cli:main"
```

**使用方式：**
```bash
pip install -e .
my-tool greet World
my-tool greet Alice --greeting="Hi"
my-tool repeat --count=3
```

### 模式 7：使用 argparse 的 CLI

```python
# src/my_package/cli.py
import argparse
import sys

def main():
    """Main CLI entry point."""
    parser = argparse.ArgumentParser(
        description="My awesome tool",
        prog="my-tool"
    )

    parser.add_argument(
        "--version",
        action="version",
        version="%(prog)s 1.0.0"
    )

    subparsers = parser.add_subparsers(dest="command", help="Commands")

    # Add subcommand
    process_parser = subparsers.add_parser("process", help="Process data")
    process_parser.add_argument("input_file", help="Input file path")
    process_parser.add_argument(
        "--output", "-o",
        default="output.txt",
        help="Output file path"
    )

    args = parser.parse_args()

    if args.command == "process":
        process_data(args.input_file, args.output)
    else:
        parser.print_help()
        sys.exit(1)

def process_data(input_file: str, output_file: str):
    """Process data from input to output."""
    print(f"Processing {input_file} -> {output_file}")

if __name__ == "__main__":
    main()
```

## 建置與發布

### 模式 8：本機建置套件

```bash
# 安裝建置工具
pip install build twine

# 建置散佈檔
python -m build

# 這會建立：
# dist/
#   my-package-1.0.0.tar.gz (原始碼散佈檔)
#   my_package-1.0.0-py3-none-any.whl (wheel)

# 檢查散佈檔
twine check dist/*
```

### 模式 9：發布到 PyPI

```bash
# 安裝發布工具
pip install twine

# 先在 TestPyPI 上測試
twine upload --repository testpypi dist/*

# 從 TestPyPI 安裝以進行測試
pip install --index-url https://test.pypi.org/simple/ my-package

# 如果一切正常，發布到 PyPI
twine upload dist/*
```

**使用 API tokens（建議）：**
```bash
# 建立 ~/.pypirc
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-...your-token...

[testpypi]
username = __token__
password = pypi-...your-test-token...
```

### 模式 10：使用 GitHub Actions 自動發布

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install build twine

      - name: Build package
        run: python -m build

      - name: Check package
        run: twine check dist/*

      - name: Publish to PyPI
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
        run: twine upload dist/*
```

## 進階模式

### 模式 11：包含資料檔案

```toml
[tool.setuptools.package-data]
my_package = [
    "data/*.json",
    "templates/*.html",
    "static/css/*.css",
    "py.typed",
]
```

**存取資料檔案：**
```python
# src/my_package/loader.py
from importlib.resources import files
import json

def load_config():
    """從套件資料載入配置。"""
    config_file = files("my_package").joinpath("data/config.json")
    with config_file.open() as f:
        return json.load(f)

# Python 3.9+
from importlib.resources import files

data = files("my_package").joinpath("data/file.txt").read_text()
```

### 模式 12：命名空間套件

**適用於分散在多個儲存庫的大型專案：**

```
# 套件 1：company-core
company/
└── core/
    ├── __init__.py
    └── models.py

# 套件 2：company-api
company/
└── api/
    ├── __init__.py
    └── routes.py
```

**不要在命名空間目錄 (company/) 中包含 __init__.py：**

```toml
# company-core/pyproject.toml
[project]
name = "company-core"

[tool.setuptools.packages.find]
where = ["."]
include = ["company.core*"]

# company-api/pyproject.toml
[project]
name = "company-api"

[tool.setuptools.packages.find]
where = ["."]
include = ["company.api*"]
```

**使用方式：**
```python
# 兩個套件都可以在相同命名空間下匯入
from company.core import models
from company.api import routes
```

### 模式 13：C 擴充功能

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel", "Cython>=0.29"]
build-backend = "setuptools.build_meta"

[tool.setuptools]
ext-modules = [
    {name = "my_package.fast_module", sources = ["src/fast_module.c"]},
]
```

**或使用 setup.py：**
```python
# setup.py
from setuptools import setup, Extension

setup(
    ext_modules=[
        Extension(
            "my_package.fast_module",
            sources=["src/fast_module.c"],
            include_dirs=["src/include"],
        )
    ]
)
```

## 版本管理

### 模式 14：語義化版本控制

```python
# src/my_package/__init__.py
__version__ = "1.2.3"

# 語義化版本控制：MAJOR.MINOR.PATCH
# MAJOR：破壞性變更
# MINOR：新功能（向後相容）
# PATCH：錯誤修正
```

**相依性中的版本限制：**
```toml
dependencies = [
    "requests>=2.28.0,<3.0.0",  # 相容範圍
    "click~=8.1.0",              # 相容版本（~= 8.1.0 表示 >=8.1.0,<8.2.0）
    "pydantic>=2.0",             # 最低版本
    "numpy==1.24.3",             # 精確版本（盡可能避免）
]
```

### 模式 15：基於 Git 的版本控制

```toml
[build-system]
requires = ["setuptools>=61.0", "setuptools-scm>=8.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
dynamic = ["version"]

[tool.setuptools_scm]
write_to = "src/my_package/_version.py"
version_scheme = "post-release"
local_scheme = "dirty-tag"
```

**建立的版本格式如：**
- `1.0.0`（來自 git tag）
- `1.0.1.dev3+g1234567`（tag 之後的第 3 個 commit）

## 測試安裝

### 模式 16：可編輯安裝

```bash
# 以開發模式安裝
pip install -e .

# 包含可選相依性
pip install -e ".[dev]"
pip install -e ".[dev,docs]"

# 現在原始碼的變更會立即反映
```

### 模式 17：在隔離環境中測試

```bash
# 建立虛擬環境
python -m venv test-env
source test-env/bin/activate  # Linux/Mac
# test-env\Scripts\activate  # Windows

# 安裝套件
pip install dist/my_package-1.0.0-py3-none-any.whl

# 測試是否正常運作
python -c "import my_package; print(my_package.__version__)"

# 測試 CLI
my-tool --help

# 清理
deactivate
rm -rf test-env
```

## 文件

### 模式 18：README.md 範本

```markdown
# My Package

[![PyPI version](https://badge.fury.io/py/my-package.svg)](https://pypi.org/project/my-package/)
[![Python versions](https://img.shields.io/pypi/pyversions/my-package.svg)](https://pypi.org/project/my-package/)
[![Tests](https://github.com/username/my-package/workflows/Tests/badge.svg)](https://github.com/username/my-package/actions)

Brief description of your package.

## Installation

```bash
pip install my-package
```

## Quick Start

```python
from my_package import something

result = something.do_stuff()
```

## Features

- Feature 1
- Feature 2
- Feature 3

## Documentation

Full documentation: https://my-package.readthedocs.io

## Development

```bash
git clone https://github.com/username/my-package.git
cd my-package
pip install -e ".[dev]"
pytest
```

## License

MIT
```

## 常見模式

### 模式 19：多架構 Wheel

```yaml
# .github/workflows/wheels.yml
name: Build wheels

on: [push, pull_request]

jobs:
  build_wheels:
    name: Build wheels on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v3

      - name: Build wheels
        uses: pypa/cibuildwheel@v2.16.2

      - uses: actions/upload-artifact@v3
        with:
          path: ./wheelhouse/*.whl
```

### 模式 20：私有套件索引

```bash
# 從私有索引安裝
pip install my-package --index-url https://private.pypi.org/simple/

# 或加入到 pip.conf
[global]
index-url = https://private.pypi.org/simple/
extra-index-url = https://pypi.org/simple/

# 上傳到私有索引
twine upload --repository-url https://private.pypi.org/ dist/*
```

## 檔案範本

### Python 套件的 .gitignore

```gitignore
# 建置產物
build/
dist/
*.egg-info/
*.egg
.eggs/

# Python
__pycache__/
*.py[cod]
*$py.class
*.so

# 虛擬環境
venv/
env/
ENV/

# IDE
.vscode/
.idea/
*.swp

# 測試
.pytest_cache/
.coverage
htmlcov/

# 散佈檔
*.whl
*.tar.gz
```

### MANIFEST.in

```
# MANIFEST.in
include README.md
include LICENSE
include pyproject.toml

recursive-include src/my_package/data *.json
recursive-include src/my_package/templates *.html
recursive-exclude * __pycache__
recursive-exclude * *.py[co]
```

## 發布檢查清單

- [ ] 程式碼已測試（pytest 通過）
- [ ] 文件已完成（README、docstrings）
- [ ] 版本號已更新
- [ ] CHANGELOG.md 已更新
- [ ] 已包含授權檔案
- [ ] pyproject.toml 已完成
- [ ] 套件建置無錯誤
- [ ] 已在乾淨環境中測試安裝
- [ ] CLI 工具正常運作（如適用）
- [ ] PyPI 詮釋資料正確（分類器、關鍵字）
- [ ] GitHub 儲存庫已連結
- [ ] 已先在 TestPyPI 上測試
- [ ] 已建立 Git tag 用於發布

## 資源

- **Python Packaging Guide**：https://packaging.python.org/
- **PyPI**：https://pypi.org/
- **TestPyPI**：https://test.pypi.org/
- **setuptools documentation**：https://setuptools.pypa.io/
- **build**：https://pypa-build.readthedocs.io/
- **twine**：https://twine.readthedocs.io/

## 最佳實踐總結

1. **使用 src/ layout**：提供更乾淨的套件結構
2. **使用 pyproject.toml**：採用現代化打包方式
3. **固定建置相依性**：在 build-system.requires 中指定
4. **適當的版本控制**：使用語義化版本控制
5. **包含所有詮釋資料**：分類器、URL 等
6. **在乾淨環境中測試安裝**：確保可重現性
7. **先使用 TestPyPI**：在發布到 PyPI 之前測試
8. **徹底的文件**：README 和 docstrings
9. **包含 LICENSE 檔案**：明確授權條款
10. **自動化發布**：使用 CI/CD
