---
name: shellcheck-configuration
description: 精通 ShellCheck 靜態分析配置與使用，提升 shell 腳本品質。適用於設置 linting 基礎設施、修復程式碼問題或確保腳本可移植性的情境。
---

# ShellCheck 配置與靜態分析

全面性指南，說明如何配置和使用 ShellCheck 來提升 shell 腳本品質、捕捉常見陷阱，並透過靜態程式碼分析強制執行最佳實務。

## 何時使用此技能

- 在 CI/CD 流水線中為 shell 腳本設置 linting
- 分析現有 shell 腳本的問題
- 理解 ShellCheck 錯誤代碼與警告
- 針對特定專案需求配置 ShellCheck
- 將 ShellCheck 整合到開發工作流程中
- 抑制誤報並配置規則集
- 強制執行一致的程式碼品質標準
- 遷移腳本以符合品質閘門要求

## ShellCheck 基礎知識

### 什麼是 ShellCheck？

ShellCheck 是一個靜態分析工具，可分析 shell 腳本並偵測有問題的模式。它支援：
- Bash、sh、dash、ksh 及其他 POSIX shell
- 超過 100 種不同的警告與錯誤
- 針對目標 shell 和旗標的配置
- 與編輯器和 CI/CD 系統整合

### 安裝

```bash
# macOS with Homebrew
brew install shellcheck

# Ubuntu/Debian
apt-get install shellcheck

# From source
git clone https://github.com/koalaman/shellcheck.git
cd shellcheck
make build
make install

# Verify installation
shellcheck --version
```

## 配置檔案

### .shellcheckrc（專案層級）

在專案根目錄建立 `.shellcheckrc`：

```
# Specify target shell
shell=bash

# Enable optional checks
enable=avoid-nullary-conditions
enable=require-variable-braces

# Disable specific warnings
disable=SC1091
disable=SC2086
```

### 環境變數

```bash
# Set default shell target
export SHELLCHECK_SHELL=bash

# Enable strict mode
export SHELLCHECK_STRICT=true

# Specify configuration file location
export SHELLCHECK_CONFIG=~/.shellcheckrc
```

## 常見 ShellCheck 錯誤代碼

### SC1000-1099：解析器錯誤
```bash
# SC1004: Backslash continuation not followed by newline
echo hello\
world  # Error - needs line continuation

# SC1008: Invalid data for operator `=='
if [[ $var =  "value" ]]; then  # Space before ==
    true
fi
```

### SC2000-2099：Shell 問題

```bash
# SC2009: Consider using pgrep or pidof instead of grep|grep
ps aux | grep -v grep | grep myprocess  # Use pgrep instead

# SC2012: Use `ls` only for viewing. Use `find` for reliable output
for file in $(ls -la)  # Better: use find or globbing

# SC2015: Avoid using && and || instead of if-then-else
[[ -f "$file" ]] && echo "found" || echo "not found"  # Less clear

# SC2016: Expressions don't expand in single quotes
echo '$VAR'  # Literal $VAR, not variable expansion

# SC2026: This word is non-standard. Set POSIXLY_CORRECT
# when using with scripts for other shells
```

### SC2100-2199：引號問題

```bash
# SC2086: Double quote to prevent globbing and word splitting
for i in $list; do  # Should be: for i in $list or for i in "$list"
    echo "$i"
done

# SC2115: Literal tilde in path not expanded. Use $HOME instead
~/.bashrc  # In strings, use "$HOME/.bashrc"

# SC2181: Check exit code directly with `if`, not indirectly in a list
some_command
if [ $? -eq 0 ]; then  # Better: if some_command; then

# SC2206: Quote to prevent word splitting or set IFS
array=( $items )  # Should use: array=( $items )
```

### SC3000-3999：POSIX 合規性問題

```bash
# SC3010: In POSIX sh, use 'case' instead of 'cond && foo'
[[ $var == "value" ]] && do_something  # Not POSIX

# SC3043: In POSIX sh, use 'local' is undefined
function my_func() {
    local var=value  # Not POSIX in some shells
}
```

## 實用配置範例

### 最小配置（嚴格 POSIX）

```bash
#!/bin/bash
# Configure for maximum portability

shellcheck \
  --shell=sh \
  --external-sources \
  --check-sourced \
  script.sh
```

### 開發配置（Bash 寬鬆規則）

```bash
#!/bin/bash
# Configure for Bash development

shellcheck \
  --shell=bash \
  --exclude=SC1091,SC2119 \
  --enable=all \
  script.sh
```

### CI/CD 整合配置

```bash
#!/bin/bash
set -Eeuo pipefail

# Analyze all shell scripts and fail on issues
find . -type f -name "*.sh" | while read -r script; do
    echo "Checking: $script"
    shellcheck \
        --shell=bash \
        --format=gcc \
        --exclude=SC1091 \
        "$script" || exit 1
done
```

### 專案的 .shellcheckrc

```
# Shell dialect to analyze against
shell=bash

# Enable optional checks
enable=avoid-nullary-conditions,require-variable-braces,check-unassigned-uppercase

# Disable specific warnings
# SC1091: Not following sourced files (many false positives)
disable=SC1091

