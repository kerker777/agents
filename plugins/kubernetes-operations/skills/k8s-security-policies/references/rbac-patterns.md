# RBAC 模式和最佳實踐

## 常見 RBAC 模式

### 模式 1: 唯讀存取

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-only
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

### 模式 2: 命名空間管理員

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: namespace-admin
  namespace: production
rules:
- apiGroups: ["", "apps", "batch", "extensions"]
  resources: ["*"]
  verbs: ["*"]
```

### 模式 3: Deployment 管理員

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### 模式 4: Secret 讀取者 (ServiceAccount)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
  resourceNames: ["app-secrets"]  # 僅限特定 secret
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-secret-reader
  namespace: production
subjects:
- kind: ServiceAccount
  name: my-app
  namespace: production
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### 模式 5: CI/CD Pipeline 存取

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cicd-deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["services", "configmaps"]
  verbs: ["get", "list", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

## ServiceAccount 最佳實踐

### 建立專用 ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  namespace: production
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      serviceAccountName: my-app
      automountServiceAccountToken: false  # 如不需要則停用
```

### 最小權限 ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-app-role
  namespace: production
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get"]
  resourceNames: ["my-app-config"]
```

## 安全性最佳實踐

1. **盡可能使用 Role 而非 ClusterRole**
2. **指定 resourceNames** 以實現細粒度存取
3. **避免在生產環境中使用萬用字元權限** (`*`)
4. **為每個應用程式建立專用 ServiceAccount**
5. **如不需要則停用 token 自動掛載**
6. **定期進行 RBAC 稽核** 以移除未使用的權限
7. **使用群組** 進行使用者管理
8. **實施命名空間隔離**
9. **使用稽核日誌監控 RBAC 使用情況**
10. **在 metadata 中記錄角色用途**

## RBAC 疑難排解

### 檢查使用者權限

```bash
kubectl auth can-i list pods --as john@example.com
kubectl auth can-i '*' '*' --as system:serviceaccount:default:my-app
```

### 檢視有效權限

```bash
kubectl describe clusterrole cluster-admin
kubectl describe rolebinding -n production
```

### 除錯存取問題

```bash
kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide | grep my-user
```

## 常見 RBAC 動詞

- `get` - 讀取特定資源
- `list` - 列出某類型的所有資源
- `watch` - 監看資源變更
- `create` - 建立新資源
- `update` - 更新現有資源
- `patch` - 部分更新資源
- `delete` - 刪除資源
- `deletecollection` - 刪除多個資源
- `*` - 所有動詞 (避免在生產環境中使用)

## 資源範圍

### 叢集範圍資源
- Nodes
- PersistentVolumes
- ClusterRoles
- ClusterRoleBindings
- Namespaces

### 命名空間範圍資源
- Pods
- Services
- Deployments
- ConfigMaps
- Secrets
- Roles
- RoleBindings
