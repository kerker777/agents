---
name: cost-optimization
description: 透過資源調整、標籤策略、預留執行個體和支出分析來優化雲端成本。適用於降低雲端費用、分析基礎設施成本，或實施成本治理政策。
---

# 雲端成本優化

針對 AWS、Azure 和 GCP 的成本優化策略與模式。

## 目的

實施系統化的成本優化策略，在維持效能與可靠性的同時降低雲端支出。

## 使用時機

- 降低雲端支出
- 調整資源規模
- 實施成本治理
- 優化多雲成本
- 滿足預算限制

## 成本優化框架

### 1. 可見性
- 實施成本分配標籤
- 使用雲端成本管理工具
- 設定預算警報
- 建立成本儀表板

### 2. 資源調整
- 分析資源使用率
- 縮減過度配置的資源
- 使用自動擴展
- 移除閒置資源

### 3. 定價模式
- 使用預留容量
- 運用 Spot/可搶佔執行個體
- 實施節省方案
- 使用承諾使用折扣

### 4. 架構優化
- 使用託管服務
- 實施快取
- 優化資料傳輸
- 使用生命週期政策

## AWS 成本優化

### Reserved Instances
```
節省：相比隨需 30-72%
期限：1 或 3 年
付款：全部預付/部分預付/不預付
彈性：標準或可轉換
```

### Savings Plans
```
Compute Savings Plans：節省 66%
EC2 Instance Savings Plans：節省 72%
適用於：EC2、Fargate、Lambda
彈性跨越：執行個體系列、區域、作業系統
```

### Spot Instances
```
節省：相比隨需最高 90%
最適合：批次作業、CI/CD、無狀態工作負載
風險：2 分鐘中斷通知
策略：與隨需混合使用以提高韌性
```

### S3 成本優化
```hcl
resource "aws_s3_bucket_lifecycle_configuration" "example" {
  bucket = aws_s3_bucket.example.id

  rule {
    id     = "transition-to-ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }
}
```

## Azure 成本優化

### Reserved VM Instances
- 1 或 3 年期
- 最高節省 72%
- 彈性調整大小
- 可交換

### Azure Hybrid Benefit
- 使用現有 Windows Server 授權
- 搭配 RI 最高節省 80%
- 適用於 Windows 和 SQL Server

### Azure Advisor 建議
- 調整 VM 規模
- 刪除未使用的資源
- 使用預留容量
- 優化儲存

## GCP 成本優化

### Committed Use Discounts
- 1 或 3 年承諾
- 最高節省 57%
- 適用於 vCPU 和記憶體
- 資源型或支出型

### Sustained Use Discounts
- 自動折扣
- 執行執行個體最高節省 30%
- 無需承諾
- 適用於 Compute Engine、GKE

### Preemptible VMs
- 最高節省 80%
- 最長執行時間 24 小時
- 最適合批次工作負載

## 標籤策略

### AWS 標籤
```hcl
locals {
  common_tags = {
    Environment = "production"
    Project     = "my-project"
    CostCenter  = "engineering"
    Owner       = "team@example.com"
    ManagedBy   = "terraform"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"

  tags = merge(
    local.common_tags,
    {
      Name = "web-server"
    }
  )
}
```

**參考：**參見 `references/tagging-standards.md`

## 成本監控

### 預算警報
```hcl
# AWS Budget
resource "aws_budgets_budget" "monthly" {
  name              = "monthly-budget"
  budget_type       = "COST"
  limit_amount      = "1000"
  limit_unit        = "USD"
  time_period_start = "2024-01-01_00:00"
  time_unit         = "MONTHLY"

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type            = "PERCENTAGE"
    notification_type         = "ACTUAL"
    subscriber_email_addresses = ["team@example.com"]
  }
}
```

### 成本異常偵測
- AWS Cost Anomaly Detection
- Azure Cost Management 警報
- GCP Budget 警報

## 架構模式

### 模式 1：Serverless 優先
- 對事件驅動使用 Lambda/Functions
- 僅為執行時間付費
- 包含自動擴展
- 無閒置成本

### 模式 2：適當調整的資料庫
```
開發環境：t3.small RDS
預備環境：t3.large RDS
正式環境：r6g.2xlarge RDS 搭配讀取副本
```

### 模式 3：多層儲存
```
熱資料：S3 Standard
溫資料：S3 Standard-IA（30 天）
冷資料：S3 Glacier（90 天）
封存：S3 Deep Archive（365 天）
```

### 模式 4：自動擴展
```hcl
resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale-up"
  scaling_adjustment     = 2
  adjustment_type        = "ChangeInCapacity"
  cooldown              = 300
  autoscaling_group_name = aws_autoscaling_group.main.name
}

resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name          = "cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "60"
  statistic           = "Average"
  threshold           = "80"
  alarm_actions       = [aws_autoscaling_policy.scale_up.arn]
}
```

## 成本優化檢查清單

- [ ] 實施成本分配標籤
- [ ] 刪除未使用的資源（EBS、EIP、快照）
- [ ] 根據使用率調整執行個體規模
- [ ] 對穩定工作負載使用預留容量
- [ ] 實施自動擴展
- [ ] 優化儲存類別
- [ ] 使用生命週期政策
- [ ] 啟用成本異常偵測
- [ ] 設定預算警報
- [ ] 每週檢視成本
- [ ] 使用 Spot/可搶佔執行個體
- [ ] 優化資料傳輸成本
- [ ] 實施快取層
- [ ] 使用託管服務
- [ ] 持續監控和優化

## 工具

- **AWS：**Cost Explorer、Cost Anomaly Detection、Compute Optimizer
- **Azure：**Cost Management、Advisor
- **GCP：**Cost Management、Recommender
- **多雲：**CloudHealth、Cloudability、Kubecost

## 參考檔案

- `references/tagging-standards.md` - 標籤慣例
- `assets/cost-analysis-template.xlsx` - 成本分析試算表

## 相關技能

- `terraform-module-library` - 用於資源配置
- `multi-cloud-architecture` - 用於雲端選擇
