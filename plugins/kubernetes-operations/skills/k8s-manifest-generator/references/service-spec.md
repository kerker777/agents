# Kubernetes Service 規格參考

Kubernetes Service 資源的完整參考指南,涵蓋服務類型、網路、負載平衡和服務探索模式。

## 概述

Service 為存取 Pod 提供穩定的網路端點。Service 透過提供服務探索和負載平衡功能,實現微服務之間的鬆散耦合。

## Service 類型

### 1. ClusterIP (預設)

在內部叢集 IP 上公開服務。只能從叢集內部存取。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  sessionAffinity: None
```

**使用案例:**
- 內部微服務通訊
- 資料庫服務
- 內部 API
- 訊息佇列

### 2. NodePort

在每個節點的 IP 上以靜態埠 (30000-32767 範圍) 公開服務。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080
    nodePort: 30080  # 可選,若省略則自動分配
    protocol: TCP
```

**使用案例:**
- 開發/測試外部存取
- 沒有負載平衡器的小型部署
- 需要直接節點存取

**限制:**
- 有限的埠範圍 (30000-32767)
- 必須處理節點故障
- 沒有跨節點的內建負載平衡

### 3. LoadBalancer

使用雲端供應商的負載平衡器公開服務。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: public-api
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
  - name: https
    port: 443
    targetPort: 8443
    protocol: TCP
  loadBalancerSourceRanges:
  - 203.0.113.0/24
```

**雲端特定註解:**

**AWS:**
```yaml
annotations:
  service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # or "external"
  service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
  service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
  service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:..."
  service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
```

**Azure:**
```yaml
annotations:
  service.beta.kubernetes.io/azure-load-balancer-internal: "true"
  service.beta.kubernetes.io/azure-pip-name: "my-public-ip"
```

**GCP:**
```yaml
annotations:
  cloud.google.com/load-balancer-type: "Internal"
  cloud.google.com/backend-config: '{"default": "my-backend-config"}'
```

### 4. ExternalName

將服務對應到外部 DNS 名稱 (CNAME 記錄)。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.external.example.com
  ports:
  - port: 5432
```

**使用案例:**
- 存取外部服務
- 服務遷移情境
- 多叢集服務參照

## 完整 Service 規格

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: production
  labels:
    app: my-app
    tier: backend
  annotations:
    description: "Main application service"
    prometheus.io/scrape: "true"
spec:
  # 服務類型
  type: ClusterIP

  # Pod 選擇器
  selector:
    app: my-app
    version: v1

  # 埠設定
  ports:
  - name: http
    port: 80           # 服務埠
    targetPort: 8080   # 容器埠 (或命名埠)
    protocol: TCP      # TCP, UDP, or SCTP

  # 會話親和性
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800

  # IP 設定
  clusterIP: 10.0.0.10  # 可選:特定 IP
  clusterIPs:
  - 10.0.0.10
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack

  # 外部流量策略
  externalTrafficPolicy: Local

  # 內部流量策略
  internalTrafficPolicy: Local

  # 健康檢查
  healthCheckNodePort: 30000

  # 負載平衡器設定 (用於 type: LoadBalancer)
  loadBalancerIP: 203.0.113.100
  loadBalancerSourceRanges:
  - 203.0.113.0/24

  # 外部 IP
  externalIPs:
  - 80.11.12.10

  # 發布策略
  publishNotReadyAddresses: false
```

## 埠設定

### 命名埠

在 Pod 中使用命名埠以提高靈活性:

**Deployment:**
```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        ports:
        - name: http
          containerPort: 8080
        - name: metrics
          containerPort: 9090
```

**Service:**
```yaml
spec:
  ports:
  - name: http
    port: 80
    targetPort: http  # 參照命名埠
  - name: metrics
    port: 9090
    targetPort: metrics
```

### 多埠

```yaml
spec:
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  - name: https
    port: 443
    targetPort: 8443
    protocol: TCP
  - name: grpc
    port: 9090
    targetPort: 9090
    protocol: TCP
```

## 會話親和性

### None (預設)

在 Pod 之間隨機分配請求。

```yaml
spec:
  sessionAffinity: None
```

### ClientIP

將來自相同客戶端 IP 的請求路由到相同的 Pod。

```yaml
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3 小時
```

**使用案例:**
- 有狀態應用程式
- 基於會話的應用程式
- WebSocket 連線

## 流量策略

### 外部流量策略

**Cluster (預設):**
```yaml
spec:
  externalTrafficPolicy: Cluster
```
- 在所有節點之間進行負載平衡
- 可能增加額外的網路跳躍
- 來源 IP 被遮罩

**Local:**
```yaml
spec:
  externalTrafficPolicy: Local
```
- 流量僅導向接收節點上的 Pod
- 保留客戶端來源 IP
- 更佳的效能 (無額外跳躍)
- 可能造成負載不均

### 內部流量策略

```yaml
spec:
  internalTrafficPolicy: Local  # or Cluster
```

控制叢集內部客戶端的流量路由。

## Headless Service

沒有叢集 IP 的服務,用於直接 Pod 存取。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  clusterIP: None  # Headless
  selector:
    app: database
  ports:
  - port: 5432
    targetPort: 5432
```

**使用案例:**
- StatefulSet Pod 探索
- 直接 Pod 對 Pod 通訊
- 自訂負載平衡
- 資料庫叢集

