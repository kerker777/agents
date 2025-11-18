---
name: k8s-manifest-generator
description: Create production-ready Kubernetes manifests for Deployments, Services, ConfigMaps, and Secrets following best practices and security standards. Use when generating Kubernetes YAML manifests, creating K8s resources, or implementing production-grade Kubernetes configurations.
---

# Kubernetes Manifest 產生器

建立生產就緒的 Kubernetes manifest 的逐步指南，包括 Deployment、Service、ConfigMap、Secret 和 PersistentVolumeClaim。

## 目的

本技能提供產生結構完善、安全且生產就緒的 Kubernetes manifest 的全面指南，遵循雲原生最佳實踐和 Kubernetes 慣例。

## 何時使用此技能

當您需要：
- 建立新的 Kubernetes Deployment manifest
- 定義網路連線的 Service 資源
- 產生用於配置管理的 ConfigMap 和 Secret 資源
- 為狀態工作負載建立 PersistentVolumeClaim manifest
- 遵循 Kubernetes 最佳實踐和命名慣例
- 實施資源限制、健康檢查和安全上下文
- 為多環境部署設計 manifest

## 逐步工作流程

### 1. 收集需求

**了解工作負載：**
- 應用程式類型（無狀態/狀態）
- 容器映像檔和版本
- 環境變數和配置需求
- 儲存需求
- 網路暴露需求（內部/外部）
- 資源需求（CPU、記憶體）
- 擴展需求
- 健康檢查端點

**要問的問題：**
- 應用程式名稱和用途是什麼？
- 將使用什麼容器映像檔和標籤？
- 應用程式是否需要持久儲存？
- 應用程式暴露哪些連接埠？
- 是否需要任何密鑰或配置檔案？
- CPU 和記憶體需求是什麼？
- 應用程式是否需要對外暴露？

### 2. 建立 Deployment Manifest

**遵循以下結構：**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <app-name>
  namespace: <namespace>
  labels:
    app: <app-name>
    version: <version>
spec:
  replicas: 3
  selector:
    matchLabels:
      app: <app-name>
  template:
    metadata:
      labels:
        app: <app-name>
        version: <version>
    spec:
      containers:
      - name: <container-name>
        image: <image>:<tag>
        ports:
        - containerPort: <port>
          name: http
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: ENV_VAR
          value: "value"
        envFrom:
        - configMapRef:
            name: <app-name>-config
        - secretRef:
            name: <app-name>-secret
```

**要套用的最佳實踐：**
- 總是設定資源 requests 和 limits
- 實施 liveness 和 readiness 探測
- 使用特定的映像檔標籤（絕不用 `:latest`）
- 為非 root 使用者套用安全上下文
- 使用標籤進行組織和選擇
- 根據可用性需求設定適當的副本數

**參考資料：**詳細的 deployment 選項請參閱 `references/deployment-spec.md`

### 3. 建立 Service Manifest

**選擇適當的 Service 類型：**

**ClusterIP（僅內部）：**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: <app-name>
  namespace: <namespace>
  labels:
    app: <app-name>
spec:
  type: ClusterIP
  selector:
    app: <app-name>
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```

**LoadBalancer（外部存取）：**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: <app-name>
  namespace: <namespace>
  labels:
    app: <app-name>
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: <app-name>
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```

**參考資料：**服務類型和網路請參閱 `references/service-spec.md`

### 4. 建立 ConfigMap

**用於應用程式配置：**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <app-name>-config
  namespace: <namespace>
data:
  APP_MODE: production
  LOG_LEVEL: info
  DATABASE_HOST: db.example.com
  # For config files
  app.properties: |
    server.port=8080
    server.host=0.0.0.0
    logging.level=INFO
```

**最佳實踐：**
- ConfigMap 僅用於非敏感資料
- 將相關配置組織在一起
- 為鍵使用有意義的名稱
- 考慮每個元件使用一個 ConfigMap
- 進行變更時對 ConfigMap 進行版本控制

**參考資料：**範例請參閱 `assets/configmap-template.yaml`

### 5. 建立 Secret

**用於敏感資料：**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <app-name>-secret
  namespace: <namespace>
type: Opaque
stringData:
  DATABASE_PASSWORD: "changeme"
  API_KEY: "secret-api-key"
  # For certificate files
  tls.crt: |
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN PRIVATE KEY-----
    ...
    -----END PRIVATE KEY-----
```

**安全考量：**
- 絕不以明文將密鑰提交到 Git
- 使用 Sealed Secrets、External Secrets Operator 或 Vault
- 定期輪換密鑰
- 使用 RBAC 限制密鑰存取
- 考慮使用 Secret 類型：`kubernetes.io/tls` 用於 TLS 密鑰

### 6. 建立 PersistentVolumeClaim（如需要）

**用於狀態應用程式：**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <app-name>-data
  namespace: <namespace>
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 10Gi
```

**在 Deployment 中掛載：**
```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        volumeMounts:
        - name: data
          mountPath: /var/lib/app
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: <app-name>-data
```

**儲存考量：**
- 根據效能需求選擇適當的 StorageClass
- 單 pod 存取使用 ReadWriteOnce
- 多 pod 共用儲存使用 ReadWriteMany
- 考慮備份策略
- 設定適當的保留策略

### 7. 套用安全最佳實踐

**在 Deployment 中新增安全上下文：**

```yaml
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
```

**安全檢查清單：**
- [ ] 以非 root 使用者執行
- [ ] 放棄所有 capabilities
- [ ] 使用唯讀根檔案系統
- [ ] 停用特權提升
- [ ] 設定 seccomp profile
- [ ] 使用 Pod Security Standards

### 8. 新增標籤和註解

