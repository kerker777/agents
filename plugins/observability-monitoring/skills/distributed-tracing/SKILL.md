---
name: distributed-tracing
description: 使用 Jaeger 和 Tempo 實作分散式追蹤，以追蹤微服務之間的請求並識別效能瓶頸。適用於除錯微服務、分析請求流程，或為分散式系統實作可觀測性。
---

# 分散式追蹤 (Distributed Tracing)

使用 Jaeger 和 Tempo 實作分散式追蹤，以在微服務之間取得請求流程的可見性。

## 目的

追蹤分散式系統中的請求，以了解延遲、依賴關係和失敗點。

## 使用時機

- 除錯延遲問題
- 了解服務依賴關係
- 識別瓶頸
- 追蹤錯誤傳播
- 分析請求路徑

## 分散式追蹤概念

### Trace 結構
```
Trace (Request ID: abc123)
  ↓
Span (frontend) [100ms]
  ↓
Span (api-gateway) [80ms]
  ├→ Span (auth-service) [10ms]
  └→ Span (user-service) [60ms]
      └→ Span (database) [40ms]
```

### 核心元件
- **Trace（追蹤）** - 端到端的請求旅程
- **Span（跨度）** - 追蹤中的單一操作
- **Context（上下文）** - 在服務之間傳播的元資料
- **Tags（標籤）** - 用於過濾的鍵值對
- **Logs（日誌）** - Span 中帶有時間戳記的事件

## Jaeger 設定

### Kubernetes 部署

```bash
# 部署 Jaeger Operator
kubectl create namespace observability
kubectl create -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.51.0/jaeger-operator.yaml -n observability

# 部署 Jaeger 實例
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: observability
spec:
  strategy: production
  storage:
    type: elasticsearch
    options:
      es:
        server-urls: http://elasticsearch:9200
  ingress:
    enabled: true
EOF
```

### Docker Compose

```yaml
version: '3.8'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"  # UI
      - "14268:14268"  # Collector
      - "14250:14250"  # gRPC
      - "9411:9411"    # Zipkin
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
```

**參考資料：** 請參閱 `references/jaeger-setup.md`

## 應用程式埋點 (Instrumentation)

### OpenTelemetry（推薦）

#### Python (Flask)
```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.resources import SERVICE_NAME, Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from flask import Flask

# Initialize tracer
resource = Resource(attributes={SERVICE_NAME: "my-service"})
provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
))
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

# Instrument Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)

@app.route('/api/users')
def get_users():
    tracer = trace.get_tracer(__name__)

    with tracer.start_as_current_span("get_users") as span:
        span.set_attribute("user.count", 100)
        # Business logic
        users = fetch_users_from_db()
        return {"users": users}

def fetch_users_from_db():
    tracer = trace.get_tracer(__name__)

    with tracer.start_as_current_span("database_query") as span:
        span.set_attribute("db.system", "postgresql")
        span.set_attribute("db.statement", "SELECT * FROM users")
        # Database query
        return query_database()
```

#### Node.js (Express)
```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { BatchSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');

// Initialize tracer
const provider = new NodeTracerProvider({
  resource: { attributes: { 'service.name': 'my-service' } }
});

const exporter = new JaegerExporter({
  endpoint: 'http://jaeger:14268/api/traces'
});

provider.addSpanProcessor(new BatchSpanProcessor(exporter));
provider.register();

// Instrument libraries
registerInstrumentations({
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation(),
  ],
});

const express = require('express');
const app = express();

app.get('/api/users', async (req, res) => {
  const tracer = trace.getTracer('my-service');
  const span = tracer.startSpan('get_users');

  try {
    const users = await fetchUsers();
    span.setAttributes({ 'user.count': users.length });
    res.json({ users });
  } finally {
    span.end();
  }
});
```

#### Go
```go
package main

import (
    "context"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/resource"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.4.0"
)

func initTracer() (*sdktrace.TracerProvider, error) {
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://jaeger:14268/api/traces"),
    ))
    if err != nil {
        return nil, err
    }

    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String("my-service"),
        )),
    )

    otel.SetTracerProvider(tp)
    return tp, nil
}

func getUsers(ctx context.Context) ([]User, error) {
    tracer := otel.Tracer("my-service")
    ctx, span := tracer.Start(ctx, "get_users")
    defer span.End()

    span.SetAttributes(attribute.String("user.filter", "active"))

    users, err := fetchUsersFromDB(ctx)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }

    span.SetAttributes(attribute.Int("user.count", len(users)))
    return users, nil
}
```

