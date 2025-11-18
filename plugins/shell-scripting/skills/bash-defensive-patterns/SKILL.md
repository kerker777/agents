---
name: bash-defensive-patterns
description: 掌握防禦性 Bash 程式設計技術，用於撰寫生產等級腳本。適用於編寫健壯的 shell 腳本、CI/CD 管線或需要容錯和安全性的系統工具。
---

# Bash 防禦性編程模式

全面指南，教你如何使用防禦性程式設計技術、錯誤處理和安全最佳實踐來撰寫生產就緒的 Bash 腳本，以預防常見陷阱並確保可靠性。

## 何時使用此技能

- 撰寫生產環境自動化腳本
- 建構 CI/CD 管線腳本
- 建立系統管理工具
- 開發具容錯能力的部署自動化
- 撰寫必須安全處理邊緣情況的腳本
- 建構可維護的 shell 腳本函式庫
- 實作完整的日誌記錄和監控
- 建立必須跨不同平台運作的腳本

## 核心防禦性原則

### 1. 嚴格模式 (Strict Mode)
在每個腳本開頭啟用 bash 嚴格模式以提早捕捉錯誤。

```bash
#!/bin/bash
set -Eeuo pipefail  # Exit on error, unset variables, pipe failures
```

**關鍵旗標：**
- `set -E`：在函式中繼承 ERR trap
- `set -e`：任何錯誤時退出（命令返回非零值）
- `set -u`：引用未定義變數時退出
- `set -o pipefail`：管線中任何命令失敗就視為失敗（不只是最後一個）

### 2. 錯誤捕捉與清理
在腳本退出或發生錯誤時實作適當的清理機制。

```bash
#!/bin/bash
set -Eeuo pipefail

trap 'echo "Error on line $LINENO"' ERR
trap 'echo "Cleaning up..."; rm -rf "$TMPDIR"' EXIT

TMPDIR=$(mktemp -d)
# Script code here
```

### 3. 變數安全性
始終為變數加上引號以防止字詞分割（word splitting）和萬用字元展開（globbing）問題。

```bash
# Wrong - unsafe
cp $source $dest

# Correct - safe
cp "$source" "$dest"

# Required variables - fail with message if unset
: "${REQUIRED_VAR:?REQUIRED_VAR is not set}"
```

### 4. 陣列處理
安全地使用陣列進行複雜的資料處理。

```bash
# Safe array iteration
declare -a items=("item 1" "item 2" "item 3")

for item in "${items[@]}"; do
    echo "Processing: $item"
done

# Reading output into array safely
mapfile -t lines < <(some_command)
readarray -t numbers < <(seq 1 10)
```

### 5. 條件判斷安全性
使用 `[[ ]]` 處理 Bash 特定功能，使用 `[ ]` 處理 POSIX 相容性。

```bash
# Bash - safer
if [[ -f "$file" && -r "$file" ]]; then
    content=$(<"$file")
fi

# POSIX - portable
if [ -f "$file" ] && [ -r "$file" ]; then
    content=$(cat "$file")
fi

# Test for existence before operations
if [[ -z "${VAR:-}" ]]; then
    echo "VAR is not set or is empty"
fi
```

## 基礎模式

### 模式 1：安全的腳本目錄偵測

```bash
#!/bin/bash
set -Eeuo pipefail

# Correctly determine script directory
SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd -P)"
SCRIPT_NAME="$(basename -- "${BASH_SOURCE[0]}")"

echo "Script location: $SCRIPT_DIR/$SCRIPT_NAME"
```

### 模式 2：完整的函式範本

```bash
#!/bin/bash
set -Eeuo pipefail

# Prefix for functions: handle_*, process_*, check_*, validate_*
# Include documentation and error handling

validate_file() {
    local -r file="$1"
    local -r message="${2:-File not found: $file}"

    if [[ ! -f "$file" ]]; then
        echo "ERROR: $message" >&2
        return 1
    fi
    return 0
}

process_files() {
    local -r input_dir="$1"
    local -r output_dir="$2"

    # Validate inputs
    [[ -d "$input_dir" ]] || { echo "ERROR: input_dir not a directory" >&2; return 1; }

    # Create output directory if needed
    mkdir -p "$output_dir" || { echo "ERROR: Cannot create output_dir" >&2; return 1; }

    # Process files safely
    while IFS= read -r -d '' file; do
        echo "Processing: $file"
        # Do work
    done < <(find "$input_dir" -maxdepth 1 -type f -print0)

    return 0
}
```

