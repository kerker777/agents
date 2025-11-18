---
description: Static Application Security Testing (SAST) 用於跨多種語言和框架的程式碼漏洞分析
globs: ['**/*.py', '**/*.js', '**/*.ts', '**/*.java', '**/*.rb', '**/*.go', '**/*.rs', '**/*.php']
keywords: [sast, static analysis, code security, vulnerability scanning, bandit, semgrep, eslint, sonarqube, codeql, security patterns, code review, ast analysis]
---

# SAST 安全外掛

靜態應用程式安全測試（SAST）用於全面檢測跨多種語言、框架和安全模式的程式碼漏洞。

## 功能特色

- **多語言 SAST**：支援 Python、JavaScript/TypeScript、Java、Ruby、PHP、Go、Rust
- **工具整合**：Bandit、Semgrep、ESLint Security、SonarQube、CodeQL、PMD、SpotBugs、Brakeman、gosec、cargo-clippy
- **漏洞模式**：SQL 注入、XSS、硬編碼機密資訊、路徑遍歷、IDOR、CSRF、不安全的反序列化
- **框架分析**：Django、Flask、React、Express、Spring Boot、Rails、Laravel
- **自訂規則撰寫**：開發 Semgrep 模式以符合組織特定的安全政策

## 使用時機

適用於程式碼審查安全分析、注入漏洞檢測、硬編碼機密資訊掃描、框架特定模式檢查、自訂安全政策執行、部署前驗證、遺留程式碼評估，以及合規性檢查（OWASP、PCI-DSS、SOC2）。

**專門工具**：使用 `security-secrets.md` 進行進階憑證掃描，使用 `security-owasp.md` 進行 Top 10 對應，使用 `security-api.md` 進行 REST/GraphQL 端點檢查。

## SAST 工具選擇

### Python：Bandit

```bash
# 安裝與掃描
pip install bandit
bandit -r . -f json -o bandit-report.json
bandit -r . -ll -ii -f json  # 僅高危/嚴重等級
```

**設定檔**：`.bandit`
```yaml
exclude_dirs: ['/tests/', '/venv/', '/.tox/', '/build/']
tests: [B201, B301, B302, B303, B304, B305, B307, B308, B312, B323, B324, B501, B502, B506, B602, B608]
skips: [B101]
```

### JavaScript/TypeScript：ESLint Security

```bash
npm install --save-dev eslint @eslint/plugin-security eslint-plugin-no-secrets
eslint . --ext .js,.jsx,.ts,.tsx --format json > eslint-security.json
```

**設定檔**：`.eslintrc-security.json`
```json
{
  "plugins": ["@eslint/plugin-security", "eslint-plugin-no-secrets"],
  "extends": ["plugin:security/recommended"],
  "rules": {
    "security/detect-object-injection": "error",
    "security/detect-non-literal-fs-filename": "error",
    "security/detect-eval-with-expression": "error",
    "security/detect-pseudo-random-prng": "error",
    "no-secrets/no-secrets": "error"
  }
}
```

### 多語言：Semgrep

```bash
pip install semgrep
semgrep --config=auto --json --output=semgrep-report.json
semgrep --config=p/security-audit --json
semgrep --config=p/owasp-top-ten --json
semgrep ci --config=auto  # CI 模式
```

**自訂規則**：`.semgrep.yml`
```yaml
rules:
  - id: sql-injection-format-string
    pattern: cursor.execute("... %s ..." % $VAR)
    message: 透過字串格式化的 SQL 注入
    severity: ERROR
    languages: [python]
    metadata:
      cwe: "CWE-89"
      owasp: "A03:2021-Injection"

  - id: dangerous-innerHTML
    pattern: $ELEM.innerHTML = $VAR
    message: 透過 innerHTML 賦值的 XSS 攻擊
    severity: ERROR
    languages: [javascript, typescript]
    metadata:
      cwe: "CWE-79"

  - id: hardcoded-aws-credentials
    patterns:
      - pattern: $KEY = "AKIA..."
      - metavariable-regex:
          metavariable: $KEY
          regex: "(aws_access_key_id|AWS_ACCESS_KEY_ID)"
    message: 偵測到硬編碼的 AWS 憑證
    severity: ERROR
    languages: [python, javascript, java]

  - id: path-traversal-open
    patterns:
      - pattern: open($PATH, ...)
      - pattern-not: open(os.path.join(SAFE_DIR, ...), ...)
      - metavariable-pattern:
          metavariable: $PATH
          patterns:
            - pattern: $REQ.get(...)
    message: 透過使用者輸入的路徑遍歷攻擊
    severity: ERROR
    languages: [python]

  - id: command-injection
    patterns:
      - pattern-either:
          - pattern: os.system($CMD)
          - pattern: subprocess.call($CMD, shell=True)
      - metavariable-pattern:
          metavariable: $CMD
          patterns:
            - pattern-either:
                - pattern: $X + $Y
                - pattern: f"...{$VAR}..."
    message: 透過 shell=True 的命令注入攻擊
    severity: ERROR
    languages: [python]
```