**參考資料：** 請參閱 `references/instrumentation.md`

## 上下文傳播 (Context Propagation)

### HTTP Headers
```
traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01
tracestate: congo=t61rcWkgMzE
```

### HTTP 請求中的傳播

#### Python
```python
from opentelemetry.propagate import inject

headers = {}
inject(headers)  # Injects trace context

response = requests.get('http://downstream-service/api', headers=headers)
```

#### Node.js
```javascript
const { propagation } = require('@opentelemetry/api');

const headers = {};
propagation.inject(context.active(), headers);

axios.get('http://downstream-service/api', { headers });
```

## Tempo 設定 (Grafana)

### Kubernetes 部署

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tempo-config
data:
  tempo.yaml: |
    server:
      http_listen_port: 3200

    distributor:
      receivers:
        jaeger:
          protocols:
            thrift_http:
            grpc:
        otlp:
          protocols:
            http:
            grpc:

    storage:
      trace:
        backend: s3
        s3:
          bucket: tempo-traces
          endpoint: s3.amazonaws.com

    querier:
      frontend_worker:
        frontend_address: tempo-query-frontend:9095
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tempo
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: tempo
        image: grafana/tempo:latest
        args:
          - -config.file=/etc/tempo/tempo.yaml
        volumeMounts:
        - name: config
          mountPath: /etc/tempo
      volumes:
      - name: config
        configMap:
          name: tempo-config
```

**參考資料：** 請參閱 `assets/jaeger-config.yaml.template`

## 採樣策略 (Sampling Strategies)

### 機率採樣 (Probabilistic Sampling)
```yaml
# 採樣 1% 的追蹤
sampler:
  type: probabilistic
  param: 0.01
```

### 速率限制採樣 (Rate Limiting Sampling)
```yaml
# 每秒最多採樣 100 個追蹤
sampler:
  type: ratelimiting
  param: 100
```

### 自適應採樣 (Adaptive Sampling)
```python
from opentelemetry.sdk.trace.sampling import ParentBased, TraceIdRatioBased

# 基於 trace ID 的採樣（確定性）
sampler = ParentBased(root=TraceIdRatioBased(0.01))
```

## Trace 分析

### 尋找慢速請求

**Jaeger 查詢：**
```
service=my-service
duration > 1s
```

### 尋找錯誤

**Jaeger 查詢：**
```
service=my-service
error=true
tags.http.status_code >= 500
```

### 服務依賴圖 (Service Dependency Graph)

Jaeger 會自動產生服務依賴圖，顯示：
- 服務關係
- 請求速率
- 錯誤率
- 平均延遲

## 最佳實踐

1. **適當採樣**（生產環境 1-10%）
2. **新增有意義的標籤**（user_id、request_id）
3. **在所有服務邊界傳播上下文**
4. **在 Span 中記錄例外狀況**
5. **使用一致的命名**來命名操作
6. **監控追蹤開銷**（CPU 影響 <1%）
7. **設定追蹤錯誤的告警**
8. **實作分散式上下文**（baggage）
9. **使用 Span 事件**標記重要里程碑
10. **記錄埋點**標準

## 與日誌整合

### 關聯日誌 (Correlated Logs)
```python
import logging
from opentelemetry import trace

logger = logging.getLogger(__name__)

def process_request():
    span = trace.get_current_span()
    trace_id = span.get_span_context().trace_id

    logger.info(
        "Processing request",
        extra={"trace_id": format(trace_id, '032x')}
    )
```

## 疑難排解

**沒有追蹤出現：**
- 檢查 collector 端點
- 驗證網路連線
- 檢查採樣設定
- 檢視應用程式日誌

**高延遲開銷：**
- 降低採樣率
- 使用批次 span 處理器
- 檢查 exporter 設定

## 參考檔案

- `references/jaeger-setup.md` - Jaeger 安裝
- `references/instrumentation.md` - 埋點模式
- `assets/jaeger-config.yaml.template` - Jaeger 設定

## 相關技能

- `prometheus-configuration` - 用於指標收集
- `grafana-dashboards` - 用於視覺化
- `slo-implementation` - 用於延遲 SLO
