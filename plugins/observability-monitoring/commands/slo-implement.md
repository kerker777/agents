# SLO 實施指南

您是一位 SLO（Service Level Objective，服務水準目標）專家，專精於實施可靠性標準和基於錯誤預算的工程實踐。設計全面的 SLO 框架、建立有意義的 SLI，並創建能平衡可靠性與功能開發速度的監控系統。

## 情境說明
使用者需要實施 SLO 來建立可靠性目標、衡量服務效能，並針對可靠性與功能開發做出數據驅動的決策。專注於與業務目標一致的實用 SLO 實施。

## 需求
$ARGUMENTS

## 操作指引

### 1. SLO 基礎

建立 SLO 基本原則和框架：

**SLO 框架設計器**
```python
import numpy as np
from datetime import datetime, timedelta
from typing import Dict, List, Optional

class SLOFramework:
    def __init__(self, service_name: str):
        self.service = service_name
        self.slos = []
        self.error_budget = None

    def design_slo_framework(self):
        """
        設計全面的 SLO 框架
        """
        framework = {
            'service_context': self._analyze_service_context(),
            'user_journeys': self._identify_user_journeys(),
            'sli_candidates': self._identify_sli_candidates(),
            'slo_targets': self._calculate_slo_targets(),
            'error_budgets': self._define_error_budgets(),
            'measurement_strategy': self._design_measurement_strategy()
        }

        return self._generate_slo_specification(framework)

    def _analyze_service_context(self):
        """分析服務特性以進行 SLO 設計"""
        return {
            'service_tier': self._determine_service_tier(),
            'user_expectations': self._assess_user_expectations(),
            'business_impact': self._evaluate_business_impact(),
            'technical_constraints': self._identify_constraints(),
            'dependencies': self._map_dependencies()
        }

    def _determine_service_tier(self):
        """確定適當的服務層級和 SLO 目標"""
        tiers = {
            'critical': {
                'description': '營收關鍵或安全關鍵服務',
                'availability_target': 99.95,
                'latency_p99': 100,
                'error_rate': 0.001,
                'examples': ['payment processing', 'authentication']
            },
            'essential': {
                'description': '核心業務功能',
                'availability_target': 99.9,
                'latency_p99': 500,
                'error_rate': 0.01,
                'examples': ['search', 'product catalog']
            },
            'standard': {
                'description': '標準功能',
                'availability_target': 99.5,
                'latency_p99': 1000,
                'error_rate': 0.05,
                'examples': ['recommendations', 'analytics']
            },
            'best_effort': {
                'description': '非關鍵功能',
                'availability_target': 99.0,
                'latency_p99': 2000,
                'error_rate': 0.1,
                'examples': ['batch processing', 'reporting']
            }
        }

        # 分析服務特性以確定層級
        characteristics = self._analyze_service_characteristics()
        recommended_tier = self._match_tier(characteristics, tiers)

        return {
            'recommended': recommended_tier,
            'rationale': self._explain_tier_selection(characteristics),
            'all_tiers': tiers
        }

    def _identify_user_journeys(self):
        """映射關鍵使用者旅程以選擇 SLI"""
        journeys = []

        # 使用者旅程映射範例
        journey_template = {
            'name': 'User Login',
            'description': '使用者驗證並存取儀表板',
            'steps': [
                {
                    'step': 'Load login page',
                    'sli_type': 'availability',
                    'threshold': '< 2s load time'
                },
                {
                    'step': 'Submit credentials',
                    'sli_type': 'latency',
                    'threshold': '< 500ms response'
                },
                {
                    'step': 'Validate authentication',
                    'sli_type': 'error_rate',
                    'threshold': '< 0.1% auth failures'
                },
                {
                    'step': 'Load dashboard',
                    'sli_type': 'latency',
                    'threshold': '< 3s full render'
                }
            ],
            'critical_path': True,
            'business_impact': 'high'
        }

        return journeys
```

### 2. SLI 選擇與測量

選擇並實施適當的 SLI：

