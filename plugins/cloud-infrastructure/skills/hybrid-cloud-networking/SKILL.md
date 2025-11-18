---
name: hybrid-cloud-networking
description: 使用 VPN 和專用連線設定地端基礎設施與雲端平台之間安全、高效能的連線。適用於建立混合雲架構、將資料中心連接到雲端，或實施安全的跨場域網路。
---

# 混合雲網路

使用 VPN、Direct Connect 和 ExpressRoute 設定地端與雲端環境之間安全、高效能的連線。

## 目的

在地端資料中心與雲端供應商（AWS、Azure、GCP）之間建立安全、可靠的網路連線。

## 使用時機

- 將地端連接到雲端
- 將資料中心擴展到雲端
- 實施混合雙活設定
- 滿足合規要求
- 逐步遷移到雲端

## 連線選項

### AWS 連線

#### 1. Site-to-Site VPN
- 透過網際網路的 IPSec VPN
- 每個通道最高 1.25 Gbps
- 適中頻寬的經濟實惠方案
- 較高延遲，依賴網際網路

```hcl
resource "aws_vpn_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags = {
    Name = "main-vpn-gateway"
  }
}

resource "aws_customer_gateway" "main" {
  bgp_asn    = 65000
  ip_address = "203.0.113.1"
  type       = "ipsec.1"
}

resource "aws_vpn_connection" "main" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.main.id
  type                = "ipsec.1"
  static_routes_only  = false
}
```

#### 2. AWS Direct Connect
- 專用網路連線
- 1 Gbps 至 100 Gbps
- 較低延遲、穩定頻寬
- 較昂貴，需要設定時間

**參考：**參見 `references/direct-connect.md`

### Azure 連線

#### 1. Site-to-Site VPN
```hcl
resource "azurerm_virtual_network_gateway" "vpn" {
  name                = "vpn-gateway"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  type     = "Vpn"
  vpn_type = "RouteBased"
  sku      = "VpnGw1"

  ip_configuration {
    name                          = "vnetGatewayConfig"
    public_ip_address_id          = azurerm_public_ip.vpn.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.gateway.id
  }
}
```

#### 2. Azure ExpressRoute
- 透過連線供應商的私有連線
- 最高 100 Gbps
- 低延遲、高可靠性
- Premium 提供全球連線

### GCP 連線

#### 1. Cloud VPN
- IPSec VPN（Classic 或 HA VPN）
- HA VPN：99.99% SLA
- 每個通道最高 3 Gbps

#### 2. Cloud Interconnect
- 專用（10 Gbps、100 Gbps）
- 合作夥伴（50 Mbps 至 50 Gbps）
- 延遲低於 VPN

## 混合網路模式

### 模式 1：中心輻射式
```
地端資料中心
         ↓
    VPN/Direct Connect
         ↓
    Transit Gateway (AWS) / vWAN (Azure)
         ↓
    ├─ Production VPC/VNet
    ├─ Staging VPC/VNet
    └─ Development VPC/VNet
```

### 模式 2：多區域混合
```
地端
    ├─ Direct Connect → us-east-1
    └─ Direct Connect → us-west-2
            ↓
        跨區域對等互連
```

### 模式 3：多雲混合
```
地端資料中心
    ├─ Direct Connect → AWS
    ├─ ExpressRoute → Azure
    └─ Interconnect → GCP
```

## 路由設定

### BGP 設定
```
地端路由器：
- AS 號碼：65000
- 通告：10.0.0.0/8

雲端路由器：
- AS 號碼：64512（AWS）、65515（Azure）
- 通告：雲端 VPC/VNet CIDR
```

### 路由傳播
- 在路由表上啟用路由傳播
- 使用 BGP 進行動態路由
- 實施路由篩選
- 監控路由通告

## 安全最佳實踐

1. **使用私有連線**（Direct Connect/ExpressRoute）
2. **為 VPN 通道實施加密**
3. **使用 VPC 端點**避免網際網路路由
4. **設定網路 ACL** 和安全群組
5. **啟用 VPC Flow Logs** 進行監控
6. **實施 DDoS 防護**
7. **使用 PrivateLink/私有端點**
8. **使用 CloudWatch/Monitor 監控連線**
9. **實施備援**（雙通道）
10. **定期安全稽核**

## 高可用性

### 雙 VPN 通道
```hcl
resource "aws_vpn_connection" "primary" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.primary.id
  type                = "ipsec.1"
}

resource "aws_vpn_connection" "secondary" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.secondary.id
  type                = "ipsec.1"
}
```

### 雙活設定
- 從不同地點建立多個連線
- 使用 BGP 進行自動容錯移轉
- 等價多路徑（ECMP）路由
- 監控所有連線的健康狀態

## 監控與疑難排解

### 關鍵指標
- 通道狀態（啟動/關閉）
- 傳入/傳出位元組
- 封包遺失
- 延遲
- BGP 工作階段狀態

### 疑難排解
```bash
# AWS VPN
aws ec2 describe-vpn-connections
aws ec2 get-vpn-connection-telemetry

# Azure VPN
az network vpn-connection show
az network vpn-connection show-device-config-script
```

## 成本優化

1. **根據流量適當調整連線規模**
2. **對低頻寬工作負載使用 VPN**
3. **透過較少的連線整合流量**
4. **降低資料傳輸成本**
5. **對高頻寬使用 Direct Connect**
6. **實施快取以減少流量**

## 參考檔案

- `references/vpn-setup.md` - VPN 設定指南
- `references/direct-connect.md` - Direct Connect 設定

## 相關技能

- `multi-cloud-architecture` - 用於架構決策
- `terraform-module-library` - 用於 IaC 實施
