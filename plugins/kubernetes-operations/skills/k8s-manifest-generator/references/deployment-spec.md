# Kubernetes Deployment 規格參考

Kubernetes Deployment 資源的全面參考，涵蓋所有關鍵欄位、最佳實踐和常見模式。

## 概述

Deployment 為 Pod 和 ReplicaSet 提供宣告式更新。它管理應用程式的所需狀態，處理推出、回滾和擴展操作。

## 完整 Deployment 規格

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
  labels:
    app.kubernetes.io/name: my-app
    app.kubernetes.io/version: "1.0.0"
    app.kubernetes.io/component: backend
    app.kubernetes.io/part-of: my-system
  annotations:
    description: "Main application deployment"
    contact: "backend-team@example.com"
spec:
  # Replica management
  replicas: 3
  revisionHistoryLimit: 10

  # Pod selection
  selector:
    matchLabels:
      app: my-app
      version: v1

  # Update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

  # Minimum time for pod to be ready
  minReadySeconds: 10

  # Deployment will fail if it doesn't progress in this time
  progressDeadlineSeconds: 600

  # Pod template
  template:
    metadata:
      labels:
        app: my-app
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      # Service account for RBAC
      serviceAccountName: my-app

      # Security context for the pod
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault

      # Init containers run before main containers
      initContainers:
      - name: init-db
        image: busybox:1.36
        command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 1; done']
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1000

      # Main containers
      containers:
      - name: app
        image: myapp:1.0.0
        imagePullPolicy: IfNotPresent

        # Container ports
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        - name: metrics
          containerPort: 9090
          protocol: TCP

        # Environment variables
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url

        # ConfigMap and Secret references
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets

        # Resource requests and limits
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

        # Liveness probe
        livenessProbe:
          httpGet:
            path: /health/live
            port: http
            httpHeaders:
            - name: Custom-Header
              value: Awesome
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3

        # Readiness probe
        readinessProbe:
          httpGet:
            path: /health/ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3

        # Startup probe (for slow-starting containers)
        startupProbe:
          httpGet:
            path: /health/startup
            port: http
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 30

        # Volume mounts
        volumeMounts:
        - name: data
          mountPath: /var/lib/app
        - name: config
          mountPath: /etc/app
          readOnly: true
        - name: tmp
          mountPath: /tmp

        # Security context for container
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          capabilities:
            drop:
            - ALL

        # Lifecycle hooks
        lifecycle:
          postStart:
            exec:
              command: ["/bin/sh", "-c", "echo Container started > /tmp/started"]
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]

      # Volumes
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: app-data
      - name: config
        configMap:
          name: app-config
      - name: tmp
        emptyDir: {}

      # DNS configuration
      dnsPolicy: ClusterFirst
      dnsConfig:
        options:
        - name: ndots
          value: "2"

      # Scheduling
      nodeSelector:
        disktype: ssd

      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - my-app
              topologyKey: kubernetes.io/hostname

      tolerations:
      - key: "app"
        operator: "Equal"
        value: "my-app"
        effect: "NoSchedule"

      # Termination
      terminationGracePeriodSeconds: 30

      # Image pull secrets
      imagePullSecrets:
      - name: regcred
```

## 欄位參考

### 元資料欄位

#### 必填欄位
- `apiVersion`：`apps/v1`（目前穩定版本）
- `kind`：`Deployment`
- `metadata.name`：命名空間內的唯一名稱

#### 建議的元資料
- `metadata.namespace`：目標命名空間（預設為 `default`）
- `metadata.labels`：用於組織的鍵值對
- `metadata.annotations`：非識別性元資料

### Spec 欄位

#### 副本管理

**`replicas`**（整數，預設：1）
- 所需的 pod 執行個體數量
- 最佳實踐：正式環境使用 3 個以上以實現高可用性
- 可手動擴展或透過 HorizontalPodAutoscaler 擴展

**`revisionHistoryLimit`**（整數，預設：10）
- 保留用於回滾的舊 ReplicaSet 數量
- 設為 0 以停用回滾功能
- 減少長期執行部署的儲存開銷

#### 更新策略

**`strategy.type`**（字串）
- `RollingUpdate`（預設）：漸進式 pod 替換
- `Recreate`：在建立新 pod 前刪除所有 pod

**`strategy.rollingUpdate.maxSurge`**（整數或百分比，預設：25%）
- 更新期間超過所需副本的最大 pod 數
- 範例：3 個副本且 maxSurge=1，更新期間最多 4 個 pod

**`strategy.rollingUpdate.maxUnavailable`**（整數或百分比，預設：25%）
- 更新期間低於所需副本的最大 pod 數
- 零停機部署設為 0
- 如果 maxSurge 為 0 則不能為 0

**最佳實踐：**
```yaml
# Zero-downtime deployment
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0

