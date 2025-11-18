---
name: deployment-pipeline-design
description: 設計具有核准關卡、安全檢查和部署編排功能的多階段 CI/CD 流水線。適用於規劃部署工作流程、建立持續交付機制或實作 GitOps 實務時使用。
---

# Deployment Pipeline Design

多階段 CI/CD 流水線的架構模式，包含核准關卡和部署策略。

## 目的

設計穩健、安全的部署流水線，透過適當的階段組織和核准工作流程，在速度與安全性之間取得平衡。

## 使用時機

- 設計 CI/CD 架構
- 實作部署關卡
- 配置多環境流水線
- 建立部署最佳實務
- 實作漸進式交付

## 流水線階段

### 標準流水線流程

```
┌─────────┐   ┌──────┐   ┌─────────┐   ┌────────┐   ┌──────────┐
│  Build  │ → │ Test │ → │ Staging │ → │ Approve│ → │Production│
└─────────┘   └──────┘   └─────────┘   └────────┘   └──────────┘
```

### 詳細階段分解

1. **Source** - 程式碼簽出
2. **Build** - 編譯、打包、容器化
3. **Test** - 單元測試、整合測試、安全掃描
4. **Staging Deploy** - 部署至測試環境
5. **Integration Tests** - E2E、冒煙測試
6. **Approval Gate** - 需要手動核准
7. **Production Deploy** - 金絲雀、藍綠、滾動部署
8. **Verification** - 健康檢查、監控
9. **Rollback** - 失敗時自動回滾

## 核准關卡模式

### 模式 1：手動核准

```yaml
# GitHub Actions
production-deploy:
  needs: staging-deploy
  environment:
    name: production
    url: https://app.example.com
  runs-on: ubuntu-latest
  steps:
    - name: Deploy to production
      run: |
        # Deployment commands
```

### 模式 2：基於時間的核准

```yaml
# GitLab CI
deploy:production:
  stage: deploy
  script:
    - deploy.sh production
  environment:
    name: production
  when: delayed
  start_in: 30 minutes
  only:
    - main
```

### 模式 3：多人核准

```yaml
# Azure Pipelines
stages:
- stage: Production
  dependsOn: Staging
  jobs:
  - deployment: Deploy
    environment:
      name: production
      resourceType: Kubernetes
    strategy:
      runOnce:
        preDeploy:
          steps:
          - task: ManualValidation@0
            inputs:
              notifyUsers: 'team-leads@example.com'
              instructions: 'Review staging metrics before approving'
```

**參考：** 請見 `assets/approval-gate-template.yml`

## 部署策略

### 1. 滾動部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
```

**特性：**
- 逐步推出
- 零停機時間
- 容易回滾
- 最適合大多數應用程式

### 2. 藍綠部署

```yaml
# Blue (current)
kubectl apply -f blue-deployment.yaml
kubectl label service my-app version=blue

# Green (new)
kubectl apply -f green-deployment.yaml
# Test green environment
kubectl label service my-app version=green

# Rollback if needed
kubectl label service my-app version=blue
```

**特性：**
- 即時切換
- 容易回滾
- 暫時性地增加一倍基礎設施成本
- 適合高風險部署

### 3. 金絲雀部署

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 5m}
      - setWeight: 25
      - pause: {duration: 5m}
      - setWeight: 50
      - pause: {duration: 5m}
      - setWeight: 100
```

**特性：**
- 漸進式流量轉移
- 風險降低
- 真實使用者測試
- 需要服務網格或類似技術

### 4. 功能開關

```python
from flagsmith import Flagsmith

flagsmith = Flagsmith(environment_key="API_KEY")

if flagsmith.has_feature("new_checkout_flow"):
    # New code path
    process_checkout_v2()
else:
    # Existing code path
    process_checkout_v1()
```

**特性：**
- 部署與發布分離
- A/B 測試
- 即時回滾
- 精細控制

## 流水線編排

### 多階段流水線範例

```yaml
name: Production Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build application
        run: make build
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push to registry
        run: docker push myapp:${{ github.sha }}

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Unit tests
        run: make test
      - name: Security scan
        run: trivy image myapp:${{ github.sha }}

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    environment:
      name: staging
    steps:
      - name: Deploy to staging
        run: kubectl apply -f k8s/staging/

  integration-test:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: Run E2E tests
        run: npm run test:e2e

  deploy-production:
    needs: integration-test
    runs-on: ubuntu-latest
    environment:
      name: production
    steps:
      - name: Canary deployment
        run: |
          kubectl apply -f k8s/production/
          kubectl argo rollouts promote my-app

  verify:
    needs: deploy-production
    runs-on: ubuntu-latest
    steps:
      - name: Health check
        run: curl -f https://app.example.com/health
      - name: Notify team
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text":"Production deployment successful!"}'
```

## 流水線最佳實務

1. **快速失敗** - 優先執行快速測試
2. **平行執行** - 同時執行獨立的任務
3. **快取** - 在執行之間快取相依套件
4. **產出物管理** - 儲存建置產出物
5. **環境一致性** - 保持各環境的一致性
6. **秘密管理** - 使用秘密存放區（Vault 等）
7. **部署時段** - 適當地安排部署時間
8. **監控整合** - 追蹤部署指標
9. **自動回滾** - 失敗時自動回滾
10. **文件化** - 記錄流水線各階段

## 回滾策略

### 自動回滾

```yaml
deploy-and-verify:
  steps:
    - name: Deploy new version
      run: kubectl apply -f k8s/

    - name: Wait for rollout
      run: kubectl rollout status deployment/my-app

    - name: Health check
      id: health
      run: |
        for i in {1..10}; do
          if curl -sf https://app.example.com/health; then
            exit 0
          fi
          sleep 10
        done
        exit 1

    - name: Rollback on failure
      if: failure()
      run: kubectl rollout undo deployment/my-app
```

### 手動回滾

```bash
# List revision history
kubectl rollout history deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific revision
kubectl rollout undo deployment/my-app --to-revision=3
```

## 監控與指標

### 關鍵流水線指標

- **Deployment Frequency** - 部署頻率
- **Lead Time** - 從提交到正式環境的前置時間
- **Change Failure Rate** - 部署失敗率
- **Mean Time to Recovery (MTTR)** - 從故障中恢復的平均時間
- **Pipeline Success Rate** - 流水線成功率
- **Average Pipeline Duration** - 流水線平均執行時間

### 與監控系統整合

```yaml
- name: Post-deployment verification
  run: |
    # Wait for metrics stabilization
    sleep 60

    # Check error rate
    ERROR_RATE=$(curl -s "$PROMETHEUS_URL/api/v1/query?query=rate(http_errors_total[5m])" | jq '.data.result[0].value[1]')

    if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
      echo "Error rate too high: $ERROR_RATE"
      exit 1
    fi
```

## 參考檔案

- `references/pipeline-orchestration.md` - 複雜流水線模式
- `assets/approval-gate-template.yml` - 核准工作流程範本

## 相關技能

- `github-actions-templates` - 用於 GitHub Actions 實作
- `gitlab-ci-patterns` - 用於 GitLab CI 實作
- `secrets-management` - 用於秘密處理
