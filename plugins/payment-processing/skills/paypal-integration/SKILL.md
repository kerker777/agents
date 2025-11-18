---
name: paypal-integration
description: 整合 PayPal 金流處理，支援快速結帳、訂閱制及退款管理。適用於實作 PayPal 付款、處理線上交易或建置電子商務結帳流程。
---

# PayPal 整合

精通 PayPal 付款整合，包括 Express Checkout（快速結帳）、IPN 處理、定期扣款及退款工作流程。

## 何時使用此技能

- 整合 PayPal 作為付款選項
- 實作快速結帳流程
- 使用 PayPal 設定定期扣款
- 處理退款及付款爭議
- 處理 PayPal webhooks（IPN）
- 支援國際付款
- 實作 PayPal 訂閱制

## 核心概念

### 1. 付款產品
**PayPal Checkout（結帳）**
- 單次付款
- 快速結帳體驗
- 訪客及 PayPal 帳戶付款

**PayPal Subscriptions（訂閱制）**
- 定期扣款
- 訂閱方案
- 自動續約

**PayPal Payouts（批次付款）**
- 發送款項給多位收款人
- 市集及平台付款

### 2. 整合方式
**客戶端（JavaScript SDK）**
- Smart Payment Buttons（智慧付款按鈕）
- 託管式付款流程
- 最少的後端程式碼

**伺服器端（REST API）**
- 完整控制付款流程
- 自訂結帳介面
- 進階功能

### 3. IPN（Instant Payment Notification，即時付款通知）
- 類似 webhook 的付款通知
- 非同步付款更新
- 需要驗證

## 快速開始

```javascript
// 前端 - PayPal Smart Buttons
<div id="paypal-button-container"></div>

<script src="https://www.paypal.com/sdk/js?client-id=YOUR_CLIENT_ID&currency=USD"></script>
<script>
  paypal.Buttons({
    createOrder: function(data, actions) {
      return actions.order.create({
        purchase_units: [{
          amount: {
            value: '25.00'
          }
        }]
      });
    },
    onApprove: function(data, actions) {
      return actions.order.capture().then(function(details) {
        // 付款成功
        console.log('Transaction completed by ' + details.payer.name.given_name);

        // 發送到後端進行驗證
        fetch('/api/paypal/capture', {
          method: 'POST',
          headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({orderID: data.orderID})
        });
      });
    }
  }).render('#paypal-button-container');
</script>
```

```python
# 後端 - 驗證並擷取訂單
from paypalrestsdk import Payment
import paypalrestsdk

paypalrestsdk.configure({
    "mode": "sandbox",  # 或 "live"
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET"
})

def capture_paypal_order(order_id):
    """擷取 PayPal 訂單。"""
    payment = Payment.find(order_id)

    if payment.execute({"payer_id": payment.payer.payer_info.payer_id}):
        # 付款成功
        return {
            'status': 'success',
            'transaction_id': payment.id,
            'amount': payment.transactions[0].amount.total
        }
    else:
        # 付款失敗
        return {
            'status': 'failed',
            'error': payment.error
        }
```

## Express Checkout 實作

### 伺服器端訂單建立
```python
import requests
import json

class PayPalClient:
    def __init__(self, client_id, client_secret, mode='sandbox'):
        self.client_id = client_id
        self.client_secret = client_secret
        self.base_url = 'https://api-m.sandbox.paypal.com' if mode == 'sandbox' else 'https://api-m.paypal.com'
        self.access_token = self.get_access_token()

    def get_access_token(self):
        """取得 OAuth 存取權杖。"""
        url = f"{self.base_url}/v1/oauth2/token"
        headers = {"Accept": "application/json", "Accept-Language": "en_US"}

        response = requests.post(
            url,
            headers=headers,
            data={"grant_type": "client_credentials"},
            auth=(self.client_id, self.client_secret)
        )

        return response.json()['access_token']

    def create_order(self, amount, currency='USD'):
        """建立 PayPal 訂單。"""
        url = f"{self.base_url}/v2/checkout/orders"
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.access_token}"
        }

        payload = {
            "intent": "CAPTURE",
            "purchase_units": [{
                "amount": {
                    "currency_code": currency,
                    "value": str(amount)
                }
            }]
        }

        response = requests.post(url, headers=headers, json=payload)
        return response.json()

    def capture_order(self, order_id):
        """擷取訂單的付款。"""
        url = f"{self.base_url}/v2/checkout/orders/{order_id}/capture"
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.access_token}"
        }

        response = requests.post(url, headers=headers)
        return response.json()

    def get_order_details(self, order_id):
        """取得訂單詳情。"""
        url = f"{self.base_url}/v2/checkout/orders/{order_id}"
        headers = {
            "Authorization": f"Bearer {self.access_token}"
        }

        response = requests.get(url, headers=headers)
        return response.json()
```

