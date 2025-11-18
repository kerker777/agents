# AWS Terraform 模組樣式

## VPC 模組
- 具有公有/私有子網路的 VPC
- Internet Gateway 和 NAT Gateway
- 路由表與關聯
- 網路 ACL
- VPC 流量日誌

## EKS 模組
- 具有受管節點群組的 EKS 叢集
- IRSA (服務帳戶的 IAM 角色)
- 叢集自動擴展
- VPC CNI 組態
- 叢集日誌記錄

## RDS 模組
- RDS 執行個體或叢集
- 自動備份
- 讀取複本
- 參數群組
- 子網路群組
- 安全群組

## S3 模組
- 具有版本控制的 S3 儲存貯體
- 靜態加密
- 儲存貯體政策
- 生命週期規則
- 複寫組態

## ALB 模組
- Application Load Balancer
- 目標群組
- 監聽器規則
- SSL/TLS 憑證
- 存取日誌

## Lambda 模組
- Lambda 函式
- IAM 執行角色
- CloudWatch Logs
- 環境變數
- VPC 組態（選用）

## Security Group 模組
- 可重複使用的安全群組規則
- 輸入/輸出規則
- 動態規則建立
- 規則描述

## 最佳實務

1. 使用 AWS provider 版本 ~> 5.0
2. 預設啟用加密
3. 使用最小權限 IAM
4. 為所有資源一致地標記標籤
5. 啟用日誌記錄與監控
6. 使用 KMS 進行加密
7. 實作備份策略
8. 盡可能使用 PrivateLink
9. 啟用 GuardDuty/SecurityHub
10. 遵循 AWS Well-Architected Framework