**SLI 實作**
```python
class SLIImplementation:
    def __init__(self):
        self.sli_types = {
            'availability': AvailabilitySLI,
            'latency': LatencySLI,
            'error_rate': ErrorRateSLI,
            'throughput': ThroughputSLI,
            'quality': QualitySLI
        }

    def implement_slis(self, service_type):
        """根據服務類型實施 SLI"""
        if service_type == 'api':
            return self._api_slis()
        elif service_type == 'web':
            return self._web_slis()
        elif service_type == 'batch':
            return self._batch_slis()
        elif service_type == 'streaming':
            return self._streaming_slis()

    def _api_slis(self):
        """API 服務的 SLI"""
        return {
            'availability': {
                'definition': '成功請求的百分比',
                'formula': 'successful_requests / total_requests * 100',
                'implementation': '''
# API 可用性的 Prometheus 查詢
api_availability = """
sum(rate(http_requests_total{status!~"5.."}[5m])) /
sum(rate(http_requests_total[5m])) * 100
"""

# 實作
class APIAvailabilitySLI:
    def __init__(self, prometheus_client):
        self.prom = prometheus_client

    def calculate(self, time_range='5m'):
        query = f"""
        sum(rate(http_requests_total{{status!~"5.."}}[{time_range}])) /
        sum(rate(http_requests_total[{time_range}])) * 100
        """
        result = self.prom.query(query)
        return float(result[0]['value'][1])

    def calculate_with_exclusions(self, time_range='5m'):
        """計算排除特定端點的可用性"""
        query = f"""
        sum(rate(http_requests_total{{
            status!~"5..",
            endpoint!~"/health|/metrics"
        }}[{time_range}])) /
        sum(rate(http_requests_total{{
            endpoint!~"/health|/metrics"
        }}[{time_range}])) * 100
        """
        return self.prom.query(query)
'''
            },
            'latency': {
                'definition': '快於閾值的請求百分比',
                'formula': 'fast_requests / total_requests * 100',
                'implementation': '''
# 具有多個閾值的延遲 SLI
class LatencySLI:
    def __init__(self, thresholds_ms):
        self.thresholds = thresholds_ms  # 例如：{'p50': 100, 'p95': 500, 'p99': 1000}

    def calculate_latency_sli(self, time_range='5m'):
        slis = {}

        for percentile, threshold in self.thresholds.items():
            query = f"""
            sum(rate(http_request_duration_seconds_bucket{{
                le="{threshold/1000}"
            }}[{time_range}])) /
            sum(rate(http_request_duration_seconds_count[{time_range}])) * 100
            """

            slis[f'latency_{percentile}'] = {
                'value': self.execute_query(query),
                'threshold': threshold,
                'unit': 'ms'
            }

        return slis

    def calculate_user_centric_latency(self):
        """從使用者角度計算延遲"""
        # 包含客戶端指標
        query = """
        histogram_quantile(0.95,
            sum(rate(user_request_duration_bucket[5m])) by (le)
        )
        """
        return self.execute_query(query)
'''
            },
            'error_rate': {
                'definition': '成功請求的百分比',
                'formula': '(1 - error_requests / total_requests) * 100',
                'implementation': '''
class ErrorRateSLI:
    def calculate_error_rate(self, time_range='5m'):
        """計算帶分類的錯誤率"""

        # 不同的錯誤類別
        error_categories = {
            'client_errors': 'status=~"4.."',
            'server_errors': 'status=~"5.."',
            'timeout_errors': 'status="504"',
            'business_errors': 'error_type="business_logic"'
        }

        results = {}
        for category, filter_expr in error_categories.items():
            query = f"""
            sum(rate(http_requests_total{{{filter_expr}}}[{time_range}])) /
            sum(rate(http_requests_total[{time_range}])) * 100
            """
            results[category] = self.execute_query(query)

        # 整體錯誤率（排除 4xx）
        overall_query = f"""
        (1 - sum(rate(http_requests_total{{status=~"5.."}}[{time_range}])) /
        sum(rate(http_requests_total[{time_range}]))) * 100
        """
        results['overall_success_rate'] = self.execute_query(overall_query)

        return results
'''
            }
        }
```

### 3. 錯誤預算計算

實施錯誤預算追蹤：

