---
name: stripe-integration
description: 實作 Stripe 金流處理，提供強健且符合 PCI 規範的支付流程，包含結帳、訂閱與 webhooks。適用於整合 Stripe 支付、建立訂閱系統，或實作安全的結帳流程。
---

# Stripe 整合

精通 Stripe 金流處理整合，實現強健且符合 PCI 規範的支付流程，包含結帳、訂閱、webhooks 與退款功能。

## 何時使用此技能

- 在網頁/行動應用程式中實作金流處理
- 建立訂閱計費系統
- 處理單次支付與定期扣款
- 處理退款與爭議
- 管理客戶的付款方式
- 為歐洲支付實作 SCA（強客戶驗證，Strong Customer Authentication）
- 使用 Stripe Connect 建立市集金流

## 核心概念

### 1. 支付流程
**Checkout Session（託管式）**
- Stripe 託管的支付頁面
- 最小化 PCI 合規負擔
- 最快速的實作方式
- 支援單次及定期支付

**Payment Intents（自訂 UI）**
- 完全控制支付 UI
- 需要 Stripe.js 以符合 PCI 規範
- 較複雜的實作方式
- 更好的客製化選項

**Setup Intents（儲存付款方式）**
- 收集付款方式但不扣款
- 用於訂閱與未來支付
- 需要客戶確認

### 2. Webhooks
**關鍵事件：**
- `payment_intent.succeeded`: 支付完成
- `payment_intent.payment_failed`: 支付失敗
- `customer.subscription.updated`: 訂閱變更
- `customer.subscription.deleted`: 訂閱取消
- `charge.refunded`: 退款已處理
- `invoice.payment_succeeded`: 訂閱付款成功

### 3. 訂閱
**組成元件：**
- **Product**: 您銷售的商品
- **Price**: 金額與頻率
- **Subscription**: 客戶的定期付款
- **Invoice**: 每個計費週期產生的發票

### 4. 客戶管理
- 建立與管理客戶記錄
- 儲存多種付款方式
- 追蹤客戶中繼資料
- 管理帳單詳情

## 快速開始

```python
import stripe

stripe.api_key = "sk_test_..."

# 建立結帳 session
session = stripe.checkout.Session.create(
    payment_method_types=['card'],
    line_items=[{
        'price_data': {
            'currency': 'usd',
            'product_data': {
                'name': 'Premium Subscription',
            },
            'unit_amount': 2000,  # $20.00
            'recurring': {
                'interval': 'month',
            },
        },
        'quantity': 1,
    }],
    mode='subscription',
    success_url='https://yourdomain.com/success?session_id={CHECKOUT_SESSION_ID}',
    cancel_url='https://yourdomain.com/cancel',
)

# 將使用者導向 session.url
print(session.url)
```

## 支付實作模式

### 模式 1: 單次支付（託管式結帳）
```python
def create_checkout_session(amount, currency='usd'):
    """建立單次支付結帳 session。"""
    try:
        session = stripe.checkout.Session.create(
            payment_method_types=['card'],
            line_items=[{
                'price_data': {
                    'currency': currency,
                    'product_data': {
                        'name': 'Purchase',
                        'images': ['https://example.com/product.jpg'],
                    },
                    'unit_amount': amount,  # 金額單位為分（cents）
                },
                'quantity': 1,
            }],
            mode='payment',
            success_url='https://yourdomain.com/success?session_id={CHECKOUT_SESSION_ID}',
            cancel_url='https://yourdomain.com/cancel',
            metadata={
                'order_id': 'order_123',
                'user_id': 'user_456'
            }
        )
        return session
    except stripe.error.StripeError as e:
        # 處理錯誤
        print(f"Stripe error: {e.user_message}")
        raise
```