**DNS 回傳:**
- 個別 Pod IP 而非服務 IP
- 格式: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`

## 服務探索

### DNS

**ClusterIP Service:**
```
<service-name>.<namespace>.svc.cluster.local
```

範例:
```bash
curl http://backend-service.production.svc.cluster.local
```

**在相同命名空間內:**
```bash
curl http://backend-service
```

**Headless Service (回傳 Pod IP):**
```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

### 環境變數

Kubernetes 將服務資訊注入 Pod:

```bash
# 服務主機和埠
BACKEND_SERVICE_SERVICE_HOST=10.0.0.100
BACKEND_SERVICE_SERVICE_PORT=80

# 對於命名埠
BACKEND_SERVICE_SERVICE_PORT_HTTP=80
```

**注意:** Pod 必須在服務之後建立才能注入環境變數。

## 負載平衡

### 演算法

Kubernetes 預設使用隨機選擇。對於進階負載平衡:

**Service Mesh (Istio 範例):**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-service
  trafficPolicy:
    loadBalancer:
      simple: LEAST_REQUEST  # or ROUND_ROBIN, RANDOM, PASSTHROUGH
    connectionPool:
      tcp:
        maxConnections: 100
```

### 連線限制

使用 Pod 中斷預算和資源限制:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

## Service Mesh 整合

### Istio Virtual Service

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - match:
    - headers:
        version:
          exact: v2
    route:
    - destination:
        host: my-service
        subset: v2
  - route:
    - destination:
        host: my-service
        subset: v1
      weight: 90
    - destination:
        host: my-service
        subset: v2
      weight: 10
```

## 常見模式

### 模式 1: 內部微服務

```yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: backend
  labels:
    app: user-service
    tier: backend
spec:
  type: ClusterIP
  selector:
    app: user-service
  ports:
  - name: http
    port: 8080
    targetPort: http
    protocol: TCP
  - name: grpc
    port: 9090
    targetPort: grpc
    protocol: TCP
```

### 模式 2: 使用負載平衡器的公開 API

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:..."
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  selector:
    app: api-gateway
  ports:
  - name: https
    port: 443
    targetPort: 8443
    protocol: TCP
  loadBalancerSourceRanges:
  - 0.0.0.0/0
```

### 模式 3: StatefulSet 搭配 Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cassandra
spec:
  clusterIP: None
  selector:
    app: cassandra
  ports:
  - port: 9042
    targetPort: 9042
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
      - name: cassandra
        image: cassandra:4.0
```

### 模式 4: 外部服務對應

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: prod-db.cxyz.us-west-2.rds.amazonaws.com
---
# 或使用 Endpoints 進行基於 IP 的外部服務
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 443
    targetPort: 443
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-api
subsets:
- addresses:
  - ip: 203.0.113.100
  ports:
  - port: 443
```

### 模式 5: 帶指標的多埠服務

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9090
    targetPort: 9090
```

## 網路策略

控制到服務的流量:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
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

## 最佳實踐

### Service 設定

1. **使用命名埠** 以提高靈活性
2. **根據公開需求設定適當的服務類型**
3. **在 Deployment 和 Service 之間一致使用標籤和選擇器**
4. **為有狀態應用程式設定會話親和性**
5. **將外部流量策略設為 Local** 以保留 IP
6. **為 StatefulSet 使用 headless service**
7. **實施網路策略** 以提升安全性
8. **新增監控註解** 以提高可觀測性

### 生產環境檢查清單

- [ ] 服務類型適合使用案例
- [ ] 選擇器符合 Pod 標籤
- [ ] 使用命名埠以提高清晰度
- [ ] 如需要,已設定會話親和性
- [ ] 已適當設定流量策略
- [ ] 已設定負載平衡器註解 (若適用)
- [ ] 已限制來源 IP 範圍 (對於公開服務)
- [ ] 已驗證健康檢查設定
- [ ] 已新增監控註解
- [ ] 已定義網路策略

### 效能調校

**對於高流量:**
```yaml
spec:
  externalTrafficPolicy: Local
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
```

**對於 WebSocket/長連線:**
```yaml
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 86400  # 24 小時
```

## 疑難排解

### 服務無法存取

```bash
# 檢查服務是否存在
kubectl get service <service-name>

# 檢查端點 (應顯示 Pod IP)
kubectl get endpoints <service-name>

# 描述服務
kubectl describe service <service-name>

# 檢查 Pod 是否符合選擇器
kubectl get pods -l app=<app-name>
```

**常見問題:**
- 選擇器不符合 Pod 標籤
- 沒有執行中的 Pod (端點為空)
- 埠設定錯誤
- 網路策略阻擋流量

### DNS 解析失敗

```bash
# 從 Pod 測試 DNS
kubectl run debug --rm -it --image=busybox -- nslookup <service-name>

# 檢查 CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### 負載平衡器問題

```bash
# 檢查負載平衡器狀態
kubectl describe service <service-name>

# 檢查事件
kubectl get events --sort-by='.lastTimestamp'

# 驗證雲端供應商設定
kubectl describe node
```

## 相關資源

- [Kubernetes Service API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#service-v1-core)
- [Service Networking](https://kubernetes.io/docs/concepts/services-networking/service/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