**錯誤預算管理器**
```python
class ErrorBudgetManager:
    def __init__(self, slo_target: float, window_days: int):
        self.slo_target = slo_target
        self.window_days = window_days
        self.error_budget_minutes = self._calculate_total_budget()

    def _calculate_total_budget(self):
        """計算總錯誤預算（以分鐘為單位）"""
        total_minutes = self.window_days * 24 * 60
        allowed_downtime_ratio = 1 - (self.slo_target / 100)
        return total_minutes * allowed_downtime_ratio

    def calculate_error_budget_status(self, start_date, end_date):
        """計算當前錯誤預算狀態"""
        # 取得實際效能
        actual_uptime = self._get_actual_uptime(start_date, end_date)

        # 計算已消耗的預算
        total_time = (end_date - start_date).total_seconds() / 60
        expected_uptime = total_time * (self.slo_target / 100)
        consumed_minutes = expected_uptime - actual_uptime

        # 計算剩餘預算
        remaining_budget = self.error_budget_minutes - consumed_minutes
        burn_rate = consumed_minutes / self.error_budget_minutes

        # 預測耗盡時間
        if burn_rate > 0:
            days_until_exhaustion = (self.window_days * (1 - burn_rate)) / burn_rate
        else:
            days_until_exhaustion = float('inf')

        return {
            'total_budget_minutes': self.error_budget_minutes,
            'consumed_minutes': consumed_minutes,
            'remaining_minutes': remaining_budget,
            'burn_rate': burn_rate,
            'budget_percentage_remaining': (remaining_budget / self.error_budget_minutes) * 100,
            'projected_exhaustion_days': days_until_exhaustion,
            'status': self._determine_status(remaining_budget, burn_rate)
        }

    def _determine_status(self, remaining_budget, burn_rate):
        """判斷錯誤預算狀態"""
        if remaining_budget <= 0:
            return 'exhausted'
        elif burn_rate > 2:
            return 'critical'
        elif burn_rate > 1.5:
            return 'warning'
        elif burn_rate > 1:
            return 'attention'
        else:
            return 'healthy'

    def generate_burn_rate_alerts(self):
        """產生多時間窗口燃燒率警報"""
        return {
            'fast_burn': {
                'description': '1 小時內 14.4 倍燃燒率',
                'condition': 'burn_rate >= 14.4 AND window = 1h',
                'action': 'page',
                'budget_consumed': '1 小時內消耗 2%'
            },
            'slow_burn': {
                'description': '6 小時內 3 倍燃燒率',
                'condition': 'burn_rate >= 3 AND window = 6h',
                'action': 'ticket',
                'budget_consumed': '6 小時內消耗 10%'
            }
        }
```

### 4. SLO 監控設置

實施全面的 SLO 監控：

**SLO 監控實作**
```yaml
# SLO 的 Prometheus 記錄規則
groups:
  - name: slo_rules
    interval: 30s
    rules:
      # 請求率
      - record: service:request_rate
        expr: |
          sum(rate(http_requests_total[5m])) by (service, method, route)

      # 成功率
      - record: service:success_rate_5m
        expr: |
          (
            sum(rate(http_requests_total{status!~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)
          ) * 100

      # 多時間窗口成功率
      - record: service:success_rate_30m
        expr: |
          (
            sum(rate(http_requests_total{status!~"5.."}[30m])) by (service)
            /
            sum(rate(http_requests_total[30m])) by (service)
          ) * 100

      - record: service:success_rate_1h
        expr: |
          (
            sum(rate(http_requests_total{status!~"5.."}[1h])) by (service)
            /
            sum(rate(http_requests_total[1h])) by (service)
          ) * 100

      # 延遲百分位數
      - record: service:latency_p50_5m
        expr: |
          histogram_quantile(0.50,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
          )

      - record: service:latency_p95_5m
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
          )

      - record: service:latency_p99_5m
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
          )

      # 錯誤預算燃燒率
      - record: service:error_budget_burn_rate_1h
        expr: |
          (
            1 - (
              sum(increase(http_requests_total{status!~"5.."}[1h])) by (service)
              /
              sum(increase(http_requests_total[1h])) by (service)
            )
          ) / (1 - 0.999) # 99.9% SLO
```

