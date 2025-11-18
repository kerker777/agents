---
name: github-actions-templates
description: 建立用於自動化測試、建置和部署應用程式的生產就緒 GitHub Actions 工作流程。適用於使用 GitHub Actions 建立 CI/CD、自動化開發工作流程或建立可重用工作流程範本時使用。
---

# GitHub Actions Templates

適用於測試、建置和部署應用程式的生產就緒 GitHub Actions 工作流程模式。

## 目的

為各種技術堆疊建立高效、安全的 GitHub Actions 工作流程，實現持續整合和部署。

## 使用時機

- 自動化測試與部署
- 建置 Docker 映像檔並推送至容器註冊表
- 部署至 Kubernetes 叢集
- 執行安全掃描
- 為多環境實作矩陣建置

## 常見工作流程模式

### 模式 1：測試工作流程

```yaml
name: Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linter
      run: npm run lint

    - name: Run tests
      run: npm test

    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage/lcov.info
```

**參考：** 請見 `assets/test-workflow.yml`

### 模式 2：建置並推送 Docker 映像檔

```yaml
name: Build and Push

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v4

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}

    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

**參考：** 請見 `assets/deploy-workflow.yml`

### 模式 3：部署至 Kubernetes

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --name production-cluster --region us-west-2

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/
        kubectl rollout status deployment/my-app -n production
        kubectl get services -n production

    - name: Verify deployment
      run: |
        kubectl get pods -n production
        kubectl describe deployment my-app -n production
```

### 模式 4：矩陣建置

```yaml
name: Matrix Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.9', '3.10', '3.11', '3.12']

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run tests
      run: pytest
```

**參考：** 請見 `assets/matrix-build.yml`

## 工作流程最佳實務

1. **使用特定的 action 版本**（@v4，而非 @latest）
2. **快取相依套件**以加速建置
3. **使用 secrets** 處理敏感資料
4. **實作狀態檢查**於 PR 上
5. **使用矩陣建置**進行多版本測試
6. **設定適當的權限**
7. **使用可重用工作流程**處理常見模式
8. **實作核准關卡**用於正式環境
9. **新增通知步驟**於失敗時
10. **使用自託管執行器**處理敏感工作負載

## 可重用工作流程

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
    secrets:
      NPM_TOKEN:
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci
    - run: npm test
```

**使用可重用工作流程：**
```yaml
jobs:
  call-test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20.x'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 安全掃描

```yaml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy results to GitHub Security
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

    - name: Run Snyk Security Scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

## 需核准的部署

```yaml
name: Deploy to Production

on:
  push:
    tags: [ 'v*' ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com

    steps:
    - uses: actions/checkout@v4

    - name: Deploy application
      run: |
        echo "Deploying to production..."
        # Deployment commands here

    - name: Notify Slack
      if: success()
      uses: slackapi/slack-github-action@v1
      with:
        webhook-url: ${{ secrets.SLACK_WEBHOOK }}
        payload: |
          {
            "text": "Deployment to production completed successfully!"
          }
```

## 參考檔案

- `assets/test-workflow.yml` - 測試工作流程範本
- `assets/deploy-workflow.yml` - 部署工作流程範本
- `assets/matrix-build.yml` - 矩陣建置範本
- `references/common-workflows.md` - 常見工作流程模式

## 相關技能

- `gitlab-ci-patterns` - 用於 GitLab CI 工作流程
- `deployment-pipeline-design` - 用於流水線架構
- `secrets-management` - 用於秘密處理
