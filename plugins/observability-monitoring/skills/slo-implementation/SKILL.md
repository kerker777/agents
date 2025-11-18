---
name: slo-implementation
description: 定義和實作服務等級指標（SLI）、服務等級目標（SLO），包含錯誤預算和告警機制。適用於建立可靠性目標、實施 SRE 實務，或衡量服務效能。
---

# SLO 實作

定義和實作服務等級指標（Service Level Indicators, SLI）、服務等級目標（Service Level Objectives, SLO）和錯誤預算的框架。

## 目的

使用 SLI、SLO 和錯誤預算實作可衡量的可靠性目標，以平衡可靠性與創新速度。

## 使用時機

- 定義服務可靠性目標
- 衡量使用者感知的可靠性
- 實作錯誤預算
- 建立基於 SLO 的告警
- 追蹤可靠性目標

## SLI/SLO/SLA 層級架構

```
SLA (Service Level Agreement, 服務等級協議)
  ↓ 與客戶的契約
SLO (Service Level Objective, 服務等級目標)
  ↓ 內部可靠性目標
SLI (Service Level Indicator, 服務等級指標)
  ↓ 實際測量值
```

## 定義 SLI

### 常見的 SLI 類型

#### 1. 可用性 SLI
```promql
# 成功請求數 / 總請求數
sum(rate(http_requests_total{status!~"5.."}[28d]))
/
sum(rate(http_requests_total[28d]))
```

#### 2. 延遲 SLI
```promql
# 低於延遲閾值的請求數 / 總請求數
sum(rate(http_request_duration_seconds_bucket{le="0.5"}[28d]))
/
sum(rate(http_request_duration_seconds_count[28d]))
```

#### 3. 持久性 SLI
```
# 成功寫入數 / 總寫入數
sum(storage_writes_successful_total)
/
sum(storage_writes_total)
```

**參考：** 請見 `references/slo-definitions.md`

## 設定 SLO 目標

### 可用性 SLO 範例

| SLO % | 每月停機時間 | 每年停機時間 |
|-------|-------------|-------------|
| 99%   | 7.2 小時    | 3.65 天     |
| 99.9% | 43.2 分鐘   | 8.76 小時   |
| 99.95%| 21.6 分鐘   | 4.38 小時   |
| 99.99%| 4.32 分鐘   | 52.56 分鐘  |

### 選擇適當的 SLO

**考量因素：**
- 使用者期望
- 業務需求
- 目前效能
- 可靠性成本
- 競爭對手基準

**SLO 範例：**
```yaml
slos:
  - name: api_availability
    target: 99.9
    window: 28d
    sli: |
      sum(rate(http_requests_total{status!~"5.."}[28d]))
      /
      sum(rate(http_requests_total[28d]))

  - name: api_latency_p95
    target: 99
    window: 28d
    sli: |
      sum(rate(http_request_duration_seconds_bucket{le="0.5"}[28d]))
      /
      sum(rate(http_request_duration_seconds_count[28d]))
```

## 錯誤預算計算

### 錯誤預算公式

```
錯誤預算 = 1 - SLO 目標值
```

**範例：**
- SLO：99.9% 可用性
- 錯誤預算：0.1% = 43.2 分鐘/月
- 當前錯誤：0.05% = 21.6 分鐘/月
- 剩餘預算：50%

### 錯誤預算政策

```yaml
error_budget_policy:
  - remaining_budget: 100%
    action: Normal development velocity
  - remaining_budget: 50%
    action: Consider postponing risky changes
  - remaining_budget: 10%
    action: Freeze non-critical changes
  - remaining_budget: 0%
    action: Feature freeze, focus on reliability
```

**參考：** 請見 `references/error-budget.md`

## SLO 實作

### Prometheus Recording Rules

```yaml
# SLI Recording Rules
groups:
  - name: sli_rules
    interval: 30s
    rules:
      # 可用性 SLI
      - record: sli:http_availability:ratio
        expr: |
          sum(rate(http_requests_total{status!~"5.."}[28d]))
          /
          sum(rate(http_requests_total[28d]))

      # 延遲 SLI (請求 < 500ms)
      - record: sli:http_latency:ratio
        expr: |
          sum(rate(http_request_duration_seconds_bucket{le="0.5"}[28d]))
          /
          sum(rate(http_request_duration_seconds_count[28d]))

  - name: slo_rules
    interval: 5m
    rules:
      # SLO 合規性 (1 = 符合 SLO, 0 = 違反)
      - record: slo:http_availability:compliance
        expr: sli:http_availability:ratio >= bool 0.999

      - record: slo:http_latency:compliance
        expr: sli:http_latency:ratio >= bool 0.99

      # 剩餘錯誤預算 (百分比)
      - record: slo:http_availability:error_budget_remaining
        expr: |
          (sli:http_availability:ratio - 0.999) / (1 - 0.999) * 100

      # 錯誤預算消耗率
      - record: slo:http_availability:burn_rate_5m
        expr: |
          (1 - (
            sum(rate(http_requests_total{status!~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          )) / (1 - 0.999)
```