**警報配置**
```yaml
# 多時間窗口多燃燒率警報
groups:
  - name: slo_alerts
    rules:
      # 快速燃燒警報（1 小時內消耗 2% 預算）
      - alert: ErrorBudgetFastBurn
        expr: |
          (
            service:error_budget_burn_rate_5m{service="api"} > 14.4
            AND
            service:error_budget_burn_rate_1h{service="api"} > 14.4
          )
        for: 2m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "{{ $labels.service }} 發生快速錯誤預算燃燒"
          description: |
            服務 {{ $labels.service }} 正以 14.4 倍的速率燃燒錯誤預算。
            當前燃燒率：{{ $value }}x
            這將在 1 小時內耗盡 2% 的月度預算。

      # 緩慢燃燒警報（6 小時內消耗 10% 預算）
      - alert: ErrorBudgetSlowBurn
        expr: |
          (
            service:error_budget_burn_rate_30m{service="api"} > 3
            AND
            service:error_budget_burn_rate_6h{service="api"} > 3
          )
        for: 15m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "{{ $labels.service }} 發生緩慢錯誤預算燃燒"
          description: |
            服務 {{ $labels.service }} 正以 3 倍的速率燃燒錯誤預算。
            當前燃燒率：{{ $value }}x
            這將在 6 小時內耗盡 10% 的月度預算。
```

### 5. SLO 儀表板

創建全面的 SLO 儀表板：

**Grafana 儀表板配置**
```python
def create_slo_dashboard():
    """產生用於 SLO 監控的 Grafana 儀表板"""
    return {
        "dashboard": {
            "title": "Service SLO Dashboard",
            "panels": [
                {
                    "title": "SLO Summary",
                    "type": "stat",
                    "gridPos": {"h": 4, "w": 6, "x": 0, "y": 0},
                    "targets": [{
                        "expr": "service:success_rate_30d{service=\"$service\"}",
                        "legendFormat": "30-day SLO"
                    }],
                    "fieldConfig": {
                        "defaults": {
                            "thresholds": {
                                "mode": "absolute",
                                "steps": [
                                    {"color": "red", "value": None},
                                    {"color": "yellow", "value": 99.5},
                                    {"color": "green", "value": 99.9}
                                ]
                            },
                            "unit": "percent"
                        }
                    }
                },
                {
                    "title": "Error Budget Status",
                    "type": "gauge",
                    "gridPos": {"h": 4, "w": 6, "x": 6, "y": 0},
                    "targets": [{
                        "expr": '''
                        100 * (
                            1 - (
                                (1 - service:success_rate_30d{service="$service"}/100) /
                                (1 - $slo_target/100)
                            )
                        )
                        ''',
                        "legendFormat": "Remaining Budget"
                    }],
                    "fieldConfig": {
                        "defaults": {
                            "min": 0,
                            "max": 100,
                            "thresholds": {
                                "mode": "absolute",
                                "steps": [
                                    {"color": "red", "value": None},
                                    {"color": "yellow", "value": 20},
                                    {"color": "green", "value": 50}
                                ]
                            },
                            "unit": "percent"
                        }
                    }
                },
                {
                    "title": "Burn Rate Trend",
                    "type": "graph",
                    "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
                    "targets": [
                        {
                            "expr": "service:error_budget_burn_rate_1h{service=\"$service\"}",
                            "legendFormat": "1h burn rate"
                        },
                        {
                            "expr": "service:error_budget_burn_rate_6h{service=\"$service\"}",
                            "legendFormat": "6h burn rate"
                        },
                        {
                            "expr": "service:error_budget_burn_rate_24h{service=\"$service\"}",
                            "legendFormat": "24h burn rate"
                        }
                    ],
                    "yaxes": [{
                        "format": "short",
                        "label": "Burn Rate (x)",
                        "min": 0
                    }],
                    "alert": {
                        "conditions": [{
                            "evaluator": {"params": [14.4], "type": "gt"},
                            "operator": {"type": "and"},
                            "query": {"params": ["A", "5m", "now"]},
                            "type": "query"
                        }],
                        "name": "High burn rate detected"
                    }
                }
            ]
        }
    }
```

### 6. SLO 報告

產生 SLO 報告和審查：

