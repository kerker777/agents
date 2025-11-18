---
name: sast-configuration
description: 設定靜態應用程式安全測試（SAST）工具，用於應用程式程式碼的自動化漏洞檢測。適用於設置安全掃描、實施 DevSecOps 實務或自動化程式碼漏洞檢測。
---

# SAST 配置

靜態應用程式安全測試（SAST）工具的設置、配置及自訂規則建立，用於跨多種程式語言的全面性安全掃描。

## 概述

此技能提供設置和配置 SAST 工具的全面性指引，包括 Semgrep、SonarQube 和 CodeQL。在以下情況下使用此技能：

- 在 CI/CD 管線中設置 SAST 掃描
- 為您的程式碼庫建立自訂安全規則
- 配置品質閘門和合規政策
- 優化掃描效能並減少誤報
- 整合多種 SAST 工具以實現深度防禦

## 核心功能

### 1. Semgrep 配置
- 使用模式匹配建立自訂規則
- 特定語言的安全規則（Python、JavaScript、Go、Java 等）
- CI/CD 整合（GitHub Actions、GitLab CI、Jenkins）
- 誤報調整和規則優化
- 組織政策執行

### 2. SonarQube 設置
- 品質閘門配置
- 安全熱點分析
- 程式碼覆蓋率和技術債務追蹤
- 語言自訂品質配置檔
- 企業級 LDAP/SAML 整合

### 3. CodeQL 分析
- GitHub Advanced Security 整合
- 自訂查詢開發
- 漏洞變體分析
- 安全研究工作流程
- SARIF 結果處理

## 快速開始

### 初始評估
1. 識別程式碼庫中的主要程式語言
2. 確定合規要求（PCI-DSS、SOC 2 等）
3. 根據語言支援和整合需求選擇 SAST 工具
4. 檢閱基準掃描以了解當前的安全態勢

### 基本設置
```bash
# Semgrep 快速開始
pip install semgrep
semgrep --config=auto --error

# 使用 Docker 執行 SonarQube
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest

# CodeQL CLI 設置
gh extension install github/gh-codeql
codeql database create mydb --language=python
```

## 參考文件

- [Semgrep Rule Creation](references/semgrep-rules.md) - 基於模式的安全規則開發
- [SonarQube Configuration](references/sonarqube-config.md) - 品質閘門和配置檔
- [CodeQL Setup Guide](references/codeql-setup.md) - 查詢開發和工作流程

## 範本與資源

- [semgrep-config.yml](assets/semgrep-config.yml) - 可用於生產環境的 Semgrep 配置
- [sonarqube-settings.xml](assets/sonarqube-settings.xml) - SonarQube 品質配置檔範本
- [run-sast.sh](scripts/run-sast.sh) - 自動化 SAST 執行腳本

## 整合模式

### CI/CD 管線整合
```yaml
# GitHub Actions 範例
- name: Run Semgrep
  uses: returntocorp/semgrep-action@v1
  with:
    config: >-
      p/security-audit
      p/owasp-top-ten
```

### Pre-commit Hook
```bash
# .pre-commit-config.yaml
- repo: https://github.com/returntocorp/semgrep
  rev: v1.45.0
  hooks:
    - id: semgrep
      args: ['--config=auto', '--error']
```

## 最佳實務

1. **從基準開始**
   - 執行初始掃描以建立安全基準
   - 優先處理關鍵和高嚴重性的發現
   - 建立補救路線圖

2. **漸進式採用**
   - 從以安全為重點的規則開始
   - 逐步新增程式碼品質規則
   - 僅對關鍵問題實施阻擋

3. **誤報管理**
   - 記錄合理的抑制項目
   - 為已知安全模式建立允許清單
   - 定期檢閱被抑制的發現

4. **效能優化**
   - 排除測試檔案和產生的程式碼
   - 對大型程式碼庫使用增量掃描
   - 在 CI/CD 中快取掃描結果

5. **團隊賦能**
   - 為開發人員提供安全訓練
   - 為常見模式建立內部文件
   - 建立安全推廣大使計畫

## 常見使用案例

### 新專案設置
```bash
./scripts/run-sast.sh --setup --language python --tools semgrep,sonarqube
```

### 自訂規則開發
```yaml
# 詳細範例請參閱 references/semgrep-rules.md
rules:
  - id: hardcoded-jwt-secret
    pattern: jwt.encode($DATA, "...", ...)
    message: JWT secret should not be hardcoded
    severity: ERROR
```

### 合規掃描
```bash
# 專注於 PCI-DSS 的掃描
semgrep --config p/pci-dss --json -o pci-scan-results.json
```

## 疑難排解

### 高誤報率
- 檢閱並調整規則敏感度
- 新增路徑過濾器以排除測試檔案
- 對於雜訊模式使用 nostmt 元資料
- 建立組織特定的規則例外

### 效能問題
- 啟用增量掃描
- 跨模組平行化掃描
- 優化規則模式以提升效率
- 快取相依性和掃描結果

### 整合失敗
- 驗證 API 權杖和憑證
- 檢查網路連線和代理設定
- 檢閱 SARIF 輸出格式相容性
- 驗證 CI/CD runner 權限

## 相關技能

- [OWASP Top 10 Checklist](../owasp-top10-checklist/SKILL.md)
- [Container Security](../container-security/SKILL.md)
- [Dependency Scanning](../dependency-scanning/SKILL.md)

## 工具比較

| 工具 | 最適用於 | 語言支援 | 成本 | 整合性 |
|------|----------|----------|------|--------|
| Semgrep | 自訂規則、快速掃描 | 30+ 種語言 | 免費/企業版 | 優秀 |
| SonarQube | 程式碼品質 + 安全性 | 25+ 種語言 | 免費/商業版 | 良好 |
| CodeQL | 深度分析、研究 | 10+ 種語言 | 免費（開源專案） | GitHub 原生 |

## 下一步

1. 完成初始 SAST 工具設置
2. 執行基準安全掃描
3. 為組織特定模式建立自訂規則
4. 整合至 CI/CD 管線
5. 建立安全閘門政策
6. 訓練開發團隊了解發現和補救措施