**標準標籤（建議）：**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: <app-name>
    app.kubernetes.io/instance: <instance-name>
    app.kubernetes.io/version: "1.0.0"
    app.kubernetes.io/component: backend
    app.kubernetes.io/part-of: <system-name>
    app.kubernetes.io/managed-by: kubectl
```

**有用的註解：**

```yaml
metadata:
  annotations:
    description: "Application description"
    contact: "team@example.com"
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
```

### 9. 組織多資源 Manifest

**檔案組織選項：**

**選項 1：使用 `---` 分隔符的單一檔案**
```yaml
# app-name.yaml
---
apiVersion: v1
kind: ConfigMap
...
---
apiVersion: v1
kind: Secret
...
---
apiVersion: apps/v1
kind: Deployment
...
---
apiVersion: v1
kind: Service
...
```

**選項 2：分開的檔案**
```
manifests/
├── configmap.yaml
├── secret.yaml
├── deployment.yaml
├── service.yaml
└── pvc.yaml
```

**選項 3：Kustomize 結構**
```
base/
├── kustomization.yaml
├── deployment.yaml
├── service.yaml
└── configmap.yaml
overlays/
├── dev/
│   └── kustomization.yaml
└── prod/
    └── kustomization.yaml
```

### 10. 驗證與測試

**驗證步驟：**

```bash
# Dry-run validation
kubectl apply -f manifest.yaml --dry-run=client

# Server-side validation
kubectl apply -f manifest.yaml --dry-run=server

# Validate with kubeval
kubeval manifest.yaml

# Validate with kube-score
kube-score score manifest.yaml

# Check with kube-linter
kube-linter lint manifest.yaml
```

**測試檢查清單：**
- [ ] Manifest 通過 dry-run 驗證
- [ ] 所有必填欄位都存在
- [ ] 資源限制合理
- [ ] 健康檢查已配置
- [ ] 安全上下文已設定
- [ ] 標籤遵循慣例
- [ ] 命名空間存在或已建立

## 常見模式

### 模式 1：簡單的無狀態 Web 應用程式

**使用場景：**標準 Web API 或微服務

**需要的元件：**
- Deployment（3 個副本用於 HA）
- ClusterIP Service
- ConfigMap 用於配置
- Secret 用於 API 金鑰
- HorizontalPodAutoscaler（可選）

**參考資料：**請參閱 `assets/deployment-template.yaml`

### 模式 2：狀態資料庫應用程式

**使用場景：**資料庫或持久儲存應用程式

**需要的元件：**
- StatefulSet（而非 Deployment）
- Headless Service
- PersistentVolumeClaim 範本
- ConfigMap 用於資料庫配置
- Secret 用於憑證

### 模式 3：背景作業或 Cron

**使用場景：**排程任務或批次處理

**需要的元件：**
- CronJob 或 Job
- ConfigMap 用於作業參數
- Secret 用於憑證
- ServiceAccount 搭配 RBAC

### 模式 4：多容器 Pod

**使用場景：**具有 sidecar 容器的應用程式

**需要的元件：**
- 具有多個容器的 Deployment
- 容器之間的共用卷
- Init 容器用於設定
- Service（如需要）

## 範本

以下範本可在 `assets/` 目錄中取得：

- `deployment-template.yaml` - 具有最佳實踐的標準 deployment
- `service-template.yaml` - Service 配置（ClusterIP、LoadBalancer、NodePort）
- `configmap-template.yaml` - 具有不同資料類型的 ConfigMap 範例
- `secret-template.yaml` - Secret 範例（要產生，不要提交）
- `pvc-template.yaml` - PersistentVolumeClaim 範本

## 參考文件

- `references/deployment-spec.md` - 詳細的 Deployment 規格
- `references/service-spec.md` - Service 類型和網路詳細資訊

## 最佳實踐摘要

1. **總是設定資源 requests 和 limits** - 防止資源耗盡
2. **實施健康檢查** - 確保 Kubernetes 能管理您的應用程式
3. **使用特定的映像檔標籤** - 避免不可預測的部署
4. **套用安全上下文** - 以非 root 執行，放棄 capabilities
5. **使用 ConfigMap 和 Secret** - 將配置與程式碼分離
6. **為所有內容加上標籤** - 啟用篩選和組織
7. **遵循命名慣例** - 使用標準 Kubernetes 標籤
8. **套用前先驗證** - 使用 dry-run 和驗證工具
9. **對 manifest 進行版本控制** - 使用版本控制保存在 Git 中
10. **使用註解記錄** - 為其他開發者添加上下文

## 疑難排解

**Pod 無法啟動：**
- 檢查映像檔拉取錯誤：`kubectl describe pod <pod-name>`
- 驗證資源可用性：`kubectl get nodes`
- 檢查事件：`kubectl get events --sort-by='.lastTimestamp'`

**Service 無法存取：**
- 驗證選擇器與 pod 標籤匹配：`kubectl get endpoints <service-name>`
- 檢查服務類型和連接埠配置
- 從叢集內測試：`kubectl run debug --rm -it --image=busybox -- sh`

**ConfigMap/Secret 未載入：**
- 驗證 Deployment 中的名稱匹配
- 檢查命名空間
- 確保資源存在：`kubectl get configmap,secret`

## 下一步

建立 manifest 後：
1. 儲存在 Git 儲存庫
2. 設定 CI/CD 管線進行部署
3. 考慮使用 Helm 或 Kustomize 進行範本化
4. 使用 ArgoCD 或 Flux 實施 GitOps
5. 新增監控和可觀測性

## 相關技能

- `helm-chart-scaffolding` - 用於範本化和封裝
- `gitops-workflow` - 用於自動化部署
- `k8s-security-policies` - 用於進階安全配置