**SLO 報告產生器**
```python
class SLOReporter:
    def __init__(self, metrics_client):
        self.metrics = metrics_client

    def generate_monthly_report(self, service, month):
        """產生全面的月度 SLO 報告"""
        report_data = {
            'service': service,
            'period': month,
            'slo_performance': self._calculate_slo_performance(service, month),
            'incidents': self._analyze_incidents(service, month),
            'error_budget': self._analyze_error_budget(service, month),
            'trends': self._analyze_trends(service, month),
            'recommendations': self._generate_recommendations(service, month)
        }

        return self._format_report(report_data)

    def _calculate_slo_performance(self, service, month):
        """計算 SLO 效能指標"""
        slos = {}

        # 可用性 SLO
        availability_query = f"""
        avg_over_time(
            service:success_rate_5m{{service="{service}"}}[{month}]
        )
        """
        slos['availability'] = {
            'target': 99.9,
            'actual': self.metrics.query(availability_query),
            'met': self.metrics.query(availability_query) >= 99.9
        }

        # 延遲 SLO
        latency_query = f"""
        quantile_over_time(0.95,
            service:latency_p95_5m{{service="{service}"}}[{month}]
        )
        """
        slos['latency_p95'] = {
            'target': 500,  # ms
            'actual': self.metrics.query(latency_query) * 1000,
            'met': self.metrics.query(latency_query) * 1000 <= 500
        }

        return slos

    def _format_report(self, data):
        """將報告格式化為 HTML"""
        return f"""
<!DOCTYPE html>
<html>
<head>
    <title>SLO 報告 - {data['service']} - {data['period']}</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 40px; }}
        .summary {{ background: #f0f0f0; padding: 20px; border-radius: 8px; }}
        .metric {{ margin: 20px 0; }}
        .good {{ color: green; }}
        .bad {{ color: red; }}
        table {{ border-collapse: collapse; width: 100%; }}
        th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
        .chart {{ margin: 20px 0; }}
    </style>
</head>
<body>
    <h1>SLO 報告：{data['service']}</h1>
    <h2>期間：{data['period']}</h2>

    <div class="summary">
        <h3>執行摘要</h3>
        <p>服務可靠性：{data['slo_performance']['availability']['actual']:.2f}%</p>
        <p>剩餘錯誤預算：{data['error_budget']['remaining_percentage']:.1f}%</p>
        <p>事件數量：{len(data['incidents'])}</p>
    </div>

    <div class="metric">
        <h3>SLO 效能</h3>
        <table>
            <tr>
                <th>SLO</th>
                <th>目標</th>
                <th>實際</th>
                <th>狀態</th>
            </tr>
            {self._format_slo_table_rows(data['slo_performance'])}
        </table>
    </div>

    <div class="incidents">
        <h3>事件分析</h3>
        {self._format_incident_analysis(data['incidents'])}
    </div>

    <div class="recommendations">
        <h3>建議</h3>
        {self._format_recommendations(data['recommendations'])}
    </div>
</body>
</html>
"""
```

### 7. 基於 SLO 的決策制定

實施 SLO 驅動的工程決策：