### 模式 3：安全的暫存檔處理

```bash
#!/bin/bash
set -Eeuo pipefail

trap 'rm -rf -- "$TMPDIR"' EXIT

# Create temporary directory
TMPDIR=$(mktemp -d) || { echo "ERROR: Failed to create temp directory" >&2; exit 1; }

# Create temporary files in directory
TMPFILE1="$TMPDIR/temp1.txt"
TMPFILE2="$TMPDIR/temp2.txt"

# Use temporary files
touch "$TMPFILE1" "$TMPFILE2"

echo "Temp files created in: $TMPDIR"
```

### 模式 4：健壯的參數解析

```bash
#!/bin/bash
set -Eeuo pipefail

# Default values
VERBOSE=false
DRY_RUN=false
OUTPUT_FILE=""
THREADS=4

usage() {
    cat <<EOF
Usage: $0 [OPTIONS]

Options:
    -v, --verbose       Enable verbose output
    -d, --dry-run       Run without making changes
    -o, --output FILE   Output file path
    -j, --jobs NUM      Number of parallel jobs
    -h, --help          Show this help message
EOF
    exit "${1:-0}"
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case "$1" in
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -d|--dry-run)
            DRY_RUN=true
            shift
            ;;
        -o|--output)
            OUTPUT_FILE="$2"
            shift 2
            ;;
        -j|--jobs)
            THREADS="$2"
            shift 2
            ;;
        -h|--help)
            usage 0
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "ERROR: Unknown option: $1" >&2
            usage 1
            ;;
    esac
done

# Validate required arguments
[[ -n "$OUTPUT_FILE" ]] || { echo "ERROR: -o/--output is required" >&2; usage 1; }
```

### 模式 5：結構化日誌記錄

```bash
#!/bin/bash
set -Eeuo pipefail

# Logging functions
log_info() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] INFO: $*" >&2
}

log_warn() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] WARN: $*" >&2
}

log_error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $*" >&2
}

log_debug() {
    if [[ "${DEBUG:-0}" == "1" ]]; then
        echo "[$(date +'%Y-%m-%d %H:%M:%S')] DEBUG: $*" >&2
    fi
}

# Usage
log_info "Starting script"
log_debug "Debug information"
log_warn "Warning message"
log_error "Error occurred"
```

### 模式 6：使用訊號進行流程編排

```bash
#!/bin/bash
set -Eeuo pipefail

# Track background processes
PIDS=()

cleanup() {
    log_info "Shutting down..."

    # Terminate all background processes
    for pid in "${PIDS[@]}"; do
        if kill -0 "$pid" 2>/dev/null; then
            kill -TERM "$pid" 2>/dev/null || true
        fi
    done

    # Wait for graceful shutdown
    for pid in "${PIDS[@]}"; do
        wait "$pid" 2>/dev/null || true
    done
}

trap cleanup SIGTERM SIGINT

# Start background tasks
background_task &
PIDS+=($!)

another_task &
PIDS+=($!)

# Wait for all background processes
wait
```

### 模式 7：安全的檔案操作

```bash
#!/bin/bash
set -Eeuo pipefail

# Use -i flag to move safely without overwriting
safe_move() {
    local -r source="$1"
    local -r dest="$2"

    if [[ ! -e "$source" ]]; then
        echo "ERROR: Source does not exist: $source" >&2
        return 1
    fi

    if [[ -e "$dest" ]]; then
        echo "ERROR: Destination already exists: $dest" >&2
        return 1
    fi

    mv "$source" "$dest"
}

# Safe directory cleanup
safe_rmdir() {
    local -r dir="$1"

    if [[ ! -d "$dir" ]]; then
        echo "ERROR: Not a directory: $dir" >&2
        return 1
    fi

    # Use -I flag to prompt before rm (BSD/GNU compatible)
    rm -rI -- "$dir"
}

# Atomic file writes
atomic_write() {
    local -r target="$1"
    local -r tmpfile
    tmpfile=$(mktemp) || return 1

    # Write to temp file first
    cat > "$tmpfile"

    # Atomic rename
    mv "$tmpfile" "$target"
}
```

### 模式 8：冪等性腳本設計