### 其他語言工具

**Java**：`mvn spotbugs:check`
**Ruby**：`brakeman -o report.json -f json`
**Go**：`gosec -fmt=json -out=gosec.json ./...`
**Rust**：`cargo clippy -- -W clippy::unwrap_used`

## 漏洞模式

### SQL 注入

**有漏洞**：在 SQL 查詢中使用字串格式化/串接搭配使用者輸入

**安全做法**：
```python
# 參數化查詢
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
User.objects.filter(id=user_id)  # ORM
```

### 跨站腳本攻擊（XSS）

**有漏洞**：使用未消毒的使用者輸入直接操作 HTML（innerHTML、outerHTML、document.write）

**安全做法**：
```javascript
// 純文字使用 textContent
element.textContent = userInput;

// React 自動跳脫
<div>{userInput}</div>

// 需要 HTML 時進行消毒
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

### 硬編碼機密資訊

**有漏洞**：在原始碼中硬編碼 API 金鑰、密碼、權杖

**安全做法**：
```python
import os
API_KEY = os.environ.get('API_KEY')
PASSWORD = os.getenv('DB_PASSWORD')
```

### 路徑遍歷

**有漏洞**：使用未消毒的使用者輸入開啟檔案

**安全做法**：
```python
import os
ALLOWED_DIR = '/var/www/uploads'
file_name = request.args.get('file')
file_path = os.path.join(ALLOWED_DIR, file_name)
file_path = os.path.realpath(file_path)
if not file_path.startswith(os.path.realpath(ALLOWED_DIR)):
    raise ValueError("Invalid file path")
with open(file_path, 'r') as f:
    content = f.read()
```

### 不安全的反序列化

**有漏洞**：對不受信任的資料使用 pickle.loads()、yaml.load()

**安全做法**：
```python
import json
data = json.loads(user_input)  # 安全
import yaml
config = yaml.safe_load(user_input)  # 安全
```

### 命令注入

**有漏洞**：使用 os.system() 或帶有 shell=True 和使用者輸入的 subprocess

**安全做法**：
```python
subprocess.run(['ping', '-c', '4', user_input])  # 陣列參數
import shlex
safe_input = shlex.quote(user_input)  # 輸入驗證
```

### 不安全的亂數

**有漏洞**：在安全關鍵操作中使用 random 模組

**安全做法**：
```python
import secrets
token = secrets.token_hex(16)
session_id = secrets.token_urlsafe(32)
```

## 框架安全

### Django

**有漏洞**：@csrf_exempt、DEBUG=True、弱 SECRET_KEY、缺少安全中介軟體

**安全做法**：
```python
# settings.py
DEBUG = False
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
X_FRAME_OPTIONS = 'DENY'
```

### Flask

**有漏洞**：debug=True、弱 secret_key、CORS 萬用字元

**安全做法**：
```python
import os
from flask_talisman import Talisman

app.secret_key = os.environ.get('FLASK_SECRET_KEY')
Talisman(app, force_https=True)
CORS(app, origins=['https://example.com'])
```

### Express.js

**有漏洞**：缺少 helmet、CORS 萬用字元、無速率限制

**安全做法**：
```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

app.use(helmet());
app.use(cors({ origin: 'https://example.com' }));
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
```

## 多語言掃描器實作

```python
import json
import subprocess
from pathlib import Path
from typing import Dict, List, Any
from dataclasses import dataclass
from datetime import datetime

@dataclass
class SASTFinding:
    tool: str
    severity: str
    category: str
    title: str
    description: str
    file_path: str
    line_number: int
    cwe: str
    owasp: str
    confidence: str