**SLO 決策框架**
```python
class SLODecisionFramework:
    def __init__(self, error_budget_policy):
        self.policy = error_budget_policy

    def make_release_decision(self, service, release_risk):
        """根據錯誤預算制定發布決策"""
        budget_status = self.get_error_budget_status(service)

        decision_matrix = {
            'healthy': {
                'low_risk': 'approve',
                'medium_risk': 'approve',
                'high_risk': 'review'
            },
            'attention': {
                'low_risk': 'approve',
                'medium_risk': 'review',
                'high_risk': 'defer'
            },
            'warning': {
                'low_risk': 'review',
                'medium_risk': 'defer',
                'high_risk': 'block'
            },
            'critical': {
                'low_risk': 'defer',
                'medium_risk': 'block',
                'high_risk': 'block'
            },
            'exhausted': {
                'low_risk': 'block',
                'medium_risk': 'block',
                'high_risk': 'block'
            }
        }

        decision = decision_matrix[budget_status['status']][release_risk]

        return {
            'decision': decision,
            'rationale': self._explain_decision(budget_status, release_risk),
            'conditions': self._get_approval_conditions(decision, budget_status),
            'alternative_actions': self._suggest_alternatives(decision, budget_status)
        }

    def prioritize_reliability_work(self, service):
        """根據 SLO 差距排定可靠性改進的優先順序"""
        slo_gaps = self.analyze_slo_gaps(service)

        priorities = []
        for gap in slo_gaps:
            priority_score = self.calculate_priority_score(gap)

            priorities.append({
                'issue': gap['issue'],
                'impact': gap['impact'],
                'effort': gap['estimated_effort'],
                'priority_score': priority_score,
                'recommended_actions': self.recommend_actions(gap)
            })

        return sorted(priorities, key=lambda x: x['priority_score'], reverse=True)

    def calculate_toil_budget(self, team_size, slo_performance):
        """根據 SLO 計算可接受的繁瑣工作量"""
        # 如果達成 SLO，可以承擔更多繁瑣工作
        # 如果未達成 SLO，需要減少繁瑣工作

        base_toil_percentage = 50  # Google SRE 建議

        if slo_performance >= 100:
            # 超越 SLO，可以承擔更多繁瑣工作
            toil_budget = base_toil_percentage + 10
        elif slo_performance >= 99:
            # 達成 SLO
            toil_budget = base_toil_percentage
        else:
            # 未達成 SLO，減少繁瑣工作
            toil_budget = base_toil_percentage - (100 - slo_performance) * 5

        return {
            'toil_percentage': max(toil_budget, 20),  # 最低 20%
            'toil_hours_per_week': (toil_budget / 100) * 40 * team_size,
            'automation_hours_per_week': ((100 - toil_budget) / 100) * 40 * team_size
        }
```

### 8. SLO 模板

提供常見服務的 SLO 模板：

**SLO 模板庫**
```python
class SLOTemplates:
    @staticmethod
    def get_api_service_template():
        """API 服務的 SLO 模板"""
        return {
            'name': 'API 服務 SLO 模板',
            'slos': [
                {
                    'name': 'availability',
                    'description': '成功請求的比例',
                    'sli': {
                        'type': 'ratio',
                        'good_events': 'requests with status != 5xx',
                        'total_events': 'all requests'
                    },
                    'objectives': [
                        {'window': '30d', 'target': 99.9}
                    ]
                },
                {
                    'name': 'latency',
                    'description': '快速請求的比例',
                    'sli': {
                        'type': 'ratio',
                        'good_events': 'requests faster than 500ms',
                        'total_events': 'all requests'
                    },
                    'objectives': [
                        {'window': '30d', 'target': 95.0}
                    ]
                }
            ]
        }

    @staticmethod
    def get_data_pipeline_template():
        """資料管道的 SLO 模板"""
        return {
            'name': '資料管道 SLO 模板',
            'slos': [
                {
                    'name': 'freshness',
                    'description': '資料在 SLA 內處理完成',
                    'sli': {
                        'type': 'ratio',
                        'good_events': 'batches processed within 30 minutes',
                        'total_events': 'all batches'
                    },
                    'objectives': [
                        {'window': '7d', 'target': 99.0}
                    ]
                },
                {
                    'name': 'completeness',
                    'description': '所有預期資料都已處理',
                    'sli': {
                        'type': 'ratio',
                        'good_events': 'records successfully processed',
                        'total_events': 'all records'
                    },
                    'objectives': [
                        {'window': '7d', 'target': 99.95}
                    ]
                }
            ]
        }
```

### 9. SLO 自動化

自動化 SLO 管理：