### 模式 2: 自訂 Payment Intent 流程
```python
def create_payment_intent(amount, currency='usd', customer_id=None):
    """建立用於自訂結帳 UI 的 payment intent。"""
    intent = stripe.PaymentIntent.create(
        amount=amount,
        currency=currency,
        customer=customer_id,
        automatic_payment_methods={
            'enabled': True,
        },
        metadata={
            'integration_check': 'accept_a_payment'
        }
    )
    return intent.client_secret  # 傳送至前端

# 前端（JavaScript）
"""
const stripe = Stripe('pk_test_...');
const elements = stripe.elements();
const cardElement = elements.create('card');
cardElement.mount('#card-element');

const {error, paymentIntent} = await stripe.confirmCardPayment(
    clientSecret,
    {
        payment_method: {
            card: cardElement,
            billing_details: {
                name: 'Customer Name'
            }
        }
    }
);

if (error) {
    // 處理錯誤
} else if (paymentIntent.status === 'succeeded') {
    // 支付成功
}
"""
```

### 模式 3: 建立訂閱
```python
def create_subscription(customer_id, price_id):
    """為客戶建立訂閱。"""
    try:
        subscription = stripe.Subscription.create(
            customer=customer_id,
            items=[{'price': price_id}],
            payment_behavior='default_incomplete',
            payment_settings={'save_default_payment_method': 'on_subscription'},
            expand=['latest_invoice.payment_intent'],
        )

        return {
            'subscription_id': subscription.id,
            'client_secret': subscription.latest_invoice.payment_intent.client_secret
        }
    except stripe.error.StripeError as e:
        print(f"Subscription creation failed: {e}")
        raise
```

### 模式 4: 客戶入口
```python
def create_customer_portal_session(customer_id):
    """建立讓客戶管理訂閱的入口 session。"""
    session = stripe.billing_portal.Session.create(
        customer=customer_id,
        return_url='https://yourdomain.com/account',
    )
    return session.url  # 將客戶導向此網址
```

## Webhook 處理

### 安全的 Webhook 端點
```python
from flask import Flask, request
import stripe

app = Flask(__name__)

endpoint_secret = 'whsec_...'

@app.route('/webhook', methods=['POST'])
def webhook():
    payload = request.data
    sig_header = request.headers.get('Stripe-Signature')

    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, endpoint_secret
        )
    except ValueError:
        # 無效的 payload
        return 'Invalid payload', 400
    except stripe.error.SignatureVerificationError:
        # 無效的簽章
        return 'Invalid signature', 400

    # 處理事件
    if event['type'] == 'payment_intent.succeeded':
        payment_intent = event['data']['object']
        handle_successful_payment(payment_intent)
    elif event['type'] == 'payment_intent.payment_failed':
        payment_intent = event['data']['object']
        handle_failed_payment(payment_intent)
    elif event['type'] == 'customer.subscription.deleted':
        subscription = event['data']['object']
        handle_subscription_canceled(subscription)

    return 'Success', 200

def handle_successful_payment(payment_intent):
    """處理成功的支付。"""
    customer_id = payment_intent.get('customer')
    amount = payment_intent['amount']
    metadata = payment_intent.get('metadata', {})

    # 更新資料庫
    # 傳送確認信
    # 履行訂單
    print(f"Payment succeeded: {payment_intent['id']}")

def handle_failed_payment(payment_intent):
    """處理失敗的支付。"""
    error = payment_intent.get('last_payment_error', {})
    print(f"Payment failed: {error.get('message')}")
    # 通知客戶
    # 更新訂單狀態

def handle_subscription_canceled(subscription):
    """處理訂閱取消。"""
    customer_id = subscription['customer']
    # 更新使用者存取權限
    # 傳送取消通知信
    print(f"Subscription canceled: {subscription['id']}")
```

### Webhook 最佳實踐
```python
import hashlib
import hmac

def verify_webhook_signature(payload, signature, secret):
    """手動驗證 webhook 簽章。"""
    expected_sig = hmac.new(
        secret.encode('utf-8'),
        payload,
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(signature, expected_sig)

def handle_webhook_idempotently(event_id, handler):
    """確保 webhook 只被處理一次。"""
    # 檢查事件是否已被處理
    if is_event_processed(event_id):
        return

    # 處理事件
    try:
        handler()
        mark_event_processed(event_id)
    except Exception as e:
        log_error(e)
        # Stripe 會重試失敗的 webhooks
        raise
```

## 客戶管理

