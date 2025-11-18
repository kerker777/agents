---
name: k8s-security-policies
description: 實施 Kubernetes 安全策略,包括 NetworkPolicy、PodSecurityPolicy 和 RBAC,以達成生產級安全性。適用於保護 Kubernetes 叢集、實施網路隔離或強制執行 Pod 安全標準。
---

# Kubernetes 安全策略

在 Kubernetes 中實施 NetworkPolicy、PodSecurityPolicy、RBAC 和 Pod Security Standards 的完整指南。

## 目的

使用網路策略、Pod 安全標準和 RBAC 為 Kubernetes 叢集實施深度防禦安全性。

## 何時使用此技能

- 實施網路分段
- 設定 Pod 安全標準
- 設定最小權限 RBAC 存取
- 建立合規性安全策略
- 實施准入控制
- 保護多租戶叢集

## Pod Security Standards

### 1. Privileged (不受限)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: privileged-ns
  labels:
    pod-security.kubernetes.io/enforce: privileged
    pod-security.kubernetes.io/audit: privileged
    pod-security.kubernetes.io/warn: privileged
```

### 2. Baseline (最低限度限制)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: baseline-ns
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/warn: baseline
```

### 3. Restricted (最嚴格限制)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

## 網路策略

### 預設拒絕全部

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 允許前端到後端

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### 允許 DNS

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

**參考:** 參見 `assets/network-policy-template.yaml`

## RBAC 設定

### Role (命名空間範圍)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### ClusterRole (叢集範圍)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: default
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**參考:** 參見 `references/rbac-patterns.md`

## Pod Security Context

### 受限制的 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

## 使用 OPA Gatekeeper 進行策略強制執行

### ConstraintTemplate

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("missing required labels: %v", [missing])
        }
```

### Constraint

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-app-label
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels: ["app", "environment"]
```

## Service Mesh 安全性 (Istio)

### PeerAuthentication (mTLS)

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

### AuthorizationPolicy

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/frontend"]
```

## 最佳實踐

1. **在命名空間層級實施 Pod Security Standards**
2. **使用網路策略** 進行網路分段
3. **對所有服務帳戶套用最小權限 RBAC**
4. **啟用准入控制** (OPA Gatekeeper/Kyverno)
5. **以非 root 身分執行容器**
6. **使用唯讀根檔案系統**
7. **除非需要,否則捨棄所有權限**
8. **實施資源配額** 和限制範圍
9. **啟用稽核日誌** 記錄安全事件
10. **定期進行映像檔安全掃描**

## 合規框架

### CIS Kubernetes Benchmark
- 使用 RBAC 授權
- 啟用稽核日誌
- 使用 Pod Security Standards
- 設定網路策略
- 實施靜態加密 Secret
- 啟用節點身分驗證

### NIST 網路安全框架
- 實施深度防禦
- 使用網路分段
- 設定安全監控
- 實施存取控制
- 啟用日誌記錄和監控

## 疑難排解

**NetworkPolicy 無法運作:**
```bash
# 檢查 CNI 是否支援 NetworkPolicy
kubectl get nodes -o wide
kubectl describe networkpolicy <name>
```

**RBAC 權限被拒絕:**
```bash
# 檢查有效權限
kubectl auth can-i list pods --as system:serviceaccount:default:my-sa
kubectl auth can-i '*' '*' --as system:serviceaccount:default:my-sa
```

## 參考檔案

- `assets/network-policy-template.yaml` - 網路策略範例
- `assets/pod-security-template.yaml` - Pod 安全策略
- `references/rbac-patterns.md` - RBAC 設定模式

## 相關技能

- `k8s-manifest-generator` - 用於建立安全的清單檔案
- `gitops-workflow` - 用於自動化策略部署