**SLO 自動化工具**
```python
class SLOAutomation:
    def __init__(self):
        self.config = self.load_slo_config()

    def auto_generate_slos(self, service_discovery):
        """為發現的服務自動產生 SLO"""
        services = service_discovery.get_all_services()
        generated_slos = []

        for service in services:
            # 分析服務特性
            characteristics = self.analyze_service(service)

            # 選擇適當的模板
            template = self.select_template(characteristics)

            # 基於觀察到的行為進行客製化
            customized_slo = self.customize_slo(template, service)

            generated_slos.append(customized_slo)

        return generated_slos

    def implement_progressive_slos(self, service):
        """實施漸進式的更嚴格 SLO"""
        return {
            'phase1': {
                'duration': '1 個月',
                'target': 99.0,
                'description': '基準建立'
            },
            'phase2': {
                'duration': '2 個月',
                'target': 99.5,
                'description': '初步改進'
            },
            'phase3': {
                'duration': '3 個月',
                'target': 99.9,
                'description': '生產環境就緒'
            },
            'phase4': {
                'duration': '持續進行',
                'target': 99.95,
                'description': '追求卓越'
            }
        }

    def create_slo_as_code(self):
        """將 SLO 定義為程式碼"""
        return '''
# slo_definitions.yaml
apiVersion: slo.dev/v1
kind: ServiceLevelObjective
metadata:
  name: api-availability
  namespace: production
spec:
  service: api-service
  description: API 服務可用性 SLO

  indicator:
    type: ratio
    counter:
      metric: http_requests_total
      filters:
        - status_code != 5xx
    total:
      metric: http_requests_total

  objectives:
    - displayName: 30 天滾動窗口
      window: 30d
      target: 0.999

  alerting:
    burnRates:
      - severity: critical
        shortWindow: 1h
        longWindow: 5m
        burnRate: 14.4
      - severity: warning
        shortWindow: 6h
        longWindow: 30m
        burnRate: 3

  annotations:
    runbook: https://runbooks.example.com/api-availability
    dashboard: https://grafana.example.com/d/api-slo
'''
```

### 10. SLO 文化與治理

建立 SLO 文化：

**SLO 治理框架**
```python
class SLOGovernance:
    def establish_slo_culture(self):
        """建立 SLO 驅動的文化"""
        return {
            'principles': [
                'SLO 是共同責任',
                '錯誤預算驅動優先順序',
                '可靠性是一項功能',
                '衡量對使用者重要的事項'
            ],
            'practices': {
                'weekly_reviews': self.weekly_slo_review_template(),
                'incident_retrospectives': self.slo_incident_template(),
                'quarterly_planning': self.quarterly_slo_planning(),
                'stakeholder_communication': self.stakeholder_report_template()
            },
            'roles': {
                'slo_owner': {
                    'responsibilities': [
                        '定義並維護 SLO 定義',
                        '監控 SLO 效能',
                        '領導 SLO 審查',
                        '與利害關係人溝通'
                    ]
                },
                'engineering_team': {
                    'responsibilities': [
                        '實施 SLI 測量',
                        '回應 SLO 違規',
                        '改善可靠性',
                        '參與審查'
                    ]
                },
                'product_owner': {
                    'responsibilities': [
                        '平衡功能與可靠性',
                        '核准錯誤預算使用',
                        '設定業務優先順序',
                        '與客戶溝通'
                    ]
                }
            }
        }

    def create_slo_review_process(self):
        """創建結構化的 SLO 審查流程"""
        return '''
# 每週 SLO 審查模板

## 議程（30 分鐘）

### 1. SLO 效能審查（10 分鐘）
- 所有服務的當前 SLO 狀態
- 錯誤預算消耗率
- 趨勢分析

### 2. 事件審查（10 分鐘）
- 影響 SLO 的事件
- 根本原因分析
- 行動項目

### 3. 決策制定（10 分鐘）
- 發布核准/延期
- 資源分配
- 優先順序調整

## 審查檢查清單

- [ ] 已審查所有 SLO
- [ ] 已分析燃燒率
- [ ] 已討論事件
- [ ] 已分配行動項目
- [ ] 已記錄決策

## 輸出模板

### 服務：[服務名稱]
- **SLO 狀態**：[綠色/黃色/紅色]
- **錯誤預算**：剩餘 [XX%]
- **關鍵問題**：[清單]
- **行動項目**：[帶負責人的清單]
- **決策**：[清單]
'''
```

## 輸出格式

1. **SLO 框架**：全面的 SLO 設計和目標
2. **SLI 實作**：用於測量 SLI 的程式碼和查詢
3. **錯誤預算追蹤**：計算和燃燒率監控
4. **監控設置**：Prometheus 規則和 Grafana 儀表板
5. **警報配置**：多時間窗口多燃燒率警報
6. **報告模板**：月度報告和審查
7. **決策框架**：基於 SLO 的工程決策
8. **自動化工具**：SLO 即程式碼和自動產生
9. **治理流程**：文化和審查流程

專注於創建有意義的 SLO，在可靠性和功能開發速度之間取得平衡，為工程決策提供清晰的信號，並培養可靠性文化。