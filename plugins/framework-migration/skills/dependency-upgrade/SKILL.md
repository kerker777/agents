---
name: dependency-upgrade
description: 透過相容性分析、分階段推出和全面測試來管理主要依賴項版本升級。適用於升級框架版本、更新主要依賴項或管理函式庫的破壞性變更。
---

# Dependency Upgrade

精通主要依賴項版本升級、相容性分析、分階段升級策略和全面測試方法。

## 何時使用此技能

- 升級主要框架版本
- 更新具有安全性漏洞的依賴項
- 現代化舊版依賴項
- 解決依賴項衝突
- 規劃漸進式升級路徑
- 測試相容性矩陣
- 自動化依賴項更新

## 語義版本控制回顧

```
MAJOR.MINOR.PATCH (例如：2.3.1)

MAJOR：破壞性變更
MINOR：新功能，向後相容
PATCH：錯誤修復，向後相容

^2.3.1 = >=2.3.1 <3.0.0 (minor updates)
~2.3.1 = >=2.3.1 <2.4.0 (patch updates)
2.3.1 = exact version
```

## 依賴項分析

### 稽核依賴項
```bash
# npm
npm outdated
npm audit
npm audit fix

# yarn
yarn outdated
yarn audit

# Check for major updates
npx npm-check-updates
npx npm-check-updates -u  # Update package.json
```

### 分析依賴樹
```bash
# See why a package is installed
npm ls package-name
yarn why package-name

# Find duplicate packages
npm dedupe
yarn dedupe

# Visualize dependencies
npx madge --image graph.png src/
```

## 相容性矩陣

```javascript
// compatibility-matrix.js
const compatibilityMatrix = {
  'react': {
    '16.x': {
      'react-dom': '^16.0.0',
      'react-router-dom': '^5.0.0',
      '@testing-library/react': '^11.0.0'
    },
    '17.x': {
      'react-dom': '^17.0.0',
      'react-router-dom': '^5.0.0 || ^6.0.0',
      '@testing-library/react': '^12.0.0'
    },
    '18.x': {
      'react-dom': '^18.0.0',
      'react-router-dom': '^6.0.0',
      '@testing-library/react': '^13.0.0'
    }
  }
};

function checkCompatibility(packages) {
  // Validate package versions against matrix
}
```

## 分階段升級策略

### 階段 1：規劃
```bash
# 1. Identify current versions
npm list --depth=0

# 2. Check for breaking changes
# Read CHANGELOG.md and MIGRATION.md

# 3. Create upgrade plan
echo "Upgrade order:
1. TypeScript
2. React
3. React Router
4. Testing libraries
5. Build tools" > UPGRADE_PLAN.md
```

### 階段 2：漸進式更新
```bash
# Don't upgrade everything at once!

# Step 1: Update TypeScript
npm install typescript@latest

# Test
npm run test
npm run build

# Step 2: Update React (one major version at a time)
npm install react@17 react-dom@17

# Test again
npm run test

# Step 3: Continue with other packages
npm install react-router-dom@6

# And so on...
```

### 階段 3：驗證
```javascript
// tests/compatibility.test.js
describe('Dependency Compatibility', () => {
  it('should have compatible React versions', () => {
    const reactVersion = require('react/package.json').version;
    const reactDomVersion = require('react-dom/package.json').version;

    expect(reactVersion).toBe(reactDomVersion);
  });

  it('should not have peer dependency warnings', () => {
    // Run npm ls and check for warnings
  });
});
```

## 破壞性變更處理

### 識別破壞性變更
```bash
# Use changelog parsers
npx changelog-parser react 16.0.0 17.0.0

# Or manually check
curl https://raw.githubusercontent.com/facebook/react/main/CHANGELOG.md
```

### 使用 Codemod 進行自動修復
```bash
# React upgrade codemods
npx react-codeshift <transform> <path>

# Example: Update lifecycle methods
npx react-codeshift \
  --parser tsx \
  --transform react-codeshift/transforms/rename-unsafe-lifecycles.js \
  src/
```

### 自訂遷移腳本
```javascript
// migration-script.js
const fs = require('fs');
const glob = require('glob');

glob('src/**/*.tsx', (err, files) => {
  files.forEach(file => {
    let content = fs.readFileSync(file, 'utf8');

    // Replace old API with new API
    content = content.replace(
      /componentWillMount/g,
      'UNSAFE_componentWillMount'
    );

    // Update imports
    content = content.replace(
      /import { Component } from 'react'/g,
      "import React, { Component } from 'react'"
    );

    fs.writeFileSync(file, content);
  });
});
```