```python
def create_customer(email, name, payment_method_id=None):
    """建立 Stripe 客戶。"""
    customer = stripe.Customer.create(
        email=email,
        name=name,
        payment_method=payment_method_id,
        invoice_settings={
            'default_payment_method': payment_method_id
        } if payment_method_id else None,
        metadata={
            'user_id': '12345'
        }
    )
    return customer

def attach_payment_method(customer_id, payment_method_id):
    """將付款方式附加至客戶。"""
    stripe.PaymentMethod.attach(
        payment_method_id,
        customer=customer_id
    )

    # 設為預設付款方式
    stripe.Customer.modify(
        customer_id,
        invoice_settings={
            'default_payment_method': payment_method_id
        }
    )

def list_customer_payment_methods(customer_id):
    """列出客戶的所有付款方式。"""
    payment_methods = stripe.PaymentMethod.list(
        customer=customer_id,
        type='card'
    )
    return payment_methods.data
```

## 退款處理

```python
def create_refund(payment_intent_id, amount=None, reason=None):
    """建立退款。"""
    refund_params = {
        'payment_intent': payment_intent_id
    }

    if amount:
        refund_params['amount'] = amount  # 部分退款

    if reason:
        refund_params['reason'] = reason  # 'duplicate', 'fraudulent', 'requested_by_customer'

    refund = stripe.Refund.create(**refund_params)
    return refund

def handle_dispute(charge_id, evidence):
    """以證據更新爭議。"""
    stripe.Dispute.modify(
        charge_id,
        evidence={
            'customer_name': evidence.get('customer_name'),
            'customer_email_address': evidence.get('customer_email'),
            'shipping_documentation': evidence.get('shipping_proof'),
            'customer_communication': evidence.get('communication'),
        }
    )
```

## 測試

```python
# 使用測試模式金鑰
stripe.api_key = "sk_test_..."

# 測試卡號
TEST_CARDS = {
    'success': '4242424242424242',
    'declined': '4000000000000002',
    '3d_secure': '4000002500003155',
    'insufficient_funds': '4000000000009995'
}

def test_payment_flow():
    """測試完整支付流程。"""
    # 建立測試客戶
    customer = stripe.Customer.create(
        email="test@example.com"
    )

    # 建立 payment intent
    intent = stripe.PaymentIntent.create(
        amount=1000,
        currency='usd',
        customer=customer.id,
        payment_method_types=['card']
    )

    # 使用測試卡片確認
    confirmed = stripe.PaymentIntent.confirm(
        intent.id,
        payment_method='pm_card_visa'  # 測試付款方式
    )

    assert confirmed.status == 'succeeded'
```

## 資源

- **references/checkout-flows.md**: 詳細的結帳實作
- **references/webhook-handling.md**: Webhook 安全性與處理
- **references/subscription-management.md**: 訂閱生命週期
- **references/customer-management.md**: 客戶與付款方式處理
- **references/invoice-generation.md**: 發票開立與計費
- **assets/stripe-client.py**: 正式環境可用的 Stripe 客戶端包裝器
- **assets/webhook-handler.py**: 完整的 webhook 處理器
- **assets/checkout-config.json**: 結帳設定範本

## 最佳實踐

1. **務必使用 Webhooks**: 不要只依賴客戶端的確認
2. **冪等性**: 以冪等的方式處理 webhook 事件
3. **錯誤處理**: 優雅地處理所有 Stripe 錯誤
4. **測試模式**: 上正式環境前使用測試金鑰徹底測試
5. **中繼資料**: 使用 metadata 將 Stripe 物件連結到您的資料庫
6. **監控**: 追蹤支付成功率與錯誤
7. **PCI 合規**: 絕不在伺服器上處理原始卡片資料
8. **SCA 就緒**: 為歐洲支付實作 3D Secure

## 常見陷阱

- **未驗證 Webhooks**: 務必驗證 webhook 簽章
- **遺漏 Webhook 事件**: 處理所有相關的 webhook 事件
- **寫死金額**: 使用分（cents）或最小貨幣單位
- **無重試邏輯**: 為 API 呼叫實作重試機制
- **忽略測試模式**: 使用測試卡片測試所有邊界案例