## IPN（Instant Payment Notification，即時付款通知）處理

### IPN 驗證與處理
```python
from flask import Flask, request
import requests
from urllib.parse import parse_qs

app = Flask(__name__)

@app.route('/ipn', methods=['POST'])
def handle_ipn():
    """處理 PayPal IPN 通知。"""
    # 取得 IPN 訊息
    ipn_data = request.form.to_dict()

    # 向 PayPal 驗證 IPN
    if not verify_ipn(ipn_data):
        return 'IPN verification failed', 400

    # 根據交易類型處理 IPN
    payment_status = ipn_data.get('payment_status')
    txn_type = ipn_data.get('txn_type')

    if payment_status == 'Completed':
        handle_payment_completed(ipn_data)
    elif payment_status == 'Refunded':
        handle_refund(ipn_data)
    elif payment_status == 'Reversed':
        handle_chargeback(ipn_data)

    return 'IPN processed', 200

def verify_ipn(ipn_data):
    """驗證 IPN 訊息真實性。"""
    # 新增 'cmd' 參數
    verify_data = ipn_data.copy()
    verify_data['cmd'] = '_notify-validate'

    # 回傳給 PayPal 進行驗證
    paypal_url = 'https://ipnpb.sandbox.paypal.com/cgi-bin/webscr'  # 或正式環境 URL

    response = requests.post(paypal_url, data=verify_data)

    return response.text == 'VERIFIED'

def handle_payment_completed(ipn_data):
    """處理已完成的付款。"""
    txn_id = ipn_data.get('txn_id')
    payer_email = ipn_data.get('payer_email')
    mc_gross = ipn_data.get('mc_gross')
    item_name = ipn_data.get('item_name')

    # 檢查是否已處理（防止重複）
    if is_transaction_processed(txn_id):
        return

    # 更新資料庫
    # 發送確認郵件
    # 履行訂單
    print(f"Payment completed: {txn_id}, Amount: ${mc_gross}")

def handle_refund(ipn_data):
    """處理退款。"""
    parent_txn_id = ipn_data.get('parent_txn_id')
    mc_gross = ipn_data.get('mc_gross')

    # 在系統中處理退款
    print(f"Refund processed: {parent_txn_id}, Amount: ${mc_gross}")

def handle_chargeback(ipn_data):
    """處理付款撤銷／拒付。"""
    txn_id = ipn_data.get('txn_id')
    reason_code = ipn_data.get('reason_code')

    # 處理拒付
    print(f"Chargeback: {txn_id}, Reason: {reason_code}")
```

## 訂閱制／定期扣款

### 建立訂閱方案
```python
def create_subscription_plan(name, amount, interval='MONTH'):
    """建立訂閱方案。"""
    client = PayPalClient(CLIENT_ID, CLIENT_SECRET)

    url = f"{client.base_url}/v1/billing/plans"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {client.access_token}"
    }

    payload = {
        "product_id": "PRODUCT_ID",  # 先建立產品
        "name": name,
        "billing_cycles": [{
            "frequency": {
                "interval_unit": interval,
                "interval_count": 1
            },
            "tenure_type": "REGULAR",
            "sequence": 1,
            "total_cycles": 0,  # 無限次
            "pricing_scheme": {
                "fixed_price": {
                    "value": str(amount),
                    "currency_code": "USD"
                }
            }
        }],
        "payment_preferences": {
            "auto_bill_outstanding": True,
            "setup_fee": {
                "value": "0",
                "currency_code": "USD"
            },
            "setup_fee_failure_action": "CONTINUE",
            "payment_failure_threshold": 3
        }
    }

    response = requests.post(url, headers=headers, json=payload)
    return response.json()

def create_subscription(plan_id, subscriber_email):
    """為客戶建立訂閱。"""
    client = PayPalClient(CLIENT_ID, CLIENT_SECRET)

    url = f"{client.base_url}/v1/billing/subscriptions"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {client.access_token}"
    }

    payload = {
        "plan_id": plan_id,
        "subscriber": {
            "email_address": subscriber_email
        },
        "application_context": {
            "return_url": "https://yourdomain.com/subscription/success",
            "cancel_url": "https://yourdomain.com/subscription/cancel"
        }
    }

    response = requests.post(url, headers=headers, json=payload)
    subscription = response.json()

    # 取得核准 URL
    for link in subscription.get('links', []):
        if link['rel'] == 'approve':
            return {
                'subscription_id': subscription['id'],
                'approval_url': link['href']
            }
```