# Fast deployment (can have brief downtime)
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 2
    maxUnavailable: 1

# Complete replacement
strategy:
  type: Recreate
```

#### Pod 範本

**`template.metadata.labels`**
- 必須包含與 `spec.selector.matchLabels` 匹配的標籤
- 為藍綠部署新增版本標籤
- 包含標準 Kubernetes 標籤

**`template.spec.containers`**（必填）
- 容器規格陣列
- 至少需要一個容器
- 每個容器需要唯一名稱

#### 容器配置

**映像檔管理：**
```yaml
containers:
- name: app
  image: registry.example.com/myapp:1.0.0
  imagePullPolicy: IfNotPresent  # or Always, Never
```

映像檔拉取策略：
- `IfNotPresent`：如果未快取則拉取（標記映像檔的預設值）
- `Always`：總是拉取（:latest 的預設值）
- `Never`：永不拉取，如果未快取則失敗

**連接埠宣告：**
```yaml
ports:
- name: http      # Named for referencing in Service
  containerPort: 8080
  protocol: TCP   # TCP (default), UDP, or SCTP
  hostPort: 8080  # Optional: Bind to host port (rarely used)
```

#### 資源管理

**Requests vs Limits：**

```yaml
resources:
  requests:
    memory: "256Mi"  # Guaranteed resources
    cpu: "250m"      # 0.25 CPU cores
  limits:
    memory: "512Mi"  # Maximum allowed
    cpu: "500m"      # 0.5 CPU cores
```

**QoS 類別（自動決定）：**

1. **Guaranteed**：所有容器的 requests = limits
   - 最高優先級
   - 最後被驅逐

2. **Burstable**：requests < limits 或僅設定 requests
   - 中等優先級
   - 在 Guaranteed 之前被驅逐

3. **BestEffort**：未設定 requests 或 limits
   - 最低優先級
   - 首先被驅逐

**最佳實踐：**
- 正式環境總是設定 requests
- 設定 limits 以防止資源壟斷
- 記憶體 limits 應為 requests 的 1.5-2 倍
- CPU limits 對於突發工作負載可以更高

#### 健康檢查

**探測類型：**

1. **startupProbe** - 用於啟動緩慢的應用程式
   ```yaml
   startupProbe:
     httpGet:
       path: /health/startup
       port: 8080
     initialDelaySeconds: 0
     periodSeconds: 10
     failureThreshold: 30  # 5 minutes to start (10s * 30)
   ```

2. **livenessProbe** - 重啟不健康的容器
   ```yaml
   livenessProbe:
     httpGet:
       path: /health/live
       port: 8080
     initialDelaySeconds: 30
     periodSeconds: 10
     timeoutSeconds: 5
     failureThreshold: 3  # Restart after 3 failures
   ```

3. **readinessProbe** - 控制流量路由
   ```yaml
   readinessProbe:
     httpGet:
       path: /health/ready
       port: 8080
     initialDelaySeconds: 5
     periodSeconds: 5
     failureThreshold: 3  # Remove from service after 3 failures
   ```

**探測機制：**

```yaml
# HTTP GET
httpGet:
  path: /health
  port: 8080
  httpHeaders:
  - name: Authorization
    value: Bearer token

# TCP Socket
tcpSocket:
  port: 3306

# Command execution
exec:
  command:
  - cat
  - /tmp/healthy

# gRPC (Kubernetes 1.24+)
grpc:
  port: 9090
  service: my.service.health.v1.Health
```

**探測時間參數：**

- `initialDelaySeconds`：第一次探測前等待時間
- `periodSeconds`：探測頻率
- `timeoutSeconds`：探測逾時
- `successThreshold`：標記為健康所需的成功次數（liveness/startup 為 1）
- `failureThreshold`：採取行動前的失敗次數

#### 安全上下文

**Pod 級安全上下文：**
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    fsGroupChangePolicy: OnRootMismatch
    seccompProfile:
      type: RuntimeDefault
```

