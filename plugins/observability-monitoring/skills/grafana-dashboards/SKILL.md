---
name: grafana-dashboards
description: 建立與管理正式環境 Grafana 儀表板，用於即時視覺化系統與應用程式指標。適用於建置監控儀表板、視覺化指標或建立操作可觀測性介面。
---

# Grafana 儀表板

建立與管理正式環境就緒的 Grafana 儀表板，以達成全面的系統可觀測性。

## 目的

設計有效的 Grafana 儀表板，用於監控應用程式、基礎架構與業務指標。

## 使用時機

- 視覺化 Prometheus 指標
- 建立自訂儀表板
- 實作 SLO（服務水準目標）儀表板
- 監控基礎架構
- 追蹤業務 KPI

## 儀表板設計原則

### 1. 資訊層級架構
```
┌─────────────────────────────────────┐
│  關鍵指標（大數字）                  │
├─────────────────────────────────────┤
│  重要趨勢（時間序列）                │
├─────────────────────────────────────┤
│  詳細指標（表格/熱力圖）             │
└─────────────────────────────────────┘
```

### 2. RED 方法（服務）
- **Rate（請求率）** - 每秒請求數
- **Errors（錯誤率）** - 錯誤率
- **Duration（持續時間）** - 延遲/回應時間

### 3. USE 方法（資源）
- **Utilization（使用率）** - 資源忙碌時間百分比
- **Saturation（飽和度）** - 佇列長度/等待時間
- **Errors（錯誤）** - 錯誤計數

## 儀表板結構

### API 監控儀表板

```json
{
  "dashboard": {
    "title": "API Monitoring",
    "tags": ["api", "production"],
    "timezone": "browser",
    "refresh": "30s",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ],
        "gridPos": {"x": 0, "y": 0, "w": 12, "h": 8}
      },
      {
        "title": "Error Rate %",
        "type": "graph",
        "targets": [
          {
            "expr": "(sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))) * 100",
            "legendFormat": "Error Rate"
          }
        ],
        "alert": {
          "conditions": [
            {
              "evaluator": {"params": [5], "type": "gt"},
              "operator": {"type": "and"},
              "query": {"params": ["A", "5m", "now"]},
              "type": "query"
            }
          ]
        },
        "gridPos": {"x": 12, "y": 0, "w": 12, "h": 8}
      },
      {
        "title": "P95 Latency",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))",
            "legendFormat": "{{service}}"
          }
        ],
        "gridPos": {"x": 0, "y": 8, "w": 24, "h": 8}
      }
    ]
  }
}
```

**參考：** 請參閱 `assets/api-dashboard.json`

## 面板類型

### 1. Stat 面板（單一數值）
```json
{
  "type": "stat",
  "title": "Total Requests",
  "targets": [{
    "expr": "sum(http_requests_total)"
  }],
  "options": {
    "reduceOptions": {
      "values": false,
      "calcs": ["lastNotNull"]
    },
    "orientation": "auto",
    "textMode": "auto",
    "colorMode": "value"
  },
  "fieldConfig": {
    "defaults": {
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {"value": 0, "color": "green"},
          {"value": 80, "color": "yellow"},
          {"value": 90, "color": "red"}
        ]
      }
    }
  }
}
```

### 2. 時間序列圖表
```json
{
  "type": "graph",
  "title": "CPU Usage",
  "targets": [{
    "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
  }],
  "yaxes": [
    {"format": "percent", "max": 100, "min": 0},
    {"format": "short"}
  ]
}
```

### 3. 表格面板
```json
{
  "type": "table",
  "title": "Service Status",
  "targets": [{
    "expr": "up",
    "format": "table",
    "instant": true
  }],
  "transformations": [
    {
      "id": "organize",
      "options": {
        "excludeByName": {"Time": true},
        "indexByName": {},
        "renameByName": {
          "instance": "Instance",
          "job": "Service",
          "Value": "Status"
        }
      }
    }
  ]
}
```