## 退款工作流程

```python
def create_refund(capture_id, amount=None, note=None):
    """為已擷取的付款建立退款。"""
    client = PayPalClient(CLIENT_ID, CLIENT_SECRET)

    url = f"{client.base_url}/v2/payments/captures/{capture_id}/refund"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {client.access_token}"
    }

    payload = {}
    if amount:
        payload["amount"] = {
            "value": str(amount),
            "currency_code": "USD"
        }

    if note:
        payload["note_to_payer"] = note

    response = requests.post(url, headers=headers, json=payload)
    return response.json()

def get_refund_details(refund_id):
    """取得退款詳情。"""
    client = PayPalClient(CLIENT_ID, CLIENT_SECRET)

    url = f"{client.base_url}/v2/payments/refunds/{refund_id}"
    headers = {
        "Authorization": f"Bearer {client.access_token}"
    }

    response = requests.get(url, headers=headers)
    return response.json()
```

## 錯誤處理

```python
class PayPalError(Exception):
    """自訂 PayPal 錯誤。"""
    pass

def handle_paypal_api_call(api_function):
    """PayPal API 呼叫的錯誤處理包裝器。"""
    try:
        result = api_function()
        return result
    except requests.exceptions.RequestException as e:
        # 網路錯誤
        raise PayPalError(f"Network error: {str(e)}")
    except Exception as e:
        # 其他錯誤
        raise PayPalError(f"PayPal API error: {str(e)}")

# 使用方式
try:
    order = handle_paypal_api_call(lambda: client.create_order(25.00))
except PayPalError as e:
    # 適當處理錯誤
    log_error(e)
```

## 測試

```python
# 使用沙箱憑證
SANDBOX_CLIENT_ID = "..."
SANDBOX_SECRET = "..."

# 測試帳號
# 在 developer.paypal.com 建立測試買家和賣家帳號

def test_payment_flow():
    """測試完整付款流程。"""
    client = PayPalClient(SANDBOX_CLIENT_ID, SANDBOX_SECRET, mode='sandbox')

    # 建立訂單
    order = client.create_order(10.00)
    assert 'id' in order

    # 取得核准 URL
    approval_url = next((link['href'] for link in order['links'] if link['rel'] == 'approve'), None)
    assert approval_url is not None

    # 核准後（使用測試帳號的手動步驟）
    # 擷取訂單
    # captured = client.capture_order(order['id'])
    # assert captured['status'] == 'COMPLETED'
```

## 資源

- **references/express-checkout.md**：Express Checkout 實作指南
- **references/ipn-handling.md**：IPN 驗證與處理
- **references/refund-workflows.md**：退款處理模式
- **references/billing-agreements.md**：定期扣款設定
- **assets/paypal-client.py**：正式環境 PayPal 客戶端
- **assets/ipn-processor.py**：IPN webhook 處理器
- **assets/recurring-billing.py**：訂閱管理

## 最佳實踐

1. **務必驗證 IPN**：永遠不要在未驗證的情況下信任 IPN
2. **冪等處理**：處理重複的 IPN 通知
3. **錯誤處理**：實作健全的錯誤處理
4. **日誌記錄**：記錄所有交易和錯誤
5. **徹底測試**：廣泛使用沙箱環境
6. **Webhook 備援**：不要僅依賴客戶端回呼
7. **幣別處理**：總是明確指定幣別

## 常見陷阱

- **未驗證 IPN**：在未驗證的情況下接受 IPN
- **重複處理**：未檢查重複交易
- **錯誤環境**：混用沙箱和正式環境的 URL／憑證
- **遺漏 Webhooks**：未處理所有付款狀態
- **硬編碼值**：未針對不同環境設定可配置選項
