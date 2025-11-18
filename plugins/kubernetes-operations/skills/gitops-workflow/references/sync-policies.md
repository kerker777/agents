# GitOps 同步策略

## ArgoCD 同步策略

### 自動同步
```yaml
syncPolicy:
  automated:
    prune: true       # Delete resources removed from Git
    selfHeal: true    # Reconcile manual changes
    allowEmpty: false # Prevent empty sync
```

### 手動同步
```yaml
syncPolicy:
  syncOptions:
  - PrunePropagationPolicy=foreground
  - CreateNamespace=true
```

### 同步視窗
```yaml
syncWindows:
- kind: allow
  schedule: "0 8 * * *"
  duration: 1h
  applications:
  - my-app
- kind: deny
  schedule: "0 22 * * *"
  duration: 8h
  applications:
  - '*'
```

### 重試策略
```yaml
syncPolicy:
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

## Flux 同步策略

### Kustomization 同步
```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
spec:
  interval: 5m
  prune: true
  wait: true
  timeout: 5m
  retryInterval: 1m
  force: false
```

### 來源同步間隔
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
spec:
  interval: 1m
  timeout: 60s
```

## 健康狀況評估

### 自訂健康檢查
```yaml
# ArgoCD
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.health.MyCustomResource: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.conditions ~= nil then
        for i, condition in ipairs(obj.status.conditions) do
          if condition.type == "Ready" and condition.status == "False" then
            hs.status = "Degraded"
            hs.message = condition.message
            return hs
          end
          if condition.type == "Ready" and condition.status == "True" then
            hs.status = "Healthy"
            hs.message = condition.message
            return hs
          end
        end
      end
    end
    hs.status = "Progressing"
    hs.message = "Waiting for status"
    return hs
```

## 同步選項

### 常見同步選項
- `PrunePropagationPolicy=foreground` - 等待刪除的資源完成刪除
- `CreateNamespace=true` - 自動建立命名空間
- `Validate=false` - 跳過 kubectl 驗證
- `PruneLast=true` - 同步後刪除資源
- `RespectIgnoreDifferences=true` - 遵循忽略差異設定
- `ApplyOutOfSyncOnly=true` - 僅套用不同步的資源

## 最佳實踐

1. 非正式環境使用自動同步
2. 正式環境需要手動核准
3. 配置同步視窗進行維護
4. 為自訂資源實施健康檢查
5. 大型應用程式使用選擇性同步
6. 配置適當的重試策略
7. 監控同步失敗並配置告警
8. 正式環境謹慎使用 prune
9. 在測試環境測試同步策略
10. 為團隊記錄同步行為