### 4. 熱力圖
```json
{
  "type": "heatmap",
  "title": "Latency Heatmap",
  "targets": [{
    "expr": "sum(rate(http_request_duration_seconds_bucket[5m])) by (le)",
    "format": "heatmap"
  }],
  "dataFormat": "tsbuckets",
  "yAxis": {
    "format": "s"
  }
}
```

## 變數

### 查詢變數
```json
{
  "templating": {
    "list": [
      {
        "name": "namespace",
        "type": "query",
        "datasource": "Prometheus",
        "query": "label_values(kube_pod_info, namespace)",
        "refresh": 1,
        "multi": false
      },
      {
        "name": "service",
        "type": "query",
        "datasource": "Prometheus",
        "query": "label_values(kube_service_info{namespace=\"$namespace\"}, service)",
        "refresh": 1,
        "multi": true
      }
    ]
  }
}
```

### 在查詢中使用變數
```
sum(rate(http_requests_total{namespace="$namespace", service=~"$service"}[5m]))
```

## 儀表板中的告警

```json
{
  "alert": {
    "name": "High Error Rate",
    "conditions": [
      {
        "evaluator": {
          "params": [5],
          "type": "gt"
        },
        "operator": {"type": "and"},
        "query": {
          "params": ["A", "5m", "now"]
        },
        "reducer": {"type": "avg"},
        "type": "query"
      }
    ],
    "executionErrorState": "alerting",
    "for": "5m",
    "frequency": "1m",
    "message": "Error rate is above 5%",
    "noDataState": "no_data",
    "notifications": [
      {"uid": "slack-channel"}
    ]
  }
}
```

## 儀表板佈建

**dashboards.yml:**
```yaml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: 'General'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/dashboards
```

## 常見儀表板模式

### 基礎架構儀表板

**關鍵面板：**
- 每個節點的 CPU 使用率
- 每個節點的記憶體使用量
- 磁碟 I/O
- 網路流量
- 依命名空間的 Pod 數量
- 節點狀態

**參考：** 請參閱 `assets/infrastructure-dashboard.json`

### 資料庫儀表板

**關鍵面板：**
- 每秒查詢數
- 連線池使用率
- 查詢延遲（P50、P95、P99）
- 活躍連線數
- 資料庫大小
- 複製延遲
- 慢查詢

**參考：** 請參閱 `assets/database-dashboard.json`

### 應用程式儀表板

**關鍵面板：**
- 請求率
- 錯誤率
- 回應時間（百分位數）
- 活躍使用者/會話
- 快取命中率
- 佇列長度

## 最佳實務

1. **從範本開始**（Grafana 社群儀表板）
2. **使用一致的命名**規範於面板與變數
3. **將相關指標分組**於列中
4. **設定適當的時間範圍**（預設值：最近 6 小時）
5. **使用變數**以增加彈性
6. **新增面板說明**以提供背景資訊
7. **正確設定單位**
8. **設定有意義的閾值**以定義顏色
9. **跨儀表板使用一致的顏色**
10. **使用不同時間範圍進行測試**

## 儀表板即程式碼

### Terraform 佈建

```hcl
resource "grafana_dashboard" "api_monitoring" {
  config_json = file("${path.module}/dashboards/api-monitoring.json")
  folder      = grafana_folder.monitoring.id
}

resource "grafana_folder" "monitoring" {
  title = "Production Monitoring"
}
```

### Ansible 佈建

```yaml
- name: Deploy Grafana dashboards
  copy:
    src: "{{ item }}"
    dest: /etc/grafana/dashboards/
  with_fileglob:
    - "dashboards/*.json"
  notify: restart grafana
```

## 參考檔案

- `assets/api-dashboard.json` - API 監控儀表板
- `assets/infrastructure-dashboard.json` - 基礎架構儀表板
- `assets/database-dashboard.json` - 資料庫監控儀表板
- `references/dashboard-design.md` - 儀表板設計指南

## 相關技能

- `prometheus-configuration` - 用於指標收集
- `slo-implementation` - 用於 SLO 儀表板
