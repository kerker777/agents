---
name: gitops-workflow
description: Implement GitOps workflows with ArgoCD and Flux for automated, declarative Kubernetes deployments with continuous reconciliation. Use when implementing GitOps practices, automating Kubernetes deployments, or setting up declarative infrastructure management.
---

# GitOps 工作流程

使用 ArgoCD 和 Flux 實施 GitOps 工作流程，實現自動化 Kubernetes 部署的完整指南。

## 目的

使用 ArgoCD 或 Flux CD 為 Kubernetes 實施宣告式、基於 Git 的持續交付，遵循 OpenGitOps 原則。

## 何時使用此技能

- 為 Kubernetes 叢集設定 GitOps
- 從 Git 自動化應用程式部署
- 實施漸進式交付策略
- 管理多叢集部署
- 配置自動同步策略
- 在 GitOps 中設定密鑰管理

## OpenGitOps 原則

1. **宣告式** - 整個系統以宣告方式描述
2. **版本化與不可變** - 所需狀態儲存於 Git
3. **自動拉取** - 軟體代理拉取所需狀態
4. **持續協調** - 代理協調實際與所需狀態

## ArgoCD 設定

### 1. 安裝

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**參考資料：**詳細設定請參閱 `references/argocd-setup.md`

### 2. 儲存庫結構

```
gitops-repo/
├── apps/
│   ├── production/
│   │   ├── app1/
│   │   │   ├── kustomization.yaml
│   │   │   └── deployment.yaml
│   │   └── app2/
│   └── staging/
├── infrastructure/
│   ├── ingress-nginx/
│   ├── cert-manager/
│   └── monitoring/
└── argocd/
    ├── applications/
    └── projects/
```

### 3. 建立 Application

```yaml
# argocd/applications/my-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops-repo
    targetRevision: main
    path: apps/production/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### 4. App of Apps 模式

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: applications
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops-repo
    targetRevision: main
    path: argocd/applications
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated: {}
```

## Flux CD 設定

### 1. 安裝

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Bootstrap Flux
flux bootstrap github \
  --owner=org \
  --repository=gitops-repo \
  --branch=main \
  --path=clusters/production \
  --personal
```

### 2. 建立 GitRepository

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/org/my-app
  ref:
    branch: main
```

### 3. 建立 Kustomization

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m
  path: ./deploy
  prune: true
  sourceRef:
    kind: GitRepository
    name: my-app
```

## 同步策略

### 自動同步配置

**ArgoCD：**
```yaml
syncPolicy:
  automated:
    prune: true      # Delete resources not in Git
    selfHeal: true   # Reconcile manual changes
    allowEmpty: false
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

**Flux：**
```yaml
spec:
  interval: 1m
  prune: true
  wait: true
  timeout: 5m
```

**參考資料：**請參閱 `references/sync-policies.md`

## 漸進式交付

### 使用 ArgoCD Rollouts 的金絲雀部署

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 1m}
      - setWeight: 50
      - pause: {duration: 2m}
      - setWeight: 100
```

### 藍綠部署

```yaml
strategy:
  blueGreen:
    activeService: my-app
    previewService: my-app-preview
    autoPromotionEnabled: false
```

## 密鑰管理

### External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: db-credentials
  data:
  - secretKey: password
    remoteRef:
      key: prod/db/password
```

### Sealed Secrets

```bash
# Encrypt secret
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# Commit sealed-secret.yaml to Git
```

## 最佳實踐

1. **針對不同環境使用不同的儲存庫或分支**
2. **實施 RBAC** 用於 Git 儲存庫
3. **啟用通知** 用於同步失敗
4. **對自訂資源使用健康檢查**
5. **為正式環境實施核准關卡**
6. **將密鑰排除於 Git 之外**（使用 External Secrets）
7. **使用 App of Apps 模式** 進行組織管理
8. **標記發布版本** 以便輕鬆回滾
9. **監控同步狀態** 並配置告警
10. **先在測試環境測試變更**

## 疑難排解

**同步失敗：**
```bash
argocd app get my-app
argocd app sync my-app --prune
```

**不同步狀態：**
```bash
argocd app diff my-app
argocd app sync my-app --force
```

## 相關技能

- `k8s-manifest-generator` - 用於建立 manifest
- `helm-chart-scaffolding` - 用於封裝應用程式