# SC2119: Use function_name instead of function_name -- (arguments)
disable=SC2119

# External files to source for context
external-sources=true
```

## 整合模式

### Pre-commit Hook 配置

```bash
#!/bin/bash
# .git/hooks/pre-commit

#!/bin/bash
set -e

# Find all shell scripts changed in this commit
git diff --cached --name-only | grep '\.sh$' | while read -r script; do
    echo "Linting: $script"

    if ! shellcheck "$script"; then
        echo "ShellCheck failed on $script"
        exit 1
    fi
done
```

### GitHub Actions 工作流程

```yaml
name: ShellCheck

on: [push, pull_request]

jobs:
  shellcheck:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Run ShellCheck
        run: |
          sudo apt-get install shellcheck
          find . -type f -name "*.sh" -exec shellcheck {} \;
```

### GitLab CI 流水線

```yaml
shellcheck:
  stage: lint
  image: koalaman/shellcheck-alpine
  script:
    - find . -type f -name "*.sh" -exec shellcheck {} \;
  allow_failure: false
```

## 處理 ShellCheck 違規

### 抑制特定警告

```bash
#!/bin/bash

# Disable warning for entire line
# shellcheck disable=SC2086
for file in $(ls -la); do
    echo "$file"
done

# Disable for entire script
# shellcheck disable=SC1091,SC2119

# Disable multiple warnings (format varies)
command_that_fails() {
    # shellcheck disable=SC2015
    [ -f "$1" ] && echo "found" || echo "not found"
}

# Disable specific check for source directive
# shellcheck source=./helper.sh
source helper.sh
```

### 常見違規與修正方法

#### SC2086：使用雙引號防止字詞分割

```bash
# Problem
for i in $list; do done

# Solution
for i in $list; do done  # If $list is already quoted, or
for i in "${list[@]}"; do done  # If list is an array
```

#### SC2181：直接檢查退出碼

```bash
# Problem
some_command
if [ $? -eq 0 ]; then
    echo "success"
fi

# Solution
if some_command; then
    echo "success"
fi
```

#### SC2015：使用 if-then 而非 && ||

```bash
# Problem
[ -f "$file" ] && echo "exists" || echo "not found"

# Solution - clearer intent
if [ -f "$file" ]; then
    echo "exists"
else
    echo "not found"
fi
```

#### SC2016：單引號中的表達式不會展開

```bash
# Problem
echo 'Variable value: $VAR'

# Solution
echo "Variable value: $VAR"
```

#### SC2009：使用 pgrep 而非 grep

```bash
# Problem
ps aux | grep -v grep | grep myprocess

# Solution
pgrep -f myprocess
```

## 效能最佳化

### 檢查多個檔案

```bash
#!/bin/bash

# Sequential checking
for script in *.sh; do
    shellcheck "$script"
done

# Parallel checking (faster)
find . -name "*.sh" -print0 | \
    xargs -0 -P 4 -n 1 shellcheck
```

### 快取結果

```bash
#!/bin/bash

CACHE_DIR=".shellcheck_cache"
mkdir -p "$CACHE_DIR"

check_script() {
    local script="$1"
    local hash
    local cache_file

    hash=$(sha256sum "$script" | cut -d' ' -f1)
    cache_file="$CACHE_DIR/$hash"

    if [[ ! -f "$cache_file" ]]; then
        if shellcheck "$script" > "$cache_file" 2>&1; then
            touch "$cache_file.ok"
        else
            return 1
        fi
    fi

    [[ -f "$cache_file.ok" ]]
}

find . -name "*.sh" | while read -r script; do
    check_script "$script" || exit 1
done
```

## 輸出格式

### 預設格式

```bash
shellcheck script.sh

# Output:
# script.sh:1:3: warning: foo is referenced but not assigned. [SC2154]
```

### GCC 格式（用於 CI/CD）

```bash
shellcheck --format=gcc script.sh

# Output:
# script.sh:1:3: warning: foo is referenced but not assigned.
```

### JSON 格式（用於解析）

```bash
shellcheck --format=json script.sh

# Output:
# [{"file": "script.sh", "line": 1, "column": 3, "level": "warning", "code": 2154, "message": "..."}]
```

### 安靜模式

```bash
shellcheck --format=quiet script.sh

# Returns non-zero if issues found, no output otherwise
```

## 最佳實務

1. **在 CI/CD 中執行 ShellCheck** - 在合併前捕捉問題
2. **為目標 shell 配置** - 不要將 bash 分析為 sh
3. **記錄排除項目** - 說明為何抑制違規
4. **處理違規** - 不要只是停用警告
5. **啟用嚴格模式** - 謹慎使用 `--enable=all` 並排除特定項目
6. **定期更新** - 保持 ShellCheck 為最新版本以獲得新檢查
7. **使用 pre-commit hook** - 在推送前於本地捕捉問題
8. **與編輯器整合** - 在開發期間獲得即時回饋

## 資源

- **ShellCheck GitHub**：https://github.com/koalaman/shellcheck
- **ShellCheck Wiki**：https://www.shellcheck.net/wiki/
- **錯誤代碼參考**：https://www.shellcheck.net/