```bash
#!/bin/bash
set -Eeuo pipefail

# Check if resource already exists
ensure_directory() {
    local -r dir="$1"

    if [[ -d "$dir" ]]; then
        log_info "Directory already exists: $dir"
        return 0
    fi

    mkdir -p "$dir" || {
        log_error "Failed to create directory: $dir"
        return 1
    }

    log_info "Created directory: $dir"
}

# Ensure configuration state
ensure_config() {
    local -r config_file="$1"
    local -r default_value="$2"

    if [[ ! -f "$config_file" ]]; then
        echo "$default_value" > "$config_file"
        log_info "Created config: $config_file"
    fi
}

# Rerunning script multiple times should be safe
ensure_directory "/var/cache/myapp"
ensure_config "/etc/myapp/config" "DEBUG=false"
```

### 模式 9：安全的命令替換

```bash
#!/bin/bash
set -Eeuo pipefail

# Use $() instead of backticks
name=$(<"$file")  # Modern, safe variable assignment from file
output=$(command -v python3)  # Get command location safely

# Handle command substitution with error checking
result=$(command -v node) || {
    log_error "node command not found"
    return 1
}

# For multiple lines
mapfile -t lines < <(grep "pattern" "$file")

# NUL-safe iteration
while IFS= read -r -d '' file; do
    echo "Processing: $file"
done < <(find /path -type f -print0)
```

### 模式 10：模擬執行 (Dry-Run) 支援

```bash
#!/bin/bash
set -Eeuo pipefail

DRY_RUN="${DRY_RUN:-false}"

run_cmd() {
    if [[ "$DRY_RUN" == "true" ]]; then
        echo "[DRY RUN] Would execute: $*"
        return 0
    fi

    "$@"
}

# Usage
run_cmd cp "$source" "$dest"
run_cmd rm "$file"
run_cmd chown "$owner" "$target"
```

## 進階防禦性技術

### 具名參數模式

```bash
#!/bin/bash
set -Eeuo pipefail

process_data() {
    local input_file=""
    local output_dir=""
    local format="json"

    # Parse named parameters
    while [[ $# -gt 0 ]]; do
        case "$1" in
            --input=*)
                input_file="${1#*=}"
                ;;
            --output=*)
                output_dir="${1#*=}"
                ;;
            --format=*)
                format="${1#*=}"
                ;;
            *)
                echo "ERROR: Unknown parameter: $1" >&2
                return 1
                ;;
        esac
        shift
    done

    # Validate required parameters
    [[ -n "$input_file" ]] || { echo "ERROR: --input is required" >&2; return 1; }
    [[ -n "$output_dir" ]] || { echo "ERROR: --output is required" >&2; return 1; }
}
```

### 依賴項檢查

```bash
#!/bin/bash
set -Eeuo pipefail

check_dependencies() {
    local -a missing_deps=()
    local -a required=("jq" "curl" "git")

    for cmd in "${required[@]}"; do
        if ! command -v "$cmd" &>/dev/null; then
            missing_deps+=("$cmd")
        fi
    done

    if [[ ${#missing_deps[@]} -gt 0 ]]; then
        echo "ERROR: Missing required commands: ${missing_deps[*]}" >&2
        return 1
    fi
}

check_dependencies
```

## 最佳實踐摘要

1. **始終使用嚴格模式** - `set -Eeuo pipefail`
2. **為所有變數加上引號** - `"$variable"` 可防止字詞分割
3. **使用 [[ ]] 條件判斷** - 比 [ ] 更健壯
4. **實作錯誤捕捉** - 優雅地捕捉和處理錯誤
5. **驗證所有輸入** - 檢查檔案存在性、權限、格式
6. **使用函式提高可重用性** - 使用有意義的前綴命名
7. **實作結構化日誌記錄** - 包含時間戳記和日誌級別
8. **支援模擬執行模式** - 讓使用者能夠預覽變更
9. **安全處理暫存檔** - 使用 mktemp，透過 trap 進行清理
10. **設計為冪等性** - 腳本應該能夠安全地重複執行
11. **記錄需求** - 列出依賴項和最低版本要求
12. **測試錯誤路徑** - 確保錯誤處理正確運作
13. **使用 `command -v`** - 比 `which` 更安全地檢查可執行檔
14. **優先使用 printf 而非 echo** - 跨系統更可預測

## 參考資源

- **Bash Strict Mode**: http://redsymbol.net/articles/unofficial-bash-strict-mode/
- **Google Shell Style Guide**: https://google.github.io/styleguide/shellguide.html
- **Defensive BASH Programming**: https://www.lifepipe.net/