## 測試策略

### 單元測試
```javascript
// Ensure tests pass before and after upgrade
npm run test

// Update test utilities if needed
npm install @testing-library/react@latest
```

### 整合測試
```javascript
// tests/integration/app.test.js
describe('App Integration', () => {
  it('should render without crashing', () => {
    render(<App />);
  });

  it('should handle navigation', () => {
    const { getByText } = render(<App />);
    fireEvent.click(getByText('Navigate'));
    expect(screen.getByText('New Page')).toBeInTheDocument();
  });
});
```

### 視覺回歸測試
```javascript
// visual-regression.test.js
describe('Visual Regression', () => {
  it('should match snapshot', () => {
    const { container } = render(<App />);
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

### E2E 測試
```javascript
// cypress/e2e/app.cy.js
describe('E2E Tests', () => {
  it('should complete user flow', () => {
    cy.visit('/');
    cy.get('[data-testid="login"]').click();
    cy.get('input[name="email"]').type('user@example.com');
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

## 自動化依賴項更新

### Renovate 設定
```json
// renovate.json
{
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    },
    {
      "matchUpdateTypes": ["major"],
      "automerge": false,
      "labels": ["major-update"]
    }
  ],
  "schedule": ["before 3am on Monday"],
  "timezone": "America/New_York"
}
```

### Dependabot 設定
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
    reviewers:
      - "team-leads"
    commit-message:
      prefix: "chore"
      include: "scope"
```

## 回滾計畫

```javascript
// rollback.sh
#!/bin/bash

# Save current state
git stash
git checkout -b upgrade-branch

# Attempt upgrade
npm install package@latest

# Run tests
if npm run test; then
  echo "Upgrade successful"
  git add package.json package-lock.json
  git commit -m "chore: upgrade package"
else
  echo "Upgrade failed, rolling back"
  git checkout main
  git branch -D upgrade-branch
  npm install  # Restore from package-lock.json
fi
```

## 常見升級模式

### 鎖定檔案管理
```bash
# npm
npm install --package-lock-only  # Update lock file only
npm ci  # Clean install from lock file

# yarn
yarn install --frozen-lockfile  # CI mode
yarn upgrade-interactive  # Interactive upgrades
```

### Peer Dependency 解析
```bash
# npm 7+: strict peer dependencies
npm install --legacy-peer-deps  # Ignore peer deps

# npm 8+: override peer dependencies
npm install --force
```

### Workspace 升級
```bash
# Update all workspace packages
npm install --workspaces

# Update specific workspace
npm install package@latest --workspace=packages/app
```

## 資源

- **references/semver.md**：語義版本控制指南
- **references/compatibility-matrix.md**：常見相容性問題
- **references/staged-upgrades.md**：漸進式升級策略
- **references/testing-strategy.md**：全面測試方法
- **assets/upgrade-checklist.md**：逐步檢查清單
- **assets/compatibility-matrix.csv**：版本相容性表
- **scripts/audit-dependencies.sh**：依賴項稽核腳本

## 最佳實務

1. **閱讀變更日誌**：了解變更內容
2. **漸進式升級**：一次升級一個主要版本
3. **徹底測試**：單元、整合、E2E 測試
4. **檢查 Peer Dependencies**：盡早解決衝突
5. **使用鎖定檔案**：確保可重現的安裝
6. **自動化更新**：使用 Renovate 或 Dependabot
7. **監控**：升級後注意執行時期錯誤
8. **撰寫文件**：保留升級筆記

## 升級檢查清單

```markdown
升級前：
- [ ] 檢視目前依賴項版本
- [ ] 閱讀破壞性變更的變更日誌
- [ ] 建立功能分支
- [ ] 備份目前狀態 (git tag)
- [ ] 執行完整測試套件（基準）

升級期間：
- [ ] 一次升級一個依賴項
- [ ] 更新 peer dependencies
- [ ] 修復 TypeScript 錯誤
- [ ] 必要時更新測試
- [ ] 每次升級後執行測試套件
- [ ] 檢查套件大小影響

升級後：
- [ ] 完整回歸測試
- [ ] 效能測試
- [ ] 更新文件
- [ ] 部署至測試環境
- [ ] 監控錯誤
- [ ] 部署至正式環境
```

## 常見陷阱

- 一次升級所有依賴項
- 每次升級後未進行測試
- 忽略 peer dependency 警告
- 忘記更新鎖定檔案
- 未閱讀破壞性變更說明
- 跳過主要版本
- 沒有回滾計畫