### SLO 告警規則

```yaml
groups:
  - name: slo_alerts
    interval: 1m
    rules:
      # 快速消耗：14.4 倍速率，1 小時視窗
      # 1 小時內消耗 2% 錯誤預算
      - alert: SLOErrorBudgetBurnFast
        expr: |
          slo:http_availability:burn_rate_1h > 14.4
          and
          slo:http_availability:burn_rate_5m > 14.4
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "偵測到快速錯誤預算消耗"
          description: "錯誤預算以 {{ $value }} 倍速率消耗中"

      # 緩慢消耗：6 倍速率，6 小時視窗
      # 6 小時內消耗 5% 錯誤預算
      - alert: SLOErrorBudgetBurnSlow
        expr: |
          slo:http_availability:burn_rate_6h > 6
          and
          slo:http_availability:burn_rate_30m > 6
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "偵測到緩慢錯誤預算消耗"
          description: "錯誤預算以 {{ $value }} 倍速率消耗中"

      # 錯誤預算耗盡
      - alert: SLOErrorBudgetExhausted
        expr: slo:http_availability:error_budget_remaining < 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "SLO 錯誤預算已耗盡"
          description: "剩餘錯誤預算：{{ $value }}%"
```

## SLO 儀表板

**Grafana 儀表板結構：**

```
┌────────────────────────────────────┐
│ SLO 合規性（當前）                  │
│ ✓ 99.95% (目標：99.9%)             │
├────────────────────────────────────┤
│ 剩餘錯誤預算：65%                   │
│ ████████░░ 65%                     │
├────────────────────────────────────┤
│ SLI 趨勢（28 天）                   │
│ [時間序列圖表]                      │
├────────────────────────────────────┤
│ 消耗率分析                          │
│ [依時間視窗的消耗率]                │
└────────────────────────────────────┘
```

**查詢範例：**

```promql
# 當前 SLO 合規性
sli:http_availability:ratio * 100

# 剩餘錯誤預算
slo:http_availability:error_budget_remaining

# 錯誤預算耗盡前的天數（以當前消耗率計算）
(slo:http_availability:error_budget_remaining / 100)
*
28
/
(1 - sli:http_availability:ratio) * (1 - 0.999)
```

## 多視窗消耗率告警

```yaml
# 結合短期和長期視窗以減少誤報
rules:
  - alert: SLOBurnRateHigh
    expr: |
      (
        slo:http_availability:burn_rate_1h > 14.4
        and
        slo:http_availability:burn_rate_5m > 14.4
      )
      or
      (
        slo:http_availability:burn_rate_6h > 6
        and
        slo:http_availability:burn_rate_30m > 6
      )
    labels:
      severity: critical
```

## SLO 審查流程

### 每週審查
- 當前 SLO 合規性
- 錯誤預算狀態
- 趨勢分析
- 事件影響

### 每月審查
- SLO 達成情況
- 錯誤預算使用情況
- 事件事後檢討
- SLO 調整

### 每季審查
- SLO 相關性
- 目標調整
- 流程改善
- 工具增強

## 最佳實務

1. **從面向使用者的服務開始**
2. **使用多個 SLI**（可用性、延遲等）
3. **設定可達成的 SLO**（不要追求 100%）
4. **實作多視窗告警**以減少雜訊
5. **持續追蹤錯誤預算**
6. **定期審查 SLO**
7. **記錄 SLO 決策**
8. **與業務目標對齊**
9. **自動化 SLO 報告**
10. **使用 SLO 進行優先順序排序**

## 參考檔案

- `assets/slo-template.md` - SLO 定義範本
- `references/slo-definitions.md` - SLO 定義模式
- `references/error-budget.md` - 錯誤預算計算

## 相關技能

- `prometheus-configuration` - 用於指標收集
- `grafana-dashboards` - 用於 SLO 視覺化