**容器級安全上下文：**
```yaml
containers:
- name: app
  securityContext:
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    runAsNonRoot: true
    runAsUser: 1000
    capabilities:
      drop:
      - ALL
      add:
      - NET_BIND_SERVICE  # Only if needed
```

**安全最佳實踐：**
- 總是以非 root 執行（`runAsNonRoot: true`）
- 放棄所有 capabilities 並僅新增需要的
- 盡可能使用唯讀根檔案系統
- 啟用 seccomp profile
- 停用特權提升

#### 卷

**卷類型：**

```yaml
volumes:
# PersistentVolumeClaim
- name: data
  persistentVolumeClaim:
    claimName: app-data

# ConfigMap
- name: config
  configMap:
    name: app-config
    items:
    - key: app.properties
      path: application.properties

# Secret
- name: secrets
  secret:
    secretName: app-secrets
    defaultMode: 0400

# EmptyDir (ephemeral)
- name: cache
  emptyDir:
    sizeLimit: 1Gi

# HostPath (avoid in production)
- name: host-data
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

#### 排程

**節點選擇：**

```yaml
# Simple node selector
nodeSelector:
  disktype: ssd
  zone: us-west-1a

# Node affinity (more expressive)
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/arch
          operator: In
          values:
          - amd64
          - arm64
```

**Pod 親和性/反親和性：**

```yaml
# Spread pods across nodes
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: my-app
      topologyKey: kubernetes.io/hostname

# Co-locate with database
affinity:
  podAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchLabels:
            app: database
        topologyKey: kubernetes.io/hostname
```

**容忍：**

```yaml
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 30
- key: "dedicated"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"
```

## 常見模式

### 高可用性 Deployment

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: my-app
            topologyKey: kubernetes.io/hostname
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: my-app
```

### Sidecar 容器模式

```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0.0
        volumeMounts:
        - name: shared-logs
          mountPath: /var/log
      - name: log-forwarder
        image: fluent-bit:2.0
        volumeMounts:
        - name: shared-logs
          mountPath: /var/log
          readOnly: true
      volumes:
      - name: shared-logs
        emptyDir: {}
```

### 用於相依性的 Init 容器

```yaml
spec:
  template:
    spec:
      initContainers:
      - name: wait-for-db
        image: busybox:1.36
        command:
        - sh
        - -c
        - |
          until nc -z database-service 5432; do
            echo "Waiting for database..."
            sleep 2
          done
      - name: run-migrations
        image: myapp:1.0.0
        command: ["./migrate", "up"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
      containers:
      - name: app
        image: myapp:1.0.0
```

## 最佳實踐

### 正式環境檢查清單

- [ ] 設定資源 requests 和 limits
- [ ] 實施所有三種探測類型（startup、liveness、readiness）
- [ ] 使用特定映像檔標籤（不用 :latest）
- [ ] 配置安全上下文（非 root、唯讀檔案系統）
- [ ] 設定副本數 >= 3 以實現高可用性
- [ ] 配置 pod 反親和性以進行分散
- [ ] 設定適當的更新策略（零停機為 maxUnavailable: 0）
- [ ] 使用 ConfigMap 和 Secret 進行配置
- [ ] 新增標準標籤和註解
- [ ] 配置優雅關機（preStop hook、terminationGracePeriodSeconds）
- [ ] 設定 revisionHistoryLimit 以實現回滾能力
- [ ] 使用具有最小 RBAC 權限的 ServiceAccount

### 效能調校

**快速啟動：**
```yaml
spec:
  minReadySeconds: 5
  strategy:
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
```

**零停機更新：**
```yaml
spec:
  minReadySeconds: 10
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

**優雅關機：**
```yaml
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: app
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15 && kill -SIGTERM 1"]
```

## 疑難排解

### 常見問題

**Pod 無法啟動：**
```bash
kubectl describe deployment <name>
kubectl get pods -l app=<app-name>
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

**ImagePullBackOff：**
- 檢查映像檔名稱和標籤
- 驗證 imagePullSecrets
- 檢查登錄表憑證

**CrashLoopBackOff：**
- 檢查容器日誌
- 驗證 liveness probe 不會太激進
- 檢查資源限制
- 驗證應用程式相依性

**Deployment 停滯不前：**
- 檢查 progressDeadlineSeconds
- 驗證 readiness probe
- 檢查資源可用性

## 相關資源

- [Kubernetes Deployment API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#deployment-v1-apps)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