class MultiLanguageSASTScanner:
    def __init__(self, project_path: str):
        self.project_path = Path(project_path)
        self.findings: List[SASTFinding] = []

    def detect_languages(self) -> List[str]:
        """自動偵測語言"""
        languages = []
        indicators = {
            'python': ['*.py', 'requirements.txt'],
            'javascript': ['*.js', 'package.json'],
            'typescript': ['*.ts', 'tsconfig.json'],
            'java': ['*.java', 'pom.xml'],
            'ruby': ['*.rb', 'Gemfile'],
            'go': ['*.go', 'go.mod'],
            'rust': ['*.rs', 'Cargo.toml'],
        }
        for lang, patterns in indicators.items():
            for pattern in patterns:
                if list(self.project_path.glob(f'**/{pattern}')):
                    languages.append(lang)
                    break
        return languages

    def run_comprehensive_sast(self) -> Dict[str, Any]:
        """執行所有適用的 SAST 工具"""
        languages = self.detect_languages()

        scan_results = {
            'timestamp': datetime.now().isoformat(),
            'languages': languages,
            'tools_executed': [],
            'findings': []
        }

        self.run_semgrep_scan()
        scan_results['tools_executed'].append('semgrep')

        if 'python' in languages:
            self.run_bandit_scan()
            scan_results['tools_executed'].append('bandit')
        if 'javascript' in languages or 'typescript' in languages:
            self.run_eslint_security_scan()
            scan_results['tools_executed'].append('eslint-security')

        scan_results['findings'] = [vars(f) for f in self.findings]
        scan_results['summary'] = self.generate_summary()
        return scan_results

    def run_semgrep_scan(self):
        """執行 Semgrep"""
        for ruleset in ['auto', 'p/security-audit', 'p/owasp-top-ten']:
            try:
                result = subprocess.run([
                    'semgrep', '--config', ruleset, '--json', '--quiet',
                    str(self.project_path)
                ], capture_output=True, text=True, timeout=300)

                if result.stdout:
                    data = json.loads(result.stdout)
                    for f in data.get('results', []):
                        self.findings.append(SASTFinding(
                            tool='semgrep',
                            severity=f.get('extra', {}).get('severity', 'MEDIUM').upper(),
                            category='sast',
                            title=f.get('check_id', ''),
                            description=f.get('extra', {}).get('message', ''),
                            file_path=f.get('path', ''),
                            line_number=f.get('start', {}).get('line', 0),
                            cwe=f.get('extra', {}).get('metadata', {}).get('cwe', ''),
                            owasp=f.get('extra', {}).get('metadata', {}).get('owasp', ''),
                            confidence=f.get('extra', {}).get('metadata', {}).get('confidence', 'MEDIUM')
                        ))
            except Exception as e:
                print(f"Semgrep {ruleset} 執行失敗：{e}")

    def generate_summary(self) -> Dict[str, Any]:
        """產生統計資料"""
        severity_counts = {'CRITICAL': 0, 'HIGH': 0, 'MEDIUM': 0, 'LOW': 0}
        for f in self.findings:
            severity_counts[f.severity] = severity_counts.get(f.severity, 0) + 1

        return {
            'total_findings': len(self.findings),
            'severity_breakdown': severity_counts,
            'risk_score': self.calculate_risk_score(severity_counts)
        }

    def calculate_risk_score(self, severity_counts: Dict[str, int]) -> int:
        """風險評分 0-100"""
        weights = {'CRITICAL': 10, 'HIGH': 7, 'MEDIUM': 4, 'LOW': 1}
        total = sum(weights[s] * c for s, c in severity_counts.items())
        return min(100, int((total / 50) * 100))
```

## CI/CD 整合

### GitHub Actions

```yaml
name: SAST Scan
on:
  pull_request:
    branches: [main]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install tools
        run: |
          pip install bandit semgrep
          npm install -g eslint @eslint/plugin-security

      - name: Run scans
        run: |
          bandit -r . -f json -o bandit.json || true
          semgrep --config=auto --json --output=semgrep.json || true

      - name: Upload reports
        uses: actions/upload-artifact@v3
        with:
          name: sast-reports
          path: |
            bandit.json
            semgrep.json
```

### GitLab CI

```yaml
sast:
  stage: test
  image: python:3.11
  script:
    - pip install bandit semgrep
    - bandit -r . -f json -o bandit.json || true
    - semgrep --config=auto --json --output=semgrep.json || true
  artifacts:
    reports:
      sast: bandit.json
```

## 最佳實踐

1. **盡早且頻繁執行** - 使用 pre-commit hooks 和 CI/CD
2. **結合多種工具** - 不同工具可捕捉不同的漏洞
3. **調整誤報** - 設定排除項目和閾值
4. **優先處理發現** - 先專注於嚴重/高危等級
5. **框架感知掃描** - 使用特定規則集
6. **自訂規則** - 組織特定的模式
7. **開發人員培訓** - 安全編碼實踐
8. **漸進式修復** - 逐步修正
9. **基線管理** - 追蹤已知問題
10. **定期更新** - 保持工具最新

## 相關工具

- **security-secrets.md** - 進階憑證偵測
- **security-owasp.md** - OWASP Top 10 評估
- **security-api.md** - API 安全測試
- **security-scan.md** - 全面安全掃描
